# Lab2: Building a Fault Tolerant Wordpress Site, Set Up EC2 Instance  

## I. Launch EC2 Instance  

1. Launch an Amazon Linux AMI.  
2. **IMPORANT STEP**: make sure to set EC2 Role to the S3 role you created.  
3. In step 3, *Configure Instance Details*, add the bashscript.sh code in the *Advanced Details* section before launch.  
4. Create or use the old .pem key.    
5. Launch.   
6. After instance is running and become available, ssh into the instance from the terminal.  
7. Upgrade to root privilege with "sudo su" command.   
8. Run "yum update"    
9.

## II. LAMP Stack Set Up  

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

#### The following steps 3-5 are necessary if you didnt run the bashscript on the instance before launch  

3. Go to /var/www/html  
4. Create a healthy.html file:  
        echo "healthy server" >> healthy.html  
5. Change permissions so media files are accessible  
        chmod -R 755 wp-content  
        chown -R apache.apache wp-content  
        service httpd start  
        chkconfig httpd  

##  III. Set Up Load Balancer to Your Instance  
1. Go to the load balancer section in your EC2 Dashboard.  
2. Click on "Edit Instance" to add the wordpress instance.  
3. Wait until the Status change from **OutOfService** to **InService**.  

## IV. Set Up WordPress in the Browser  
1. Fill in the database name, username, and password you saved from RDS before.  
2. First, go to RDS to get the endpoint url for the **Database Host** field.  
3. Click on the RDS instance and copy and paste the endpoint url to the clipboard.  
4. Update this url to the **Database Host** field.  
5. The screen will say it cannot write in the wp-config.php file, but still give you the script.  Copy and paste this script and update it in the terminal for the wp-config.php file.  






            
