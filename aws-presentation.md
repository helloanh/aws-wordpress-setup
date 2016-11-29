# AWS Wordpress for Fault Tolerance Site Logistics and Overview  

The purpose of this guide is to break down the logistics and price breakdown for setting up a WordPress site on the AWS EC2 service.  The site is configured to be fault tolerant using Route53 as domain registar,  Elastic Load Balancer with auto scalinng group, and the media files will be served over CloudFront.  The database setup uses RDS setup.  

### Architecture Overview  

![img](https://s17.postimg.org/y4asrtggv/aws_fault_torelant_arch.png) 

Estimate Hours to Set Up AWS instance: 1  
Estimate Hours to Set Up CloudFront and RDS: 1  
Estimate Hours to migrate preexisting communitychange.org site: 2-3  

### Cost Breakdown  

![reserved_spot_instances.png](https://s17.postimg.org/bswh6rzqn/reserved_spot_instances.png)
