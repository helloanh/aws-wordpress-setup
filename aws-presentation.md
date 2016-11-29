# AWS Wordpress for Fault Tolerance Site Logistics and Overview  

The purpose of this guide is to break down the logistics and price breakdown for setting up a WordPress site on the AWS EC2 service.  The site is configured to be fault tolerant using Route53 as domain registar,  Elastic Load Balancer with auto scalinng group, and the media files will be served over CloudFront.  The database setup uses RDS setup.  

### Architecture Overview  

<img src="https://camo.githubusercontent.com/278cc5fe248c3e2f0a64eb91875c00d3e17e682a/68747470733a2f2f7332322e706f7374696d672e6f72672f35616a7373636330312f53637265656e5f53686f745f323031365f31315f30385f61745f375f3320202020325f33365f504d2e706e67" style="width: 200px;">  


