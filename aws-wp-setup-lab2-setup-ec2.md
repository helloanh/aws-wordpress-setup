# Lab2: Building a Fault Tolerant Wordpress Site, Set Up EC2 Instance  

## 1. Launch EC2 Instance  

1. Launch an Amazon Linux AMI.  
2. **IMPORANT STEP**: make sure to set EC2 Role to the S3 role you created.  
3. In step 3, *Configure Instance Details*, add the bashscript.sh code in the *Advanced Details* section before launch.  
4. Create or use the old .pem key.    
5. Launch.   
6. After instance is running and become available, ssh into the instance from the terminal.  
7. Upgrade to root privilege with "sudo su" command.   
8. Run "yum update"    
9.

## 2. LAMP Stack Set Up  

1. Run "yum install httpd php php-mysql stress"  
2. Configure Apache to use URL rewrite:  
        - cd /etc/httpd/conf  
        - cp httpd.conf httpd.conf.bak  
        - make a backup of the httpd config file:    
                cp httpd.conf httpdconfbackup.conf  
                vim httpd.conf:  
                    scroll down, look for line to allow url rewrite    
                    this allows large img files to be distributed using Cloudfront and not from our instance  
                    find the line with <Directory "var/www/html"> and find AllowOverride All  
                    set **AllowOverride None** to **AllowOverride All**  





            
