
# 📘📘 Real-Time Implementation of AWS VPC Endpoints (Gateway & Interface) 📘📘

This guide provides a step-by-step implementation of both Gateway and Interface VPC Endpoints, ensuring that no steps are skipped. These two endpoint types are interconnected, so we will first configure a Gateway Endpoint for S3, followed by Interface Endpoints for AWS Systems Manager (SSM) to enable secure and private connectivity within our VPC.

---

## 📍 Step 1: Configure Gateway Endpoint for Amazon S3

### 1.1 Create an S3 Bucket

1. Navigate to AWS Management Console → S3 → Create Bucket  
2. Provide a unique bucket name (e.g., my-vpc-endpoint-bucket).  
3. Set ACLs enabled → Keep default settings.  
4. Under Block Public Access Settings, uncheck all options (for testing purposes).  
5. Click Create Bucket.  

---

### 1.2 Upload an Image to S3

1. Open the newly created S3 bucket → Click Upload.  
2. Select an image file from your local system.  
3. Click Upload.  

---

### 1.3 Launch Public and Private EC2 Instances

#### 1.3.1 Create a Public EC2 Instance

1. Navigate to AWS Management Console → EC2 Dashboard → Launch Instance.  
2. Select Amazon Linux 2 AMI.  
3. Instance Type: t2.micro.  
4. Choose an existing VPC and select a Public Subnet.  
5. Auto-assign Public IP: Enabled.  
6. Configure Security Group:  
   - ✅ Allow SSH (22) from Anywhere (0.0.0.0/0).  
   - ✅ Allow HTTP/HTTPS (for testing).  
7. Choose an existing key pair or create a new one.  
8. Click Launch.  

#### 1.3.2 Create a Private EC2 Instance

📘📘 Follow the same steps as above with these changes:  
- Select a Private Subnet instead of a Public Subnet.  
- Auto-assign Public IP: Disabled.  

---

### 1.4 Test S3 Access from the Public Instance

📘📘 SSH into the Public Instance:
```bash
ssh -i my-key.pem ec2-user@<public-instance-public-ip>
````

📘📘 Try downloading the image from S3:

```bash
wget https://my-vpc-endpoint-bucket.s3.amazonaws.com/my-image.jpg
```

✅ The download should succeed since the Public Instance has internet access.

---

### 1.5 Transfer Key Pair and SSH into Private Instance

📘📘 Copy the key pair from Public to Private Instance:

```bash
scp -i my-key.pem my-key.pem ec2-user@<private-instance-private-ip>:~
```

📘📘 SSH into the Private Instance from the Public Instance:

```bash
ssh -i my-key.pem ec2-user@<private-instance-private-ip>
```

📘📘 Try downloading the S3 image from the Private Instance:

```bash
wget https://my-vpc-endpoint-bucket.s3.amazonaws.com/my-image.jpg
```

❌ The download will fail because the Private Instance does not have internet access.

---

### 1.6 Create a Gateway Endpoint for Amazon S3

1. Navigate to VPC Dashboard → Endpoints → Create Endpoint.
2. Name: s3-gateway-endpoint.
3. Service Category: AWS Services.
4. Service Name: Search for `com.amazonaws.<region>.s3` and select it.
5. VPC: Select your existing VPC.
6. Route Table: Choose the Private Subnet’s Route Table.
7. Click Create Endpoint.

---

### 1.7 Validate Gateway Endpoint

📘📘 SSH into the Private Instance and try downloading the image again:

```bash
wget https://my-vpc-endpoint-bucket.s3.amazonaws.com/my-image.jpg
```

✅ Now, the download should succeed! The Gateway Endpoint enables Private Instance to access S3 without a NAT Gateway.

---

## 📍 Step 2: Configure Interface Endpoints for AWS Systems Manager

### 2.1 Create an IAM Role for EC2

1. Go to IAM → Roles → Create Role.
2. Trusted Entity Type: AWS Service.
3. Use Case: Select EC2.
4. Click Next and attach the following policies:

   * ✅ AmazonSSMManagedInstanceCore
   * ✅ AmazonSSMFullAccess
5. Role Name: SSM-Role.
6. Click Create Role.

---

### 2.2 Attach IAM Role to EC2 Instances

1. Go to EC2 Dashboard → Select Public Instance → Actions → Security → Modify IAM Role.
2. Select SSM-Role and attach it.
3. Repeat the same for the Private Instance.

---

### 2.3 Verify AWS Systems Manager (SSM) Session Manager

1. Navigate to AWS Systems Manager → Session Manager.
2. Click Start Session.

❌ You will only see the Public Instance (Private Instance is missing due to lack of internet access).

---

### 2.4 Create Interface Endpoints for SSM

#### 2.4.1 Create Endpoint for `ec2messages`

1. Navigate to VPC Dashboard → Endpoints → Create Endpoint.
2. Name: ec2messages-endpoint.
3. Service Category: AWS Services.
4. Service Name: `com.amazonaws.<region>.ec2messages`.
5. VPC: Select your existing VPC.
6. Subnet: Choose the Private Subnet.
7. Enable DNS Name Resolution: ✅ Yes.
8. Click Create Endpoint.

#### 2.4.2 Create Endpoint for `ssmmessages`

📘📘 Repeat the above steps but with:

* Service Name: `com.amazonaws.<region>.ssmmessages`
* Name: ssmmessages-endpoint

#### 2.4.3 Create Endpoint for `ssm`

📘📘 Repeat the same steps but with:

* Service Name: `com.amazonaws.<region>.ssm`
* Name: ssm-endpoint

---

### 2.5 Restart Private Instance

1. Navigate to EC2 Dashboard → Select Private Instance → Instance State → Reboot.

---

### 2.6 Validate AWS Systems Manager (SSM) Access

1. Go to AWS Systems Manager → Session Manager.
2. Click Start Session.

✅ Now, both the Public and Private Instances should appear!
✅ You can now access the Private Instance via SSM Session Manager, eliminating the need for SSH or a Public IP.

---

## ✅ Final Summary

✔ Public & Private EC2 Instances Created
✔ S3 Gateway Endpoint Configured
✔ Private Instance Can Now Access S3 Without NAT
✔ SSM Interface Endpoints Configured
✔ Private Instance Can Now Be Accessed via AWS Systems Manager

By implementing VPC Endpoints, we have successfully secured and optimized access to Amazon S3 and AWS Systems Manager, without using a NAT Gateway or Public IPs.

---

## 🔐🔧 Conclusion

📘📘

With this implementation, we have successfully configured VPC Endpoints to enable secure and cost-effective access to AWS services without relying on the public internet. By setting up Gateway Endpoints for S3 and Interface Endpoints for AWS Systems Manager, we have ensured seamless communication for private instances while enhancing security and reducing costs.
