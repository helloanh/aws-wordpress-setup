# Building a Fault Tolerant Wordpress Site: Part 1, Set Up Environment  

<img src="https://s22.postimg.org/5ajsscc01/Screen_Shot_2016_11_08_at_7_32_36_PM.png" style="height: 300px; width: 175px;"> 


## I. Create S3 Role in IAM  

1. Go to IAM, create a new role    
2. Name the role "s3role"    
3. Connect the s3 role with your EC2 instance  
4. Select AmazonS3FullAccess  

## II. Create Two Security Groups in your VPC  

### Create Web DMZ  
Inbound:  
        -   SSH(22) 0.0.0.0/0  
        -   HTTP(80) 0.0.0.0/0  

### Create RDS Security Group  
Inbound:  
        -   MySQL/Aurora(3306) 0.0.0.0/0 

## III. Create Two S3 Buckets  
Bucket 1: for the wordpress core files    
Bucket 2: for the wordpress media files  

### Create CDN to serve the media bucket  

1. Go into CloudFront  
2. Create Distribution  
3. On *Origin Settings* under *Origin Domain Name*, select the name of the media bucket you just created. 
4. Set *Restrict Bucket Access* as On.  
5. Select *Create A New Identity*   
6. Select *Yes, Update Bucket Policy* on Grant Read Permissions on Bucket  
7. Leave everything the way it is and click on "Create Distribution"  

## IV. Create RDS Instance  

1. Go to RDS in AWS Console  
2. Select MySQL Community Edition  
3. Select the appropriate DB Instance Class  
4. Select Multi A-Z Deployment (not eligible for free-tier)  
5. Fill in the DB Instance Identifier, Master Username, and Master Password  
6. Make sure under the *Network and Security* section has *Publicly Accessible* to **No**.   
7. For *VPC Security Group(s)*, select the rds secruity grou pyou created in Step 2.    
8. Name the database and launch.  

## V. Set Up Load Balancer in EC2  

1. Name load balancer something like mywordpressLB.  
2. Go to the next page and assign the web-dmz security group you created earlier in Step 2.  
3. Configure heatlh check.  Just name the *Ping Path* file to healthy.html  
4. Lower the health check interval to 10 seconds, healthy threshold to 3.  So now the instance will come into service within 30 seconds.  
8. Name the database and launch.  


## VI. Set Up DNS in Route53  

1. Create Record Set  
2. Set *Alias* to *Yes*  
3. Set *Alis Target* to the load balancer you made in the step 5.    
4. Leave everything else as standard and hit create.  



