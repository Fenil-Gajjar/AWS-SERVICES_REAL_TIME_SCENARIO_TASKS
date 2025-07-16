# Real-Time Task: Step-by-Step Implementation of IAM Policies

Welcome to this hands-on AWS IAM implementation task, where we focus on securing access to AWS resources by applying fine-grained permissions using IAM policies.

In today’s cloud-driven world, Identity and Access Management (IAM) is a critical component for managing user permissions and securing AWS environments. By mastering IAM, you can ensure that users and services have the right level of access—no more, no less—following security best practices.

## What You’ll Learn in This Task?

*   How to launch an EC2 instance with proper tagging.
*   How to create an IAM user with AWS Management Console access.
*   How to design a custom IAM policy with restricted permissions.
*   How to attach the policy to the IAM user for controlled access.
*   How to verify IAM permissions by logging in as the user and testing access.

This real-world scenario will help you understand how IAM policies work, ensuring secure access management for AWS resources.

Let’s dive in and implement this step-by-step!

## 🛠 Step 1: Launch an EC2 Instance with Tags

### 🔹 1.1 Log in to the AWS Console and Launch an Instance

1.  Open the AWS Management Console and navigate to EC2.
2.  Click on Launch Instance.
3.  Choose an Amazon Machine Image (AMI):
    *   Example: Amazon Linux 2 AMI or Ubuntu 20.04.
4.  Select an instance type (e.g., t2.micro for testing).
5.  Click Next: Configure Instance Details.
6.  Ensure the following configurations:
    *   Select the desired VPC (Security Group should allow all traffic).
    *   Choose a public subnet for easy access.
7.  Click Next: Add Storage, then Next: Add Tags.

### 🔹 1.2 Add Tags to the Instance

*   **Tag 1:**
    *   Key: `owner`
    *   Value: `testuser1`
*   **Tag 2:**
    *   Key: `Name`
    *   Value: `iam-instance`

8.  Click Next: Configure Security Group.
9.  Select the default security group (allows all traffic).
10. Click Review and Launch, then Launch the instance.

✅ Select (or create) a key pair and click Launch Instances.

## 🛠 Step 2: Create an IAM User

### 🔹 2.1 Create a User (testuser1)

1.  Open the AWS Management Console and navigate to IAM.
2.  Click on Users in the left-hand panel.
3.  Click Add users and enter the username: `testuser1`.
4.  Select Access Type:
    *   Enable AWS Management Console access.
5.  Set Console Password:
    *   Select Custom password and enter a secure password.
    *   Uncheck "User must create a new password at next sign-in" (for easy login).
6.  Click Next: Permissions.

### 🔹 2.2 Create User Without Group Membership

7.  On the Set Permissions page, choose Attach policies directly but do not select any policies at this stage.
8.  Click Next: Tags (Optional: Add tags for better tracking).
9.  Click Next: Review, verify the details, and click Create User.

## 🛠 Step 3: Create a Custom IAM Policy

### 🔹 3.1 Navigate to IAM Policies and Create a New Policy

1.  In the IAM Console, click on Policies in the left menu.
2.  Click Create policy.
3.  Select the JSON tab and paste the following policy JSON:

```json
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": [
"ec2:DescribeInstanceTypes",
"ec2:DescribeInstances",
"ec2:DescribeInstanceStatus",
"route53:*",
"s3:*"
],
"Resource": "*"
}
]
}
```

### 🔹 Policy Explanation:

*   Allows describing EC2 instances, instance types, and status.
*   Grants full access to Route 53.
*   Grants full access to S3.

4.  Click Next: Tags (Optional).
5.  Click Next: Review.
6.  Policy Name: `testuser-policy`.
7.  (Optional: Add a description).
8.  Click Create policy.

## 🛠 Step 4: Attach the Custom Policy to the User

### 🔹 4.1 Assign Policy to testuser1

1.  In the IAM Console, go to Users.
2.  Select the user `testuser1`.
3.  Click on the Permissions tab.
4.  Click Add Permissions → Attach Policies Directly.
5.  Search for `testuser-policy` and select it.
6.  Click Next: Review, then Add permissions.

✅ Permissions are now assigned!

## 🛠 Step 5: Verify User Permissions

### 🔹 5.1 Log in as testuser1 and Validate Access

1.  Open a new browser window (or Incognito mode).
2.  Navigate to the AWS Console Sign-in Page.
3.  Enter the AWS Account ID or Alias.
4.  Enter Username: `testuser1` and the Custom Password you set earlier.
5.  Log in and go to the IAM Dashboard → Permissions for `testuser1`.
6.  Verify that the user has access to EC2 descriptions, Route 53, and S3 as per the policy.

## 🎯 Task Completed Successfully! 🚀

You have successfully:

*   Launched an EC2 instance and tagged it.
*   Created an IAM user (`testuser1`).
*   Defined a custom IAM policy.
*   Assigned the policy to the user.
*   Verified that the user has the required permissions.

This step-by-step task helps in real-world IAM implementation, ensuring secure and controlled access to AWS resources.

