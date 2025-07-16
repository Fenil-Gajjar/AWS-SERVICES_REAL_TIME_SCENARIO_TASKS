
# 📌 Step 9: Validate Load Balancer Traffic Distribution

Run the following script in Git Bash or any shell:

```bash
while true
do
curl -sL https://www.your_domain_name.com | grep -i 'US-EAST'
sleep 10
done
```

🔹 This ensures requests are load balanced across all zones.


