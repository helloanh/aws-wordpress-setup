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

### The following steps 3-5 are necessary if you didnt run the bashscript on the instance before launch  

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

## V. Set up S3 Bucket with EC2 WordPress Instance  
1. See all you buckets: aws s3 ls  
2. Copy all WordPress Core code to S3 bucket:  
        aws s3 cp --recursive /var/www/html s3:yourbucketnamehere  
3. Check to see if all your WP files are in the S3 bucket:  
        aws s3 ls yourbucketnamehere  
4. Do something dangerous -- delete all your WP Core files in html/  
        cd ..  
        rm -rf html/  
5. Go to the browser and see the white screen of death when refresh the page  
        you just simulate a failure of your EC2  
6. Just recover by downloading the same bucket again:  
        aws s3 cp --recursive s3://yourbucketnamehere /var/www/html/  
7. Go back to the browser and refresh the screen, and the site should be up again.  
8. Last thing, change permission and user since you just download the files in the html again:  
        chmod -R 755 wp-content  
        chown -R apache.apache wp-content  

## RECAP  
        +  Set up EC2 instance with S3 permissions  
        +  Install WordPress with RDS database  
        +  Upload the code to S3  
        +  Simulate a failure from the server and fix it by downloading the same code from S3 bucket  






            
