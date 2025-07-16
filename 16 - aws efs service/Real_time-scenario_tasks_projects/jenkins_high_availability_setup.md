# Welcome to the AWS EFS-based Jenkins High Availability Setup

## Introduction

In modern DevOps environments, ensuring high availability and reliability of Jenkins is crucial for seamless CI/CD workflows. This project focuses on setting up AWS Elastic File System (EFS) with Jenkins, enabling multiple Jenkins instances to share the same data and configurations. By implementing a shared storage solution using AWS EFS, we ensure that Jenkins can be accessed from multiple instances while maintaining data consistency.

## Project Overview

This project guides you through the step-by-step implementation of an AWS EFS-backed Jenkins setup, covering:

*   Provisioning EC2 instances for Jenkins deployment.
*   Configuring AWS Elastic File System (EFS) to provide shared storage across multiple instances.
*   Mounting EFS on Jenkins-primary and Jenkins-secondary instances.
*   Installing and configuring Jenkins on both instances.
*   Verifying shared storage functionality by running Jenkins jobs across both instances.
*   Setting up log rotation to manage system logs efficiently.

By the end of this project, you will have a fully functional Jenkins setup with shared storage, ensuring data consistency, fault tolerance, and seamless job execution across multiple nodes.

## Why Use AWS EFS for Jenkins?

*   **High Availability:** Ensures Jenkins data is accessible across multiple instances.
*   **Scalability:** Automatically grows as your storage needs increase.
*   **Resilience:** Prevents data loss even if an EC2 instance goes down.
*   **Easy Maintenance:** No need for complex NFS server management.

## Who Should Follow This Guide?

This project is ideal for:

*   DevOps engineers looking to enhance their Jenkins setup with high availability and scalability.
*   AWS enthusiasts exploring EFS and EC2 integration for real-world applications.
*   Anyone interested in CI/CD best practices and infrastructure automation.

## Real time Task on Step By Step Implementation of EFS

### Part 1: Launch EC2 Instances for Jenkins

#### Step 1.1: Launch Jenkins-primary Instance

1.  Open the AWS Management Console and navigate to EC2.
2.  Click **Launch Instance**.
3.  **Choose an AMI:**
    *   Select an Ubuntu AMI (e.g., Ubuntu 20.04 LTS).
4.  **Choose Instance Type:**
    *   Select `t2.medium`.
5.  **Configure Instance Details:**
    *   Choose your desired VPC (with a security group that allows all traffic from anywhere).
    *   Choose a Public Subnet.
    *   Enable **Auto-assign Public IP**.
6.  **Add Tags:**
    *   Add a tag with Key: `Name` and Value: `Jenkins-primary`.
7.  **Configure Security Group:**
    *   Ensure the security group allows required ports (SSH on 22, HTTP if needed, etc.).
8.  **User Data:**
    In the Advanced Details section, paste the following script:
    ```bash
    #!/bin/bash
    sudo apt update
    sudo apt install -y nfs-common openjdk-8-jdk
    ```
9.  Click **Review and Launch**, then **Launch**.
10. Select your PEM key pair when prompted.
11. Wait until the instance is in a running state.

#### Step 1.2: Launch Jenkins-secondary Instance

1.  Repeat the above steps to launch a second EC2 instance.
2.  **Name:** Tag it as `Jenkins-secondary`.
3.  Use the same Ubuntu AMI and instance type (`t2.medium`).
4.  For User Data, use the same script:
    ```bash
    #!/bin/bash
    sudo apt update
    sudo apt install -y nfs-common openjdk-8-jdk
    ```
5.  Complete the launch and wait until this instance is running.

### Part 2: Create and Configure the EFS File System

#### Step 2.1: Create an EFS File System

1.  In the AWS Management Console, navigate to EFS (Elastic File System).
2.  Click **Create file system**.
3.  **Configure Basic Settings:**
    *   Give the file system a name (for example, `jenkins-efs`).
    *   Select the VPC used by your EC2 instances.
4.  Accept the default options and click **Create file system**.
5.  Wait for the file system status to become **Available**.

#### Step 2.2: Configure the EFS Network Settings

1.  In the EFS console, click on your new file system.
2.  Go to the **Network** tab.
3.  Under **Manage security groups**, click **Manage**.
4.  Remove any default security groups.
5.  Attach the security group(s) that are used by your EC2 instances (ensuring that the security group allows NFS traffic on port 2049).
6.  Click **Save**.
7.  Copy the DNS name of the EFS file system (it will look similar to `fs-xxxxxx.efs.<region>.amazonaws.com`). You will need this DNS name later.

### Part 3: Mount the EFS File System on Both Jenkins Instances

#### Step 3.1: On Jenkins-primary Instance

Connect via SSH or RDP (for Ubuntu, use SSH; if you have a GUI, you can use it, but typically SSH is used):

`ssh -i your-key.pem ubuntu@<Jenkins-primary_Public_IP>`

Create the mount directory:

`sudo mkdir -p /var/lib/jenkins`

Edit the `/etc/fstab` file to add the EFS mount entry:

`sudo nano /etc/fstab`

Add the following line at the end of the file (replace `paste_the_dns_name_that_we_copied_from_file_system` with the actual DNS name of your EFS file system):

`paste_the_dns_name_that_we_copied_from_file_system:/ /var/lib/jenkins nfs defaults,_netdev 0 0`

1.  Save and exit (Ctrl+X, then Y, then Enter if using nano).

Mount all filesystems specified in fstab:

`sudo mount -a`

Verify the mount:

`df -h`

*   Confirm that the EFS file system is mounted on `/var/lib/jenkins`.

#### Step 3.2: On Jenkins-secondary Instance

Connect to the Jenkins-secondary instance via SSH:

`ssh -i your-key.pem ubuntu@<Jenkins-secondary_Public_IP>`

Create the mount directory:

`sudo mkdir -p /var/lib/jenkins`

Edit `/etc/fstab`:

`sudo nano /etc/fstab`

Add the same line as before:

`paste_the_dns_name_that_we_copied_from_file_system:/ /var/lib/jenkins nfs defaults,_netdev 0 0`

1.  Save and exit.

Mount the file system:

`sudo mount -a`

Verify the mount:

`df -h`

*   The EFS file system should be mounted on `/var/lib/jenkins`.

#### Step 3.3: Verify Shared File System Access

On Jenkins-primary, navigate to the mount point:

`cd /var/lib/jenkins`

Create an empty file:

`touch primary_file.txt`

On Jenkins-secondary, navigate to the same directory:

`cd /var/lib/jenkins`
`ls -l`

*   You should see `primary_file.txt` created on Jenkins-primary.

Similarly, on Jenkins-secondary, create an empty file:

`touch secondary_file.txt`

On Jenkins-primary, check the directory:

`cd /var/lib/jenkins`
`ls -l`

*   You should see `secondary_file.txt`.

Delete both files to clean up:

`rm primary_file.txt secondary_file.txt`

### Part 4: Install and Configure Jenkins on Both Instances

#### Step 4.1: Install Jenkins on Jenkins-primary Instance

1.  Connect to Jenkins-primary via SSH (if not already connected).

Install Jenkins using the following commands:

```bash
# Update package list
sudo apt update
# Install Java (if not already installed)
sudo apt install openjdk-8-jdk -y
# Add the Jenkins repository and key
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
# Update package list again
sudo apt update
# Install Jenkins
sudo apt install jenkins -y
# Start Jenkins
sudo systemctl start jenkins
# Enable Jenkins to start on boot
sudo systemctl enable jenkins
```

2.  Wait a few minutes for Jenkins to fully start.

#### Step 4.2: Access Jenkins on Jenkins-primary

1.  Copy the Public IP of the Jenkins-primary instance.

Open a web browser and navigate to:

`http://<Jenkins-primary_Public_IP>:8080`

2.  Follow the Jenkins setup instructions:

Enter the `initialAdminPassword` (retrieve it by running):

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

*   Complete the installation steps (install suggested plugins, create an admin user, etc.).

#### Step 4.3: Run a Freestyle Job on Jenkins-primary

1.  After Jenkins is set up, create a new Freestyle Job:

*   In the Jenkins dashboard, click on **New Item**.
*   Enter a name (e.g., `TestJob`), select **Freestyle Project**, and click **OK**.

In the job configuration, add a build step (e.g., Execute shell) with the following command:

`echo 


we are testing"

2.  Save the job.
3.  Click **Build Now** three times (or build three times sequentially).
4.  Verify that the job runs successfully.

#### Step 4.4: Install Jenkins on Jenkins-secondary Instance

1.  Connect to Jenkins-secondary via SSH.

Repeat the same installation steps as for Jenkins-primary:

```bash
sudo apt update
sudo apt install openjdk-8-jdk -y
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

2.  Wait for Jenkins-secondary to start.

#### Step 4.5: Access Jenkins on Jenkins-secondary

1.  Copy the Public IP of Jenkins-secondary.

Open a web browser and navigate to:

`http://<Jenkins-secondary_Public_IP>:8080`

2.  You will be prompted for a username and password.

*   **Username:** Enter `admin`.
*   **Password:** Obtain the password by logging into Jenkins-primary, and running:

    `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

    *   Copy that password and use it for Jenkins-secondary.

3.  Log in to Jenkins-secondary.
4.  Verify that the Jenkins dashboard on Jenkins-secondary shows the same previously run Freestyle job (e.g., `TestJob`). This demonstrates that both Jenkins nodes share the same `/var/lib/jenkins` directory via EFS.

### Part 5: Set Up a Log Rotation Script on Jenkins-primary

#### Step 5.1: Create the Log Rotation Script

1.  On Jenkins-primary, open a terminal or connect via SSH.
2.  Navigate to the directory where you want to store the script (e.g., `/root` or `/var/lib/jenkins`).

Create a new file called `rotating.sh` using a text editor (e.g., `vim` or `nano`):

`sudo vim /root/rotating.sh`

Paste the following script into the file:

```bash
#!/bin/bash

f="/var/log/syslog"

if [ ! -f "$f" ]; then
echo "$f does not exist!"
exit 1
fi

# Ensure the file exists by touching it
touch "$f"
MAXSIZE=$((1 * 1024))

# Maximum size in bytes (1 KB in this example)

size=$(du -b "$f" | tr -s '\t' ' ' | cut -d' ' -f1)
if [ "$size" -gt "$MAXSIZE" ]; then
echo "Rotating!"
timestamp=$(date +%s)
mv "$f" "/var/lib/jenkins/backup.$timestamp"
touch "$f"
fi
```

3.  Save and close the file.

#### Step 5.2: Test the Log Rotation Script

Run the script manually:

`bash /root/rotating.sh`

Change directory to the mount point:

`cd /var/lib/jenkins`
`ls -al`

*   You should see a backup file (e.g., `backup.<timestamp>`) if `/var/log/syslog` was larger than the defined threshold.

2.  Remove any backup files if needed for a clean test.

#### Step 5.3: Automate the Log Rotation via Crontab

Open the crontab editor:

`crontab -e`

Add the following line to schedule the script to run every minute:

`* * * * * bash /root/rotating.sh`

1.  Save and exit the editor.

## Final Verification and Summary

### Verification Steps:

*   **EFS Mounting Verification:**
    *   On both Jenkins-primary and Jenkins-secondary, ensure that `/var/lib/jenkins` is mounted correctly by running `df -h` and checking for the EFS mount.
    *   Create files on one instance and verify they appear on the other.
*   **Jenkins Installation Verification:**
    *   Access the Jenkins UI on both instances via their public IP addresses on port 8080.
    *   Confirm that the initial job (`TestJob`) runs on Jenkins-primary and appears on Jenkins-secondary (indicating a shared file system).
*   **Log Rotation Verification:**
    *   Manually run the rotation script and check for backup files.
    *   Verify that crontab has been set up correctly to run the script every minute.

### What This Scenario Accomplishes:

*   **Shared File System with EFS:**
    *   Both Jenkins-primary and Jenkins-secondary mount the same EFS file system at `/var/lib/jenkins`, ensuring that job data, configuration, and logs are shared across nodes.
*   **Jenkins High Availability:**
    *   Installing Jenkins on two separate instances with shared storage ensures that if one node fails, the other still has access to the same job configurations and data.
*   **Centralized Log Rotation:**
    *   A custom log rotation script on Jenkins-primary manages and rotates logs (in this case, `/var/log/syslog`), with the backup files stored on the shared EFS. This script is automated via crontab.
*   **Real-World Jenkins Setup:**
    *   Demonstrates how to set up a redundant Jenkins environment using AWS services (EC2, EFS) for high availability and data consistency.

This complete setup ensures that both Jenkins instances share the same file system (via EFS), can see and manage the same job configurations and logs, and have automated log rotation to manage disk usage.

## Final Thoughts & Goodbye

In this document, we embarked on a comprehensive journey into AWS Elastic File System (EFS), covering both theoretical concepts and practical implementation in detail.

### What We Covered

We began with the theoretical foundation of EFS, understanding:

*   What EFS is and how it works
*   Its key features, including scalability, elasticity, and shared access
*   Use cases and benefits in cloud-native architectures
*   How EFS differs from other AWS storage solutions like EBS and S3

With this solid foundation, we transitioned into a real-world hands-on task, where we:

*   Launched EC2 instances for Jenkins setup.
*   Created and configured an EFS file system for shared storage.
*   Mounted the EFS file system on multiple Jenkins instances for high availability.
*   Installed and configured Jenkins across instances.
*   Tested job execution and data sharing across nodes.
*   Implemented automated log rotation to optimize system performance.

By completing both the theory and practical aspects, we now have a deep understanding of AWS EFS—both in concept and in action.

### What’s Next?

We are committed to posting daily AWS and DevOps tasks to help you gain hands-on experience and stay ahead in your cloud journey. If you found this guide useful, follow me for more real-world projects and learning content.

🚀 Keep building, keep automating, and see you in the next task! 🔥💡


