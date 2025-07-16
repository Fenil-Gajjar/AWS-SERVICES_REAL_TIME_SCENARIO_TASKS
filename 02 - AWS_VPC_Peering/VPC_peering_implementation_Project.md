
# 🛠️ Step-by-Step Implementation of VPC Peering Across Two AWS Regions

---

## 🧱 Step 1: Create VPCs in Two AWS Regions

### 1.1 Create VPCs in North Virginia (us-east-1)

Navigate to VPC Dashboard → Create VPC and create the following VPCs:

✅ **App VPC**  
● Name: App VPC  
● CIDR: 10.0.0.0/16  

✅ **Web VPC**  
● Name: Web VPC  
● CIDR: 10.1.0.0/16  

### 1.2 Create VPC in Ohio (us-east-2)

Navigate to VPC Dashboard → Create VPC and create the following VPC:

✅ **DB VPC**  
● Name: DB VPC  
● CIDR: 10.2.0.0/16

---

## 🔐 Step 2: Configure Security Groups

Navigate to Security Groups under the VPC Dashboard and modify the default security group for each VPC:

| VPC      | Security Group | Rule                        |
|----------|----------------|-----------------------------|
| App VPC  | Default SG     | Allow all traffic (0.0.0.0/0) |
| Web VPC  | Default SG     | Allow all traffic (0.0.0.0/0) |
| DB VPC   | Default SG     | Allow all traffic (0.0.0.0/0) |

This configuration ensures unrestricted communication between the instances for testing purposes.

---

## 🚀 Step 3: Launch EC2 Instances in Each VPC

### 3.1 Launch `app_server` in App VPC

Navigate to EC2 Dashboard → Launch Instance:

✅ AMI: Amazon Linux 2  
✅ Instance Type: t2.micro  
✅ VPC: App VPC  
✅ Subnet: Choose any available subnet in App VPC  
✅ Security Group: Attach App VPC’s Default SG  
✅ Launch the instance

### 3.2 Launch `web_server` in Web VPC

✅ AMI: Amazon Linux 2  
✅ Instance Type: t2.micro  
✅ VPC: Web VPC  
✅ Subnet: Choose any available subnet in Web VPC  
✅ Security Group: Attach Web VPC’s Default SG  
✅ Launch the instance

### 3.3 Launch `db_server` in DB VPC

✅ Region: Ohio (us-east-2)  
✅ AMI: Amazon Linux 2  
✅ Instance Type: t2.micro  
✅ VPC: DB VPC  
✅ Subnet: Choose any available subnet in DB VPC  
✅ Security Group: Attach DB VPC’s Default SG  
✅ Launch the instance

---

## 🔄 Step 4: Create VPC Peering Connections & Modify Route Tables

### 4.1 App VPC to Web VPC (App_to_Web) – Same Region

**Create VPC Peering Connection:**

1. Navigate to North Virginia (us-east-1) → VPC Dashboard → Peering Connections  
2. Click Create Peering Connection  
3. Requester VPC: App VPC  
4. Accepter VPC: Web VPC  
5. Click Create, then accept the request in Peering Connections  

**Modify Route Tables:**

- **App VPC Route Table**  
  ● Destination: 10.1.0.0/16 (Web VPC CIDR)  
  ● Target: App_to_Web Peering Connection  

- **Web VPC Route Table**  
  ● Destination: 10.0.0.0/16 (App VPC CIDR)  
  ● Target: App_to_Web Peering Connection  

**Test Connectivity:**

```bash
# SSH into app_server and ping web_server
ping <web_server_private_IP>

# SSH into web_server and ping app_server
ping <app_server_private_IP>
````

---

### 4.2 DB VPC to App VPC (DB\_to\_App) – Cross-Region

**Create VPC Peering Connection:**

1. Navigate to Ohio (us-east-2) → VPC Dashboard → Peering Connections
2. Click Create Peering Connection
3. Requester VPC: DB VPC
4. Accepter VPC: App VPC (North Virginia)
5. Click Create, then switch to North Virginia and accept the request

**Modify Route Tables:**

* **DB VPC Route Table**
  ● Destination: 10.0.0.0/16 (App VPC CIDR)
  ● Target: DB\_to\_App Peering Connection

* **App VPC Route Table**
  ● Destination: 10.2.0.0/16 (DB VPC CIDR)
  ● Target: DB\_to\_App Peering Connection

**Test Connectivity:**

```bash
# SSH into db_server and ping app_server
ping <app_server_private_IP>

# SSH into app_server and ping db_server
ping <db_server_private_IP>
```

---

### 4.3 Web VPC to DB VPC (Web\_to\_DB) – Cross-Region

**Create VPC Peering Connection:**

1. Navigate to North Virginia (us-east-1) → VPC Dashboard → Peering Connections
2. Click Create Peering Connection
3. Requester VPC: Web VPC
4. Accepter VPC: DB VPC (Ohio)
5. Click Create, then switch to Ohio and accept the request

**Modify Route Tables:**

* **Web VPC Route Table**
  ● Destination: 10.2.0.0/16 (DB VPC CIDR)
  ● Target: Web\_to\_DB Peering Connection

* **DB VPC Route Table**
  ● Destination: 10.1.0.0/16 (Web VPC CIDR)
  ● Target: Web\_to\_DB Peering Connection

**Test Connectivity:**

```bash
# SSH into web_server and ping db_server
ping <db_server_private_IP>

# SSH into db_server and ping web_server
ping <web_server_private_IP>
```

---

## ✅ Final Validation

✔ App Server ↔ Web Server (VPC Peering: App\_to\_Web)
✔ DB Server ↔ App Server (VPC Peering: DB\_to\_App)
✔ Web Server ↔ DB Server (VPC Peering: Web\_to\_DB)

---

## 🏁 Congratulations!

We have successfully set up multi-region VPC Peering with proper security groups and route tables for seamless communication between instances.

---

## 📌 Conclusion

In this real-time VPC Peering implementation task, we successfully established secure and efficient connectivity between three VPCs across two AWS regions:

✅ App VPC ↔ Web VPC (Same Region Peering: App\_to\_Web)
✅ DB VPC ↔ App VPC (Cross-Region Peering: DB\_to\_App)
✅ Web VPC ↔ DB VPC (Cross-Region Peering: Web\_to\_DB)

By carefully configuring VPCs, security groups, route tables, and peering connections, we achieved seamless communication between EC2 instances in different VPCs.

