# Lab 3: Automation and Setting Up Our AMI  
1. Navigate to WP directory wp-content/uploads.  
2. Upload a test image.  
3. Make a post with the test image.  
4. Navigate to the wp-content/uploads directory and you should see a new directory based on the current year.  
        *  for example, mine is wp-content/uploads/

Trying to show you that the file is stored locally in EC2 instance.  Now we are going to automate it in a fashion such that all our uplaods go to an S3 bucket, so our CloudFront CDN can serve the media files and not the EC2 instance.  

## I. Use CloudFront and S3 to server media contents  

1. Synchonizing contents from uploads directory to S3 bucket:  
        *  aws s3 ls
        *  aws s3 cp --recursive /var/www/html/wp-content/uploads/  s3://yourmediabucketname    

The commands list the buckets, the copy contents from the uploads folder from EC2 instance to S3 bucket.  

2. Use "aws s3 sync /var/www/html/wp-content/uploads s3://yourmediabuketname --delete" when you have more image uploads.  This is better than the copy command, you you have to reupload everything again.  The sync will upload the new image only.  

### Force WordPress to Use CloudFront by Rewriting the URL in the .htaccess  

1. Go to /var/www/html/ 
2. Make a .htaccess or download it here:  
        *  wget https://s3-eu-west-1.amazonaws.com/acloudguru/config/htaccess  
        *  make sure rename htaccess to .htaccess  
*There is also a copy in this repo*  
3. Look into the .htaccess file and see that the rewriteryle line is using the endpoint of the CloudFront Domain name.  So go to your AWS Dashboard, find CloudFront, and copy your own CloudFront Domain name.  
4. Replace the default domain name in the sample .htaccess with your own CloudFront Domain Name.  
5. Restart httpd service:  
        *  service httpd restart  


What should happen now is the image in the EC2 WordPress is served front CloudFront.  You can check by checking the WordPress image by clicking on the source of the image on a post.  

**NOTE:** If your cloudfront distribution is not working, try putting the line with rewriterule after the #END WordPress, save the .htaccess, and restart httpd server again.    

Also try to access /etc/httpd/conf/httpd.conf and make sure "AllowOverride All" setting is on under the section on "DocumentRoot /var/www/html"  

**The next step to how to automate this whole sync process.**   

## II. Automating Files to Serve from CloudFront  

1. Navigate to /etc directory.  
2. Vim to crontab file, which is basically the linux version of scheduled task script for the system.  
        * vim crontab  
        * make these changes:  
                    1. sync our website code to s3 bucket  
                    2. sync our upload directory to our cdn  

First line means sync every two minutes every hour to everyday of the month, on every month on everyday of the week.

```sh
# file /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
#put this at the last line --> this sync all the files from EC2 to your bucket and delete the old file in the bucket  
*/2 * * * * root aws s3 sync --delete /var/www/html/ s3://yourbucketnamecodehere/  
*/2 * * * * root aws s3 sync --delete /var/www/htnl/wp-content/uploads s3://yourbucketnamemediahere/    
#download the contents from the S3 code bucket back to EC2 instance.  Why?  
#bc if you have multiple EC2 instances and make a change on one, you want to make those changes back to your instance.  This is a bit messy, normally you would have a dedicated instance to make changes to.  download back the code to our /var/www/html  
#for production, have a dedicated EC2 instance so you only need to run the top two lines of code, but if you have a fleet of instances, you would need to run the bottom code for all of them  

*/3 * * * * root aws s3 sync --delete s3://yourbucketnamecodehere/ /var/www/html/
```  

**Now we have set up everything and can make an AMI of this instance to use it for autoscaling for future instances**  

## II. Create an AMI  

1. Go to your EC2 Console, select our EC2, go to action and click on "Create Image"  
2. For the "No Reboot" option.  If you check this box, it won't reboot your EC2 instance in the image creation process.  This can cause issue with data consistency.  Recommended practice to leave this option unchecked.  
3. When the AMI is available, launch the AMI, set the IAM role as S3, and run this bootstrap:  

```sh
#!/bin/bash
yum update -y
aws s3 sync --delete s3://yourwordpresscodedirectoryhere/ /var/www/html/  

``` 
4. Put the security group as Web-DMZ and launch this.  
5. When instance come live, grab the public IP address and go to the browser to see the set up is still there (including CloudFront server).  

### NEXT STEPS: CREATE AUTOSCALING AND CONFIGURATION GROUP ...  

