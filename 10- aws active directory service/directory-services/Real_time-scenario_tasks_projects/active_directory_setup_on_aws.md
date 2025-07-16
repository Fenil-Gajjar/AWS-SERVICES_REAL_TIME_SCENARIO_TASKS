# Real time Task on Step By Step Implementation of Active Directory Setup

## Scenario Task: Active Directory Setup on AWS

In this scenario, you will:

*   Launch a Windows Server 2019 EC2 instance.
*   Connect to it via Remote Desktop (RDP).
*   Configure the network settings and disable the firewall.
*   Install Active Directory Domain Services (AD DS) using Server Manager, promote the server to a domain controller, and create a new forest.

**Important:**

*   Ensure your VPC’s security group is configured to allow all traffic (or at least RDP, DNS, and necessary management ports) from anywhere.
*   Replace any placeholder values (e.g., domain name, password) with your own desired values.

## Step 1: Launch a Windows Server 2019 EC2 Instance

### 1.1 Launch the Instance

1.  Open the AWS Management Console and navigate to EC2 in your desired region (e.g., North Virginia).
2.  Click on Launch Instance.
3.  Choose an Amazon Machine Image (AMI):
    *   Select Microsoft Windows Server 2019 Base.
4.  Choose an Instance Type:
    *   For example, select `t3.medium` (or another instance type that meets your requirements).
5.  Click Next: Configure Instance Details.
    *   Ensure the instance is launched in the proper VPC and Public Subnet.
    *   Confirm that the security group attached to the instance allows all traffic (or at least RDP) from anywhere.
6.  Click Next: Add Storage and then Next: Add Tags.
7.  (Optional) Add tags to name the instance (e.g., `Name: iam-ad-server`).
8.  Click Next: Configure Security Group.
    *   Verify that the security group allows inbound RDP (port 3389).
9.  Click Review and Launch.
10. Click Launch.
11. When prompted, select your existing PEM key pair (used during launch) and acknowledge that you have access to the private key.
12. Click Launch Instances.

## Step 2: Retrieve the Administrator Password

### 2.1 Obtain the Windows Administrator Password

1.  In the EC2 Dashboard, locate your newly launched instance.
2.  Select the instance and click the Connect button.
3.  In the Connect dialog, choose RDP Client.
4.  Copy the Public DNS (or Public IP) of the instance; you'll need it for RDP.
5.  Click the Get Password tab.
6.  Click Browse and upload the PEM key file you used during instance launch.
7.  Click Decrypt Password.
8.  Copy the generated Administrator password and save it securely.

## Step 3: Connect to the Windows Server via RDP

### 3.1 Launch Remote Desktop Connection

1.  On your local Windows machine, search for and open Remote Desktop Connection.
2.  In the Computer field, paste the Public DNS (or IP) of the EC2 instance.
3.  Click Connect.
4.  When prompted with a security warning, click More choices (if applicable) and then Use another account.
5.  Enter the Username: `Administrator`
6.  Paste the Administrator password you decrypted earlier.
7.  Click OK to log in.

## Step 4: Configure the Windows Server Networking and Firewall

### 4.1 Configure DNS Settings via Command Prompt and Network Connections

1.  Open Command Prompt:
    *   Click on the Start Menu, type `cmd`, and press Enter.

Run the command:

```bash
ipconfig /all
```

2.  Note down the IPv4 Address and the DNS server addresses displayed.
3.  Open Network Connections:
    *   Press Win + R, type `ncpa.cpl`, and press Enter.
4.  In the Network Connections window:
    *   Right-click on the Ethernet adapter and select Properties.
    *   Select Internet Protocol Version 4 (TCP/IPv4) and click Properties.
    *   Choose Use the following DNS server addresses.
        *   Preferred DNS server: Enter the IPv4 address (the main IP from `ipconfig`).
        *   Alternate DNS server: Enter one of the DNS server IPs shown in `ipconfig`.
    *   Click OK and close the window.
5.  Verify the settings by running `ipconfig` again.
6.  Disable Windows Firewall:
    *   Press Win + R, type `firewall.cpl`, and press Enter.
    *   Click Turn Windows Defender Firewall off for both private and public networks.
    *   Click OK.

## Step 5: Install Active Directory Domain Services

### 5.1 Open Server Manager and Add Roles and Features

1.  Click on the Start Menu and open Server Manager.
2.  In Server Manager, click Manage (top-right) and then Add Roles and Features.
3.  On the Before you begin page, click Next.
4.  On the Installation Type page, select Role-based or feature-based installation and click Next.
5.  On the Server Selection page, ensure your current server is selected, then click Next.
6.  On the Server Roles page:
    *   Check Active Directory Domain Services.
    *   When prompted, click Add Features.
7.  On the Features page:
    *   Also check Telnet Client (if not already selected).
    *   Click Next.
8.  On the AD DS page, read the overview, then click Next.
9.  Click Install on the Confirmation page.
10. Wait for the installation to complete (this may take a few minutes).

### 5.2 Promote the Server to a Domain Controller

1.  After installation, a notification flag will appear in Server Manager. Click on the notification flag and then Promote this server to a domain controller.
2.  On the Deployment Configuration page:
    *   Select Add a new forest.
    *   In Root domain name, enter your desired domain name (e.g., `mytestdomain.com`).
    *   Click Next.
3.  On the Domain Controller Options page:
    *   Choose the Domain Functional Level and Forest Functional Level (default is Windows Server 2016 or 2019; select as needed).
    *   Set a Directory Services Restore Mode (DSRM) password (make sure to remember it).
    *   Click Next.
4.  On the DNS Options page:
    *   Accept any warnings and click Next.
5.  On the Additional Options page:
    *   Verify the NetBIOS domain name and click Next.
6.  On the Paths page:
    *   Leave the defaults for Database, Log files, and SYSVOL.
    *   Click Next.
7.  On the Review Options page:
    *   Review your settings, and click Next.
8.  On the Prerequisites Check page:
    *   Wait for the prerequisites check to complete; if all checks pass, click Install.
9.  The server will automatically reboot once the promotion process completes.

## Final Testing and Verification

### After Reboot:

1.  Log in again via RDP with your Administrator credentials.
2.  Open Server Manager.
    *   You should now see an Active Directory Domain Services icon and additional information about your new domain controller.
3.  Verify that the domain controller is functioning properly by opening Active Directory Users and Computers (you can find it in the Tools menu in Server Manager) and checking that the new domain (e.g., `mytestdomain.com`) is listed.

## Summary

*   **Step 1:** Launched a Windows Server 2019 EC2 instance.
*   **Step 2:** Retrieved the Administrator password using the RDP Connect process.
*   **Step 3:** Connected to the instance using Remote Desktop.
*   **Step 4:** Configured network settings (IP configuration, DNS settings) and disabled the Windows Firewall.
*   **Step 5:** Used Server Manager to add the Active Directory Domain Services role and installed additional features (Telnet Client).
*   **Step 6:** Promoted the server to a domain controller by creating a new forest with a random domain name, setting the DSRM password, and accepting defaults.
*   The server rebooted automatically to complete the promotion.
*   **Final Verification:** After logging back in, you can verify that the server is now acting as a domain controller via Server Manager and Active Directory tools.

