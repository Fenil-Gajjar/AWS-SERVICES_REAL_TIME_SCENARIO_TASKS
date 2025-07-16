# Welcome to the AWS RDS with MySQL Implementation Project

## Objective

In this project, we will set up Amazon RDS with MySQL from scratch and connect it to both a Windows Workbench instance and an Ubuntu instance for database operations. This step-by-step guide ensures you gain hands-on experience with AWS RDS, EC2, MySQL Workbench, and Python-based database access.

## What You Will Learn

*   How to create an RDS MySQL database instance
*   Setting up a DB Subnet Group for high availability
*   Launching EC2 instances for database interaction
*   Using MySQL Workbench to manage RDS
*   Connecting an Ubuntu instance to RDS using Python
*   Running SQL queries and automating database interactions

## Why This Project?

Managing a relational database in the cloud is a crucial skill for cloud engineers, DevOps professionals, and database administrators. This hands-on project will help you understand cloud-based database management, security best practices, and real-world database connectivity.

## Prerequisites

*   AWS Account with necessary permissions
*   Basic knowledge of AWS services (EC2, RDS, VPC, Security Groups)
*   Familiarity with MySQL and Python

## Real time Task on Step By Step Implementation of RDS with MYSql

### Part 1: Create RDS MySQL Database

#### Step 1: Create a DB Subnet Group

1.  Open the AWS Management Console and navigate to the RDS service.
2.  In the left navigation pane, click on Subnet groups.
3.  Click Create DB Subnet Group.
4.  Fill in the details:
    *   Name: `subnet-db`
    *   Description: (Optional description, e.g., "DB subnet group for RDS MySQL")
    *   VPC: Select the VPC where you want your RDS instance.
    *   Availability Zones: Select 3 AZs.
    *   Subnets: Choose all 3 public subnets in the selected AZs.
5.  Click Create.

#### Step 2: Create the RDS MySQL Database

1.  In the RDS console, click on Databases in the left-hand menu.
2.  Click Create database.
3.  Database Creation Method: Select Standard Create.
4.  Engine Options:
    *   Engine: Choose MySQL.
    *   Engine Version: Select the latest available version.
5.  Templates: Choose Production.
6.  Availability & Durability:
    *   Select Multi-AZ DB instance.
7.  Settings:
    *   DB Instance Identifier: Enter `main-db`.
    *   Master Username: Enter `admin`.
    *   Credentials Management: Select Self managed, then enter and confirm your desired Master Password.
8.  DB Instance Class:
    *   Choose Burstable classes and select `db.t3.medium`.
9.  Storage:
    *   Storage Type: Select `gp2`.
    *   Allocated Storage: Enter `20` (GB).
10. Connectivity:
    *   Under Connectivity & security, select Don't connect to EC2 computer resource.
    *   Choose your VPC.
    *   DB Subnet Group: Select `subnet-db`.
    *   Public access: Select Yes.
    *   Security Group: Select the appropriate security group (one that allows access on port 3306, if required).
11. Additional Configuration:
    *   Performance Insights: Uncheck Turn on Performance insights.
    *   Monitoring: Check Enable Enhanced Monitoring.
    *   Automated Backups: Uncheck Enable automated backups.
    *   Encryption: Uncheck Enable Encryption.
    *   Auto Minor Version Upgrade: Uncheck Enable auto minor version upgrade.
    *   Deletion Protection: Uncheck Enable deletion protection.
12. Click Create database.
13. Wait until the DB instance is available.
14. Copy the RDS Endpoint (found on the DB instance details page); you will need this for later connections.

### Part 2: Launch EC2 Instances

#### Step 3: Launch a Windows Instance ("workbench") for MySQL Workbench

##### 3.1 Launch Workbench Instance

1.  In the EC2 console (ensure you are in the same region as your RDS DB, e.g., North Virginia), click Launch Instance.
2.  Choose an AMI:
    *   Select a Windows Server 2019 AMI.
3.  Select an Instance Type:
    *   Choose `t2.medium`.
4.  Configure Instance Details:
    *   Select your VPC.
    *   Choose a Public Subnet.
    *   Enable Auto-assign Public IP.
5.  Add Tags:
    *   Add a tag: Key: `Name`, Value: `workbench`.
6.  Configure Security Group:
    *   Use a security group that allows inbound RDP (port 3389).
7.  Launch the Instance:
    *   Click Review and Launch, then Launch.
    *   When prompted, select your PEM key pair.
8.  Wait for the instance to launch.

#### Step 4: Launch an Ubuntu Instance for Python Testing

##### 4.1 Launch Ubuntu Instance

1.  In the EC2 console, click Launch Instance.
2.  Choose an AMI:
    *   Select an Ubuntu 20.04 LTS AMI.
3.  Select an Instance Type:
    *   Choose `t2.micro`.
4.  Configure Instance Details:
    *   Select your VPC.
    *   Choose a Public Subnet.
    *   Enable Auto-assign Public IP.
5.  Add Tags:
    *   Add a tag: Key: `Name`, Value: `ubuntu`.
6.  Configure Security Group:
    *   Use a security group that allows SSH (port 22).
7.  Launch the Instance:
    *   Click Review and Launch, then Launch.
    *   Select your PEM key pair.
8.  Wait for the instance to launch.

### Part 3: Configure the Workbench (Windows) Instance

#### Step 5: Connect to the Workbench Instance via RDP

1.  In the EC2 console, select the `workbench` instance.
2.  Click Connect.
3.  Choose RDP Client.
4.  Copy the Public DNS (or Public IP) of the instance.
5.  Click the Get Password tab.
6.  Click Upload Private Key File, select the PEM file you used during launch, and click Decrypt Password.
7.  Copy the generated Administrator password.
8.  On your Windows computer, open Remote Desktop Connection.
9.  Paste the Public DNS in the Computer field and click Connect.
10. In the pop-up, click More choices and then Use a different account.
11. Enter Username: `Administrator` and paste the copied password.
12. Click OK to log in to the workbench server.

#### Step 6: Configure the Workbench Instance

##### 6.1 Disable Windows Defender Firewall

1.  Open the Command Prompt.
2.  Run: `firewall.cpl`
3.  In the Windows Firewall settings, select Turn Windows Defender Firewall off for both private and public networks.
4.  Click OK.

##### 6.2 Disable Enhanced Security Configuration

1.  Open Server Manager.
2.  Click on Local Server.
3.  Find Enhanced Security Configuration.
4.  Click on it and turn it off for both Administrators and Users.
5.  Close the window.

##### 6.3 Download and Install Google Chrome

1.  Open Internet Explorer on the server.
2.  Search for Chrome download.
3.  Download the installer and run it to install Google Chrome.

##### 6.4 Install Visual C++ Redistributable (x64)

1.  Open Google Chrome.
2.  Search for Visual C++ Redistributable x64.
3.  Visit the official Microsoft page and download the x64 installer.
4.  Run the installer and complete the installation.

##### 6.5 Install MySQL Workbench

1.  Using Google Chrome, search for MySQL Workbench on the official MySQL website.
2.  Download the installer for Windows.
3.  Run the installer and complete the installation.

##### 6.6 Configure MySQL Workbench to Connect to the RDS MySQL Instance

1.  Open MySQL Workbench.
2.  Click on the `+` (Add) icon to create a new connection.
3.  In the Setup New Connection window, enter:
    *   Connection Name: `mainDB`
    *   Connection Method: Standard (TCP/IP)
    *   Hostname: Paste the RDS endpoint you copied earlier from the `main-db` instance.
    *   Port: `3306`
    *   Username: `admin`
4.  Click Store in Vault… and enter the Master Password that you set during RDS creation.
5.  Click Test Connection.
6.  Once the test succeeds, click OK.
7.  In MySQL Workbench, select the new connection (`mainDB`), click to open it.
8.  In the workbench, click on Schemas.
9.  Open a new SQL file, paste SQL queries that create two tables with some sample data, and run the queries.
10. Run additional SQL queries to retrieve and verify the data.

### Part 4: Configure the Ubuntu Instance for Python Database Access

#### Step 7: Connect to the Ubuntu Instance via SSH

1.  On your local computer, open a terminal.
2.  Connect using: `ssh -i your-key.pem ubuntu@<Public_IP_of_Ubuntu_Instance>`

#### Step 8: Install Python3, Pip, and Necessary Libraries

1.  Update the package lists: `sudo apt update`
2.  Install Python 3 and pip: `sudo apt install python3 python3-pip -y`
3.  Install required Python packages (pymysql and SQLAlchemy): `sudo pip3 install pymysql sqlalchemy`

#### Step 9: Create and Run a Sample Python Script

##### 9.1 Create a File Named `sample.py`

1.  Use a text editor (like nano): `nano sample.py`
2.  Paste the following sample code (update the connection string with your actual RDS endpoint, admin username, and Master Password):

```python
from sqlalchemy import create_engine

# Replace with your actual password and RDS endpoint
engine = create_engine("mysql+pymysql://admin:YourMasterPassword@<RDS_ENDPOINT>:3306/main-db")

# Establish a connection
connection = engine.connect()

# Example query: Create two sample tables and insert data (adjust as needed)
create_tables_query = """
CREATE TABLE IF NOT EXISTS table1 (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50)
);
CREATE TABLE IF NOT EXISTS table2 (
    id INT PRIMARY KEY AUTO_INCREMENT,
    description VARCHAR(255)
);
INSERT INTO table1 (name) VALUES (\'Alice\'), (\'Bob\');
INSERT INTO table2 (description) VALUES (\'Description for table2 row1\'), (\'Description for table2 row2\');
"""
connection.execute(create_tables_query)

# Retrieve data from one table
result = connection.execute("SELECT * FROM table1;")
rows = result.fetchall()

# Close the connection
connection.close()

# Print the fetched data
for row in rows:
    print(row)
```

3.  Save the file and exit (Ctrl+X, then Y, then Enter if using nano).

##### 9.2 Run the Python Script

1.  Execute the script using: `python3 sample.py`
2.  Verify that the output displays the rows fetched from `table1`.

## Final Verification and Summary

### What We Have Achieved:

*   **RDS MySQL Database:**
    *   Created a DB subnet group (`subnet-db`) with 3 public subnets (across 3 AZs).
    *   Launched a Multi-AZ RDS MySQL instance (`main-db`) using the Production template with specific configuration (`db.t3.medium`, 20 GB `gp2` storage, public access enabled, and Enhanced Monitoring enabled).
*   **EC2 Instances:**
    *   Launched a Windows (`workbench`) instance (`t2.medium`) to serve as a client for MySQL Workbench.
    *   Launched an Ubuntu instance (`t2.micro`) for Python-based database access.
*   **MySQL Workbench Configuration on Windows:**
    *   Connected to the RDS MySQL instance using the provided endpoint, admin username, and Master Password.
    *   Created tables and inserted sample data, then retrieved and verified the data.
*   **Python Database Access on Ubuntu:**
    *   Installed Python 3, pip, and necessary libraries (`pymysql` and `sqlalchemy`).
    *   Created and ran a Python script to connect to the RDS MySQL instance, execute SQL queries (table creation and data retrieval), and print the retrieved data.

### How to Verify:

*   **MySQL Workbench (Windows):**
    *   Open MySQL Workbench and connect to the `mainDB` connection.
    *   Check the Schemas panel for the created tables and run `SELECT` queries to see the inserted data.
*   **Python Script (Ubuntu):**
    *   Run `python3 sample.py` and confirm that the console output shows the expected data from the database.

## Explanation of the Task: Deploying & Connecting to an AWS RDS MySQL Database

This task sets up an AWS RDS MySQL database, connects to it using MySQL Workbench from a Windows Workbench instance, and retrieves data programmatically from an Ubuntu instance using Python. Below is a breakdown of what exactly this task does and why each step is necessary.

### 1. Setting Up an RDS MySQL Database

#### What are we doing?

We create an Amazon RDS (Relational Database Service) MySQL database with a highly available setup using Multi-AZ deployment and a custom DB subnet group for networking.

#### Why are we doing it?

*   RDS provides a managed database service, reducing operational overhead.
*   Multi-AZ ensures high availability by maintaining a standby replica.
*   DB subnet groups control where the RDS instance is launched within our AWS VPC.

#### Steps Involved:

1.  Create a DB subnet group (Subnet groups define in which subnets the RDS instance can launch).
2.  Launch an RDS MySQL instance using this subnet group.
3.  Enable necessary configurations (such as multi-AZ, security groups, and public access).
4.  Obtain the RDS endpoint for connecting to the database.

### 2. Launching Two EC2 Instances

We create two instances for interacting with the database:

1.  **Workbench Instance (Windows Server 2019)**
    *   Used for graphical database access using MySQL Workbench.
    *   We disable security features to allow easy software installation.
    *   We install Google Chrome, Visual C++ Redistributable, and MySQL Workbench.
2.  **Ubuntu Instance**
    *   Used for programmatically connecting to the RDS database using Python.
    *   We install Python, pymysql, and SQLAlchemy to run scripts that fetch data.

#### Why are we doing this?

*   The Workbench instance allows manual database management via MySQL Workbench.
*   The Ubuntu instance helps automate database interactions using Python scripts.

### 3. Connecting to the RDS MySQL Database from Workbench

#### What are we doing?

After setting up the Workbench instance, we:

1.  Connect to the RDS MySQL database using MySQL Workbench.
2.  Create tables and insert sample data using SQL queries.
3.  Retrieve data manually to verify the database is working.

#### Why are we doing it?

*   Workbench is a GUI-based tool for managing MySQL databases.
*   This step ensures the database is working correctly before using it programmatically.

### 4. Connecting to RDS MySQL from Ubuntu via Python

#### What are we doing?

We:

1.  Install Python and required libraries (pymysql, sqlalchemy).
2.  Write a Python script to:
    *   Establish a connection with the MySQL RDS database.
    *   Retrieve and print data from tables created in Workbench.
    *   Close the connection after execution.

#### Why are we doing it?

*   This step simulates real-world application behavior where backend services interact with a database.
*   Validates programmatic access to the RDS database.

### Summary: What Does This Task Achieve?

*   Creates an AWS RDS MySQL database with a Multi-AZ setup.
*   Launches two EC2 instances (Windows for GUI access, Ubuntu for automation).
*   Connects to MySQL Workbench on Windows to manually manage the database.
*   Programmatically connects to RDS MySQL from Ubuntu using Python scripts.

This setup simulates a real-world cloud-based database environment where different applications and users access the same database for data management and retrieval.

