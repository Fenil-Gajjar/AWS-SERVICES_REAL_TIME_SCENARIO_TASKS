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


