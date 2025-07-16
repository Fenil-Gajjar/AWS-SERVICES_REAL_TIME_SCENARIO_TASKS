
# 📌 Step 1: Issue an SSL/TLS Certificate in AWS Certificate Manager (ACM)

🔹 Navigate to ACM (AWS Certificate Manager):
1️⃣ Go to AWS Management Console → AWS Certificate Manager (ACM)
2️⃣ Click on Request a Certificate
3️⃣ Select Request a public certificate → Click Next
4️⃣ Enter your domain name (e.g., your_domain_name.com)
5️⃣ Select DNS Validation (recommended) → Click Next
6️⃣ Select Amazon-issued certificate → Click Review & Request
7️⃣ In the Validation Method, copy the CNAME record
8️⃣ Go to Route 53 → Select the hosted zone → Create a new record
9️⃣ Paste the copied CNAME record → Click Save

🔟 Once AWS validates the DNS entry, the certificate will be issued.


