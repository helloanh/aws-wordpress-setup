# Lab 4: Autoscaling and Load Testing 

1. Create Autoscaling group  
        *  Create a launch configuration first, and use the AMI you created in the last lab.  
        * Make sure you set the IAM role to connect with S3 permission.    
        * run bash script to grab the wp file:  

```sh 
#!/bin/bash  
yum update -y
aws s3 sync --delete s3://yourbucketcodefromAMI/  /var/www/html/
```
2. Set Web-DMZ Security Group and launch.  
3. Now create the Auto Scaling Group:  
        * fill out Group name: WP-ASG-1 or something similar  
        * Group Size: 2  
        * Select all subnet regions for *Subnet* field  
4. Click on *Advanced Details* tab to expand the Load Balancer options:  
        * check on *Load Balancing* box  
        * select the load balancer you created earlier in the *Classic Load Balancers* field  
        * Health Check Type: ELB option  
        * leave everything else as standard  
5. Configure Scaling Policy:  
        * Scale between 2 to 4 instances  
        * Just click on create simple policy for both Group Size  
        * Add an alarm - set whenever *average* of *cpu utilization* is *>=* 60 percent, for at lease 1 consecutive periods of *1 minute*, then name the alarm something more human-friendly like *CPU-Above-60*.  Configure **Take the Action** to add 1 instance.  
        * Add the second alarm - set when CPU is less than or equal to 20% for same parameters as last alarm, but remove 1 instance.  
[Screen Shot 2016-11-14 at 12.41.16 AM.png](https://postimg.org/image/hr6f6zz2x/)  

6. [Optional]  Add email notification if you prefer.  
7. Configure tag and launch.  
8. Simulate failover from on site to another on RDS, and go to *Instance Actions* and hit reboot.  Go to our WordPress site to see.  It does take about 20 seconds to get the site up again.  

### Lastly, we are going to run a stress check on our instances that was set up in teh auto scaling group.  

Stress is a little program to stress out your CPU.  So log into your instance and type this command:  

```sh
stress --cpu 100
```

Now go to the instances list and see if the CPU utilization in the next 10 minutes increased.  There should be a massive spike and the ASG should notice that spike and create another instance.  

Now the next question is how to have **regional fault torelance** not just mulitiple availability fault torelance.  





