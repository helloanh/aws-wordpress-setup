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


