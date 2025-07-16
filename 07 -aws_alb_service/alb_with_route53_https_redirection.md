
# Real Time Task Step-by-Step Implementation of Application Load Balancer (ALB) with Route 53 & HTTPS Redirection

## Introduction
In this project, we implement a highly available, secure, and scalable application architecture using AWS services. We configure an Application Load Balancer (ALB) with Route 53 for domain resolution and enable HTTPS redirection to ensure secure communication.

This setup demonstrates:

*   SSL/TLS Certificate Management with AWS Certificate Manager (ACM)
*   VPC Design with public and private subnets across multiple Availability Zones
*   Application Load Balancer (ALB) for traffic distribution and secure access
*   Path-Based Routing to serve different application pages from separate backend instances
*   Automatic HTTP to HTTPS Redirection for enhanced security
*   AWS Route 53 for custom domain management

## Project Overview
We deploy a web application with three different sections:
*   Homepage (Default landing page)
*   Movies Section (/movies)
*   Shows Section (/shows)

Each section is hosted on separate EC2 instances within private subnets. The ALB manages traffic distribution, and Route 53 resolves domain names while ensuring secure HTTPS access.

## Why This Project?

*   **Security:** HTTPS encryption with ACM-issued certificates
*   **Scalability:** Load balancer distributes traffic efficiently
*   **Reliability:** Multi-AZ architecture for high availability
*   **User Experience:** Smooth redirections and efficient path-based routing

## Technologies Used

*   AWS Certificate Manager (ACM) – SSL/TLS certificate issuance
*   Amazon Route 53 – Domain name resolution
*   AWS VPC – Custom networking setup with private/public subnets
*   Amazon EC2 – Hosting NGINX-based web servers
*   Application Load Balancer (ALB) – Load balancing and path-based routing
*   Security Groups & NAT Gateway – Secure and controlled network access

By the end of this project, you will have a fully functional, secure, and well-architected AWS infrastructure with proper HTTPS redirection, domain management, and efficient load balancing.

Let’s get started!

## Step 1: Issue an SSL/TLS Certificate in AWS Certificate Manager (ACM)
Before setting up the ALB, we need to issue an SSL/TLS certificate for HTTPS traffic.

### 1.1 Navigate to AWS ACM (Certificate Manager)
*   Open the AWS Management Console.
*   Search for Certificate Manager in AWS services.
*   Click on Request a certificate.

### 1.2 Request a Public Certificate
*   Select Request a public certificate.
*   Click Next.

### 1.3 Configure the Certificate
*   In the Fully Qualified Domain Name (FQDN) field, enter:
    *   `your_domain_name` (e.g., example.com)
    *   `www.your_domain_name` (e.g., www.example.com)
*   Click Next.

### 1.4 Choose Validation Method
*   Choose DNS Validation.
*   Click Next.

### 1.5 Review and Request
*   Click on Request.

### 1.6 Validate the Certificate
*   Open Route 53.
*   Go to your hosted zone.
*   Create CNAME records using the values given in ACM.
*   Wait for validation to complete (it may take a few minutes).
*   Once validated, the certificate status should change to Issued.

## Step 2: Create a VPC in North Virginia Region
We will create a VPC with public and private subnets across three Availability Zones.

### 2.1 Create a New VPC
1.  Go to the AWS Console → VPC.
2.  Click Create VPC.
3.  Select VPC and more.
4.  Set Name as `alb-vpc`.
5.  IPv4 CIDR Block: `10.0.0.0/16`.
6.  Number of Availability Zones (AZs): 3.
7.  Subnets:
    *   3 Public Subnets (one per AZ)
    *   3 Private Subnets (one per AZ)
8.  NAT Gateway:
    *   Enable NAT Gateway in one AZ.
9.  Click Create VPC.

## Step 3: Modify Security Group for VPC

### 3.1 Modify Default Security Group
1.  Go to VPC → Security Groups.
2.  Select the default security group for `alb-vpc`.
3.  Edit Inbound Rules:
    *   All TCP (0-65535) → Anywhere (`0.0.0.0/0`)
    *   All UDP (0-65535) → Anywhere (`0.0.0.0/0`)
    *   HTTP (80) → Anywhere (`0.0.0.0/0`)
    *   HTTPS (443) → Anywhere (`0.0.0.0/0`)
    *   SSH (22) → Anywhere (`0.0.0.0/0`)
4.  Click Save Rules.

## Step 4: Launch EC2 Instances in Private Subnets
We will create three EC2 instances, each running NGINX with different content.

### 4.1 Instance 1: Homepage
1.  Go to EC2 Dashboard.
2.  Click Launch Instance.
3.  Set Instance Name: `Homepage`.
4.  AMI: Select Ubuntu 22.04.
5.  Instance Type: `t2.micro`.
6.  VPC: Select `alb-vpc`.
7.  Subnet: Select Private Subnet in AZ 1-a.
8.  Auto-assign Public IP: Disable.
9.  Security Group: Use default security group from `alb-vpc`.
10. User Data:
    ```bash
    #!bin/bash
    sudo apt update
    sudo apt install nginx -y
    sudo sed -i 's/<h1> Welcome to nginx!<\/h1>/<h1> Welcome to Homepage<\/h1>/' /var/www/html/index.nginx-debian.html
    echo '<a href="https://www.your_domain_name/movies/">Visit for Movies</a>' | sudo tee -a /var/www/html/index.nginx-debian.html
    echo '<a href="https://www.your_domain_name/shows/">Visit for Shows</a>' | sudo tee -a /var/www/html/index.nginx-debian.html
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```
11. Click Launch.

### 4.2 Instance 2: Movies
Repeat the above steps with:
*   Instance Name: `Movies`
*   Subnet: Private Subnet in AZ 1-b
*   User Data:
    ```bash
    #!bin/bash
    sudo apt update
    sudo apt install nginx -y
    sudo mkdir -p /var/www/html/movies
    sudo mv /var/www/html/index.nginx-debian.html /var/www/html/movies/index.nginx-debian.html
    sudo sed -i 's/<h1> Welcome to nginx!<\/h1>/<h1> Welcome to Movies<\/h1>/' /var/www/html/movies/index.nginx-debian.html
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```

### 4.3 Instance 3: Shows
Repeat the above steps with:
*   Instance Name: `Shows`
*   Subnet: Private Subnet in AZ 1-c
*   User Data:
    ```bash
    #!bin/bash
    sudo apt update
    sudo apt install nginx -y
    sudo mkdir -p /var/www/html/shows
    sudo mv /var/www/html/index.nginx-debian.html /var/www/html/shows/index.nginx-debian.html
    sudo sed -i 's/<h1> Welcome to nginx!<\/h1>/<h1> Welcome to Shows<\/h1>/' /var/www/html/shows/index.nginx-debian.html
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```

## Step 5: Create Target Groups for ALB
1.  Go to EC2 Dashboard → Target Groups.
2.  Click Create Target Group.
3.  Set Target Type: Instances.
4.  Create 3 Target Groups:
    *   `homepage-tg` → Add Homepage instance.
    *   `movies-tg` → Add Movies instance, set Health Check Path: `/movies`.
    *   `shows-tg` → Add Shows instance, set Health Check Path: `/shows`.

## Step 6: Create Application Load Balancer (ALB)
1.  Go to EC2 Dashboard → Load Balancers.
2.  Click Create Load Balancer.
3.  Select Application Load Balancer.
4.  Name: `alb-lb`.
5.  Scheme: Internet-facing.
6.  VPC: `alb-vpc`.
7.  Subnets: Select 3 public subnets.
8.  Listeners:
    *   HTTP (80) → Default Target Group: `homepage-tg`
    *   HTTPS (443) → Use ACM certificate, Default Target Group: `homepage-tg`.
9.  Click Create Load Balancer.

## Step 7: Configure Route 53
1.  Go to Route 53 → Hosted Zones.
2.  Create a Public Hosted Zone.
3.  Click Create Record:
    *   Name: `www`
    *   Type: `A`
    *   Alias: Select ALB (`alb-lb`).
4.  Click Create Record.

## Step 8: Validate Domain Access and Identify Issues
After setting up Route 53 and configuring the ALB, we need to test if the domain resolves correctly and behaves as expected.

### 8.1 Open Your Browser and Test Domain Resolution
*   Open a web browser (Chrome, Firefox, or Edge).
*   Enter `www.your_domain_name` in the address bar and press Enter.

**Expected Result:**
You should see the Homepage with the message:

`Welcome to Homepage`

Below this message, you should see two clickable links:

`Visit for Movies`
`Visit for Shows`

### 8.2 Click on the Links (Identify the Issue)

*   Click on Visit for Movies → Expected to load `www.your_domain_name/movies`.
*   Click on Visit for Shows → Expected to load `www.your_domain_name/shows`.

**Actual Behavior (Issue Identified)**
*   The links will not work, and you may see a 404 Not Found or a blank page.
*   This is because path-based routing is not configured on the ALB yet.

### 8.3 Test HTTP to HTTPS Redirection
*   In the browser, manually change `https://www.your_domain_name` to `http://www.your_domain_name`.
*   Press Enter.

**Actual Behavior (Issue Identified)**
*   The website will not automatically redirect to `https://www.your_domain_name`.
*   This is because HTTP to HTTPS redirection is not yet enabled in the ALB listener rules.

Now, we will configure HTTP to HTTPS redirection and path-based routing for movies and shows.

## Step 9: Configure HTTP to HTTPS Redirection in ALB
We need to ensure that any HTTP request (port 80) is redirected to HTTPS (port 443).

### 9.1 Navigate to ALB Listener Rules
1.  Go to AWS Console → EC2 Dashboard.
2.  Click on Load Balancers in the left panel.
3.  Select `alb-lb` (your Application Load Balancer).
4.  Click on the Listeners tab.
5.  Find the HTTP (80) Listener and click on the Rules tab.

### 9.2 Edit Default Rule for HTTP (80) Listener
1.  Click on HTTP:80 rule to expand it.
2.  Click on Actions → Edit Rule.
3.  In the Default actions section:
    *   Select Redirect to URL.
    *   Choose HTTPS (443).
    *   Select Full URL.
    *   Click Save Changes.

### 9.3 Test the Redirection
*   Open your browser and enter `http://www.your_domain_name`.
*   It should now automatically redirect to `https://www.your_domain_name`.

**Now, HTTP to HTTPS redirection is working!**

## Step 10: Configure Path-Based Routing for Movies and Shows
To allow the Movies and Shows pages to work, we need to route traffic based on the URL path.

### 10.1 Navigate to ALB Listener Rules
1.  Go to AWS Console → EC2 Dashboard.
2.  Click on Load Balancers in the left panel.
3.  Select `alb-lb` (your Application Load Balancer).
4.  Click on the Listeners tab.
5.  Find the HTTPS (443) Listener and click on the Rules tab.

### 10.2 Add Rule for Movies Path
1.  Click on HTTPS:443 rule.
2.  Click Manage Rules → Add Rule.
3.  Rule Name: `routetomovies`.
4.  Click Add Condition → Select "Path".
    Enter Path Pattern:
    `/movies/*`
5.  Routing Actions:
    *   Select Forward to Target Group.
    *   Choose `movies-tg` (the Target Group for Movies instance).
6.  Priority: 1 (Higher priority than the default rule).
7.  Click Save Rule.

**Now, requests to /movies/ will be routed to the Movies EC2 instance.**

### 10.3 Add Rule for Shows Path
1.  Click on HTTPS:443 rule.
2.  Click Manage Rules → Add Rule.
3.  Rule Name: `routetoshows`.
4.  Click Add Condition → Select "Path".
    Enter Path Pattern:
    `/shows/*`
5.  Routing Actions:
    *   Select Forward to Target Group.
    *   Choose `shows-tg` (the Target Group for Shows instance).
6.  Priority: 2 (Higher than the default rule but lower than `/movies/*`).
7.  Click Save Rule.

**Now, requests to /shows/ will be routed to the Shows EC2 instance.**

## Step 11: Final Testing
Now that we have configured HTTPS redirection and path-based routing, let’s verify everything.

### 11.1 Test Main Domain
Open a browser and enter:

`https://www.your_domain_name`

You should see:

`Welcome to Homepage`
`Visit for Movies`
`Visit for Shows`

### 11.2 Test Movies Page
Click on Visit for Movies, or enter:
`https://www.your_domain_name/movies`

You should see:

`Welcome to Movies`

### 11.3 Test Shows Page
Click on Visit for Shows, or enter:

`https://www.your_domain_name/shows`

You should see:

`Welcome to Shows`

### 11.4 Test HTTP to HTTPS Redirection
*   Enter `http://www.your_domain_name` in the browser.
*   It should automatically redirect to `https://www.your_domain_name`.

**Everything is working correctly!**

## Summary of What We Accomplished
1.  Issued an SSL/TLS Certificate using AWS ACM.
2.  Created a VPC with public and private subnets.
3.  Launched EC2 Instances (Homepage, Movies, Shows) in private subnets.
4.  Created Target Groups for ALB.
5.  Created an ALB (Application Load Balancer) and linked it to EC2 instances.
6.  Configured Route 53 to map a domain to the ALB.
7.  Enabled HTTP to HTTPS Redirection in ALB rules.
8.  Configured Path-Based Routing to forward requests to `/movies/` and `/shows/`.
9.  Performed Testing to validate redirections and routing.

**Your ALB is now fully functional, secure, and correctly routing traffic!**

## What to Expect After Implementing This Project
After successfully implementing this project, users can expect the following outcomes:

### 1. Secure and Scalable Web Traffic Management

*   **HTTPS-Enabled Website:** All traffic will be securely redirected from HTTP to HTTPS using an SSL/TLS certificate issued by AWS Certificate Manager (ACM).
*   **Automatic Redirection:** Any HTTP request will automatically be redirected to HTTPS, ensuring encrypted communication.

### 2. High Availability and Load Balancing

*   **Application Load Balancer (ALB) in Multi-AZ:** The ALB will distribute incoming requests across multiple EC2 instances deployed in different Availability Zones for high availability and fault tolerance.
*   **Efficient Traffic Distribution:** The ALB ensures that application requests are evenly distributed, preventing overload on any single instance.

### 3. Domain Name Resolution with Route 53

*   **Custom Domain Name Integration:** Users can access the application via a friendly domain name instead of an IP address.
*   **Reliable and Fast DNS Resolution:** AWS Route 53 provides low-latency, highly available DNS resolution for seamless domain-to-ALB mapping.

### 4. Path-Based Routing for Different Sections

*   **Separate URLs for Different Application Sections:**
    *   `/` → Homepage (Default Page)
    *   `/movies` → Movie Section
    *   `/shows` → TV Shows Section
*   **Efficient Routing via ALB:** The ALB directs requests to the appropriate backend EC2 instance based on the requested path.

### 5. Secure and Well-Architected AWS Infrastructure

*   **Private and Public Subnet Segmentation:** EC2 instances are securely deployed in private subnets, while the ALB operates in public subnets.
*   **Improved Security with Security Groups:** Strict security group rules ensure that only ALB can communicate with EC2 instances, reducing direct exposure to the internet.
*   **NAT Gateway for Secure Outbound Internet Access:** EC2 instances in private subnets can securely access external resources if needed (e.g., for software updates).

### 6. Enhanced User Experience

*   **Seamless Access with HTTPS:** Users can securely browse the application without worrying about insecure connections.
*   **Improved Performance:** The ALB optimizes request handling and ensures low latency for users accessing different sections of the site.
*   **Resilient and Fault-Tolerant Setup:** Even if one EC2 instance fails, the ALB will automatically redirect traffic to healthy instances, minimizing downtime.

## Final Outcome
After completing this project, users will have a fully functional, production-ready AWS setup with:

*   A secure, SSL-enabled web application
*   Seamless HTTP to HTTPS redirection
*   Custom domain resolution via AWS Route 53
*   Path-based request routing for different sections
*   A highly available and scalable infrastructure

This project not only demonstrates best practices in AWS architecture but also prepares users for real-world cloud deployments where security, scalability, and high availability are critical.

## Conclusion & What’s Next?
Congratulations on successfully implementing this AWS project!

You have taken another significant step toward mastering AWS infrastructure by setting up a robust, secure, and scalable environment with Route 53, ALB, and ACM.

### Key Takeaways from This Project
Through this hands-on implementation, you have learned how to:

*   Configure Route 53 for domain name resolution and routing traffic efficiently.
*   Set up an Application Load Balancer (ALB) for high availability and traffic distribution.
*   Secure your applications with AWS Certificate Manager (ACM) and enforce HTTPS.
*   Implement path-based routing to optimize request management.
*   Enhance security with private and public subnet architectures.

This project not only strengthens your AWS skills but also prepares you for real-world cloud environments where reliability, security, and scalability are key factors.

### What’s Next?
This is just one of many AWS projects to come!

AWS is vast, and to truly master it, we will explore new services and advanced concepts every day with practical real-world implementations.

#### Upcoming AWS Projects:

*   Amazon S3 – Scalable and cost-effective object storage
*   AWS Lambda – Serverless computing for event-driven automation
*   Amazon RDS – Managed relational databases with high availability
*   AWS IAM – Secure access management for AWS resources

And many more to come :

Each project will include a step-by-step detailed guide just like this one, ensuring you gain practical expertise in AWS and DevOps.

Follow along as we unlock one AWS service at a time, turning concepts into real-world implementations!

Stay tuned – A new AWS project drops daily!
Let’s keep learning, keep building, and take your AWS journey to the next level!

