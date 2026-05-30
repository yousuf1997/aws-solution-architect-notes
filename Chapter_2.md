# Chapter: AWS Identity and Access Management (IAM)

## 1. Introduction to IAM

AWS Identity and Access Management (IAM) is a global service that acts as the cornerstone of security and access control within your Amazon Web Services environment. When you first create an AWS account, a **Root account** is created by default. As a fundamental security practice, this root account should only be used for initial setup and should never be used for daily tasks or shared with others.

## 2. Users and Groups

IAM allows you to mirror your real-world organizational structure inside AWS.

* **Users:** Users represent the physical people within your organization. As a best practice, one physical user should map to exactly one AWS user.


* **Groups:** Users can be organized into Groups.


* **Group Rules:** Groups can only contain users; they cannot contain other groups.


* **User Flexibility:** Users are not required to belong to a group, and a single user can simultaneously belong to multiple groups.



## 3. Permissions and Policies

To give users or groups the ability to perform actions in AWS, you must assign them permissions.

* Users or Groups can be assigned JSON documents called **policies**.


* These policies strictly define the permissions of the users.


* AWS operates on the **least privilege principle**: you should never give a user more permissions than they explicitly need to do their job.


* **Inheritance:** Users automatically inherit the policies of any group they are placed in, but they can also have direct, "inline" policies assigned exclusively to them.



### IAM Policy Structure

IAM policies are written in JSON and follow a highly specific structure. A policy consists of the following elements:

* **Version:** The policy language version, which must always be set to `"2012-10-17"`.


* **Id:** An identifier for the policy (this is optional).


* **Statement:** One or more individual statements (this is strictly required).



Each individual Statement contains the following building blocks:

* **Sid:** An identifier for the specific statement (optional).


* **Effect:** Dictates whether the statement allows or denies access (can be `Allow` or `Deny`).


* **Principal:** The account, user, or role to which this policy is being applied.


* **Action:** The specific list of API actions this policy either allows or denies.


* **Resource:** The list of AWS resources to which the defined actions apply.


* **Condition:** Specific conditions dictating when this policy is in effect (optional).



## 4. Securing Your Account

Because users have access to your account and can change configurations or delete resources, securing identities is paramount.

### Password Policy

Implementing a strong password policy leads to higher security. In AWS, you can configure your password policy to:

* Set a minimum password length.


* Require specific character types, including uppercase letters, lowercase letters, numbers, and non-alphanumeric characters.


* Allow all IAM users to change their own passwords.


* Require users to change their password after a set period (password expiration).


* Prevent password re-use.



### Multi-Factor Authentication (MFA)

You must protect your Root Accounts and IAM users. MFA operates on the principle of requiring a password you know *plus* a security device you own. The primary benefit of enforcing MFA is that even if a password is stolen or hacked, the account cannot be compromised without physical access to the device.

AWS supports several MFA device options:

* **Virtual MFA devices:** Apps like Google Authenticator or Authy, which are phone-only. Authy supports multiple tokens on a single device.


* **Universal 2nd Factor (U2F) Security Key:** Devices like the YubiKey by Yubico (a 3rd party), which support multiple root and IAM users on a single physical key.


* **Hardware Key Fob MFA Device:** Physical fobs provided by 3rd parties like Gemalto (Thales) or SurePassID.



## 5. Accessing AWS

Users have three primary options to access and interact with AWS:

1. **AWS Management Console:** The web-based graphical interface, protected by a password and MFA.


2. **AWS Command Line Interface (CLI):** Protected by access keys. The CLI is an open-source tool that enables you to interact with AWS services using commands in your command-line shell. It provides direct access to the public APIs of AWS services and allows you to develop scripts to manage your resources.


3. **AWS Software Developer Kit (SDK):** Protected by access keys, this is used for writing code. The SDK consists of language-specific APIs (a set of libraries) embedded within your application that enable you to manage AWS services programmatically. AWS supports various SDKs (JavaScript, Python, PHP, .NET, Java, etc.), Mobile SDKs (Android, iOS), and IoT Device SDKs.



**Understanding Access Keys:**
To use the CLI or SDK, you need Access Keys.

* Access Keys are generated through the AWS Console, and users manage their own keys.


* They act exactly like a username and password: The **Access Key ID** acts as the username, and the **Secret Access Key** acts as the password.


* Access keys are strictly confidential and must never be shared.



## 6. IAM Roles for Services

Sometimes, AWS services themselves will need to perform actions on your behalf (for example, a virtual server needing to read a file from a database). To do this securely, we assign permissions directly to AWS services using **IAM Roles**. Common examples include:

* EC2 Instance Roles.


* Lambda Function Roles.


* Roles for CloudFormation.



## 7. IAM Security and Auditing Tools

AWS provides built-in tools to help you audit the permissions within your account:

* **IAM Credentials Report (account-level):** This is a comprehensive report that lists all users in your account alongside the status of their various credentials.


* **IAM Access Advisor (user-level):** This tool shows the specific service permissions granted to a user and, crucially, when those services were last accessed. You can use this data to revise your policies and remove unused permissions, enforcing the principle of least privilege.



## Chapter Summary & Best Practices Checklist

To succeed with IAM, enforce these core best practices:

* **Do not use the root account** except for initial AWS account setup.


* Assign users to groups and **assign permissions to the groups**, rather than to individual users.


* Create a strong password policy and **enforce the use of MFA** across your organization.


* Use **Roles** to give permissions to AWS services.


* Audit account permissions regularly using the **IAM Credentials Report** and **IAM Access Advisor**.


* **Never share** IAM users or Access Keys.
