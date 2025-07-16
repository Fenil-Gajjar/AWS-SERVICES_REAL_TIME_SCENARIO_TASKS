
# Welcome to the AWS Managed Microsoft AD Deployment Project

## Overview

This project focuses on deploying and configuring AWS Managed Microsoft Active Directory (AD) and integrating it with IAM Identity Center (formerly AWS SSO). By the end of this project, you will have a fully functional Windows Server joined to a domain, along with synchronized user and group management through AWS IAM Identity Center.

## Project Objectives

*   Launch and configure a Windows Server 2019 instance on AWS.
*   Set up AWS Managed Microsoft AD for domain services.
*   Join the Windows Server to the Active Directory domain.
*   Configure IAM Identity Center to sync with AWS Managed AD.
*   Create users and groups in Active Directory and synchronize them with AWS IAM Identity Center.
*   Assign permissions to users for seamless AWS access.

## Key Benefits

*   **Centralized Identity Management** – Manage user authentication and permissions efficiently.
*   **Seamless Integration with AWS** – Extend AD functionality to AWS services.
*   **Secure Access Control** – Utilize IAM Identity Center for user role-based access.
*   **Scalability & Automation** – Automate user provisioning and domain integration.

## Project Breakdown

*   Part A – Launch & Configure Windows Server for Managed AD
*   Part B – Join Windows Server to the Domain
*   Part C – Configure IAM Identity Center for Managed AD
*   Part D – Verify Domain Functionality
*   Part E – Create Users & Groups in Active Directory
*   Part F – Configure IAM Identity Center Sync

By following this step-by-step guide, you will gain hands-on experience in Windows Server administration, AWS Directory Services, and identity management with IAM Identity Center.

Let’s get started!

## Part A: Launch and Configure the Windows Server for Managed AD

### Step A1: Launch a Windows Server 2019 Instance

1.  Open the AWS Management Console and navigate to EC2 in your desired region (e.g., North Virginia).
2.  Click Launch Instance.
3.  Choose an AMI:
    *   Select Microsoft Windows Server 2019 Base.
4.  Select an Instance Type:
    *   For example, choose `t3.medium` (or a size that meets your requirements).
5.  Configure Instance Details:
    *   Select your VPC (ensure its security group allows all traffic from anywhere, or at least RDP).
    *   Choose a Public Subnet.
    *   Enable Auto-assign Public IP.
6.  Add Tags:
    *   Add a tag with Key: `Name` and Value: `corp.example.com`.
7.  Configure Security Group:
    *   Ensure the security group allows inbound RDP (port 3389) from your IP.
8.  Click Review and Launch and then Launch.
    *   Select your existing PEM key pair when prompted.
9.  Wait for the instance to launch.

### Step A2: Set Up AWS Managed Microsoft AD via Directory Service

1.  Open the AWS Management Console and navigate to Directory Service.
2.  Click Set up directory.
3.  Under Select a directory type, choose AWS Managed Microsoft AD.
4.  Edition: Choose Standard Edition.
5.  Directory DNS Name: Enter `corp.example.com`.
6.  Directory NetBIOS Name: (Typically the short version, e.g., `CORP`).
7.  Administrator Password: Enter a secure password and confirm it.
8.  Click Next and then Create directory.
9.  Wait for the directory creation to complete.
10. Once created, click on the new directory and then select the Networking & Security tab.
    *   Copy the two DNS addresses displayed; you will use them in later steps.

### Step A3: Retrieve the Windows Administrator Password and Connect via RDP

1.  In the EC2 Dashboard, select your `corp.example.com` instance.
2.  Click Connect.
3.  Choose RDP Client.
4.  Copy the Public DNS (or Public IP) of the instance.
5.  Click the Get Password tab.
6.  Click Upload Private Key File, select your PEM file used during launch, and click Decrypt Password.
7.  Copy the generated Administrator password.

### Step A4: Log in to the Windows Server via RDP

1.  On your Windows desktop, open Remote Desktop Connection (search for "RDC").
2.  Paste the Public DNS (or IP) into the Computer field and click Connect.
3.  When prompted by a security dialog, click More choices, then Use a different account.
4.  Enter the Username: `Administrator` and paste the password copied earlier.
5.  Click OK to log in.

### Step A5: Configure Network Settings on the Windows Server

#### A5.1 Configure DNS

1.  Once logged in, open Command Prompt.

Run the command:

```bash
ipconfig /all
```

2.  Note the current IPv4 address and DNS server addresses.
3.  Press Win+R, type `ncpa.cpl`, and press Enter to open the Network Connections window.
4.  Right-click on Ethernet and select Properties.
5.  Select Internet Protocol Version 4 (TCP/IPv4) and click Properties.
6.  Select Use the following DNS server addresses:
    *   Preferred DNS server: Enter the first DNS address you copied from the Directory Service’s Networking & Security tab.
    *   Alternate DNS server: Enter the second DNS address.
7.  Click OK and then Close.
8.  Run `ipconfig` again to verify that the new DNS settings are applied.
9.  Open Windows Firewall by running `firewall.cpl` and turn off the firewall (for testing purposes).

### Step A6: Install Active Directory Domain Services (AD DS) on the Windows Server

#### A6.1 Install AD DS and AD LDS Tools via Server Manager

1.  Open Server Manager:
    *   Click on the Start Menu.
    *   Search for and open Server Manager.
2.  Launch the Add Roles and Features Wizard:
    *   In the Server Manager window, click on Manage in the top-right corner.
    *   Select Add Roles and Features from the drop-down menu.
3.  Begin the Wizard:
    *   On the Before You Begin page, review any prerequisites if needed, then click Next.
4.  Installation Type:
    *   On the Installation Type page, select Role-based or feature-based installation.
    *   Click Next.
5.  Server Selection:
    *   On the Server Selection page, ensure that your current Windows Server (the one you logged into) is selected.
    *   Click Next.
6.  Skip Server Roles:
    *   On the Server Roles page, do not select any additional roles (unless needed for other purposes).
    *   Click Next.
7.  Select Features:
    *   On the Features page, scroll down until you see Remote Server Administrator Tools.
    *   Expand Remote Server Administrator Tools.
    *   Expand Role Administrator Tools.
    *   Find and check the box for AD DS and AD LDS Tools.
    *   (This feature installs the tools needed to manage Active Directory Domain Services and Active Directory Lightweight Directory Services.)
    *   Click Next.
8.  Confirm Installation:
    *   On the Confirmation page, review the selected features.
    *   Click Install.
    *   Wait for the installation process to complete.
    *   Once finished, click Close.

## Part B: Join the Windows Server to the Domain

### Step B1: Join the Domain

On the Windows Server, open Command Prompt or Run dialog (Win+R) and type:

```bash
sysdm.cpl
```

1.  In the System Properties window, click on the Computer Name tab, then click Change….
2.  In the Domain field, enter `corp.example.com`.
3.  Click OK.
    *   You will be prompted to provide credentials for joining the domain.
4.  Enter Username: `admin` (or the account you configured during AD DS promotion) and the corresponding password which we have created during AD DS (`corp.example.com`) directory .
5.  Click OK.
6.  A welcome message should appear indicating that the computer has joined the domain.
7.  Click OK and then Restart the server when prompted.

## Part C: Configure IAM Identity Center to Use Your Managed AD

### Step C1: Enable IAM Identity Center

1.  In the AWS Management Console, navigate to IAM Identity Center (formerly AWS Single Sign-On).
2.  Click on Enable if it’s not already enabled.
3.  Wait for IAM Identity Center to be enabled.

### Step C2: Change the Identity Source to Your Managed AD

1.  In IAM Identity Center, go to the Dashboard.
2.  Click on Settings.
3.  Under the Identity source tab, click Actions → Change Identity source.
4.  Select Active Directory as the identity source.
5.  Choose Existing Directory and select your directory (which has the DNS name `corp.example.com`).
6.  Click Create identity source.
7.  Once created, click Resume sync.
    *   This step configures IAM Identity Center to synchronize users and groups from your managed AD.

## Part D: Verify Domain Functionality on the Windows Server

### Step D1: Log in as a Domain Member

1.  After the reboot, log in via RDP using:
    *   Username: `admin@corp.example.com`
    *   Password: The same password you used during the creation of directory service(`corp.example.com`) .
2.  Confirm that you are now logged into the domain.

### Step D2: Verify Domain Membership

Open Command Prompt and run:

```bash
whoami
```

1.  You should see your username appended with `corp.example.com`.

Then Run:

```bash
dsa.msc
```

2.  This opens Active Directory Users and Computers.
    *   Expand your domain (`corp.example.com`) and verify that you can see the organizational structure.

## Part E: Create Users and Groups in Active Directory

### Step E1: Create New Users

1.  In Active Directory Users and Computers (opened via `dsa.msc`):
    *   Expand your domain (`corp.example.com`).
    *   Right-click on the Users container (or a specific OU if you created one) and select New → User.
2.  Create User1:
    *   User logon name: `user1`
    *   Follow the prompts to set a password.
    *   Occasionally, double-click on `user1`, go to the General tab, and add an Email address (e.g., `user1@gmail.com`).
    *   Click Finish.
3.  Create User2:
    *   Repeat the above steps with:
        *   User logon name: `user2`
        *   Set a password and add an Email (e.g., `user2@gmail.com`).

### Step E2: Create a Group and Add Users

1.  In Active Directory Users and Computers:
    *   Right-click on the Users container (or your chosen OU) and select New → Group.
    *   Group Name: Enter `awsadmingroup`.
    *   Group scope: Global (default) is typically fine.
    *   Click OK.
2.  To add users to the group:
    *   Double-click on the group `awsadmingroup`.
    *   Navigate to the Members tab.
    *   Click Add.
    *   Type `user1` and `user2` (or select them from the list) and click OK.

## Part F: Configure IAM Identity Center to Synchronize with Your Managed AD

### Step F1: Assign Users and Groups in IAM Identity Center

#### F1.a Assign Users for Sync

1.  In the Master Account, open IAM Identity Center.
2.  Navigate to AWS Accounts and select your current management account.
3.  Click Assign users or groups.
4.  In the Users tab, click Manage sync.
5.  Click Add users and groups.
6.  Search for and add `user1` and `user2`.
7.  Click Submit.
8.  Now, in the IAM Identity Center Users section, confirm that both users appear.

#### F1.b Create Permission Sets for These Users

1.  In IAM Identity Center → AWS Accounts:
    *   Select your current management account.
    *   Click Assign users or groups again.
2.  Select Users tab, choose both `user1` and `user2`.
3.  Click Create Permission Set and choose `AdministratorAccess`.
4.  Click Submit.
    *   This grants both `user1` and `user2` administrator-level access when they sign in through the AWS access portal.

#### F1.c Verify Access via AWS Access Portal

1.  In IAM Identity Center Settings, under the Identity source tab, click to open the AWS access portal URL.
2.  Use the credentials for `user1` (username and password as set in Active Directory) to log in.
3.  Verify that `user1` now sees a dashboard with administrator access.

## Final Verification and Summary

### Verification Steps:

1.  **Domain Join Verification:**
    *   Log in to the `corp.example.com` Windows server using RDP with the domain credentials (`admin@corp.example.com`).
    *   Run `whoami` and open Active Directory Users and Computers (`dsa.msc`) to verify domain membership and view your domain structure.
2.  **User and Group Verification:**
    *   Confirm that `user1` and `user2` exist in Active Directory.
    *   Confirm that they are members of the `awsadmingroup` group.
3.  **IAM Identity Center Synchronization:**
    *   In the AWS access portal (accessed via the URL provided in IAM Identity Center settings), log in as `user1`.
    *   Verify that `user1` has the assigned `AdministratorAccess` permission set and can see and manage AWS resources.

### What This Setup Achieves:

*   **Managed AD Deployment:**
    *   The Windows server is promoted to a domain controller with AWS Managed Microsoft AD (`corp.example.com`).
*   **DNS Configuration:**
    *   The server’s DNS settings are updated to use the AD-provided DNS servers.
*   **Domain Join:**
    *   The server successfully joins the domain (`corp.example.com`).
*   **IAM Identity Center Integration:**
    *   The managed AD is set as the identity source in IAM Identity Center.
*   **User & Group Creation:**
    *   Users (`user1` and `user2`) are created in Active Directory and grouped into `awsadmingroup`.
*   **IAM Identity Center Assignment:**
    *   The users are synchronized with IAM Identity Center, and permission sets (`AdministratorAccess`) are assigned.
*   **Access Verification:**
    *   When logging in via the AWS access portal using AD credentials, the user sees full administrative access as intended.

## Understanding the Entire Task: AWS Managed Microsoft AD Integration with IAM Identity Center

This task sets up AWS Managed Microsoft Active Directory (AD) on Windows Server 2019 and integrates it with IAM Identity Center (formerly AWS SSO) for centralized authentication and access control in AWS.

### Why Do This Setup?

1.  **Centralized User & Group Management:**
    *   Instead of manually managing IAM users in AWS, we integrate AWS Managed AD with IAM Identity Center.
    *   Users authenticate using their Active Directory (AD) credentials to access AWS services.
2.  **Single Sign-On (SSO) for AWS Accounts:**
    *   Employees use the same Active Directory credentials to log in to AWS, simplifying authentication.
    *   AWS Identity Center synchronizes users and groups from Managed AD.
3.  **Domain Controller Setup on Windows Server:**
    *   A Windows Server instance is configured as part of AWS Managed AD for domain membership.
    *   Users and groups are created in Active Directory for authentication.

### Step-by-Step Breakdown of What Happens

#### Part A: Launch & Configure the Windows Server

**Goal:** Set up a Windows Server, configure networking, and connect it to AWS Managed AD.

*   **Step A1: Launch Windows Server EC2 Instance**
    *   A Windows Server 2019 instance is launched in AWS.
    *   It is configured with:
        *   A public subnet (so you can access it via RDP).
        *   A security group allowing RDP (port 3389).
        *   A public IP for remote connectivity.

*   **Step A2: Set Up AWS Managed Microsoft AD**
    *   AWS Managed Microsoft AD is created instead of setting up a domain controller manually.
    *   AWS Directory Service is used to create `corp.example.com` as a Managed Microsoft AD domain.
    *   Two DNS addresses are provided by AWS, which are needed for domain joining.
    *   AWS handles the domain controllers automatically, so no manual promotion of Windows Server is needed.

*   **Step A3 & A4: Connect to Windows Server via RDP**
    *   The Windows Admin password is decrypted using the `.pem` key.
    *   Remote Desktop Connection (RDP) is used to log in to the instance.

*   **Step A5: Configure DNS on Windows Server**
    *   The Windows Server must use AWS Managed AD's DNS servers for domain authentication.
    *   The Windows Server’s DNS settings are updated to use the two AWS Managed AD-provided DNS addresses.
    *   This allows the Windows instance to locate and communicate with the AWS AD domain (`corp.example.com`).

*   **Step A6: Install Active Directory Domain Services (AD DS)**
    *   This installs the management tools for handling Active Directory on the Windows Server.
    *   AD DS and AD LDS tools are installed (but the server itself is NOT promoted to a domain controller).
    *   These tools allow managing users, groups, and policies in AWS Managed AD.

#### Part B: Join Windows Server to the Domain

*   The Windows Server must be part of the AWS Managed AD domain (`corp.example.com`).
*   The Windows Server is joined to the AWS Managed AD domain using the system properties (`sysdm.cpl`).
*   The Admin credentials from AWS Managed AD are used to authenticate.
*   After joining, the server restarts to apply domain settings.

#### Part C: Configure IAM Identity Center with AWS Managed AD

*   This enables IAM Identity Center (AWS SSO) to use Active Directory for authentication.
*   IAM Identity Center (AWS SSO) is enabled in AWS.
*   The identity source is changed to Active Directory.
*   AWS IAM Identity Center syncs users and groups from AWS Managed AD.
*   Users can log in to AWS using their Active Directory credentials instead of IAM usernames and passwords.

#### Part D: Verify Domain Functionality

*   Ensures the Windows Server is now using the domain authentication.
*   The user logs in using `admin@corp.example.com`.
*   The domain membership is verified using `whoami`.
*   `dsa.msc` (Active Directory Users & Computers) confirms that the domain is recognized.

#### Part E: Create Users & Groups in Active Directory

*   Users and groups are added to the domain to manage authentication in AWS IAM Identity Center.

*   **Step E1: Create Active Directory Users**
    *   `user1` and `user2` are created in Active Directory (`corp.example.com`).
    *   Their email addresses are also configured.

*   **Step E2: Create a Group & Assign Users**
    *   A group called `awsadmingroup` is created.
    *   `user1` and `user2` are added to this group.

#### Part F: Synchronize Users & Groups with IAM Identity Center

*   Users and groups from AD are assigned AWS permissions for centralized access control.

*   **Step F1: Assign Users & Groups in IAM Identity Center**
    *   IAM Identity Center syncs `user1` and `user2` from Active Directory.
    *   A permission set (`AdministratorAccess`) is assigned to these users.

*   **Step F2: Verify AWS Access via Identity Center**
    *   `user1` logs in via the AWS Access Portal using their AD credentials.
    *   AWS Identity Center confirms Administrator access.

### Final Outcome of This Setup

*   **Windows Server is part of AWS Managed AD**
    *   It is not a domain controller itself but is domain-joined for authentication.

*   **AWS Managed Microsoft AD handles authentication centrally**
    *   Users and groups are created in Active Directory.
    *   The domain is managed by AWS, reducing administrative overhead.

*   **IAM Identity Center integrates with AD for AWS Authentication**
    *   Users log in to AWS using their Active Directory credentials.
    *   Instead of manually creating IAM users, we sync users from Active Directory.
    *   Group-based access control is implemented via IAM Identity Center.

*   **Single Sign-On (SSO) for AWS Management Console**
    *   `user1` and `user2` use Active Directory credentials to log in to AWS.
    *   They get `AdministratorAccess` permissions through IAM Identity Center.

### Summary: What This Setup Achieves

1.  Windows Server is launched & joined to AWS Managed AD.
2.  Active Directory is used to manage users & groups.
3.  IAM Identity Center is configured to sync with AWS Managed AD.
4.  Users can log in to AWS with their AD credentials.
5.  Centralized authentication & group-based access control for AWS resources.

This setup enables enterprises to manage AWS access using existing Active Directory credentials, reducing the need for separate IAM user management.

When `user1` and `user2` log in through the AWS IAM Identity Center (SSO portal) using their Active Directory (AD) credentials, they are granted `AdministratorAccess` to the AWS Management Account (or the specific AWS account where they were assigned this permission set).

### What does `AdministratorAccess` mean in AWS IAM Identity Center?

The `AdministratorAccess` permission set is equivalent to attaching the `AdministratorAccess` AWS IAM policy to an IAM user or role. This means the assigned users get full access to all AWS services and resources in the AWS account(s) they are assigned to.

### What do `user1` and `user2` have access to?

Since they are assigned `AdministratorAccess` to the Management Account, they can:

*   Manage all AWS services (EC2, RDS, S3, VPC, IAM, etc.).
*   Create, modify, and delete AWS resources.
*   Manage IAM users, roles, and permissions.
*   Access billing and cost management.
*   Create and delete other IAM Identity Center users and assign permissions.

### Will they have access to their own accounts or the Main Account?

*   They will have `AdministratorAccess` to the AWS Management Account (or any AWS account where they were explicitly assigned access).
*   If your organization has multiple AWS accounts under AWS Organizations, you can configure IAM Identity Center to grant them `AdministratorAccess` to other linked AWS accounts as well.
*   They do not get personal AWS accounts; instead, they log into the AWS Management Console of the assigned AWS account using their Active Directory credentials.

### How do they log in?

1.  They go to the AWS IAM Identity Center access portal URL (found in IAM Identity Center settings).
2.  They enter their Active Directory username and password (e.g., `user1@corp.example.com`).
3.  Once authenticated, they see a dashboard listing all AWS accounts where they have permissions.
4.  They select the AWS account they have access to, and it logs them in with `AdministratorAccess`.

### Key Takeaway

*   `user1` and `user2` are not getting separate AWS accounts.
*   They are getting full admin access to the Management Account (or any assigned AWS account) via IAM Identity Center (SSO).
*   They do not need separate IAM users; they authenticate with their Active Directory credentials.
*   The organization can control access centrally through IAM Identity Center and AWS Organizations.

