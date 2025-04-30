# -Week-9-Assignmen
# AWS Web Application Deployment with ALB, ASG, and S3

This document outlines the steps taken to deploy a web application on AWS using S3 for static assets, an Auto Scaling Group (ASG) for NGINX web servers, and an Application Load Balancer (ALB) for traffic distribution.

## Scenario

The goal is to deploy the Clarusway Bootcamp website (provided as `index.html` and logo files) in a highly available architecture on AWS.

## Part 1: S3 Setup (Static Assets)

### 1.  Create S3 Bucket

* An S3 bucket was created in the `eu-north-1` region to store static assets.
* Bucket Name: `yourname-clarusway-assets` (Replace `yourname` with your unique identifier)

### 2.  Upload Files

* The following files were uploaded to the S3 bucket:
    * `index.html` (Provided HTML file)
    * Logo files (`logo.png`, `sda.png`)

### 3.  Configure Bucket

* **Static website hosting:** Enabled to serve the `index.html` file as the website's index document.
* **Bucket policy:** Configured to allow public read access to the assets.

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": "*",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::yourname-clarusway-assets/*"  # Replace with your bucket name
        }
      ]
    }
    ```

### Deliverables (Part 1)

* Screenshot of the S3 website URL.
* Output of `curl` command showing a `200 OK` response.

## Part 2: Auto Scaling Group (ASG)

### 1.  Create Launch Template

* A Launch Template was created to define the configuration of EC2 instances for the ASG.
    * AMI: Amazon Linux 2
    * Instance Type: `t3.micro`
    * Name: `clarusway-launch-template`
    * IAM Role: `EC2S3ReadOnlyRole` (Role with `AmazonS3ReadOnlyAccess` policy attached)
    * **User Data Script:**

        ```bash
        #!/bin/bash
        yum update -y
        yum install nginx -y
        systemctl start nginx
        systemctl enable nginx
        aws s3 cp s3://yourname-clarusway-assets/index.html /usr/share/nginx/html/index.html # Replace with your bucket name
        ```

### 2.  Configure ASG

* An Auto Scaling Group was configured to manage the NGINX web servers.
    * Name: `clarusway-asg`
    * Launch Template: Selected the template created in the previous step.
    * VPC: Default VPC
    * Subnets: All available subnets selected for high availability.
    * Capacity:
        * Min: 1
        * Max: 3
        * Desired: 2
    * Health Checks: Enabled both EC2 and ELB health checks.
    * Scaling Policies: No scaling policies were implemented in this assignment (manual scaling only).

### Deliverables (Part 2)

* Screenshot showing 2 running instances in the ASG.
* Screenshot of ASG configuration details.

## Part 3: Application Load Balancer (ALB)

### 1.  Create Internet-facing ALB

* An Application Load Balancer was created to distribute traffic across the NGINX instances.
    * Type: Application Load Balancer
    * Name: `clarusway-alb`
    * Scheme: Internet-facing
    * Listeners: HTTP on port 80
    * VPC: Default VPC
    * Subnets: All available subnets selected.
    * Target Group:
        * Name: `clarusway-tg`
        * Type: Instance
        * Health check path: `/`

### 2.  Verify

* The website was accessed using the ALB's DNS name.
* Traffic distribution was verified (as much as possible with a small number of instances).

### Deliverables (Part 3)

* Screenshot of the webpage accessible via ALB DNS name.
* Screenshot of the output of the following command:

    ```bash
    for i in {1..5}; do curl -s http://your-alb-dns | grep Welcome; done # Replace your-alb-dns
    ```

* Screenshot of the instances running in the ASG.

## Success Criteria

* Website is accessible via both:
    * S3 endpoint (static version)
    * ALB endpoint (dynamic via ASG)
* ASG automatically replaces terminated instances (This was tested manually).
* All assets (HTML and logos) load correctly.

## Cleanup

* S3 bucket (`yourname-clarusway-assets`) was deleted.
* ASG (`clarusway-asg`) was terminated (which automatically deleted the EC2 instances).
* ALB (`clarusway-alb`) was deleted.

## Additional Notes

* Replace placeholders like `yourname` and `your-alb-dns` with your actual values.
* Screenshots are named clearly and included in the repository.
