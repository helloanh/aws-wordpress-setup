# AWS Wordpress for Fault Tolerance Site Logistics and Overview  

The purpose of this guide is to break down the logistics and price breakdown for setting up a WordPress site on the AWS EC2 service.  The site is configured to be fault tolerant using Route53 as domain registar,  Elastic Load Balancer with auto scalinng group, and the media files will be served over CloudFront.  The database setup uses RDS setup.  

## I.  Architecture Overview  

![img](https://s17.postimg.org/y4asrtggv/aws_fault_torelant_arch.png) 

Estimate Hours to Set Up AWS instance: 1  
Estimate Hours to Set Up CloudFront and RDS: 1  
Estimate Hours to migrate preexisting communitychange.org site: 2-3  

## II. Cost Breakdown  

The cost breakdown is based on Amazon's Pricing [here](https://s22.postimg.org/9eoo53psx/reserved_spot_instances.png://s17.postimg.org/bswh6rzqn/reserved_spot_instances.png).  

For a t2.small instance with standard reserved setup, here is the price breakdown:  
    **one year, no reserve**  
    0.018 per hr x 24 hrs x  30 days  = $12.96 per month  
    $155.22 per year  

    **three year, with reserve**  
    0.004 per hr x 24 hrs * 30 days = $8.64 per month  
    $103.68 per year  but you must purchase for three years up front  


![reserved_spot_instances.png](https://s17.postimg.org/bswh6rzqn/reserved_spot_instances.png)
