# 01 - Real-Time Task: Step-by-Step Implementation of AWS Transit Gateway with Multi-Region Connectivity

This guide provides a step-by-step implementation of AWS Transit Gateway with Multi-Region Connectivity between North Virginia and Ohio.

## 1️⃣ Create VPCs in North Virginia Region

### Create VPC1 (North Virginia)

1.  Go to AWS Console > North Virginia Region
2.  Navigate to VPC > Your VPCs > Create VPC and more
3.  Configure the VPC as follows:
    *   Name: `vpc1`
    *   AZs: `1`
    *   Number of Public Subnets: `1`
    *   Number of Private Subnets: `0` (No private subnet)
    *   NAT Gateway: `None`
    *   VPC Endpoints: `None`
4.  Click `Create VPC`

### Create VPC2 (North Virginia)

1.  Navigate to VPC > Your VPCs > Create VPC and more
2.  Configure the VPC as follows:
    *   Name: `vpc2`
    *   AZs: `1`
    *   Number of Public Subnets: `1`
    *   Number of Private Subnets: `0` (No private subnet)
    *   NAT Gateway: `None`
    *   VPC Endpoints: `None`
3.  Click `Create VPC`

## 2️⃣ Modify Security Groups in VPC1 & VPC2

1.  Go to VPC > Security Groups
2.  Select the default security group of VPC1
3.  Edit inbound rules:
    *   Allow all traffic from anywhere (`0.0.0.0/0`)
4.  Save changes
5.  Repeat the same for VPC2

## 3️⃣ Create a Transit Gateway in North Virginia

1.  Go to VPC > Transit Gateway
2.  Click `Create Transit Gateway`
3.  Configure:
    *   Name: `tgw-nv`
    *   Leave all other settings as default
4.  Click `Create Transit Gateway`

## 4️⃣ Attach Transit Gateway to VPC1 and VPC2

### Attach VPC1

1.  Go to VPC > Transit Gateway Attachments
2.  Click `Create Transit Gateway Attachment`
3.  Configure:
    *   Name: `tgw-attach-vpc1`
    *   Transit Gateway: `tgw-nv`
    *   Attachment Type: `VPC`
    *   VPC: `vpc1`
    *   Subnet: Select the public subnet of VPC1
4.  Click `Create Attachment`

### Attach VPC2

1.  Click `Create Transit Gateway Attachment`
2.  Configure:
    *   Name: `tgw-attach-vpc2`
    *   Transit Gateway: `tgw-nv`
    *   Attachment Type: `VPC`
    *   VPC: `vpc2`
    *   Subnet: Select the public subnet of VPC2
3.  Click `Create Attachment`

## 5️⃣ Launch EC2 Instances in VPC1 & VPC2

### Launch Instance in VPC1

1.  Go to EC2 > Launch Instance
2.  Configure:
    *   Name: `tgw-vpc1`
    *   VPC: `vpc1`
    *   Subnet: Public subnet of VPC1
    *   Enable Auto-Assign Public IP
3.  Click `Launch`

### Launch Instance in VPC2

1.  Go to EC2 > Launch Instance
2.  Configure:
    *   Name: `tgw-vpc2`
    *   VPC: `vpc2`
    *   Subnet: Public subnet of VPC2
    *   Enable Auto-Assign Public IP
3.  Click `Launch`

## 6️⃣ Modify Route Tables in VPC1 & VPC2

### Modify Route Table of VPC1

1.  Go to VPC > Route Tables
2.  Select the Route Table of VPC1
3.  Edit routes:
    *   Destination: CIDR range of VPC2
    *   Target: Transit Gateway (`tgw-attach-vpc1`)
4.  Save changes

### Modify Route Table of VPC2

1.  Select the Route Table of VPC2
2.  Edit routes:
    *   Destination: CIDR range of VPC1
    *   Target: Transit Gateway (`tgw-attach-vpc2`)
3.  Save changes

## 7️⃣ Verify Connectivity Between VPC1 & VPC2

1.  Login to `tgw-vpc1` instance
2.  Ping the private IP of `tgw-vpc2`
3.  Login to `tgw-vpc2` instance
4.  Ping the private IP of `tgw-vpc1`
5.  Ensure successful connectivity

## 8️⃣ Create VPC3 in Ohio Region

1.  Switch to Ohio Region
2.  Go to VPC > Your VPCs > Create VPC and more
3.  Configure:
    *   Name: `vpc3`
    *   AZs: `1`
    *   Number of Public Subnets: `1`
    *   Number of Private Subnets: `0` (No private subnet)
    *   NAT Gateway: `None`
    *   VPC Endpoints: `None`
4.  Click `Create VPC`

## 9️⃣ Modify Security Group for VPC3

1.  Go to VPC > Security Groups
2.  Select the default security group of VPC3
3.  Edit inbound rules:
    *   Allow all traffic from anywhere (`0.0.0.0/0`)
4.  Save changes

## 🔟 Launch EC2 Instance in VPC3

1.  Go to EC2 > Launch Instance
2.  Configure:
    *   Name: `tgw-vpc3`
    *   VPC: `vpc3`
    *   Subnet: Public subnet of VPC3
    *   Enable Auto-Assign Public IP
3.  Click `Launch`

## 1️⃣1️⃣ Create Transit Gateway in Ohio

1.  Go to VPC > Transit Gateway
2.  Click `Create Transit Gateway`
3.  Configure:
    *   Name: `tgw-ohio`
    *   Leave all other settings as default
4.  Click `Create Transit Gateway`

## 1️⃣2️⃣ Attach VPC3 to Transit Gateway

1.  Go to VPC > Transit Gateway Attachments
2.  Click `Create Transit Gateway Attachment`
3.  Configure:
    *   Name: `tgw-attach-vpc3`
    *   Transit Gateway: `tgw-ohio`
    *   Attachment Type: `VPC`
    *   VPC: `vpc3`
    *   Subnet: Public subnet of VPC3
4.  Click `Create Attachment`

## 1️⃣3️⃣ Modify Route Table of VPC3

1.  Go to VPC > Route Tables
2.  Select the Route Table of VPC3
3.  Edit routes:
    *   Destination: CIDR of VPC1 & VPC2
    *   Target: Transit Gateway (`tgw-attach-vpc3`)
4.  Save changes

## 1️⃣4️⃣ Create Peering Between Ohio and North Virginia

1.  Go to Ohio Region > Transit Gateway Attachments
2.  Click `Create Transit Gateway Attachment`
3.  Configure:
    *   Name: `ohio-peering`
    *   Transit Gateway: `tgw-ohio`
    *   Attachment Type: `Peering Connection`
    *   Region: North Virginia
    *   Transit Gateway Acceptor: `tgw-nv`
4.  Click `Create`

## 1️⃣5️⃣ Accept Peering Request in North Virginia

1.  Switch to North Virginia
2.  Go to VPC > Transit Gateway Attachments
3.  Accept the `ohio-peering` request

## Step 16: Configure Static Routes in Transit Gateway Route Table (Ohio Region)

Now, we need to configure static routes in the Transit Gateway Route Table of the Ohio region (`tgw-ohio`) to ensure that traffic from VPC3 (Ohio) can reach VPC1 and VPC2 (North Virginia).

**Steps to Add Static Routes in Ohio Transit Gateway Route Table:**

1.  Navigate to the AWS Management Console.
2.  Go to the VPC Service in the Ohio region.
3.  In the left-hand navigation pane, click on "Transit Gateway Route Tables."
4.  Find the Transit Gateway Route Table associated with `tgw-ohio`.
5.  Click on this Transit Gateway Route Table to open its details.
6.  Go to the "Routes" tab.
7.  Click on the "Create static route" button.
    *   **First Route:**
        *   Destination CIDR: Enter the CIDR range of VPC1 (North Virginia) (e.g., `10.0.0.0/16`).
        *   Target: Select `ohio-peering` (the Transit Gateway Peering Attachment to North Virginia).
        *   Click "Create static route."
    *   **Second Route:**
        *   Destination CIDR: Enter the CIDR range of VPC2 (North Virginia) (e.g., `10.1.0.0/16`).
        *   Target: Select `ohio-peering` (the Transit Gateway Peering Attachment to North Virginia).
        *   Click "Create static route."
8.  Verify that both routes are listed in the "Routes" tab.
9.  This ensures that traffic from VPC3 in Ohio can route to VPC1 and VPC2 in North Virginia.

## Step 17: Configure Static Routes in Transit Gateway Route Table (North Virginia Region)

Now, we need to configure static routes in the Transit Gateway Route Table of North Virginia (`tgw-nv`) to ensure that traffic from VPC1 and VPC2 can reach VPC3 in Ohio.

**Steps to Add Static Routes in North Virginia Transit Gateway Route Table:**

1.  Navigate to the AWS Management Console.
2.  Go to the VPC Service in the North Virginia region.
3.  In the left-hand navigation pane, click on "Transit Gateway Route Tables."
4.  Find the Transit Gateway Route Table associated with `tgw-nv`.
5.  Click on this Transit Gateway Route Table to open its details.
6.  Go to the "Routes" tab.
7.  Click on the "Create static route" button.
    *   **First Route:**
        *   Destination CIDR: Enter the CIDR range of VPC3 (Ohio) (e.g., `10.2.0.0/16`).
        *   Target: Select `ohio-peering` (the Transit Gateway Peering Attachment to Ohio).
        *   Click "Create static route."
8.  Verify that the route is listed in the "Routes" tab.
9.  This ensures that traffic from VPC1 and VPC2 in North Virginia can reach VPC3 in Ohio.

## Final Verification Step: Test Connectivity Between Instances

Now, we will verify that instances from all VPCs can successfully communicate with each other using their private IPs.

**Steps to Verify Connectivity:**

1.  Login to the VPC1 Instance (`tgw-vpc1`)
    *   Navigate to the EC2 Service in North Virginia.
    *   Select the `tgw-vpc1` instance.
    *   Click "Connect" and choose EC2 Instance Connect (browser-based SSH).
    *   Run the following command to ping the VPC3 instance in Ohio:
        ```bash
        ping <Private-IP-of-tgw-vpc3>
        ```
    *   If successful, you should see responses from the target instance.

2.  Login to the VPC2 Instance (`tgw-vpc2`)
    *   Navigate to the EC2 Service in North Virginia.
    *   Select the `tgw-vpc2` instance.
    *   Click "Connect" and choose EC2 Instance Connect (browser-based SSH).
    *   Run the following command to ping the VPC3 instance in Ohio:
        ```bash
        ping <Private-IP-of-tgw-vpc3>
        ```
    *   If successful, you should see responses from the target instance.

3.  Login to the VPC3 Instance (`tgw-vpc3`) in Ohio
    *   Navigate to the EC2 Service in Ohio.
    *   Select the `tgw-vpc3` instance.
    *   Click "Connect" and choose EC2 Instance Connect (browser-based SSH).
    *   Run the following command to ping the VPC1 instance in North Virginia:
        ```bash
        ping <Private-IP-of-tgw-vpc1>
        ```
    *   Run the following command to ping the VPC2 instance in North Virginia:
        ```bash
        ping <Private-IP-of-tgw-vpc2>
        ```
    *   If successful, you should see responses from both instances.

**Expected Outcome:**

✅ All instances should be able to communicate with each other using private IPs, proving that our Transit Gateway peering setup is working correctly! 🚀🔥

---

*Authored by Manus AI*

