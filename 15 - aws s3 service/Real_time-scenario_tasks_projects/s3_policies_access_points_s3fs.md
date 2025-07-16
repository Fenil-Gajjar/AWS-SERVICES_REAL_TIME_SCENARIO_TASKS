
# Real Time Task on Step by Step Implementation of S3 Policies, Access Points and S3FS

## About This Guide

In this hands-on task, we will explore the powerful capabilities of Amazon S3 by implementing key security and access management features. This step-by-step guide will help you gain practical experience with:

*   **S3 Bucket Policies** – Secure and manage permissions at the bucket level.
*   **Pre-Signed URLs** – Enable temporary and secure access to S3 objects.
*   **S3 Access Points** – Implement fine-grained access control for different users.
*   **S3FS Integration** – Mount an S3 bucket as a filesystem on an EC2 instance.

## What You'll Achieve:

*   Launch and configure an Ubuntu EC2 instance.
*   Create and configure an S3 bucket with security best practices.
*   Generate and test pre-signed URLs for secure temporary access.
*   Implement S3 Access Points to manage access for different IAM users.
*   Mount your S3 bucket as a filesystem on an EC2 instance using s3fs.

This task will provide you with real-world AWS experience and enhance your understanding of secure storage management in the cloud. By the end, you will have a fully functional S3 setup integrated with IAM users, access points, and EC2.

Let’s get started and dive into AWS storage security and access management!

## Part 1: Launch an Ubuntu EC2 Instance

1.  **Launch an EC2 Instance:**
    *   Open the AWS Management Console and navigate to EC2.
    *   Click **Launch Instance**.
    *   Choose an Ubuntu AMI (for example, Ubuntu 20.04 LTS).
    *   Select the instance type (e.g., `t2.micro` for testing).
    *   Under **Configure Instance Details**, select your desired VPC (ensure its security group allows all traffic from anywhere) and choose a Public Subnet.
    *   Enable **Auto-assign Public IP**.
    *   Under **Add Tags**, add a tag such as:
        *   Key: `Name`
        *   Value: `Testing`
    *   Proceed to **Configure Security Group** ensuring that necessary ports (e.g., SSH on port 22) are open.
    *   Review and launch the instance.
    *   Select your PEM key pair when prompted.
    *   Wait for the instance to be in the running state.

## Part 2: Create and Configure an S3 Bucket

### 2.1 Create an S3 Bucket and Upload Files

1.  **Create the Bucket:**
    *   In the AWS Console, navigate to S3.
    *   Click **Create bucket**.
    *   **Bucket Name:** Enter a unique name (for example, `my-example-bucket`).
    *   Leave the default settings and click **Create bucket**.
2.  **Upload Files:**
    *   Open the newly created bucket.
    *   Click **Upload** and add some files (for example, images) including an `index.html` file.
    *   Complete the upload process.

### 2.2 Modify Bucket Permissions to Allow Public Read (Temporarily)

1.  **Disable Block Public Access:**
    *   In your bucket, go to the **Permissions** tab.
    *   Click on **Edit** in the **Block public access (bucket settings)** section.
    *   Uncheck the option **Block all public access**.
    *   Click **Save changes** (confirm if prompted).
2.  **Add a Bucket Policy for Public Read:**
    *   In the same **Permissions** tab, scroll to **Bucket Policy**.
    *   Click **Edit** and paste the following JSON (replace `arn:aws:s3:::my-example-bucket/*` with your bucket ARN pattern):

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "PublicReadGetObject",
          "Effect": "Allow",
          "Principal": "*",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::my-example-bucket/*"
        }
      ]
    }
    ```
    *   Click **Save changes**.
3.  **Test Public Access:**
    *   Copy the URL of one of your files (e.g., `https://my-example-bucket.s3.amazonaws.com/index.html`) and paste it in a browser.
    *   Confirm that you can view the file.
4.  **Revert Public Access Settings:**
    *   For security reasons, after testing, go back to the **Permissions** tab.
    *   Re-enable **Block all public access**.
    *   Delete the bucket policy you just created.

## Part 3: Generate a Pre-signed URL

1.  **Generate a Pre-signed URL:**
    *   In the S3 Console, navigate to your bucket.
    *   Select any file (e.g., one of your images).
    *   Click on **Actions** and choose **Share with pre-signed URL**.
    *   Specify the expiration time (e.g., 60 minutes).
    *   Click **Create pre-signed URL**.
2.  **Test the Pre-signed URL:**
    *   Copy the pre-signed URL and paste it into a browser to ensure you can access the file.
3.  **Clean Up:**
    *   Optionally, delete all files in this bucket if they were only for testing.

## Part 4: Configure S3 Access Points

### 4.1 Create Folders for Granular Access

1.  Open your S3 bucket.
2.  Click **Create folder** and create two folders:
    *   `folder1`
    *   `folder2`
    *   (The scenario is designed such that you want to give different permissions to different developers for each folder.)

### 4.2 Create an IAM User for Developer Access

1.  In the AWS Console, navigate to IAM.
2.  Go to **Users** and click **Add users**.
3.  **Username:** Enter `developer1`.
4.  **Access Type:** Choose **AWS Management Console access** (or only programmatic access if you plan to use CLI).
5.  Set a custom password and create the user without attaching any policies.
6.  Click **Create user**.

### 4.3 Create an S3 Access Point

1.  Go to your S3 bucket.
2.  Click on the **Access Points** tab.
3.  Click **Create access point**.
4.  **Name:** Enter `accesspointdev1`.
5.  **Network Origin:** Select **Internet**.
6.  Leave other settings as default.
7.  Click **Create access point**.

### 4.4 Update Bucket Policy for Access Point Usage

1.  Go back to your S3 bucket’s **Permissions** tab.
    Under **Bucket Policy**, add (or update) a policy that allows AWS users to perform actions on this bucket only when using a data access point from your AWS account. For example:

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "AllowActionsViaAccessPoint",
          "Effect": "Allow",
          "Principal": "*",
          "Action": "s3:*",
          "Resource": "arn:aws:s3:::my-example-bucket/*",
          "Condition": {
            "StringEquals": {
              "s3:DataAccessPointAccount": "<YOUR_AWS_ACCOUNT_ID>"
            }
          }
        }
      ]
    }
    ```
2.  Click **Save changes**.

### 4.5 Attach an Access Point Policy for Developer1

1.  In the S3 bucket, click on the **Access Points** tab.
2.  Select the access point you created (e.g., `accesspointdev1`).
3.  Click on the **Permissions** section for the access point.
    Edit the Access Point Policy to allow the IAM user `developer1` to perform any action on all objects under `folder1`. For example:

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "Developer1FullAccessToFolder1",
          "Effect": "Allow",
          "Principal": {
            "AWS": "arn:aws:iam::<YOUR_AWS_ACCOUNT_ID>:user/developer1"
          },
          "Action": "s3:*",
          "Resource": [
            "arn:aws:s3:::my-example-bucket/folder1/*"
          ]
        }
      ]
    }
    ```
4.  Click **Save**.

### 4.6 Test Developer1 Access via S3 Access Point

On your local machine (or via Git Bash), configure the AWS CLI as `developer1`:

`aws configure`

1.  Enter the credentials (Access Key ID, Secret Access Key) for `developer1`.
    Create an empty file on your local machine:

    `echo "Test file for access point" > testfile.txt`

    Copy the file to your S3 bucket using the access point URL. The S3 URI will follow a format similar to:

    `aws s3 cp testfile.txt s3://<accesspointdev1-identifier>/folder1/testfile.txt`

2.  The exact S3 access point URL format can be found in the Access Point details.
3.  Verify that the file is uploaded successfully into the `folder1` directory.

## Part 5: Use s3fs to Mount the S3 Bucket

### 5.1 Create an IAM User with Full S3 Access

1.  In the AWS Console, navigate to IAM.
2.  Create a new IAM user named `s3full` with **Programmatic Access**.
3.  Attach the `AmazonS3FullAccess` policy to this user.
4.  After creation, note down the Access Key ID and Secret Access Key.

### 5.2 Configure the EC2 Instance for s3fs

Log in to your Ubuntu EC2 instance (launched earlier) via SSH:

`ssh -i your-key.pem ubuntu@<Ubuntu_Public_IP>`

Update the package index:

`sudo apt update`

Install required packages:

`sudo apt install s3fs unzip awscli -y`

Configure AWS CLI with the credentials of the `s3full` user:

`aws configure`

1.  Enter the Access Key ID and Secret Access Key for `s3full`.
2.  Set the default region as appropriate.
3.  Set output format to `json` (or your choice).

Create a directory to mount the S3 bucket:

`mkdir ~/mys3`

Mount the S3 bucket using s3fs:

`s3fs my-example-bucket ~/mys3 -o use_cache=/tmp`

1.  Replace `my-example-bucket` with your S3 bucket name.

Verify the mount:

`df -h`

1.  You should see the S3 bucket mounted at `~/mys3`.

## Summary: What This Scenario Does

*   **S3 Bucket Setup:**
    *   A bucket is created and files (such as images, an `index.html`) are uploaded.
    *   Public access is enabled temporarily by modifying the bucket policy, and later reverted.
    *   A pre-signed URL is generated to provide temporary access to an object.
*   **S3 Access Points:**
    *   Two folders (`folder1` and `folder2`) are created in the bucket.
    *   An S3 Access Point (`accesspointdev1`) is created to allow granular access.
    *   A bucket policy is configured so that actions on the bucket are allowed only via the Access Point, and a separate access point policy is set to give a specific IAM user (`developer1`) permissions on `folder1`.
*   **Testing Access via CLI:**
    *   The `developer1` IAM user uses AWS CLI to upload a file to `folder1` using the Access Point URL, verifying that the policy is working.
*   **Using s3fs:**
    *   An IAM user (`s3full`) with full S3 access is created.
    *   An EC2 Ubuntu instance is configured with `s3fs` and `awscli`, and the S3 bucket is mounted as a local filesystem for further operations.

This scenario demonstrates different ways to manage S3 access:

*   Bucket policies for public object access.
*   Pre-signed URLs for temporary secure sharing.
*   S3 Access Points for granular, per-folder permissions.
*   s3fs for mounting S3 buckets on Linux systems for file system-like access.

## Understanding the S3 Policies, Access Points, and S3fs Scenario Task

This scenario is a step-by-step implementation of different ways to access, secure, and mount an Amazon S3 bucket while implementing IAM permissions and S3 Access Points. Let’s break down what this task actually achieves.

### 1. Launch EC2 Instance with Ubuntu AMI

We start by launching an EC2 instance that will be used to interact with S3. This is necessary because we need a server to perform AWS CLI operations and later mount S3 as a filesystem.

### 2. Basic S3 Bucket Access and Public Permissions Testing

Here, we create an S3 bucket and explore different ways of accessing objects stored in it.

**Steps and Purpose:**

1.  **Create an S3 bucket** – This is a storage container for files (images, logs, backups, etc.).
2.  **Upload files** (e.g., images) to the S3 bucket – This helps us test object accessibility.
3.  **Modify bucket permissions:**
    *   Disable "Block all public access" – This allows public access to objects.
    *   Add a bucket policy – We create a policy that allows anyone (public) to retrieve objects (`s3:GetObject` action).
    *   Test access via URL – Now, we try to access an object’s URL from a browser to verify it's publicly accessible.
    *   Re-enable "Block all public access" – We remove public access to secure the bucket.

**What this achieves:**

*   We learn how to make an S3 bucket public (for testing) and later revert it to private.
*   We see how Bucket Policies work for granting public access.

### 3. Pre-Signed URL for Temporary Access

Instead of making files public, we generate a pre-signed URL for an object.

**Steps and Purpose:**

1.  **Select an S3 object** → Generate a Pre-Signed URL
    *   This URL is time-limited (e.g., expires after 10 minutes or 1 hour).
    *   The object remains private, but anyone with the pre-signed URL can access it temporarily.
2.  **Test access in a browser** – Open the pre-signed URL and verify access.
3.  **Delete all files from the bucket** – Cleanup before moving to the next step.

**What this achieves:**

*   Pre-signed URLs allow temporary access to private S3 objects without changing bucket policies.
*   A common use case is sharing private files securely for a limited time.

### 4. Implementing S3 Access Points for Developer-Specific Access

S3 Access Points help manage permissions for specific users and folders within a bucket.

**Scenario:**

We assume two developers, and we want to grant each developer access to only their respective folder.

**Steps and Purpose:**

**(a) Create Folders in the S3 Bucket**

*   `Folder1` and `Folder2` – Representing different project areas for different developers.

**(b) Create IAM User (Developer1)**

*   No permissions are granted initially.

**(c) Create an S3 Access Point**

*   We create an Access Point (`accesspointdev1`) for the S3 bucket.
*   This allows controlled access only through the Access Point.

**(d) Apply an Access Point-Specific Policy**

*   **Modify S3 Bucket Policy:**
    *   Ensures that all access to the bucket happens only via the Access Point.
    *   This prevents direct S3 access via bucket URLs.
*   **Modify Access Point Policy:**
    *   Grants `Developer1` access to `Folder1` only.

**(e) Developer1 Uploads a File to S3 via Access Point**

*   `Developer1` configures AWS CLI (`aws configure`).
*   Uploads a file to `Folder1` using the Access Point URL.

**What this achieves:**

*   Restricts access to specific users based on Access Points.
*   Ensures access is managed centrally via the Access Point, not the bucket.
*   Prevents unintended access to other folders.

### 5. Mounting S3 Bucket as a Filesystem Using S3fs

Finally, we mount the S3 bucket as a local filesystem on an EC2 instance.

**Steps and Purpose:**

**(a) Create IAM User (s3full) with AmazonS3FullAccess**

*   This user has full access to S3.

**(b) Install Required Packages on the EC2 Instance**

*   `s3fs` – A tool that allows mounting S3 as a filesystem.
*   `AWS CLI` – To interact with S3.
*   `Unzip` – Helps extract files if needed.

**(c) Configure AWS Credentials on EC2 (aws configure)**

*   We enter the Access Key & Secret Key for `s3full`.

**(d) Mount the S3 Bucket to a Local Directory**

*   Create a directory (`mys3/`) to serve as the mount point.
*   Run `s3fs` to mount the bucket.
*   Verify with `df -h` – The bucket should appear as a mounted drive.

**What this achieves:**

*   We can now treat the S3 bucket like a local filesystem.
*   Developers can read, write, and manage files on S3 using standard Linux commands (`ls`, `cp`, `mv`, etc.).
*   Useful for backup storage, logging, and cloud file management.

## Wrapping Up: AWS S3 Service Comprehensive Guide

We started this journey by understanding the theoretical concepts behind S3 Policies, Access Points, and S3FS, ensuring you have a solid foundation before diving into the hands-on part.

Then, we stepped into real-world implementation with a practical approach:

*   Launching an Ubuntu EC2 Instance to serve as our workspace
*   Creating and Configuring an S3 Bucket with proper permissions
*   Managing S3 Access Control with Policies and Access Points
*   Generating Pre-Signed URLs for temporary access
*   Mounting S3 Storage with s3fs to use it like a local drive

This structured approach helped you learn, apply, and validate your skills with AWS S3 in a real-world scenario. By completing this task, you've gained hands-on experience in securely managing cloud storage using AWS best practices.

## What's Next?

The learning never stops! Follow me for more real-time DevOps and Cloud tasks daily. Every day, we explore new, industry-relevant projects that will take your skills to the next level.

Stay consistent, keep practicing, and get ready for the next challenge!

Until next time—Happy Learning & Keep Building!

