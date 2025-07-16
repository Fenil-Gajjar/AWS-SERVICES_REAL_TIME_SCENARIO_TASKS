# Welcome to the AWS FSx, AWS WorkSpaces, and Active Directory Implementation Guide

## Introduction

In modern cloud environments, managing enterprise file storage, user authentication, and secure remote access is crucial for ensuring scalability, security, and efficiency. This guide provides a step-by-step approach to implementing AWS FSx, Amazon WorkSpaces, and AWS Managed Microsoft Active Directory to create a seamless and integrated cloud-based IT infrastructure.

Whether you are an IT administrator, DevOps engineer, or cloud architect, this guide will help you understand how to:

*   Set up AWS Managed Microsoft Active Directory (AD) for centralized user authentication.
*   Deploy AWS FSx for Windows File Server to provide scalable, high-performance storage.
*   Integrate Amazon WorkSpaces to enable secure remote access for domain users.
*   Ensure security and access control by configuring Active Directory users, groups, and permissions.
*   Perform real-world testing by simulating user access to file shares and WorkSpaces.

By following this guide, you will gain hands-on experience in building a cloud-based Windows environment that supports seamless authentication, file sharing, and remote desktop access.

## Why This Setup?

This implementation is designed for organizations looking to migrate traditional on-premises Active Directory and file-sharing solutions to AWS. It provides:

*   **Scalability:** Easily scale your directory and file storage as your organization grows.
*   **Security:** Centralized authentication and access control using Active Directory.
*   **High Availability:** FSx Multi-AZ deployment ensures data redundancy and resilience.
*   **Cost-Effectiveness:** Use auto-stop WorkSpaces to optimize costs while maintaining productivity.
*   **Seamless Integration:** WorkSpaces, FSx, and Managed AD work together to provide a fully functional enterprise environment.

## Prerequisites

Before proceeding, ensure you have the following:

1.  An AWS Account with appropriate IAM permissions to create and manage resources.
2.  A properly configured VPC with public and private subnets.
3.  Security Groups that allow necessary traffic (SMB, RDP, NFS, etc.).
4.  A Windows-based local machine to test Remote Desktop and WorkSpaces access.

## Implementation Overview

This guide is divided into multiple phases, each focusing on a crucial component of the setup:

*   **Part A: Setting Up AWS Managed Microsoft Active Directory**
    *   Create and configure an AWS Managed AD domain.
    *   Obtain DNS details for integrating with FSx and WorkSpaces.
*   **Part B: Deploying Amazon FSx for Windows File Server**
    *   Set up a high-performance Windows file system with Multi-AZ deployment.
    *   Integrate FSx with the Managed AD domain.
*   **Part C: Launching a Testing EC2 Instance**
    *   Deploy a Windows EC2 instance to validate connectivity and domain integration.
    *   Configure Windows settings, network policies, and security rules.
*   **Part D: Configuring the EC2 Instance for AD and FSx**
    *   Join the EC2 instance to the Managed AD domain.
    *   Update DNS settings and configure firewall policies.
    *   Install necessary administrative tools for Active Directory management.
*   **Part E: Managing Users and Groups in Active Directory**
    *   Create Active Directory users and assign them to groups.
    *   Set up access permissions for FSx shared folders.
*   **Part F: Deploying Amazon WorkSpaces for Remote Access**
    *   Register the Managed AD directory with WorkSpaces.
    *   Assign WorkSpaces to users and enable access.
*   **Part G: Configuring FSx File Shares and Permissions**
    *   Set up shared folders with custom permissions.
    *   Restrict access based on Active Directory group membership.
*   **Part H: Testing Access from Amazon WorkSpaces**
    *   Validate user access to shared folders.
    *   Test network drive mapping and WorkSpaces connectivity.

## Expected Outcome

After completing this guide, you will have:

*   A fully functional AWS environment with centralized authentication via AWS Managed AD.
*   An integrated FSx file system that provides secure and scalable file storage.
*   A secure remote working solution with Amazon WorkSpaces.
*   A properly configured Active Directory structure with users and group policies.
*   Hands-on experience in enterprise IT infrastructure deployment in AWS.

## Next Steps

Follow each step carefully, ensuring that you configure the resources correctly. Remember:

*   Replace all placeholder values (domain names, passwords, and instance names) with your own.
*   Wait for each resource to be fully active before proceeding to the next step.
*   Use best security practices while configuring security groups, permissions, and credentials.

Once you are ready, proceed with Part A: Setting Up AWS Managed Microsoft Active Directory.

## Real time Task on Step By Step Implementation of AWS FSx, AWS Workspace and Integration with Active Directory

**Important:**

*   Replace placeholder values (for example, adminapi.in, the DNS names, passwords, and file system names) with your own.
*   Ensure your VPC and security groups allow required traffic (RDP, NFS, SMB as needed).
*   Wait for each resource (directory, FSx, WorkSpaces) to become active before proceeding.

On your local machine, download and install the Amazon WorkSpaces client from clients.amazonworkspaces.com.

### Part A: Set Up Managed Active Directory

**A1. Create AWS Managed Microsoft AD**

1.  Log in to the AWS Management Console and navigate to Directory Service.
2.  Click Set up directory.
3.  Choose AWS Managed Microsoft AD.
4.  **Edition:** Select Standard Edition.
5.  **Directory DNS Name:** Enter adminapi.in (or your chosen domain name).
6.  **Directory NetBIOS Name:** (e.g., ADMINAPI – usually the short version).
7.  **Administrator Password:** Enter a secure password and confirm it.
8.  **VPC & Subnets:**
    *   Select your VPC and choose all your public subnets.
9.  Click Create Directory.
10. Wait until the directory is fully active.
11. Once active, open the directory details, click on the Networking & Security tab, and copy both DNS addresses that are displayed. You will need these later.

### Part B: Set Up Amazon FSx for Windows File Server

**B1. Create an FSx File System**

1.  In the AWS Console, navigate to Amazon FSx.
2.  Click Create file system.
3.  **File System Type:** Choose Amazon FSx for Windows File Server.
4.  **Creation Method:** Select Quick create.
5.  **File System Name:** Enter a name (e.g., fsx-jenkins).
6.  **Deployment Type:** Select Multi-AZ.
7.  **Storage Capacity:** Enter 32 GB.
8.  **VPC & Subnets:**
    *   Select your VPC and choose all public subnets.
9.  **Directory Service Integration:**
    *   Under Directory, select the AWS Managed Microsoft AD you created earlier (with DNS name adminapi.in).
10. Click Create file system.
11. Once the file system is created, open its details and copy its DNS name. (It typically appears similar to fs-xxxxxx.fsx.<region>.amazonaws.com.)
12. Next, click on the Network tab, then click Manage.
    *   Remove all default security groups.
    *   Attach the security group(s) that are used by your EC2 instances (ensuring these groups allow SMB access, typically on TCP port 445).
    *   Save the changes.

### Part C: Launch a Testing EC2 Instance

**C1. Launch an EC2 Instance Named "Testing"**

1.  In the EC2 Console, click Launch Instance.
2.  **Name:** Tag the instance as Testing.
3.  **AMI:** Choose Windows Server 2019 Base.
4.  **Instance Type:** Select t2.large.
5.  **Configure Instance Details:**
    *   Select your VPC (make sure its security group allows all traffic from anywhere or at least allows RDP and SMB traffic).
    *   Choose a Public Subnet.
    *   Enable Auto-assign Public IP.
6.  **Add Tags:**
    *   (For example, add a Name tag with value Testing.)
7.  **Configure Security Group:**
    *   Ensure inbound rules allow RDP (port 3389) and any other required ports.
8.  Click Review and Launch, then Launch.
9.  Select your PEM key pair when prompted.
10. Wait until the instance is running.

### Part D: Configure the Testing Instance to Join the Managed AD and Mount FSx

**D1. Connect to the Testing Instance via RDP**

1.  In the EC2 Dashboard, select the Testing instance.
2.  Click Connect → RDP Client.
3.  Copy the Public DNS (or Public IP).
4.  Click on the Get Password tab.
5.  Click Upload Private Key File, choose your PEM file, and click Decrypt Password.
6.  Copy the generated Administrator password.
7.  On your Windows machine, open Remote Desktop Connection.
8.  Paste the Public DNS into the Computer field and click Connect.
9.  In the pop-up, click More choices and then Use a different account.
10. Enter Username: Administrator and paste the copied password.
11. Click OK to log in.

**D2. Configure Windows Firewall and DNS**

a) Disable Windows Defender Firewall

1.  Open Command Prompt.
    Run:
    `firewall.cpl`
2.  In the firewall settings, choose to Turn off Windows Defender Firewall for both private and public networks, then save changes.

b) Update DNS Settings to Use Managed AD DNS

1.  Open Command Prompt and run:
    `ipconfig /all`
    *   Copy the DNS Servers listed.
2.  Press Win+R, type `ncpa.cpl`, and press Enter.
3.  In the Network Connections window, right-click on Ethernet and choose Properties.
4.  Select Internet Protocol Version 4 (TCP/IPv4) and click Properties.
5.  Select Use the following DNS server addresses:
    *   **Preferred DNS server:** Enter the first DNS address you copied from the Managed AD directory details (from Part A).
    *   **Alternate DNS server:** Enter the second DNS address.
6.  Click Advanced, go to the DNS tab, click Add, and paste the DNS server IP that you noted from `ipconfig /all`.
7.  Save all changes and close the dialogs.
8.  Run `ipconfig /all` again to verify that the new DNS settings are applied.

c) Disable Enhanced Security in Server Manager

1.  Open Server Manager.
2.  Click Local Server.
3.  Under Enhanced Security Configuration, click and turn off both options for Administrators and Users.
4.  Close the window.

**D3. Install Active Directory Management Tools**

1.  In Server Manager, click Manage in the top-right corner.
2.  Click Add Roles and Features.
3.  In the wizard:
    *   On Before You Begin, click Next.
    *   On Installation Type, select Role-based or feature-based installation and click Next.
    *   On Server Selection, ensure your current server is selected and click Next.
    *   On Server Roles, skip (do not select additional roles) and click Next.
    *   On Features, scroll down to Remote Server Administrator Tools.
    *   Expand Remote Server Administrator Tools → Role Administrator Tools.
    *   Check AD DS and AD LDS Tools.
    *   Click Next, then Install on the Confirmation page.
4.  Wait for the installation to complete, then click Close.

**D4. Join the Testing Instance to the Managed AD Domain**

Open Command Prompt and type:
`sysdm.cpl`

1.  In the System Properties window, go to the Computer Name tab and click Change….
2.  In the Domain field, enter adminapi.in.
3.  Click OK.
4.  When prompted, enter the username (e.g., admin) and the password you set during Managed AD creation.
5.  You should see a welcome message confirming that the computer has joined the domain.
6.  Click OK and restart the instance when prompted.

**D5. Log In to the Domain**

1.  After the server restarts, open Remote Desktop Connection again.
2.  Connect using the Public DNS.
3.  At the login prompt, choose Use a different account.
4.  Enter Username: admin@adminapi.in and the corresponding password.
5.  Log in. You should now be logged into the domain.

### Part E: Create AD Users and Groups

**E1. Create Users and Group in Active Directory**

Open Command Prompt and type:
`dsa.msc`

1.  This opens Active Directory Users and Computers.
2.  In the left pane, expand your domain (adminapi.in).
3.  **Create User1:**
    *   Right-click on Users (or your preferred OU) and select New → User.
    *   Set User logon name: user1.
    *   Follow the prompts to set a password.
    *   After creation, double-click on user1, go to the General tab, and set the Email to (e.g., user1@gmail.com).
4.  **Create User2:**
    *   Repeat the above steps to create a second user with the logon name user2 and an Email (e.g., user2@gmail.com).
5.  **Create a Group:**
    *   Right-click on Users (or your chosen OU) and select New → Group.
    *   Enter Group Name: awsadmingroup.
    *   Click OK.
6.  **Add Users to the Group:**
    *   Double-click on the group awsadmingroup.
    *   Go to the Members tab.
    *   Click Add… and select both user1 and user2 from the directory, then click OK.

### Part F: Set Up Amazon WorkSpaces

**F1. Register the Managed AD with WorkSpaces**

1.  In the AWS Management Console, navigate to Amazon WorkSpaces.
2.  Go to Directories.
3.  Find your directory (adminapi.in), click on Actions, and select Register.
4.  Select your public subnets and enable Self-service permissions.
5.  Click Register.

**F2. Create WorkSpaces for the AD Users**

1.  In the WorkSpaces console, click Create WorkSpaces.
2.  **Directory:** Select adminapi.in.
3.  **Create Users:** Do not create new users here.
4.  **Identify Users:**
    *   Select only the usernames you created in AD (i.e., user1 and user2).
5.  **Select Bundle:**
    *   Choose Standard and the appropriate Windows operating system (select a free tier if available).
6.  **WorkSpaces Configuration:**
    *   Select AutoStop to save costs.
7.  Click Create WorkSpaces.
8.  After creation, copy the Registration Codes for user1 and user2. (You will use these when logging into WorkSpaces.)

### Part G: Add AD Users to the "Remote Desktop Users" Group on the Testing Instance

**G1. Prepare the File Share on FSx**

1.  Search and open command prompt.
2.  Run `lusrmgr.msc`.
3.  Then select Groups.
4.  Then select "Remote Desktop Users" and double click on it. And add those two users (user1 and user2) and apply it.

**G2. Configure FSx Integration with AD and File Sharing**

1.  In the AWS Console, navigate to Amazon FSx.
2.  Create a new FSx file system (if not already created) that is integrated with your Managed AD.
    *   (If you already created the FSx file system in a previous step, copy its DNS name.)
3.  Ensure the FSx file system is accessible via the domain (it will be joined to your AD).

**G3. Configure File Share Permissions on FSx via the Testing Instance**

1.  On the Testing instance, open File Explorer.
    In the address bar, type:
    `\\<DNS_Name_of_FSx>`
2.  This should display a shared folder.
3.  **Create a Share Folder:**
    *   In the shared FSx folder, create a folder (e.g., name it `share`).
4.  Inside the `share` folder, create three folders:
    *   Folder named `user1`
    *   Folder named `user2`
    *   Folder named `common`
5.  **Set Permissions:**
    *   **For the `common` folder:**
        *   Right-click, choose Properties, then Security → Advanced.
        *   Click Edit, add the group `awsadmingroup` with Full Control, and apply changes.
        *   Disable inheritance and remove all other inherited permissions so that only `awsadmingroup` has access.
    *   **For the `user1` folder:**
        *   Right-click, choose Properties, then Security → Advanced.
        *   Disable inheritance and convert inherited permissions to explicit.
        *   Remove all entries except the explicit entry for `user1` (granting Full Control).
    *   **For the `user2` folder:**
        *   Follow the same steps as above, granting Full Control only to `user2`.

### Part H: Test Access via Amazon WorkSpaces

**H1. Test WorkSpaces Access for user1**

1.  On your local machine, open the Amazon WorkSpaces client.
2.  Enter the Registration Code for `user1` and click Register.
3.  Log in using:
    *   **Username:** `user1`
    *   **Password:** The password you set for `user1` in AD.
    Once logged in, press Windows + R, type:
    `\\<DNS_Name_of_FSx>`
4.  And press Enter.
5.  Verify that you can access:
    *   The `user1` folder (should be accessible to `user1`).
    *   The `user2` folder (should be restricted).
    *   The `common` folder (accessible to members of `awsadmingroup`).
6.  Create an empty file in the `common` folder and confirm that the file is visible to `user1`.

**H2. Test WorkSpaces Access for user2**

1.  Log out from the first WorkSpace.
2.  Open the Amazon WorkSpaces client again and enter the Registration Code for `user2`.
3.  Log in using:
    *   **Username:** `user2`
    *   **Password:** The password you set for `user2`.
    Once logged in, press Windows + R, type:
    `\\<DNS_Name_of_FSx>`
4.  And press Enter.
5.  Verify that you can access:
    *   The `user2` folder (accessible to `user2`).
    *   The `common` folder (accessible as per group permissions).
    *   The `user1` folder should be restricted.
6.  **Map the network drive:**
    *   Open This PC, click on Map network drive.
    Enter the folder:
    `\\<DNS_Name_of_FSx>\share`
    *   Click Finish.
    *   The drive should be mounted and visible under This PC.

## Final Summary

This comprehensive scenario demonstrates how to set up an FSx for Windows File Server integrated with AWS Managed Microsoft AD, and how to share that file system across two EC2 instances (Jenkins-primary and Jenkins-secondary) for a high-availability Jenkins setup. Key tasks include:

*   Launching two Windows instances for Jenkins (primary and secondary).
*   Creating an AWS Managed Microsoft AD directory (with DNS name adminapi.in).
*   Creating an Amazon FSx file system that integrates with the Managed AD.
*   Configuring the Testing instance to join the domain, update DNS settings, and install AD management tools.
*   Mounting the FSx share on both Jenkins instances via SMB.
*   Configuring folder-level permissions on the FSx share so that two AD users (user1 and user2) have isolated access to their own folders, while a common folder is shared.
*   Deploying Amazon WorkSpaces for user1 and user2, which use the shared FSx storage.
*   Mapping a network drive on WorkSpaces and verifying file sharing across users.
*   Optionally, configuring a log rotation script on Jenkins-primary to manage log files stored on the shared FSx.

### What Does This Setup Do, in Simple Terms?

*   **File Sharing Across Multiple Servers:**
    FSx is set up as a shared file storage solution accessible from both Jenkins nodes. This means any files created on one server are visible on the other.
*   **Active Directory Integration:**
    The FSx file system is integrated with a managed AD, enabling you to create users and groups (user1, user2, awsadmingroup) and control access at a granular level (each user gets access only to their folder, and a common folder is shared for both).
*   **Unified Access via WorkSpaces:**
    Amazon WorkSpaces are configured for user1 and user2. When they log in, they use their AD credentials and access the FSx file share. This setup demonstrates that different users (even if on separate WorkSpaces) see the same shared file structure and permissions.
*   **Jenkins High Availability:**
    By installing Jenkins on both the primary and secondary instances and storing Jenkins data on FSx, you achieve high availability: if one instance fails, the other still has access to the same Jenkins jobs and configurations.
*   **Log Management (Optional):**
    A log rotation script is set up on Jenkins-primary to automatically rotate log files stored on FSx, ensuring that log files do not grow too large over time.

Overall, this scenario shows how to build a robust, domain-integrated shared file system using FSx, integrated with AD and WorkSpaces, while also supporting a high-availability Jenkins setup with shared data and proper log management.

### What the Scenario Does

1.  **Creates a Managed Active Directory and FSx File System:**
    *   It starts by setting up an AWS Managed Microsoft AD (with a domain like adminapi.in). This is used to centrally manage users and groups.
    *   It then creates an Amazon FSx file system that is integrated with this AD. FSx is used to provide a shared file system that can be accessed by multiple Windows instances.
2.  **Deploys a Testing Windows Instance:**
    *   A Windows Server 2019 instance (named “Testing”) is launched. This instance will join the domain (using the Managed AD) and then be used to configure and test file sharing.
    *   On this instance, you update the DNS settings so it uses the AD-provided DNS servers, disable the firewall for testing, and install necessary AD management tools.
3.  **Joins the Instance to the Domain:**
    *   The Testing instance is joined to the AD domain (adminapi.in), meaning it becomes part of the centralized directory. This allows it to leverage domain authentication and centralized user management.
4.  **Creates Active Directory Users and Groups:**
    *   In the domain, you create two users (user1 and user2) and a group (awsadmingroup). These users will later be used to access the shared file system.
    *   You add user1 and user2 to the group to manage common permissions.
5.  **Configures Remote Desktop Access:**
    *   On the Testing instance, you open the local Users and Groups manager (`lusrmgr.msc`) and add user1 and user2 to the “Remote Desktop Users” group. This step ensures that these AD users are allowed to log in remotely (via RDP) to the machine.
6.  **File Sharing Setup via FSx:**
    *   You then use the FSx file share by mapping its network path. Within the FSx share, you create folders for user1, user2, and a common folder.
    *   You set permissions so that each user has exclusive access to their own folder, while the common folder is shared (only accessible to members of the awsadmingroup).
7.  **Client Access via Amazon WorkSpaces:**
    *   Amazon WorkSpaces are configured so that when user1 or user2 log in using their AD credentials, they can access the shared FSx storage (and, for example, map it as a network drive). This shows that file sharing works correctly across multiple client sessions.

### What It Achieves

*   **Centralized Authentication and Access Control:**
    By integrating with AWS Managed AD, all users (user1, user2, and groups like awsadmingroup) are managed centrally. This means you can control who gets access to resources such as the FSx share through the AD, rather than managing separate credentials on each server.
*   **Shared File Storage:**
    The FSx file system provides a shared, high-performance storage solution. Any changes (like file creation or deletion) on one instance are reflected on all instances that mount the same file system.
*   **Remote Access for Domain Users:**
    Adding user1 and user2 to the "Remote Desktop Users" group allows these domain users to remotely log in to the Testing instance. This is crucial for scenarios where users need remote management or access to shared resources.
*   **Consistent Environment Across WorkSpaces:**
    When users log into their Amazon WorkSpaces (or the Testing instance), they see the same shared files, which ensures consistency and high availability of data across the environment.
*   **Improved Collaboration and Data Management:**
    The setup supports scenarios like shared Jenkins environments, where job configurations, logs, and artifacts are stored on a common file system that both Jenkins nodes can access. It’s a robust solution for collaborative work and centralized storage.

### In Simple Terms

The entire scenario creates a shared file storage system (FSx) that is securely managed through a central Active Directory. It then configures a Windows server to join this domain and share the FSx file system. By adding specific users (user1 and user2) to the Remote Desktop Users group, it ensures that these users can log in remotely and access their designated file folders (and a common folder). Finally, it demonstrates that changes in one place (like creating or deleting a file) are visible across all instances that access this shared storage. This setup is essential for environments requiring centralized storage, high availability, and controlled user access—all managed from one central directory.

### What Does This Scenario Achieve?

This scenario sets up a fully managed Windows-based Active Directory environment on AWS, integrating various AWS services to enable seamless authentication, file sharing, and virtual desktop infrastructure.

**Key Achievements of This Setup:**

1.  **AWS Managed Microsoft AD Setup**
    *   Creates an Active Directory domain (adminapi.in) managed by AWS, providing centralized authentication and access control.
    *   Enables user and group management via Active Directory.
2.  **Amazon FSx for Windows File Server Integration**
    *   Provides a scalable, high-performance Windows file system for domain users.
    *   Allows secure file sharing across multiple Windows servers and WorkSpaces.
3.  **EC2 Instance as an AD Domain Member**
    *   Launches a Windows Server 2019 EC2 instance, joins it to the domain, and configures it as an Active Directory management node.
    *   Enables centralized user authentication and access control.
4.  **Active Directory User & Group Management**
    *   Creates users and security groups within Active Directory.
    *   Facilitates role-based access control for users within the organization.
5.  **Amazon WorkSpaces for Virtual Desktops**
    *   Registers the AWS Managed AD with Amazon WorkSpaces.
    *   Provides domain-joined virtual desktops (VDI) for remote access by AD users.
    *   Simplifies user access to corporate resources from any location.
6.  **File Sharing with Amazon FSx**
    *   Configures FSx file shares accessible by AD users.
    *   Enables seamless, centralized storage with domain-based authentication.

**Overall Purpose:**

This setup mimics an enterprise IT infrastructure on AWS, providing:

*   Centralized authentication using AWS Managed Microsoft AD
*   Secure file sharing via Amazon FSx for Windows File Server
*   Virtual desktops for remote employees using Amazon WorkSpaces
*   Scalable, managed, and cost-efficient Windows-based infrastructure

### Step-by-Step Breakdown of What Each Step Does

This task sets up a cloud-based Windows Active Directory environment with file sharing and remote desktop access using AWS services. Below is what each step does in simple terms:

**1. Download & Install Amazon WorkSpaces Client**

*   **What?** You install Amazon WorkSpaces Client on your local machine.
*   **Why?** It allows you to connect to virtual Windows desktops that will be created later.

**2. Launch a Windows Server 2019 EC2 Instance**

*   **What?** Creates a Windows Server 2019 instance (testing) in AWS.
*   **Why?** This server will be used for managing Active Directory and as a domain-joined machine.

**3. Create an AWS Managed Active Directory (AD)**

*   **What?** Sets up an Active Directory domain (adminapi.in) in AWS Directory Services.
*   **Why?**
    *   Provides centralized user authentication for Windows-based systems in AWS.
    *   Allows users to log in to Windows machines using domain credentials instead of local accounts.

*   **Key Configurations:**
    *   Chose AWS Managed Microsoft AD (Standard Edition) → AWS manages the AD for you.
    *   Defined a DNS name (adminapi.in) → The domain name used for authentication.
    *   Set an Admin password → Used for managing AD.
    *   Selected VPC & subnets → AD needs network connectivity.
    *   Copied DNS addresses → Required for domain-joining machines.

**4. Create an FSx for Windows File Server**

*   **What?** Deploys Amazon FSx for Windows, a fully managed file system for Windows.
*   **Why?** Allows domain-joined users to share files securely.

*   **Key Configurations:**
    *   Selected Multi-AZ for high availability
    *   32GB storage
    *   Linked it to the Active Directory → Users will access FSx using domain credentials.
    *   Copied FSx DNS name → Used to access shared files later.

**5. Connect to the Windows Server (Testing Instance)**

*   **What?** Connects to the Windows EC2 instance using RDP (Remote Desktop Protocol).
*   **Why?** Needed to join the machine to the domain and manage AD settings.

*   **Steps Taken:**
    *   Retrieved the Public DNS of the EC2 instance.
    *   Decrypted Windows admin password using the .pem key file.
    *   Used Remote Desktop Connection (RDP) to log in to the Windows instance.

**6. Configure Network Settings on the Windows Server**

*   **What?** Updates the DNS settings on the Windows server to use the Active Directory DNS.
*   **Why?** Required for joining the machine to the domain (adminapi.in).

*   **Steps Taken:**
    *   Disabled Windows Firewall (to prevent network issues).
    *   Checked DNS settings (`ipconfig /all`).
    *   Updated DNS manually → Set the AD's DNS as the primary and secondary.
    *   Verified DNS settings after the update.

**7. Install Active Directory Administration Tools**

*   **What?** Installs AD management tools (RSAT) on the Windows Server.
*   **Why?** Required to manage users and groups within the AD domain.

*   **Steps Taken:**
    *   Installed AD DS and AD LDS Tools via Server Manager.

**8. Join Windows Server to Active Directory Domain**

*   **What?** Connects the Windows Server to the Active Directory domain (adminapi.in).
*   **Why?** Enables domain-based authentication instead of local Windows login.

*   **Steps Taken:**
    *   Opened System Properties (`sysdm.cpl`).
    *   Changed Computer Name & Domain to adminapi.in.
    *   Provided admin credentials for authentication.
    *   Restarted the machine to apply changes.

**9. Reconnect to the Windows Server as a Domain User**

*   **What?** Logs into the Windows Server as a domain admin (`admin@adminapi.in`) instead of the local administrator.
*   **Why?** Confirms the server is successfully joined to Active Directory.

**10. Create Users & Groups in Active Directory**

*   **What?** Adds two users (`user1`, `user2`) and a group (`awsadmingroup`) in Active Directory.
*   **Why?** Required for user-based authentication and role-based access control.

*   **Steps Taken:**
    *   Opened Active Directory Users & Computers (`dsa.msc`).
    *   Created users (`user1`, `user2`) and assigned passwords.
    *   Created a group (`awsadmingroup`) and added both users to it.

**11. Register Active Directory with AWS WorkSpaces**

*   **What?** Links the AWS Managed AD to Amazon WorkSpaces (virtual desktops).
*   **Why?** Allows domain users to log into WorkSpaces using AD credentials.

*   **Steps Taken:**
    *   Selected `adminapi.in` AD and registered it with WorkSpaces.
    *   Assigned it to public subnets.

**12. Create WorkSpaces for User1 & User2**

*   **What?** Creates virtual desktops (VDI) for `user1` and `user2`.
*   **Why?** Enables secure, remote access to the corporate environment.

*   **Steps Taken:**
    *   Selected Standard Windows WorkSpace (free-tier).
    *   Assigned WorkSpaces to `user1` and `user2`.
    *   Copied their registration codes (needed for login).

**13. Configure Remote Desktop Access for Users**

*   **What?** Grants `user1` and `user2` permission to log in via Remote Desktop.
*   **Why?** Without this, they cannot access their WorkSpaces.

*   **Steps Taken:**
    *   Opened Local Users & Groups (`lusrmgr.msc`).
    *   Added `user1` & `user2` to Remote Desktop Users.

**14. Configure FSx File Shares & Access Permissions**

*   **What?** Creates shared folders on Amazon FSx and assigns access permissions.
*   **Why?** Controls who can access what files.

*   **Steps Taken:**
    *   Mapped FSx as a network drive (`\\FSx-DNS-Name\share`).
    *   Created folders for each user (`user1`, `user2`, `common`).
    *   Assigned permissions:
        *   `user1` → Access only `user1` folder.
        *   `user2` → Access only `user2` folder.
        *   `awsadmingroup` (both users) → Full access to `common` folder.

**15. Verify WorkSpaces Login & File Access**

*   **What?** Tests whether `user1` & `user2` can log into their WorkSpaces and access FSx file shares.
*   **Why?** Confirms everything is working correctly.

*   **Steps Taken:**
    *   Logged into WorkSpaces as `user1` → Checked if the user could access their private folder & common folder.
    *   Logged into WorkSpaces as `user2` → Verified the same.
    *   Tested file creation & access.
    *   Mapped FSx share as a network drive for convenience.

## Conclusion & Final Thoughts

Congratulations! You have successfully completed this comprehensive guide on AWS FSx, AWS WorkSpaces, and AWS Managed Active Directory. Throughout this journey, we have covered both theoretical concepts and hands-on practical tasks, helping you build a strong foundation in deploying and managing Windows-based workloads in AWS.

### What We Have Learned in This Guide

**Theoretical Concepts**

*   **Introduction to AWS Managed Active Directory (AD)**
    *   What AWS Managed AD is and why enterprises use it.
    *   How AWS integrates with on-premises AD for hybrid cloud setups.
    *   Key benefits like automated maintenance, high availability, and security.

*   **Understanding Amazon FSx for Windows File Server**
    *   What FSx is and how it provides a fully managed Windows file system.
    *   The role of SMB protocol, NTFS file system, and AWS Managed AD integration.
    *   Use cases for FSx in enterprise file storage and application workloads.

*   **Introduction to Amazon WorkSpaces**
    *   How Amazon WorkSpaces provides a secure, scalable, and managed VDI solution.
    *   Key features like persistent desktops, Active Directory integration, and pay-as-you-go pricing.
    *   Comparison of AWS WorkSpaces vs traditional remote desktop solutions.

*   **Identity & Access Management in AWS AD & FSx**
    *   How users and groups are managed in AWS Managed AD.
    *   Best practices for group policies, access controls, and security configurations.

**Hands-On Practical Implementation**

*   **Setting Up AWS Managed Microsoft Active Directory**
    *   Created AWS Managed AD and configured DNS settings.
    *   Ensured secure authentication and domain management.

*   **Deploying Amazon FSx for Windows File Server**
    *   Created an FSx file system and integrated it with AWS Managed AD.
    *   Configured network settings, security groups, and SMB file sharing.

*   **Launching & Configuring an EC2 Instance**
    *   Launched a Windows EC2 instance and joined it to the Active Directory domain.
    *   Installed Active Directory Management Tools and tested domain authentication.

*   **Setting Up Amazon WorkSpaces with AWS AD**
    *   Registered AWS Managed AD with Amazon WorkSpaces.
    *   Created WorkSpaces for domain users (e.g., user1, user2).
    *   Verified connectivity and remote desktop access.

*   **Creating & Managing Shared File System on FSx**
    *   Configured folder shares and NTFS permissions for different users.
    *   Tested access restrictions via EC2 instance and WorkSpaces.

*   **Validating the Complete Setup**
    *   Ensured all components (AD, FSx, EC2, and WorkSpaces) were properly integrated.
    *   Tested user authentication, file sharing, and remote desktop access.

### What’s Next?

Now that you have successfully deployed a fully functional Windows-based cloud infrastructure, you can explore further:

*   **Security Enhancements** – Implement multi-factor authentication (MFA), conditional access policies, and auditing.
*   **Automation** – Use PowerShell scripts, AWS Lambda, or AWS Systems Manager to automate routine tasks.
*   **Scaling & Optimization** – Extend your environment by adding more users, integrating AWS SSO, or connecting with on-premises AD.

This guide has given you both practical skills and theoretical knowledge to work with AWS FSx, WorkSpaces, and Managed AD in a real-world enterprise setup. Keep experimenting, optimizing, and scaling your infrastructure!


