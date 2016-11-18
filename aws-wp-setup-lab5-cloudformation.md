# Lab 5: Cloud Formation  

1. Go to Auto Scaling Group, delete the entire ASG.  
2. Delete our RDS instances.  
3. Elastic Load Balancers cost you money, so delete them as well.  
4. Now the entire environment is shut down.  
5. Change region.  Go to Cloud Formation.  

## Set Up Stack  

1. Click on **Create a Stack**.  
2. On **Choose a template** section, click on Multi-AZ Samples and select the *WordPress blog* option.  
3. Click on *View/Edit Template own Designer*.  This will take you to a page with view of a network diagram with view of JSON version as well.  
4. Click on template1 tag to view the JSON full code.  Just hit close and go back to the *Create Stack* page.  We are just using the default template, so nothing will change.  
5. Click *Next* button to go to the next *Specify Details* section.  
6. Configurations:  
        + Stackname - just going to call it something like CCC-WP-ASG-CF.  
        + DBClass - db.t2.small   
        + DBName - wordpressdbccc  
        + DBPassword - ???  
        + DBUser cccadmin  
        + KeyName - assign from your existing key  
        + MultiAZDatabase - true  
        + SSHLocation  0.0.0.0/0  
        + WebServer Capacity 2 


Now you have the script in a complete new region.  You can use the same script in different regions.  

Cheap to do and not have to rent data centers around the world.  

7.  After 20 mins or so, the status should say **CREATE_COMPLETE**.  Click on the bottom "OutPuts" tab and grab the website URL.  You can go to the Dashboard and set the site up now.  You can also see the instances listed in the EC2 section.  


8. You still need to set up user permissions and change the owner of the /var/www/ directory to make basic functions like upload or download plugins work on the dashboard.  Follow this tutorial [here](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-LAMP.html).  

To allow user *ec2-user* (for Amazon AWS) write access to the public web directory /var/www/html enter the following commands:  
```sh
# for Amazon AMI, make sure to sudo su first
chown -R ec2-user:ec2-user /var/www/html  
chmod -R 755 /var/www/html  

# for ubuntu
sudo chown -R ubuntu:ubuntu /var/www/html 
sudo chmod -R 755 /var/www/html
```


