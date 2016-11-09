# Building a Fault Tolerant Wordpress Site: Part 1, Set Up Environment  

![network diagram](https://s22.postimg.org/5ajsscc01/Screen_Shot_2016_11_08_at_7_32_36_PM.png)  


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



