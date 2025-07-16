
# Real time Task on Step By Step Implementation of IAM Switching Roles

## Introduction

In modern cloud environments, secure access management is crucial for handling multiple AWS accounts efficiently. IAM Role Switching allows users and services to assume different roles dynamically, ensuring temporary and controlled access without directly assigning excessive permissions.

This real-time task will guide you through a step-by-step implementation of IAM Role Switching across multiple AWS accounts:

*   Master Account (centralized user management)
*   QA Account (for testing and validation)
*   Staging Account (for pre-production workflows)

## What You’ll Learn in This Task

*   How to create and configure IAM roles in multiple AWS accounts
*   How to set up cross-account access securely using IAM policies
*   How to create IAM users and groups in the master account
*   How to update trust relationships to allow role assumptions
*   How to switch roles using the AWS Console and CLI

## Why is This Important?

*   **Enhances Security:** Users get permissions only when needed (Least Privilege Principle).
*   **Simplifies Multi-Account Access:** No need to manage multiple credentials across environments.
*   **Ensures Better Access Control:** Only authorized users can assume privileged roles.
*   **Facilitates DevOps & CI/CD Practices:** Helps teams manage environments efficiently.

## Who Should Follow This Task?

*   DevOps Engineers managing AWS infrastructure across multiple accounts
*   Cloud Administrators implementing best practices for IAM security
*   Security Engineers looking to enforce controlled role-based access
*   Developers who need temporary elevated access to QA and Staging environments

## Next Steps

Let's dive into the step-by-step implementation and set up secure, cross-account IAM Role Switching!

## Part A: Create Roles in QA and Staging Accounts

### A.1 In the QA Account: Create an IAM Role (QA-Role)

1.  Log in to the QA account’s AWS Management Console.
2.  Navigate to IAM → Roles.
3.  Click on Create role.
4.  Select Trusted Entity Type:
    *   Choose AWS account.
5.  For “An AWS account” section:
    *   Select Another AWS account.
    *   Enter the Master Account ID (for example, `<MASTER_ACCOUNT_ID>`).
6.  Click Next: Permissions.
7.  Attach Policy:
    *   From the list, search for and select the `AdministratorAccess` policy. (This grants full administrative permissions when the role is assumed.)
8.  Click Next: Tags (adding tags is optional).
9.  Click Next: Review.
10. Name the Role: Enter `QA-Role`.
11. Click Create role.

### A.2 In the Staging Account: Create an IAM Role (Staging-Role)

1.  Log in to the Staging account’s AWS Management Console.
2.  Navigate to IAM → Roles.
3.  Click on Create role.
4.  Select Trusted Entity Type:
    *   Choose AWS account.
5.  In the An AWS account section, select Another AWS account and enter the Master Account ID.
6.  Click Next: Permissions.
7.  Attach Policy:
    *   Search for and select `AdministratorAccess`.
8.  Click Next: Tags (optional) and then Next: Review.
9.  Name the Role: Enter `Staging-Role`.
10. Click Create role.

## Part B: Create IAM Users and an Admin Group in the Master Account

### B.1 Create IAM Users in the Master Account

#### B.1.a Create User “admin1”

1.  Log in to the Master account’s AWS Management Console.
2.  Navigate to IAM → Users.
3.  Click Add users.
4.  User Name: Enter `admin1`.
5.  Access Type: Check AWS Management Console access.
6.  For Console password, select Custom password and enter your desired password.
7.  Uncheck the option “User must create a new password at next sign in”.
8.  Click Next: Permissions.
9.  Do not attach any policies (we’ll manage permissions via group policies later).
10. Click Next: Tags (optional).
11. Click Next: Review, then click Create user.

#### B.1.b Create User “admin2”

1.  In the IAM Console → Users, click Add users again.
2.  User Name: Enter `admin2`.
3.  Follow the same steps as for `admin1` (custom password, no policies attached).
4.  Click Create user.

### B.2 Create an Admin User Group in the Master Account

1.  In the IAM Console → User groups.
2.  Click Create user group.
3.  Group Name: Enter `admin-group`.
4.  In the list of users, select both `admin1` and `admin2` to add them to the group.
5.  Click Create group.

### B.3 Attach an Inline Policy to the Admin Group (admin-group) to Allow AssumeRole

1.  In the IAM Console, navigate to User groups and select `admin-group`.
2.  Go to the Permissions tab.
3.  Click Add permissions → Create inline policy.
4.  Choose the JSON tab.

Paste the following policy JSON (update the ARNs with the actual role ARNs from QA and Staging accounts):

```json
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": "sts:AssumeRole",
"Resource": [
"arn:aws:iam::<QA_ACCOUNT_ID>:role/QA-Role",
"arn:aws:iam::<STAGING_ACCOUNT_ID>:role/Staging-Role"
]
}
]
}
```

5.  Click Next: Tags (optional).
6.  Policy Name: Enter `UG-Policy`.
7.  Click Create policy.

This inline policy allows any user in the `admin-group` to assume the roles `QA-Role` (in QA Account) and `Staging-Role` (in Staging Account).

## Part C: Update the Trust Policy for the Roles in QA and Staging Accounts

### C.1 In the QA Account: Modify Trust Policy for QA-Role

1.  Log in to the QA account’s AWS Console.
2.  Navigate to IAM → Roles and select `QA-Role`.
3.  Click on the Trust relationships tab.
4.  Click Edit trust policy.

Replace (or merge) with the following JSON (replace `<MASTER_ACCOUNT_ID>` with your actual master account ID):

```json
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Principal": {
"AWS":
"arn:aws:iam::<MASTER_ACCOUNT_ID>:user/admin1"
},
"Action": "sts:AssumeRole"
}
]
}
```

5.  Note: You may also include additional services (EC2, Lambda, SSM) if desired as secondary principals.
6.  Click Update Policy.

### C.2 In the Staging Account: Modify Trust Policy for Staging-Role

1.  Log in to the Staging account’s AWS Console.
2.  Navigate to IAM → Roles and select `Staging-Role`.
3.  Click on the Trust relationships tab.
4.  Click Edit trust policy.

Update the JSON to allow the Master account’s user (e.g., `admin1`) to assume the role:

```json
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Principal": {
"AWS":
"arn:aws:iam::<MASTER_ACCOUNT_ID>:user/admin1"
},
"Action": "sts:AssumeRole"
}
]
}
```

5.  Click Update Policy.

These updates allow the Master account’s users (in our case, members of `admin-group`, especially `admin1`) to assume the QA and Staging roles.

## Part D: Accessing/Switching Roles

### D.1 First Way – Switching Roles via the AWS Management Console

1.  Log in to the Master account as the user `admin1` (who is part of `admin-group`).
2.  In the console, click on your account name at the top right.
3.  Select Switch Role from the dropdown.
4.  In the Switch Role form:
    *   Account ID: Enter the Staging Account ID.
    *   Role: Enter the Role Name that exists in the Staging account (i.e., `Staging-Role`).
    *   (Optionally, you can also specify a display color and icon for the switched role.)
5.  Click Switch Role.
6.  You are now logged in with the temporary credentials and permissions of `Staging-Role`.
    *   Verify by navigating to resources (e.g., EC2 Dashboard) and confirming that you see what an administrator in the Staging account would see.

### D.2 Second Way – Directly Granting AdministratorAccess via Inline Policy

1.  In the Master Account, go to IAM → User Groups and select the group `admin-group`.
2.  Click Add Permissions → Create inline policy.
3.  Use the Visual editor or JSON tab and select the `AdministratorAccess` policy.
4.  Policy Name: Enter `Admin-Direct-Access`.
5.  Click Create policy.
6.  Now, any user in the `admin-group` (for example, `admin2`) will have full `AdministratorAccess`.
7.  Log in as `admin2`.
8.  Navigate to various AWS services (such as EC2, S3, etc.) to confirm that you have the same level of access as in the Master account.

## Final Summary and Verification

### Verification for Switching Roles (Method 1)

*   Log in to the Master account as `admin1`.
*   Use Switch Role to assume the `Staging-Role` in the Staging account.
*   Verify that your permissions, resource view, and AWS Management Console experience reflect those of the Staging account (for example, viewing EC2 instances in the Staging account).

### Verification for Direct Admin Access (Method 2)

*   Log in as `admin2` (member of `admin-group` with the direct inline `AdministratorAccess` policy).
*   Verify that you can see and manage all resources (EC2, S3, etc.) just as if you were in the Master account.

## What Does This Entire Setup Achieve?

*   **Centralized Administration:**
    *   The Master account’s users (`admin1` and `admin2`) can assume roles in other accounts (QA and Staging) to perform administrative tasks without requiring separate credentials for each account.
*   **Security & Least Privilege:**
    *   Instead of granting permanent, broad permissions to users, you enable role assumption, so users get temporary, controlled access only when needed.
*   **Flexible Access Control:**
    *   Users can switch roles via the AWS Management Console, or be granted direct administrative access through an inline policy, providing two methods to access resources across accounts.

Let’s break it down:

### 1. Admin1 User - Role Switching to Staging Account

*   When `admin1` logs into the Master Account, they initially have no permissions to do anything.
*   But when they switch roles to the `Staging-Role` in the Staging Account, they effectively assume the permissions of that role.
*   Since the `Staging-Role` in the Staging Account has `AdministratorAccess`, `admin1` gets full control over the Staging Account while they are in that session.
*   They can:
    *   View and modify resources (EC2, S3, IAM, etc.) in the Staging Account
    *   Perform administrative actions as if they are a user inside the Staging Account
    *   But once they switch back to the Master Account, they lose those permissions.

**Conclusion:** When `admin1` switches roles, they see the Staging Account's console and can make changes there because they assumed a role with `AdministratorAccess` in that account.

### 2. Admin2 User - Full Admin in Master Account

*   `admin2` is in the Master Account, but we attached `AdministratorAccess` directly to the `admin-group`.
*   Since `admin2` is in that group, they now have full admin rights in the Master Account itself.
*   `admin2` will:
    *   See the Master Account’s EC2, IAM, S3, and all resources.
    *   Have full control over everything inside the Master Account, including user management.

**Conclusion:** `admin2` remains inside the Master Account but sees and controls everything in the Master Account because of `AdministratorAccess`.

### Key Takeaways

*   `admin1` can switch roles to the Staging Account and get `AdministratorAccess` there.
*   `admin2` stays in the Master Account but has full admin control over it.
*   Role switching isolates permissions per session, while direct `AdministratorAccess` gives full access to the account itself.

### What Happens After Switching Roles?

*   `Admin1` or `Admin2` sees the AWS console of the QA/Staging Account.
*   They now have `AdministratorAccess` permissions inside that account.
*   They cannot access the Master Account’s resources unless they switch back.

### Final Confirmation

Yes, all users in the `admin-group` can switch roles to access both QA and Staging accounts using the AWS Console or CLI.

