# Welcome to the Route 53 Routing Policies Implementation Project

## Project Overview

This project provides a step-by-step guide to implementing various AWS Route 53 routing policies using EC2 instances across multiple AWS regions. By following this structured approach, you will gain hands-on experience with:

*   Failover Routing – Ensuring high availability by automatically redirecting traffic in case of failures.
*   Latency-Based Routing – Optimizing performance by directing users to the closest and lowest-latency server.
*   Weighted Routing – Controlling traffic distribution between multiple servers based on assigned weights.

## Key Objectives

*   Deploy and configure EC2 instances using a custom AMI across different AWS regions.
*   Set up Nginx web servers to serve content and validate routing behavior.
*   Configure Route 53 health checks to monitor server availability.
*   Implement Failover, Latency, and Weighted routing policies in Route 53.
*   Test and verify the correctness of each routing mechanism.

## Why This Project?

*   **Practical Hands-On Experience:** Deep dive into AWS Route 53 configurations in a real-world scenario.
*   **High Availability & Resilience:** Learn how to ensure seamless traffic failover and regional optimization.
*   **Performance Optimization:** Understand latency-based routing and weighted traffic distribution.
*   **Infrastructure as Code (IaC) Ready:** This setup can be further automated using Terraform or AWS CloudFormation.

## Who Should Follow This Guide?

*   DevOps Engineers – Looking to strengthen their AWS networking skills.
*   Cloud Engineers – Interested in mastering Route 53 routing mechanisms.
*   System Administrators – Managing high-availability deployments in AWS.
*   Students & Enthusiasts – Exploring AWS networking and traffic management concepts.

By the end of this project, you will have a fully functional multi-region deployment with intelligent routing mechanisms using AWS Route 53.

## Prerequisites for the Route 53 Implementation Project

Before starting this project, ensure you meet the following requirements:

### 1. AWS Account Setup

*   An active AWS account with administrator permissions.
*   AWS IAM user with the necessary permissions to manage EC2, Route 53, VPCs, and security groups.

### 2. Basic Knowledge Requirements

*   Understanding of AWS EC2, VPC, Security Groups, and AMIs.
*   Familiarity with basic Linux commands and working with the CLI.
*   Basic experience with web servers (Nginx/Apache) and configuring them.
*   Understanding of DNS concepts (e.g., A records, TTL, health checks).

### 3. Infrastructure Requirements

*   **Custom AMI:** You must create a custom Amazon Machine Image (AMI) for launching multiple EC2 instances.
*   **Two AWS Regions:** The project requires instances in North Virginia (us-east-1) and Mumbai (ap-south-1).
*   **Route 53 Hosted Zone:** A registered domain name in Route 53 for setting up DNS records.
*   **Security Groups:** Ensure port 80 (HTTP) is open for web server access.

### 4. Software & Tools Required

*   AWS CLI installed and configured.
*   SSH Client (such as OpenSSH, PuTTY) to connect to EC2 instances.
*   Text Editor (VS Code, Nano, Vim) for modifying configuration files.
*   Web Browser for testing DNS resolution and failover behavior.

## Real time Task Step-by-Step Implementation for Route 53 and Policies

### Step 1: Create a Custom AMI

#### 1.1 Launch an EC2 Instance with the Desired Configuration

1.  Open AWS Management Console → EC2 Dashboard.
2.  Click Launch Instance.
3.  Choose an OS (select Ubuntu 20.04).
4.  Choose an instance type (e.g., `t2.micro`).
5.  Under Configure Instance, select your VPC (ensure the security group allows all traffic from anywhere).
6.  Under Configure Storage, leave default settings.

In the Advanced Details (User Data) section, paste the following script (make sure to update the folder paths if necessary):

```bash
#!/bin/bash
apt update && apt install -y nginx
apt update && sudo apt install certbot python3-certbot-nginx
sudo mkdir -p /var/www/your_domain_name.in/html
sudo chown -R $USER:$USER /var/www/your_domain_name.in/html
sudo chmod -R 755 /var/www/your_domain_name.in/html
# Open editor to create an index file (use nano if preferred)
cat << 'EOF' | sudo tee /var/www/your_domain_name.in/html/index.html
<html>
<head>
<title>Welcome to your_domain_name!</title>
</head>
<body>
<h1>Success! The your_domain_name server block is working!</h1>
</body>
</html>
EOF
sudo nano /etc/nginx/sites-available/your_domain_name
# Paste the following into the file:
# -------------------------------------------------server {
listen 80;
listen [::]:80;

root /var/www/your_domain_name/in/html;
index index.html index.htm index.nginx-debian.html;

server_name your_domain_name failover.your_domain_name latency.your_domain_name weighted.your_domain_name www.your_domain_name;

location / {
try_files $uri $uri/ =404;
}
}
# -------------------------------------------------sudo ln -s /etc/nginx/sites-available/your_domain_name /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

sudo certbot certonly \
--agree-tos \
--no-eff-email \
--email your_email@gmail.com \
--manual \
--preferred-challenges=dns \
-d *.your_domain_name \
--server https://acme-v02.api.letsencrypt.org/directory
# Optionally, you may run certbot --nginx after certificate issuance.
```

7.  Complete the launch process and log in to verify that the instance is working as expected.
8.  Once verified, create an AMI from this instance:
    *   In the EC2 Console, select the instance.
    *   Choose Actions > Image and templates > Create image.
    *   Image Name: `asg-ami`
    *   Click Create Image and wait for the AMI to be available.

### Step 2: Launch EC2 Instances Using the Custom AMI

#### 2.1 Launch Instance in North Virginia Region (AMI-NV)

1.  Go to EC2 Dashboard in the North Virginia (`us-east-1`) region.
2.  Click Launch Instance.
3.  Name the instance: `ami-nv`
4.  Select AMI: Choose the custom AMI `asg-ami` you created.
5.  Select Instance Type: (as desired; defaults can be used)
6.  Select VPC: Use the VPC with your security group (allows all traffic).
7.  Select Subnet: Choose a public subnet.
8.  Auto-assign Public IP: Enable
9.  Complete launch and note the public IP.

#### 2.2 Launch Instance in Mumbai Region (AMI-MB)

1.  Switch to the Mumbai region (`ap-south-1`).
2.  Go to EC2 Dashboard and click Launch Instance.
3.  Name the instance: `ami-mb`
4.  Select AMI: Use the same custom AMI `asg-ami` (ensure it’s available or copy it to Mumbai if needed).
5.  Select Instance Type, VPC, and Subnet: Use a public subnet.
6.  Auto-assign Public IP: Enable
7.  Complete launch and note the public IP.

### Step 3: Create Route 53 Health Checks

#### 3.1 Create Health Check for US Server (North Virginia Instance)

1.  In the AWS Management Console, go to Route 53.
2.  Click on Health Checks in the left-hand menu.
3.  Click Create health check.
4.  Health Check Name: `us-server`
5.  What to Monitor: Choose Endpoint.
6.  Specify Endpoint:
    *   Endpoint Type: IP address.
    *   Protocol: TCP.
    *   IP Address: Enter the public IP of the `ami-nv` instance (North Virginia).
    *   Port: `80`
7.  In the Advanced Configuration, set the Failure threshold to `1` (for Fast and Future threshold).
8.  Click Create Health Check.

#### 3.2 Create Health Check for India Server (Mumbai Instance)

1.  Click Create health check again.
2.  Health Check Name: `india-server`
3.  Endpoint Type: IP address.
4.  Protocol: TCP.
5.  IP Address: Enter the public IP of the `ami-mb` instance (Mumbai).
6.  Port: `80`
7.  In Advanced Configuration, set Failure threshold to `1`.
8.  Click Create Health Check.

### Step 4: Configure Failover Routing in Route 53

#### 4.1 Create Failover Records in Your Hosted Zone

1.  Go to Route 53 → Hosted Zones.
2.  Select your public hosted zone for your domain (e.g., `your_domain_name.com`).

##### 4.1.a Create Primary Failover Record

1.  Click Create record.
2.  Record Name: Enter `failover` (this creates `failover.your_domain_name.com`).
3.  Record Type: A – IPv4 address.
4.  Routing Policy: Select Failover.
5.  Under Failover record settings:
    *   Failover Record Type: Primary.
    *   Value/Route traffic to: Enter the public IP of the `ami-nv` instance (North Virginia).
    *   Health Check ID: Select the `us-server` health check.
    *   Record ID: Enter a name like `US`.
6.  Click Create Record.

##### 4.1.b Create Secondary Failover Record

1.  Click Create record again.
2.  Record Name: Use the same, `failover` (creating an additional failover record for the same domain).
3.  Record Type: A – IPv4 address.
4.  Routing Policy: Failover.
5.  Under Failover record settings:
    *   Failover Record Type: Secondary.
    *   Value/Route traffic to: Enter the public IP of the `ami-mb` instance (Mumbai).
    *   Health Check ID: Select the `india-server` health check.
    *   Record ID: Enter a name like `INDIA`.
6.  Click Create Record.

### Step 5: Test Failover Configuration

Open a web browser and navigate to:

```
http://failover.your_domain_name.com
```

1.  **Expected Result:**
    *   Initially, the browser should load the content from the `ami-nv` instance (North Virginia) since it is set as Primary.
2.  **To simulate failover:**
    *   SSH into the `ami-nv` instance.

Stop the Nginx service:

```bash
sudo systemctl stop nginx
```

*   Wait for the health check to fail (allow a minute or so).

Open the browser again and navigate to:

```
http://failover.your_domain_name.com
```

3.  Now, traffic should be routed to the `ami-mb` instance (Mumbai) because the Primary health check will be failing.
4.  Verify that the content is now served from the Mumbai instance.

### Step 6: Configure Latency Routing in Route 53

#### 6.1 Create Latency Records in Your Hosted Zone

1.  Go to Route 53 → Hosted Zones → Select your hosted zone.
2.  Click Create record.
3.  Record Name: Enter `latency` (this creates `latency.your_domain_name.com`).
4.  Record Type: A – IPv4 address.
5.  Routing Policy: Select Latency.

##### 6.1.a Add Latency Record for US (North Virginia)

1.  Under Define Latency Record, choose Value/Route traffic to:
    *   Enter the public IP of the `ami-nv` instance (North Virginia).
    *   Health Check ID: Select `us-server`.
    *   Record ID: `US`.
2.  Click Add Record.

##### 6.1.b Add Latency Record for India (Mumbai)

1.  Again, click Create Record (or add a new record within the same record group if allowed).
2.  Record Name: `latency` (the same record name).
3.  Record Type: A – IPv4 address.
4.  Routing Policy: Latency.
5.  Under Define Latency Record, choose:
    *   Value/Route traffic to: Enter the public IP of the `ami-mb` instance (Mumbai).
    *   Health Check ID: Select `india-server`.
    *   Record ID: `INDIA`.
6.  Click Create Record.

#### 6.2 Test Latency-Based Routing

Open a web browser and navigate to:

```
http://latency.your_domain_name.com
```

1.  Based on your network latency:
    *   Requests from locations with lower latency to Mumbai should go there.
    *   Requests from locations with lower latency to North Virginia should go there.
2.  You can also test by using tools like `curl` from different networks to verify routing.

### Step 7: Configure Weighted Routing in Route 53

#### 7.1 Create Weighted Records in Your Hosted Zone

1.  Go to Route 53 → Hosted Zones → Select your hosted zone.
2.  Click Create record.
3.  Record Name: Enter `weighted` (this creates `weighted.your_domain_name.com`).
4.  Record Type: A – IPv4 address.
5.  Routing Policy: Select Weighted.

##### 7.1.a Define Weighted Record for US (North Virginia)

1.  Under Define Weighted Record, set:
    *   Value/Route traffic to: Enter the public IP of the `ami-nv` instance (North Virginia).
    *   Weight: Enter `1`.
    *   Health Check ID: Select `us-server`.
    *   Record ID: Enter `US`.
2.  Click Add Record.

##### 7.1.b Define Weighted Record for India (Mumbai)

1.  Again, click Add Record (or create another weighted record under the same record name).
2.  Set:
    *   Value/Route traffic to: Enter the public IP of the `ami-mb` instance (Mumbai).
    *   Weight: Enter `254`.
    *   Health Check ID: Select `india-server`.
    *   Record ID: Enter `INDIA`.
3.  Click Create Record.

#### 7.2 Test Weighted Routing

Open a browser and navigate to:

```
http://weighted.your_domain_name.com
```

1.  Since the weight for the Mumbai instance is much higher (`254`) compared to North Virginia (`1`), the majority of traffic should be routed to the Mumbai server.
2.  Verify by refreshing the page several times; the displayed content should primarily be from the Mumbai instance.

### Step 8: Final Testing and Verification

#### 8.1 Verify Failover Configuration

Open your browser and type:

```
http://failover.your_domain_name.com
```

*   Initially, you should see the content from the North Virginia instance.
*   After stopping Nginx on the North Virginia instance, verify that the traffic fails over to the Mumbai instance.

#### 8.2 Verify Latency-Based Routing

Open your browser and navigate to:

```
http://latency.your_domain_name.com
```

*   The ALB should direct you to the instance (North Virginia or Mumbai) with the lowest latency for your location.

#### 8.3 Verify Weighted Routing

Open your browser and navigate to:

```
http://weighted.your_domain_name.com
```

*   You should see that the traffic is predominantly routed to the Mumbai instance due to its higher weight.

