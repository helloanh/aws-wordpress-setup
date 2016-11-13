# Lab 3: Automation and Setting Up Our AMI  
1. Navigate to WP directory wp-content/uploads.  
2. Upload a test image.  
3. Make a post with the test image.  
4. Navigate to the wp-content/uploads directory and you should see a new directory based on the current year.  
    +  for example, mine is wp-content/uploads/

Trying to show you that the file is stored locally in EC2 instance.  Now we are going to automate it in a fashion such that all our uplaods go to an S3 bucket, so our CloudFront CDN can serve the media files and not the EC2 instance.  

## I. Use CloudFront and S3 to server media contents  

1. Synchonizing contents from uploads directory to S3 bucket:  
        +  aws s3 ls
        +  aws s3 cp --recursive /var/www/html/wp-content/uploads/  s3://yourmediabucketname    

The commands list the buckets, the copy contents from the uploads folder from EC2 instance to S3 bucket.  

2. Use "aws s3 sync /var/www/html/wp-content/uploads s3://yourmediabuketname --delete" when you have more image uploads.  This is better than the copy command, you you have to reupload everything again.  The sync will upload the new image only.  

### Force WordPress to Use CloudFront by Rewriting the URL in the .htaccess  

1. Go to /var/www/  
2. Make a .htaccess or download it here:  
        +  wget https://s3-eu-west-1.amazonaws.com/acloudguru/config/htaccess  
        +  make sure rename htaccess to .htaccess  
*There is also a copy in this repo*  
3. Look into the .htaccess file and see that the rewriteryle line is using the endpoint of the CloudFront Domain name.  So go to your AWS Dashboard, find CloudFront, and copy your own CloudFront Domain name.  

4.




