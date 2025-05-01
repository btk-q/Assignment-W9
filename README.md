# Assignment-W9
# Week 9 Assignment - AWS Web Application Deployment

This project demonstrates how to deploy the Clarusway Bootcamp website using AWS services with high availability and scalability features.

## Objective

Deploy a static website using the following AWS components:
- Amazon S3 for static asset hosting
- EC2 instances running NGINX behind an Auto Scaling Group
- Application Load Balancer to distribute traffic

---

## Part 1: S3 Setup for Static Assets

### Step-by-Step:

1. **Create the S3 Bucket**
   - Name it: `sda1029abdelmohsen-alhummimes-clarusway-assets` 
   - Region: `eu-north-1`

2. **Enable Static Website Hosting**
   - Go to **Properties** tab
   - Scroll to **Static website hosting**
   - Enable it and set:
     - **Index document:** `index.html`

3. **Upload Files**
   - Upload `index.html`, `logo.png`, and `sda.png` into the bucket.

4. **Set Bucket Policy for Public Access**
   - Use this JSON policy:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [{
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::sda1029abdelmohsen-alhummimes-clarusway-assets/*"
       }]
     }
     ```

5. **Test the S3 Website**
   - Copy the S3 website endpoint URL and open it in your browser.
   - Run `curl` to verify HTTP 200:
     ```bash
     curl -I http://sda1029abdelmohsen-alhummimes-clarusway-assets.s3-website.eu-north-1.amazonaws.com
     ```

---

## Part 2: Launch Template & Auto Scaling Group

### Step-by-Step:

1. **Create a Launch Template**
   - Use Amazon Linux 2 AMI
   - Add the following User Data script:
     ```bash
     #!/bin/bash
     yum update -y
     yum install nginx -y
     systemctl start nginx
     systemctl enable nginx
     aws s3 cp s3://sda1029abdelmohsen-alhummimes-clarusway-assets/index.html /usr/share/nginx/html/
     ```

2. **Create an Auto Scaling Group**
   - Attach the Launch Template
   - Set:
     - Min: 1
     - Max: 3
     - Desired Capacity: 2
   - Enable Health Checks: **EC2 and ELB**
   - Select **subnets** in at least two Availability Zones

3. **Verify**
   - Ensure 2 instances are running
   - Check that NGINX is serving the website

---

## Part 3: Application Load Balancer

### Step-by-Step:

1. **Create ALB**
   - Type: Internet-facing
   - Listener: Port 80 (HTTP)
   - Subnets: At least two across AZs

2. **Create a Target Group**
   - Type: Instance
   - Protocol: HTTP
   - Health Check Path: `/`

3. **Attach Target Group to ALB**
   - Register your ASG with the target group

4. **Verify Load Balancer**
   - Visit the ALB DNS name in browser
   - Run:
     ```bash
     for i in {1..5}; do curl -s YOUR_ALB_DNS | grep "hostname"; done
     ```
   - Confirm round-robin traffic between instances