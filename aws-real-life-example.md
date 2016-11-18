# REAL LIFE EXAMPLE: WORDPRESS SET UP  

Designing a highly scalable WordPress site on the cloud in AWS.  This is great for clients who still have their sites on traditional hosting providers such a Bluehost GoDaddy, etc.  

Make sure you start from a clean slate.  So delete delete everything you started -- CloudFront, VPC, EC2, all security groups.  The only thing left is the IAM S3 Bucket rule.  [Or to can keep those groups but just make new configuration from below]  

1. Create IAM role and configure it to S3 Full Access, if you don't have the s3 role from before.  

2. Create security groups in VPC Dashboard:    
        -  web-dmz and set it up my default VPC.  alow inbound rules: http traffic from 0.0.0.0/0 and do the same for https.  You might want to lock down ssh to your own IP address.  

[webdmz](https://s15.postimg.org/6wg100nrf/Screen_Shot_2016_11_18_at_2_42_26_PM.png)  

        - rds-security-group and set mysql traffic 3306 port with *source* from the web-dmz security group.  this is important.  Make sure the MySQL (3306) with TCP protocal and Port Range 3306 is linked to the Souce *sg-somenumberhere* and that source is from the webdmz you jsut created!  







