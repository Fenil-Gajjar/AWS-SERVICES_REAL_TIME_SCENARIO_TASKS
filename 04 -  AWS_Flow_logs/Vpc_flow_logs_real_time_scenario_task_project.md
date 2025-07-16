
# ✅ Real time Task on Step-by-Step Implementation of AWS VPC Flow Logs

## Overview

AWS VPC Flow Logs provide deep insights into network traffic, helping you monitor, troubleshoot, and secure your infrastructure. This guide walks you through the end-to-end setup of VPC Flow Logs, storing logs in an S3 bucket, and validating them by generating real-time network traffic.

By the end of this guide, you will have:
✅ A fully configured VPC Flow Logs setup  
✅ Traffic logs stored in an S3 bucket  
✅ Verified logs by generating and analyzing network traffic  

Let’s dive into the hands-on implementation!

---

## 1. Launch an EC2 Instance for Testing

To generate network traffic, we will launch an EC2 instance, install Nginx, and use it for testing.

### Step 1: Launch an EC2 Instance

1. Navigate to **AWS Management Console → EC2 → Instances**  
2. Click **Launch Instance**  
3. Configure the instance:  
   - **Instance Name** → `flowlogs`  
   - **Amazon Machine Image (AMI)** → Amazon Linux 2 (or Ubuntu)  
   - **Instance Type** → `t2.micro` (Free Tier)  
   - **VPC** → Select an existing VPC or create a new one  
   - **Subnet** → Select a Public Subnet  
   - **Auto-assign Public IP** → ✅ Enabled  
4. **Security Group Configuration**:  
   - Create a new security group `flowlogs-sg`  
   - Allow SSH (port 22) from your IP  
   - Allow HTTP (port 80) from anywhere (0.0.0.0/0)  
5. **Key Pair**:  
   - Select an existing key pair or create a new one  
6. Click **Launch Instance**

### Step 2: Install Nginx on the EC2 Instance

```bash
# Connect to the instance via SSH:
ssh -i your-key.pem ec2-user@your-instance-public-ip

# Update the system:
sudo yum update -y

# Install and start Nginx:
sudo amazon-linux-extras enable nginx1
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify Nginx is running:
systemctl status nginx
````

* **Test the web server**:
  Open your browser and visit:
  `http://your-instance-public-ip`
  ✅ You should see the Nginx default page.

---

## 2. Create an S3 Bucket for Storing VPC Flow Logs

To store flow logs, we will create an S3 bucket.

### Step 1: Create an S3 Bucket

1. Navigate to **AWS Management Console → S3**
2. Click **Create bucket**
3. Configure the bucket:

   * **Bucket Name**: `my-vpc-flowlogs-bucket-12345` (Ensure uniqueness)
   * **Block Public Access**: ✅ Enabled (Recommended for security)
4. Click **Create bucket**
   ✅ The S3 bucket is now ready to store VPC Flow Logs.

---

## 3. Enable VPC Flow Logs and Store Logs in S3

Now, we will enable VPC Flow Logs for our VPC and direct the logs to the S3 bucket.

### Step 1: Identify the VPC

1. Navigate to **AWS Management Console → VPC**
2. Click **Your VPCs**
3. Locate the VPC where the `flowlogs` instance is running

### Step 2: Create a VPC Flow Log

1. Click on the **VPC ID** of your selected VPC
2. Go to the **Flow Logs** tab
3. Click **Create Flow Log**
4. Configure the flow log:

   * **Flow Log Name**: `my-vpc-flowlogs`
   * **Filter**: `All` (Logs both accepted and rejected traffic)
   * **Maximum Aggregation Interval**: 1 minute
   * **Destination**: Send to an Amazon S3 bucket
   * **S3 Bucket ARN**:
     Find your bucket ARN in **S3 → Your Bucket → Properties**
     `arn:aws:s3:::my-vpc-flowlogs-bucket-12345`
5. Click **Create Flow Log**
   ✅ Flow Logs are now enabled and will be stored in S3.

---

## 4. Generate Network Traffic to Capture Flow Logs

Since VPC Flow Logs only capture active network traffic, we need to generate traffic to verify that logs are being collected.

### Step 1: Verify S3 Logs Before Generating Traffic

1. Navigate to **AWS Management Console → S3**
2. Open the flow logs bucket
3. You will see no logs yet (because no traffic has been detected)

### Step 2: Generate Traffic

```bash
# Send outgoing traffic from the instance:
curl http://example.com

# Send incoming traffic to the instance:
# Open your browser and visit:
http://your-instance-public-ip
# Refresh the page multiple times to simulate inbound requests

# Test SSH access from another machine:
ssh ec2-user@your-instance-public-ip
```

✅ This will create traffic logs that VPC Flow Logs will capture.

---

## 5. Verify Flow Logs in S3

After some time, logs will start appearing in the S3 bucket.

### Step 1: Check the S3 Bucket for Logs

1. Navigate to **AWS Management Console → S3**
2. Open your flow logs bucket
3. Navigate through the log directories:
   `AWSLogs/{account-id}/vpcflowlogs/{region}/{year}/{month}/{day}/{log-file}.gz`
4. Download and extract the log file:

```bash
gunzip your-log-file.gz
```

5. View the logs:

```bash
cat your-log-file
```

### Sample VPC Flow Log Entry

```
2 111122223333 vpc-abc123 172.31.20.1 192.168.1.1 443 51234 6 10 840 1625259374 1625259434 ACCEPT OK
```

### Breaking Down the Log Entry

| Field       | Description                 |
| ----------- | --------------------------- |
| 172.31.20.1 | Source IP (EC2 instance)    |
| 192.168.1.1 | Destination IP              |
| 443         | Port number (HTTPS request) |
| ACCEPT      | The request was allowed     |

✅ Flow Logs successfully captured network activity!

---

## ✅ Final Verification Checklist

✅ VPC Flow Logs enabled
✅ Logs stored in S3
✅ Traffic data captured and analyzed

---

## ✅ VPC Flow Logs setup is now successfully completed!

---

## Conclusion

In this guide, we successfully implemented VPC Flow Logs to monitor network traffic within an AWS VPC. We covered:

✔ Launching an EC2 instance and installing Nginx to generate network traffic.
✔ Creating an S3 bucket to store VPC Flow Logs.
✔ Enabling and configuring VPC Flow Logs for real-time traffic monitoring.
✔ Generating and verifying network traffic to ensure logs are being captured.
✔ Accessing and analyzing logs stored in S3.

By implementing VPC Flow Logs, we gained visibility into network activity, helping with troubleshooting, security analysis, and compliance monitoring. This setup enables better network auditing, performance optimization, and anomaly detection within your AWS environment.

```