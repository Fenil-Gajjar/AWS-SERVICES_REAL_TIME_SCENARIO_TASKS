
# 📌 Step 3: Modify the Default Security Group for the nlb-vpc

🔹 Configure Security Group Rules:
1️⃣ Go to VPC → Security Groups
2️⃣ Select the default security group of nlb-vpc
3️⃣ Click Edit inbound rules → Add rules:
- All Traffic: 0.0.0.0/0
- All TCP: 0.0.0.0/0
- HTTP (80): 0.0.0.0/0
- HTTPS (443): 0.0.0.0/0
- SSH (22): 0.0.0.0/0
4️⃣ Click Save rules


