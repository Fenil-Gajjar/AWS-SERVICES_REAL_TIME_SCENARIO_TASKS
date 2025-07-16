
# 📌 Step 4: Launch Three EC2 Instances in Private Subnets (Private-1A, Private-1B, Private-1C)

🔹 Repeat the steps below for each instance (Private-1A, Private-1B, Private-1C)

🔹 Create Private Instance 1 (Private-1A)
1️⃣ Go to AWS EC2 → Launch Instance
2️⃣ Name: Private-1A
3️⃣ Select Amazon Linux 2 or Ubuntu
4️⃣ Instance Type: t2.micro
5️⃣ Select VPC: nlb-vpc
6️⃣ Select Subnet: private-subnet-1A (First Private Subnet)
7️⃣ Disable Auto-Assign Public IP
8️⃣ Select Default Security Group (configured in Step 3)
9️⃣ Under Advanced Details → User Data → Paste the script below:
```bash
#!/bin/bash
sudo apt update
sudo apt install nginx -y
sudo systemctl restart nginx
sudo systemctl enable nginx
echo "<h1> $(cat /etc/hostname)</h1>" >> /var/www/html/index.nginx-debian.html
echo "<h1> US-EAST-1A-SERVERS</h1>" >> /var/www/html/index.nginx-debian.html
```
🔟 Click Launch Instance

🔹 Create Private Instance 2 (Private-1B)
Repeat the steps for Private-1A, but:
- Name it Private-1B
- Select private-subnet-1B
- Change US-EAST-1A to US-EAST-1B in the script.

🔹 Create Private Instance 3 (Private-1C)
Repeat the steps for Private-1A, but:
- Name it Private-1C
- Select private-subnet-1C
- Change US-EAST-1A to US-EAST-1C in the script.


