# REAL LIFE EXAMPLE: WORDPRESS SET UP  

Designing a highly scalable WordPress site on the cloud in AWS.  This is great for clients who still have their sites on traditional hosting providers such a Bluehost GoDaddy, etc.  

Make sure you start from a clean slate.  So delete delete everything you started -- CloudFront, VPC, EC2, all security groups.  The only thing left is the IAM S3 Bucket rule.  [Or to can keep those groups but just make new configuration from below]  

### 1. Create IAM role and configure it to S3 Full Access, if you don't have the s3 role from before.  

### 2. Create security groups in VPC Dashboard:    
*  web-dmz and set it up my default VPC.  alow inbound rules: http traffic from 0.0.0.0/0 and do the same for https.  You might want to lock down ssh to your own IP address.  

![webdmz](https://s15.postimg.org/6wg100nrf/Screen_Shot_2016_11_18_at_2_42_26_PM.png)  

* rds-security-group and set mysql traffic 3306 port with *source* from the web-dmz security group.  this is important.  Make sure the MySQL (3306) with TCP protocal and Port Range 3306 is linked to the Souce *sg-somenumberhere* and that source is from the webdmz you jsut created  

### 3. Configure RDS Instance  

* Go into RDS, hit get started and use MySQL instance.  You can select Multi-AZ Deloyment or not.  

* Configure DB Instnace Identifier, Username, Pw, etc.  

* In *Network & Security* section, make sure to put **Publicly Accessible** as No and **VPC Security Group(s)** to the RDS security group you just created that was connected to the webdmz security group.  

![rds-mysql](https://s12.postimg.org/drqu55jgt/Screen_Shot_2016_11_18_at_3_01_13_PM.png)  

* Launch your db and now we are going to the Elastic Load Balancer.  

### 4.  Configure your ELB:  
        - go to EC2, click on *Load Balancers* tab, and create a new Load Balancer.  
        - configure only HTTP, because if you also select HTTPS, it asks for an SSL certificate.  so just do HTTP at port 80 for now.  
        - for healthcheck, set ping path to /elb.html  
        - response timeout: 5 seconds  
        - health check interval: 30 seconds
        - unhealthy threshold: 4 (mins)    

We are setting the acceptable amount of down time based on the existence of the /elb.html file.   

![health-check](https://s17.postimg.org/67wvu56sf/Screen_Shot_2016_11_18_at_3_21_27_PM.png) 

*  Put security group as webdmz that you created earlier.  
*  There's no instances to add at this stage yet.  
*  Create a keypair name and hit create.  

### 5. Set Up Domain Name in Route53  

* Go to Route53 to DNS service.  Go to Hosted Zones, and go to *Create Record Set*  and associate root domain with the ELB.  (You need to purchase a domain name first before this could work.)  

* What you want to edit is the two type A records.  

* What to change **Alias Target** to the name of your load balancer you created for all A records type.  

### 6. Create Two S3 Buckets  

* example: I created goodjobsforallcdn and goodjobsforallwpcode  

* the cdn is to serve media contents in the wp-content/uploads/ folder  (publicly available bucket for the world to view)  

* the wpcode is to serve and save the core wp code (private bucket for the wordpress site)    

### 7. Create CloudFront Distribution  

* Go to CF and click on create distribution.  There are two types, webs and RTMP.  99% of the time you will go for the web.  

* Click on the web distribution.  

**Origin Domain Name**: goodjobsforallcdn.s3.amazon.aws.com  (or the name of the cdn bucket you created earlier)  

**Origin Path** leave as it   
**Origin ID** leave as it    
**Restrict Bucket Access** Yes, because we want users to force them to use CDN and not S3.  This speeds up the latency.  
**Origin Access Identity** Select Create A New Identity  
**Grant Read Permissions on Bucket** Yes, Update Bucket Policy  

Everything else leave as default and hit create distribution.  Now you can see the distribution is set up (might take some time for the process to be completed).  


### 8. EC2 Instance and WordPress Installation  

* Create a new Amazon AMI Linux instance, set the S3 IAM role.  We are not going to use the Advance Details section to add the bootstrapshell script, but will do it in the terminal later.    

* I call it something like CCCTemplateWebServer-WordPress.  
* Configure Security Group as the webdmz (should have SSH, HTTP, HTTPS).  If you look at the rds security group, you should see that the webdmz you have selected is connected to the mysql.  So just click on only the webdmz and launch.  Assign tis to existing PEM key-pair.  

* Copy the Public IP address and ssh into the instance you just created.  Make sure you RDS is also live.  

* In RDS dashboard if you set **Multi AZ** as No, then you don't manage the connection with the endpoit url.  
* If **Multi AZ** is Yes, use the endpoint, so if the primary zone is done, AWS will set up the failover to another zone.  

### 9. Connect EC2 Instance to Load Balancer    

* Go to the load balancer you created, and add the instance.  

[![Screen Shot 2016-11-18 at 4.34.53 PM.png](https://s17.postimg.org/5akribwj3/Screen_Shot_2016_11_18_at_4_34_53_PM.png)](https://postimg.org/image/4l1z5yvzf/)  

* Currently the status is *OutOfService*, when it is up in service then it will start service traffic to your EC2 instance.  

### 10. SSH to Instance to Set Up WordPress Environment  

```sh
sudo su
clear

#install apache, php, and php-mysql (just for our instance to communicate to our rds server)  

yum install httpd php php-mysql -y
yum update -y

# edit settings of httpd.conf file to allow us to do url rewrite
cd /etc/httpd/conf/
cp httpd.conf mycopyhttpd.conf
vim httpd.conf
```  

```sh
#file name: httpd.conf
#find section with DocumentRoot "/var/www/html"

...
<Directory "/var/www/html">
# look for the line AllowOverride None, and change it to All  

# AllowOverride controls what directives may be placed in .htaccess files.  
# It can be "All", "None", or any combination of the keywords:  
#   Iotuis FileInfo AuthConfig Limit 

    AllowOverride None

# Controls who can get stuff from this server.
# 

...

</Directory>

...

```  

* After you edited the httpd.conf file, you need to start apache:  

```sh
service httpd start
# go to html directory
cd /var/www/html
# create elb.html file for healthcheck  
echo "Success. Elastic Load Balancer is healthy." > elb.html 
clear
```

* Go to the browser and type in the ipadress/elb.html to see the file  

* Now download wordpress  

```sh
cd /var/www/  
wget https://wordpress.org/latest.tar.gz  
tar -xzf latest.tar.gz
rm -rf html
mv wordpress html
ls
# now we just have html and have all the wp files
cd html
# reprovision elb
echo "success" > elb.html  
# clone the config file
cp wp-config-sample.php wp-config.php
vim wp-config.php
# now configure the db and username

```

* Need to run thees two commands to allow WP to write to the folder:  

```sh
chown -R apache.apache html/
chmod -R 755 html/
```

* Now set up WP in the browser, remove wp-config.php first
```php
#file wp-config.php
# you can also do this using the GUI installation with the browser
```
* All the info is available in RDS instance if you forget  
* Copy the RDS endpoint to the **Database Host** field  

### 11. Configure Site to Recover from the lost of an EC2 Instance  

* Now we can make this site highly available  
* Go into Post, there is a default Hello World! post  
* Test media upload my adding image to the post  
* Now image is served from the EC2 and not CloudFront  

```sh
cd html/
# just list the content
aws s3 ls
# copy the contents of my html directory into my bucket  
aws s3 cp --recursive /var/www/html/ s3://yourbucketwordpresscode  
# check if the files are copy by listing the bucket 
aws s3 ls yourbucketwordpresscode 
# test to see if this work, by rm all contents in html
cd ..
rm -rf html
# now check the website on the browser, it should be dead
# now recover by copy the content again
aws s3 cp s3://yourbucketwordpresscode /var/www/html --recursive  
# now it should downlaod all the files again and if you check the browser, the site should be up  

```

* Make it so that everytime we update our site, the contents will be updated into the aws s3 bucket.  This is done using cronjob to automate this process  

```sh
cd /etc
ls
# you should see a file called crontab
# now go to crontab and edit the following at the end of the document  

# every 2 mins for every hour of month of the week  
*/2 * * * * root aws s3 sync  /var/www/html/ s3://youbucketwordpresscode  

```

* Now restart cron service
```sh
service crond restart
```

* Now create a new test file called mytestfile.html in the /var/www/html  
```sh
echo "hello this is a test" > mytestfile.html
```

* Wait for two mins, and check the aws s3 bucket to see if the file is synced there  

* If it is not syncing, try taking out the --recursive tag, else add --region tag  

### 12. Configure CloudFront to Serve Media Files from S3 Bucket  

* Go back to the wordpress site, you can see the uploads are still from EC2 instances  
* First step is to copy the current contents from the uploads folder to our media bucket:  

```sh
aws s3 cp /var/www/thml/wp-content/uploads s3://yourwordpressmediabuckethere/  
```

* Now enable it by url rewrite so it is served from the bucket  
```sh
#this file is provided by acloudguru
aws s3 cp s3://acloudguru/htaccess /var/www/html/  
```

* Ignore everything and change his url with our own cloudfront ur

#### [Optional] Migration An Old Site With All-In-One Migration  

* Install the All-In-One-Migration plugin in the dashboard  
* You might need to download the .wpress backup file through the command line.  I saved it in my S3 bucket and copy it to my folder at var/www/html/wp-content/ai1wm-backups/  

* Make sure you have enough memory!  usually t2.micro are not enough for live site migration.  This crashed the site for me on first try (a reboot recovers it).  

* You need to reset the chmod an chown again to apache  
* You also need to copy the contents fromt html/ up to the s3 bucket again before running the cron job in crontab script  


#### Setting Up CloudFront  

```sh
# first, copy up the files in the cloud to bucket 3  
aws s3 cp /var/www/html/wp-content/uploads s3://yourwordpresscdnbucket 

# copy the code from this file
wget https://s3-eu-west-1.amazonaws.com/acloudguru/config/htaccess

# go to the htaccess and switch the http url to cloudfront **Domain Name**    
```

```htaccess
Options +FollowSymlinks
RewriteEngine on
rewriterule ^wp-content/uploads/(.*)$ http://d2jmrva8tfzxcq.cloudfront.net/$1 [r=301,nc]

# BEGIN WordPress

# END WordPress
```

* rename htaccess to .htaccess  
* restart apache service  

```sh
# in html/
mv htaccess .htaccess
service apache restart
```

What should happen now is the image in the EC2 WordPress is served front CloudFront. You can check by checking the WordPress image by clicking on the source of the image on a post.

NOTE: If your cloudfront distribution is not working, try putting the line with rewriterule after the #END WordPress, save the .htaccess, and restart httpd server again.

Also try to access /etc/httpd/conf/httpd.conf and make sure "AllowOverride All" setting is on under the section on "DocumentRoot /var/www/html"  

#### Syncing Upload Folder with Cron Job  

```sh
#file /etc/crontab
#add these two lines after the first cron job:  

#put this at the last line --> this sync all the files from EC2 to your bucket and delete the old file in the bucket  

*/2 * * * * root aws s3 sync --delete /var/www/html/ s3://yourbucketnamecodehere/  

*/2 * * * * root aws s3 sync --delete /var/www/htnl/wp-content/uploads s3://yourbucketnamemediahere/    

#download the contents from the S3 code bucket back to EC2 instance.  Why?  
#bc if you have multiple EC2 instances and make a change on one, you want to make those changes back to your instance.  This is a bit messy, normally you would have a dedicated instance to make changes to.  download back the code to our /var/www/html  
#for production, have a dedicated EC2 instance so you only need to run the top two lines of code, but if you have a fleet of instances, you would need to run the bottom code for all of them  

*/5  * * * * root aws s3 sync --delete s3://yourbucketnamecodehere/ /var/www/html/
```

### 12. Create an AMI Image  

* First test the cron job to see if it works  
* upload a file in your bucket at aws s3 dashboard  
* wait to see if the cron job is working  
* Second test an image upload in WP dashboard  

* Now delete html directory, create a snapshot, and turn it into an AMI  

```sh
service crond stop
rm -rf html/
mkdir html/
clear
chown -R apache.apache html/
chmod -R 755 html/
cd html/
ls
# should be empty  
chkconfig crond on
chkconfig httpd on
```

* Go back to your EC2 Console  
* Click on "Create An Image", call it something like MyWPDbWeb  
* Notice that you will lose access to your instance  
* Go to AMI, it should be pending, and wait until it is available  
* Now it is ready for autoscaling group  

### 13. Autoscaling & Testing  

* go to ec3 dashboard amd click on create launch configuration  
* select the ami we created  
* give it s3 access  
* under **Advanced Details**  make sure to add this sccript:  
```sh
#!/bin/bash
yum update -y
aws s3 cp --recursive s3://yourwordpresscode /var/www/html/
service crond start
service httpd start 

```

* add storage  
* use web-dmz security group  
* launch configuration  
* auto scaling group --> starts with 2 instances  
* add subnets  
* **Advanced Details**: 300 seconds, from eLB, receive traffic check  
* Use scaling policies  


### [Optional] Set Up Sass and Compass  

Setting up [sass](http://stackoverflow.com/questions/10129235/amazon-linux-latest-ruby).  
[More](https://forums.aws.amazon.com/message.jspa?messageID=565015).  
[Danny Nguyen's Linux Set Up](https://gist.github.com/dannguyen/5415628).  

