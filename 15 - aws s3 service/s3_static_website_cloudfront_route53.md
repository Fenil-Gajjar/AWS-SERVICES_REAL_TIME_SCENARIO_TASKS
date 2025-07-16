# Real Time Task on Step By Step Implementation of S3 Static Website with Cloudfront and Route53

## About This Guide

In today’s digital world, fast, secure, and scalable websites are essential. AWS offers a powerful and cost-effective way to host static websites using Amazon S3, CloudFront, and Route 53. This guide will walk you through a real-world implementation of setting up a highly available, secure, and globally distributed static website step by step.

## What You’ll Learn

By following this guide, you will:

*   Set up an S3 bucket for website hosting
*   Configure CloudFront for global content delivery & performance optimization
*   Integrate Route 53 for domain name mapping
*   Enforce HTTPS security for a secure browsing experience
*   Deploy a fully functional static website with AWS best practices

## Why This Setup?

*   **Scalability** – Handles thousands of requests seamlessly.
*   **Speed & Performance** – CloudFront’s CDN accelerates content delivery worldwide.
*   **Security** – HTTPS encryption and access control enhance website protection.
*   **Cost-Effective** – Pay only for what you use, with minimal operational overhead.

## Let’s Get Started!

This hands-on task ensures you not only learn the theory but also implement a production-grade static website using AWS services. Follow the steps carefully, and by the end, you’ll have a fully operational website with a custom domain, secured with SSL, and delivered at blazing speed across the globe!

Let’s dive in!

## Step 1: Create and Configure the S3 Bucket

### 1.1 Create the S3 Bucket

1.  Log in to the AWS Management Console and navigate to the S3 service.
2.  Click **Create bucket**.
3.  In the **Bucket name** field, enter a unique name (e.g., `my-example-bucket`).
4.  Under **Access Control List (ACL) settings**, select the option that enables ACLs (if not already enabled).
5.  **Block Public Access Settings:**
    *   Uncheck **Block all public access**.
    *   Confirm your intent if prompted.
6.  **Bucket Versioning:**
    *   Enable Bucket versioning to keep versions of objects.
7.  Click **Create bucket**.

### 1.2 Upload Website Content to the S3 Bucket

1.  Open the newly created bucket.
2.  Click **Upload**.
3.  Upload your website file(s) – at a minimum, an `index.html` file containing your website content.
4.  After uploading, select the uploaded object(s) and configure permissions:
    *   Grant `public-read` access so that the content is available publicly.
5.  Confirm that your object now shows `public-read` permissions.

### 1.3 Enable Static Website Hosting on the S3 Bucket

1.  In your S3 bucket, click on the **Permissions** tab.
2.  Scroll down to the **Static website hosting** section.
3.  Select **Enable**.
4.  For the **Index document**, enter `index.html`.
5.  (Optionally, specify an error document.)
6.  Click **Save changes**.

## Step 2: Create a CloudFront Distribution

### 2.1 Launch a New CloudFront Distribution

1.  Open the AWS Management Console and navigate to CloudFront.
2.  Click **Create Distribution**.
3.  Under **Web** (the distribution method for web content), click **Get Started**.

### 2.2 Configure the Origin Settings

1.  **Origin Domain Name:**
    *   From the dropdown, select your S3 bucket (it will appear as something like `my-example-bucket.s3.amazonaws.com`).
2.  **Origin Access:**
    *   Under **Origin Access**, choose **Legacy OAI** and select or create an Origin Access Identity (OAI) that grants CloudFront permission to access your bucket.
    *   (Ensure that your S3 bucket policy allows access from this OAI if needed.)

### 2.3 Configure Viewer Protocol Policy

1.  In the **Default Cache Behavior Settings**, find **Viewer Protocol Policy**.
2.  Select **Redirect HTTP to HTTPS** to ensure all requests use a secure connection.

### 2.4 Enable Web Application Firewall (WAF) (Optional)

1.  Under **Web Application Firewall (WAF)**, choose **Enable security protections** if you want to attach a WAF for additional security.

### 2.5 Configure Alternate Domain Names and SSL Certificate

1.  In the **Settings** section, locate **Alternate Domain Names (CNAMEs)**.
2.  Enter your domain name with the `www` prefix (e.g., `www.example.com`).
3.  Under **SSL Certificate**, select **Custom SSL Certificate (example.com)**.
    *   Choose the ACM certificate that covers `www.example.com`.

### 2.6 Set Default Root Object

1.  For **Default Root Object**, enter `index.html`.

### 2.7 Create the Distribution

1.  Review all your settings.
2.  Click **Create Distribution**.
3.  Wait for the distribution to deploy (this might take up to 20–30 minutes).
4.  Once deployed, copy the **Distribution Domain Name** (e.g., `d123456abcdef8.cloudfront.net`); you will use this in Route 53.

## Step 3: Configure Route 53 DNS

### 3.1 Create a Record to Point to CloudFront

1.  Open the AWS Management Console and navigate to Route 53.
2.  Go to **Hosted Zones** and select your hosted zone (e.g., `example.com`).
3.  Click **Create Record**.
4.  In the **Record name** field, enter `www` (this creates `www.example.com`).
5.  **Record Type:** Choose `A – IPv4 address`.
6.  **Alias:** Enable the alias option.
7.  **Alias Target:**
    *   In the dropdown, select the CloudFront distribution you created (e.g., `d123456abcdef8.cloudfront.net`).
8.  Click **Create Record**.
9.  Wait until the record status shows as `insync`.

## Step 4: Verify the Setup

1.  **DNS Propagation:**
    Open a terminal or use an online DNS checker and run:
    `nslookup www.example.com`
    *   Ensure it resolves to your CloudFront distribution.
2.  **Access the Website:**
    *   Open your web browser and navigate to `https://www.example.com`.
    *   You should see the content of your `index.html` file (hosted in your S3 bucket).

## Summary: What This Scenario Does

*   **S3 Bucket:**
    *   Creates a bucket that hosts your static website content (e.g., `index.html`).
    *   Configured for static website hosting with public-read permissions.
*   **CloudFront Distribution:**
    *   Serves as a global content delivery network (CDN) that caches and delivers your website content.
    *   Redirects HTTP requests to HTTPS and uses your ACM certificate for secure connections.
    *   Uses an Origin Access Identity (OAI) to securely access the S3 bucket.
*   **Route 53 DNS:**
    *   Creates a DNS record (`www.example.com`) that points to the CloudFront distribution.
    *   Ensures that when users visit your domain, they are routed to your CloudFront distribution, which in turn serves the content from S3.
*   **Overall Outcome:**
    *   A secure, highly available, and globally distributed static website hosted on S3, served through CloudFront, and accessed via your custom domain (`www.example.com`).

## What Does This Scenario Do? (In Simple Terms)

This scenario sets up a static website using AWS services S3, CloudFront, and Route 53, making it secure, fast, and accessible via a custom domain.

Here’s how it works step by step:

1.  **Store Website Files in S3 (Simple Storage Service)**
    *   You create an S3 bucket and upload your website files, including an `index.html` file.
    *   You enable public access so the website can be viewed by anyone.
    *   You enable static website hosting in S3.
2.  **Use CloudFront (Content Delivery Network) for Faster & Secure Delivery**
    *   You create a CloudFront distribution that fetches your website content from S3.
    *   CloudFront caches the content across multiple locations worldwide, making it load faster.
    *   It enforces HTTPS, ensuring your website is secure.
    *   You link an SSL certificate (ACM) for a custom domain (`www.example.com`).
3.  **Route the Custom Domain to CloudFront Using Route 53**
    *   You create a DNS record in Route 53 that maps `www.example.com` to the CloudFront distribution.
    *   This allows users to visit your site using your own domain name instead of an AWS URL.

**Final Outcome:**

When someone types `www.example.com` in their browser:

*   Route 53 directs the request to CloudFront.
*   CloudFront fetches and serves the website files from S3.
*   The website loads quickly, securely (HTTPS), and globally with reduced latency.

This setup makes your website:

*   **Fast** (via CloudFront caching)
*   **Secure** (via HTTPS & SSL certificate)
*   **Scalable** (AWS handles traffic spikes)
*   **Cost-effective** (S3 + CloudFront is cheaper than traditional hosting)


