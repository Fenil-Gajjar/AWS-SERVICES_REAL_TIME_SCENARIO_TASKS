
# Real Time Task on Step By Step Implementation IAM Roles and IAM Assume Roles

## Part 1: Accessing a Specific S3 Bucket via an IAM Role

### Step 1: Create Two EC2 Instances (Testing1 and Testing2)

#### 1.1 Launch Testing1 Instance

1.  Log in to the AWS Management Console and navigate to EC2.
2.  Click Launch Instance.
3.  Choose an appropriate AMI (e.g., Ubuntu 20.04 LTS).
4.  Choose an instance type (e.g., t2.micro).
5.  Under Configure Instance Details:
    *   Select your VPC (ensure its security group allows all traffic from anywhere).
    *   Select a Public Subnet.
    *   Ensure Auto-assign Public IP is enabled.
6.  Under Add Tags, add:
    *   Tag Key: `Name` → Value: `Testing1`
7.  Complete the launch process.

#### 1.2 Launch Testing2 Instance

1.  Repeat the steps above to launch a second EC2 instance.
2.  Under Add Tags, add:
    *   Tag Key: `Name` → Value: `Testing2`
3.  Complete the launch process.

### Step 2: Create an S3 Bucket (testing-s3)

1.  Go to the AWS Management Console → S3.
2.  Click Create Bucket.
3.  Bucket Name: Enter `testing-s3` (ensure the name is unique across AWS).
4.  Leave the default settings (you may leave Block Public Access as-is if you do not need public access; for our testing, access is controlled via IAM roles).
5.  Click Create Bucket.

### Step 3: Create an IAM Role for EC2 (main-role1)

1.  Open the AWS Management Console and navigate to IAM.
2.  In the left-hand menu, click on Roles.
3.  Click Create role.
4.  Under Select trusted entity, choose AWS service.
5.  Under Use Case for other AWS services, select EC2.
6.  Click Next: Permissions (do not attach any policies yet).
7.  Click Next: Tags (optional to add tags).
8.  Click Next: Review.
9.  Role Name: Enter `main-role1`.
10. Click Create role.

### Step 4: Attach an Inline Policy to main-role1 for S3 Bucket Access

1.  In the IAM console, navigate to Roles and select `main-role1`.
2.  Click the Permissions tab.
3.  Click Add permissions → Create inline policy.

Select the JSON tab and paste the following policy (update the bucket name if necessary):

```json
{
"Version": "2012-10-17",
"Statement": [
{
"Sid": "AllowFullAccessToTestingS3",
"Effect": "Allow",
"Action": "s3:*",
"Resource": [
"arn:aws:s3:::testing-s3",
"arn:aws:s3:::testing-s3/*"
]
}
]
}
```

4.  Click Next: Tags (optional).
5.  Policy Name: Enter `s3-full-access-main`.
6.  Click Create policy.

### Step 5: Attach the IAM Role (main-role1) to Testing1 Instance

1.  Navigate to EC2 → Instances.
2.  Select the instance named `Testing1`.
3.  Click on Actions → Security → Modify IAM Role.
4.  Choose the IAM role `main-role1` from the dropdown.
5.  Click Update IAM role.

### Step 6: Test S3 Access from Testing1 Instance

1.  SSH into `Testing1` (using your key pair and public IP).

On the instance, create an empty file:

```bash
echo "Test file content" > testfile.txt
```

Copy the file to the S3 bucket:

```bash
aws s3 cp testfile.txt s3://testing-s3/
```

2.  If the IAM role is correctly attached and the policy is in effect, the file should be uploaded successfully.

## Part 2: Attaching the Role to a User and Assuming the Role

### Step 7: Create an IAM User (main-user)

1.  In the IAM Console, click on Users.
2.  Click Add users.
3.  User Name: Enter `main-user`.
4.  Select Access type:
    *   Check Programmatic access (to get access keys).
5.  Click Next: Permissions.
6.  Choose Attach policies directly (for now, we will attach an inline policy later) and click Next: Tags.
7.  (Optional) Add tags.
8.  Click Next: Review.
9.  Click Create user.
10. Note down the Access Key ID and Secret Access Key (or download the CSV file).

### Step 8: Create an Inline Policy for main-user to Allow Assuming main-role1

1.  In the IAM Console, navigate to Users and select `main-user`.
2.  Go to the Permissions tab.
3.  Click Add permissions → Create inline policy.

Select the JSON tab and paste the following:

```json
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": "sts:AssumeRole",
"Resource":
"arn:aws:iam::<YOUR_ACCOUNT_ID>:role/main-role1"
}
]
}
```

4.  Replace `<YOUR_ACCOUNT_ID>` with your AWS account ID.
5.  Click Next: Tags (optional).
6.  Policy Name: Enter `assuming-role`.
7.  Click Create policy.

### Step 9: Update the Trust Relationship for main-role1

1.  In the IAM Console, navigate to Roles and select `main-role1`.
2.  Click on the Trust relationships tab.
3.  Click Edit trust policy.

Replace the trust policy with the following (update the ARN for `main-user` and add services if needed):

```json
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Principal": {
"AWS":
"arn:aws:iam::<YOUR_ACCOUNT_ID>:user/main-user"
},
"Action": "sts:AssumeRole"
},
{
"Effect": "Allow",
"Principal": {
"Service": [
"ec2.amazonaws.com",
"lambda.amazonaws.com",
"ssm.amazonaws.com"
]
},
"Action": "sts:AssumeRole"
}
]
}
```

4.  Click Update Policy.

### Step 10: Generate Access Keys for main-user

1.  In the IAM Console, go to Users → Select `main-user`.
2.  Click on the Security Credentials tab.
3.  Under Access keys (access key ID and secret access key), click Create access key.
4.  Choose Other and click Create access key.
5.  Copy the Access Key ID and Secret Access Key (or download the CSV file).

### Step 11: Assume Role from Testing2 Instance

#### 11.1 Configure AWS CLI on Testing2 Instance

1.  SSH into `Testing2` (using your key pair and public IP).

Run the command to configure AWS CLI:

```bash
aws configure
```

2.  Enter the Access Key ID and Secret Access Key of `main-user`.
    *   Set the default region as your region (e.g., `us-east-1`).
    *   Set the default output format as `json` (or your preference).

Verify the configuration:

```bash
aws sts get-caller-identity
```

3.  This should return details for `main-user`.

#### 11.2 Assume the Role main-role1 Using AWS CLI

Run the following command on `Testing2`:

```bash
aws sts assume-role --role-arn \
"arn:aws:iam::<YOUR_ACCOUNT_ID>:role/main-role1" \
--role-session-name "testSession"
```

The command returns temporary credentials (AccessKeyId, SecretAccessKey, SessionToken).

For example:

```json
{
"Credentials": {
"AccessKeyId": "ASIA...",
"SecretAccessKey": "abc123...",
"SessionToken": "IQoJb3JpZ2luX2VjE...",
"Expiration": "2024-04-01T12:00:00Z"
},
"AssumedRoleUser": {
"AssumedRoleId": "AROA...",
"Arn":
"arn:aws:iam::<YOUR_ACCOUNT_ID>:role/main-role1/testSession"
}
}
```

1.  Export these temporary credentials as environment variables:

```bash
export AWS_ACCESS_KEY_ID=<Temporary AccessKeyId>
export AWS_SECRET_ACCESS_KEY=<Temporary SecretAccessKey>
export AWS_SESSION_TOKEN=<Temporary SessionToken>
```

#### 11.3 Test S3 Access Using Assumed Role Credentials

On `Testing2`, create an empty file:

```bash
echo "Hello from assumed role" > testfile.txt
```

Copy the file to the S3 bucket:

```bash
aws s3 cp testfile.txt s3://testing-s3/
```

1.  If the assumed role permissions are correctly applied, the file should successfully copy into the S3 bucket.

## Understanding Part 2: IAM Role Attachment & Assuming a Role via an IAM User

In Part 2, we are setting up IAM Role Assumption, which allows an IAM user (instead of an EC2 instance) to temporarily assume a role with higher permissions. This is useful for granting users temporary elevated access to AWS resources without directly assigning those permissions to them.

### Breakdown of Part 2: IAM Role Assumption

The goal of Part 2 is to allow a user (`main-user`) to assume the IAM role (`main-role1`) and access the `testing-s3` bucket from an EC2 instance (`Testing2`), without directly assigning the S3 access policy to the user.

### Why Do We Need This?

*   Instead of granting permanent S3 permissions to the `main-user`, we allow them to "assume" the role `main-role1` dynamically when needed.
*   This follows the Principle of Least Privilege: The user gets temporary permissions only when needed.
*   This is a common AWS security best practice.

### How Does This Work?

1.  **Create an IAM User (`main-user`)**
    *   This user does not have S3 access directly.
    *   Instead, they will have permission to assume `main-role1`.
2.  **Modify `main-role1` to Allow `main-user` to Assume It**
    *   Edit the Trust Relationship of `main-role1` to allow `main-user` to assume the role.
    *   This means AWS allows `main-user` to "borrow" the permissions of `main-role1`.
3.  **Generate AWS Credentials for `main-user`**
    *   We generate Access Key and Secret Key for `main-user`.
4.  **Configure AWS CLI on `Testing2`**
    *   Use `aws configure` to set up `main-user` on `Testing2`.
    *   Confirm identity using `aws sts get-caller-identity`.
5.  **Assume the Role (`main-role1`) Using AWS CLI**
    *   Run the `aws sts assume-role` command to get temporary credentials.
    *   Use these credentials to access the `testing-s3` bucket.

### Final Outcome

*   `Testing1` (EC2 instance) has direct access to the `testing-s3` bucket via `main-role1`.
*   `main-user` (IAM user) does not have direct access but can assume `main-role1` to get temporary access.
*   `Testing2` (EC2 instance) can act as `main-user` and assume `main-role1` dynamically to access `testing-s3`.

This approach ensures that the IAM user doesn't have persistent permissions but can get temporary access whenever needed.

### Why is This Useful in Real-Life?

1.  **Security Best Practices** → Instead of assigning permissions to users directly, they assume a role only when needed.
2.  **Temporary Access** → Users can access specific AWS resources without permanent credentials.
3.  **Multi-User Access** → Any user who has permission to assume `main-role1` can access `testing-s3`, without directly modifying IAM permissions.
4.  **Separation of Concerns** → IAM users and IAM roles remain separate entities.

### Duration of Temporary Credentials

Temporary credentials obtained using the `aws sts assume-role` command have a default validity of 1 hour (3600 seconds). However, you can specify a different duration using the `--duration-seconds` flag.

*   **Default:** 1 hour (3600 seconds)
*   **Minimum:** 15 minutes (900 seconds)
*   **Maximum:** 12 hours (43,200 seconds) (Only if the IAM role has been configured to allow a longer session duration.)

### Checking the Role's Maximum Session Duration

Each IAM role has a maximum session duration setting. Even if you request a longer session duration using `--duration-seconds`, it cannot exceed this configured limit.

To check the maximum session duration for `main-role1`:

```bash
aws iam get-role --role-name main-role1 --query "Role.MaxSessionDuration"
```

By default, this value is 1 hour (3600 seconds) unless changed in the IAM settings.

### Extending the Session Duration

If you want the session to last longer, follow these steps:

1.  Go to IAM → Roles → `main-role1`.
2.  Click Edit under Maximum session duration.
3.  Increase it up to 12 hours.
4.  Save changes.

When assuming the role, specify the session duration:

```bash
aws sts assume-role --role-arn \
"arn:aws:iam::YOUR_AWS_ACCOUNT_ID:role/main-role1" \
--role-session-name "main-user-session" --duration-seconds 10800
```

5.  This example requests 3 hours (10,800 seconds) of validity.

### How to Check Expiry Time of Temporary Credentials

Once you assume a role, AWS provides a response containing the expiration timestamp:

```json
{
"Credentials": {
"AccessKeyId": "AKIA...",
"SecretAccessKey": "wJal...",
"SessionToken": "FQoGZXIvYXdz...",
"Expiration": "2025-02-02T12:30:00Z"
}
}
```

You can also check the expiry time with:

```bash
aws sts get-session-token
```

### What Happens After Expiry?

*   Once the session expires, the credentials stop working.
*   You need to assume the role again to get new credentials.
*   This is why temporary credentials are safer than long-term IAM user credentials.

### Key Takeaways

*   By default, temporary credentials last for 1 hour.
*   You can extend them up to 12 hours by modifying the IAM role settings.
*   After expiration, you must re-assume the role to get fresh credentials.

## Understanding the `aws sts assume-role` Command

This command is used to assume an IAM role and get temporary security credentials (Access Key, Secret Key, and Session Token) to perform actions permitted by that role.

### Breaking Down the Command

```bash
aws sts assume-role --role-arn \
"arn:aws:iam::YOUR_AWS_ACCOUNT_ID:role/main-role1" \
--role-session-name "main-user-session"
```

*   `aws sts assume-role` → Uses the AWS Security Token Service (STS) to assume a specified IAM role.
*   `--role-arn "arn:aws:iam::YOUR_AWS_ACCOUNT_ID:role/main-role1"` → Specifies the ARN (Amazon Resource Name) of the role (`main-role1`) that we want to assume.
*   `--role-session-name "main-user-session"` → Assigns a session name for the assumed role session. This is just a label for tracking purposes.

### Why Do We Use This Command?

1.  **To Access AWS Resources Using a Role Instead of IAM User Credentials**
    *   IAM roles do not have long-term credentials like IAM users.
    *   You must assume a role to obtain temporary credentials.
2.  **To Get Temporary Credentials**

This command returns a response with temporary security credentials:

```json
{
"Credentials": {
"AccessKeyId": "ASIA...",
"SecretAccessKey": "wJalrX...",
"SessionToken": "FQoGZXIvYXdz...",
"Expiration": "2025-02-02T12:30:00Z"
}
}
```

*   These credentials expire after a set duration (default 1 hour, can be extended up to 12 hours).

3.  **To Perform Actions Permitted by the Role**
    *   After assuming the role, you can use the temporary credentials to access AWS services based on the permissions attached to the role.
    *   For example, in our scenario, `main-role1` has full access to a specific S3 bucket, so after assuming the role, we can upload files to that bucket.

### Example: Using Assumed Role Credentials

Once you assume the role and get the temporary credentials, you need to set them in your environment to use them:

```bash
export AWS_ACCESS_KEY_ID="ASIA..."
export AWS_SECRET_ACCESS_KEY="wJalrX..."
export AWS_SESSION_TOKEN="FQoGZXIvYXdz..."
```

Now, you can run AWS CLI commands as the assumed role:

```bash
aws s3 cp myfile.txt s3://testing-s3/
```

*   Since `main-role1` has full access to `testing-s3`, this will work.

### Real-Life Scenario: Why Use Assume Role Instead of IAM Users?

*   Suppose a company has developers who need access to S3.
*   Instead of giving them permanent IAM user credentials, they create an IAM role with limited permissions.
*   Developers assume the role only when needed, and the credentials expire after some time, reducing security risks.

### Key Takeaways

*   `aws sts assume-role` is used to get temporary credentials for an IAM role.
*   These credentials expire after 1 hour by default (can be extended to 12 hours).
*   It enhances security by avoiding long-term IAM user credentials.
*   After assuming the role, you must export the temporary credentials to use them.

