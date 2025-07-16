Here is your content, formatted for clarity and ready to copy into a document:

---

# **Website in VPC (Real Time Task)**

---

## **1. Create a VPC**

### **Step 1: Create a VPC**

1. Log in to AWS Console and go to the **VPC Dashboard**.
2. Click on **Create VPC**.
3. Provide the following details:

   * **Name tag**: `MyVPC`
   * **IPv4 CIDR block**: `10.0.0.0/16` (This defines the IP range for the VPC).
   * **IPv6 CIDR block**: Leave it as `No IPv6 CIDR Block` unless you want to enable IPv6.
   * **Tenancy**: `Default` (Unless you need dedicated instances).
4. Click on **Create VPC**.

---

## **2. Launch a Bastion Host Instance in the Public Subnet**

### **Step 2: Create Public Subnet**

1. In the VPC Dashboard, go to **Subnets** and click on **Create subnet**.
2. Choose your VPC (`MyVPC`) and provide:

   * **Subnet name**: `PublicSubnet`
   * **Availability Zone**: Choose one, e.g., `us-east-1a`.
   * **CIDR block**: `10.0.1.0/24`
3. Click on **Create subnet**.

### **Step 3: Create an Internet Gateway**

1. Go to the **VPC Dashboard** and click on **Internet Gateways**.
2. Click **Create internet gateway**.
3. Provide a name tag: `MyInternetGateway`.
4. After creation, select the gateway and click on **Actions > Attach to VPC**.

   * Choose `MyVPC` and click **Attach**.

### **Step 4: Create a Route Table for Public Subnet**

1. In the **VPC Dashboard**, go to **Route Tables** and click on **Create route table**.
2. Name it `PublicRouteTable`, and associate it with `MyVPC`.
3. After creation, select the route table and click on the **Routes** tab.
4. Click **Edit routes** and add a route:

   * **Destination**: `0.0.0.0/0` (all traffic).
   * **Target**: Select the Internet Gateway `MyInternetGateway`.
5. Click **Save routes**.
6. Go to the **Subnet Associations** tab and click **Edit subnet associations**.
7. Select the `PublicSubnet` and click **Save**.

### **Step 5: Launch Bastion Host Instance in Public Subnet**

1. Go to **EC2 Dashboard** and click **Launch Instance**.
2. Choose the **Amazon Linux 2 AMI**.
3. Select `t2.micro` (or any instance type as per your need).
4. In the **Network** section, choose `MyVPC` and `PublicSubnet`.
5. Under **Auto-assign Public IP**, choose `Enable`.
6. Configure the **Security Group** for the Bastion Host:

   * **Inbound Rules**:

     * **Type**: SSH
     * **Protocol**: TCP
     * **Port**: 22
     * **Source**: My IP (your local IP)
7. Launch the Instance and use your key pair to access it.

---

## **3. Launch Instance in Private Subnet (For Website)**

### **Step 6: Create Private Subnet**

1. Go to **Subnets** and click **Create subnet**.
2. Choose `MyVPC` and provide:

   * **Subnet name**: `PrivateSubnet`
   * **Availability Zone**: Choose the same AZ as the public subnet (or different).
   * **CIDR block**: `10.0.2.0/24`
3. Click **Create subnet**.

### **Step 7: Launch Instance in Private Subnet (for Website)**

1. Go to **EC2 Dashboard** and click **Launch Instance**.
2. Choose **Amazon Linux 2 AMI** (or any Linux-based AMI).
3. Select `t2.micro` (or your preferred type).
4. In the **Network** section, choose `MyVPC` and `PrivateSubnet`.
5. Under **Auto-assign Public IP**, select `Disable` (since it is a private instance).
6. Create a **Security Group**:

   * **Inbound Rules**:

     * **Type**: SSH
     * **Protocol**: TCP
     * **Port**: 22
     * **Source**: Select the Bastion Host security group (so only the Bastion Host can access it).
7. Launch the Instance using your key pair.

---

## **4. SSH from Bastion Host to Private Instance**

### **Step 8: SSH into Bastion Host**

1. Use your key pair to SSH into the Bastion Host in the public subnet.

### **Step 9: Copy the Key Pair to Bastion Host for Private Instance Access**

1. On the Bastion Host, upload the private key of the private instance.
2. SSH into the Private Instance using the command:

```
ssh -i /path/to/private-key.pem ec2-user@<Private-Instance-IP>
```

---

## **5. Install Web Server on Private Instance**

### **Step 10: Install Packages on Private Instance**

After logging into the private instance, run:

```bash
sudo yum update -y
sudo yum install -y httpd wget unzip
```

### **Step 11: Download and Install Website Template**

1. Go to [tooplate.com](https://www.tooplate.com) (or another template site) and choose a template you like. Copy the URL of the template's ZIP file.
2. On the private instance, run:

```bash
wget <template-url> -O /tmp/website-template.zip
```

3. Unzip the downloaded file:

```bash
sudo unzip /tmp/website-template.zip -d /tmp/
```

### **Step 12: Move the Website Files to the Web Server Directory**

```bash
sudo cp -r /tmp/<template-folder>/* /var/www/html/
```

### **Step 13: Restart and Enable HTTPD Service**

```bash
sudo systemctl restart httpd
sudo systemctl enable httpd
```

---

## **6. Set Up Load Balancer and Target Group**

### **Step 14: Create Target Group**

1. In the **EC2 Dashboard**, go to **Target Groups** and click **Create target group**.
2. Select `Instances` as the target type.
3. Choose **Protocol** as `HTTP` and **Port** as `80`.
4. Choose **VPC**: `MyVPC`.
5. Click **Next** and register the private instance (created earlier) as the target in the group.

### **Step 15: Create Load Balancer**

1. Go to the **Load Balancers** section in the EC2 Dashboard and click **Create Load Balancer**.
2. Choose **Application Load Balancer**.
3. Set the following details:

   * **Name**: `MyALB`
   * **Scheme**: Internet-facing
   * **IP address type**: IPv4
   * **Listeners**: Add a listener on port 80 (HTTP)
   * **VPC**: Choose `MyVPC`
   * **Availability Zones**: Choose the zones where your instances are located
4. Under **Security Groups**, create a security group that allows **HTTP (port 80)** from anywhere (`0.0.0.0/0`).
5. Click **Create Load Balancer**.

---

## **7. Update Private Instance Security Group to Allow Traffic from Load Balancer**

### **Step 16: Modify Security Group of Private Instance**

1. In the **EC2 Dashboard**, go to **Security Groups**.
2. Select the security group of your **Private Instance**.
3. Add an inbound rule:

   * **Type**: HTTP
   * **Port**: 80
   * **Source**: Select the Load Balancer security group
4. Click **Save rules**.

---

## **8. Access the Website via Load Balancer**

### **Step 17: Access the Website**

1. In the **EC2 Dashboard**, go to **Load Balancers**.
2. Copy the **DNS name** of your Load Balancer.
3. Open a web browser and paste the DNS name in the address bar.
4. **Your website should now be accessible through the Load Balancer's DNS name.**

---

**Now we can access our Website on the browser.**