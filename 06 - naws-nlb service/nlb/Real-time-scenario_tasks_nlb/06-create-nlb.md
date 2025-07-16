
# 📌 Step 6: Create the Network Load Balancer (NLB)
🔹 Navigate to AWS EC2 → Load Balancers

1️⃣ Click Create Load Balancer → Network Load Balancer
2️⃣ Name: nlb-lb
3️⃣ Scheme: Internet-facing
4️⃣ VPC: nlb-vpc
5️⃣ Subnets: All 3 Public Subnets (1A, 1B, 1C)
6️⃣ Listeners & Routing:
- Listener 1 → TCP Port 80 → Target Group: nlb-tg
- Listener 2 → TLS Port 443 → Target Group: nlb-tg
7️⃣ Select SSL/TLS Certificate → Choose the issued ACM Certificate
8️⃣ Click Create Load Balancer


