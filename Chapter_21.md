## Chapter 1: Advanced Identity and Account Management in AWS

Welcome to the definitive guide on advanced identity and account administration within the Amazon Web Services (AWS) ecosystem. In this chapter, we will comprehensively cover everything you need to know for your exam regarding AWS Organizations, IAM configurations, IAM Identity Center (formerly SSO), Directory Services, and AWS Control Tower.

---

### 1. AWS Organizations

AWS Organizations is a global service that allows you to manage multiple AWS accounts centrally. When you build an organization, the primary account is designated as the management account, while all other enrolled accounts are known as member accounts. It is important to note that member accounts can only be part of one organization at a time.

Using AWS Organizations provides significant billing and structural advantages:

* **Consolidated Billing:** You can manage payments across all accounts using a single payment method.


* **Pricing Benefits:** Your accounts benefit from aggregated usage, which unlocks volume discounts for services like Amazon EC2 and Amazon S3.


* **Discount Sharing:** Reserved instances and Savings Plans discounts are shared across all accounts within the organization.


* **Automation:** An API is available to automate the creation of new AWS accounts.



#### Organizational Units (OUs)

To logically group your accounts, you can create Organizational Units (OUs). OUs exist under the Root OU and can be structured based on several strategies:

* **Business Unit:** Grouping by departments, such as a Sales OU, Retail OU, and Finance OU, which then contain their respective member accounts.


* **Environmental Lifecycle:** Grouping by deployment stages, such as a Prod OU, Dev OU, and Test OU.


* **Project-Based:** Grouping by specific initiatives, like Project 1 OU, Project 2 OU, and Project 3 OU.



```text
       [ Root Organizational Unit (OU) ]
                     |
                     +---> Management Account
                     |
                     +---> [ OU (Dev) ] 
                     |        +---> Member Accounts
                     |
                     +---> [ OU (Prod) ]
                     |        +---> Member Accounts
                     |
                     +---> [ OU (Finance) ]
                              +---> Member Accounts

```

*An ASCII representation of a basic AWS Organizations structure.*

#### Advantages of AWS Organizations

Employing a multi-account strategy using AWS Organizations is generally preferred over using a single account with multiple VPCs. It allows you to:

* Use tagging standards strictly for billing categorization.


* Enable AWS CloudTrail across all accounts and send logs to a central S3 bucket account for auditing.


* Send CloudWatch Logs to a centralized logging account.


* Establish cross-account IAM roles for administrative access.


* Enforce security baselines using Service Control Policies (SCPs).



---

### 2. Service Control Policies (SCPs)

SCPs are essentially IAM policies applied at the OU or Account level to restrict the permissions of Users and Roles within those member accounts.

* **Crucial Exam Fact:** SCPs do *not* apply to the management account, which always retains full administrative power.


* **Default Deny:** Like standard IAM, SCPs do not allow anything by default; there must be an explicit allow from the root, flowing through each OU in the direct path down to the target account.



#### SCP Hierarchy and Inheritance

If an explicit Deny is placed anywhere in the hierarchy, it applies to all resources beneath it.

```text
[ OU (Root) - FullAWSAccess ]
    |
    +-- Management Account (SCPs do not apply here)
    |
    +-- [ OU (Sandbox) - FullAWSAccess + Deny Athena ]
    |       +-- Account A (Can do anything EXCEPT Athena)
    |
    +-- [ OU (Workloads) - FullAWSAccess + Deny S3 ]
            |
            +-- [ OU (Test) - Allow EC2 ]
            |       +-- Account B (Can access EC2, Denied S3)
            |
            +-- [ OU (Prod) - FullAWSAccess ]
                    +-- Account C (Can do anything EXCEPT S3)

```

#### Blocklist vs. Allowlist Strategies

You can configure SCPs using two main strategies:

* **Blocklist Strategy:** You allow all actions by default (using `Action: "*"` and `Effect: "Allow"`) but specify certain services to explicitly deny (e.g., `Effect: "Deny"`, `Action: "dynamodb:*"`)..


* **Allowlist Strategy:** You only allow specific actions (e.g., `Effect: "Allow"`, `Action: ["ec2:*", "cloudwatch:*"]`). By default, everything else is implicitly denied.



---

### 3. Advanced IAM Features

#### AWS Organizations - Tag Policies

Tag policies help you standardize resource tags across an entire AWS Organization.

* They ensure consistent tagging, assist in auditing resources, and maintain proper categorization.


* You can explicitly define tag keys and their allowed values (e.g., forcing a "CostCenter" tag to only accept values "100" or "200").


* This greatly helps with AWS Cost Allocation Tags and Attribute-Based Access Control (ABAC).


* Tag policies can prevent non-compliant tagging operations on specified services and resources, though they have no effect on resources that are entirely without tags.


* Administrators can generate reports listing all tagged or non-compliant resources and use Amazon EventBridge to actively monitor for non-compliant tags.



#### Important IAM Conditions

IAM conditions allow you to enforce specific rules when API calls are made:

* `aws:SourceIp`: Restricts the client IP address from which the API calls are being made.


* `aws:RequestedRegion`: Restricts the specific AWS region the API calls are made to.


* `ec2:ResourceTag`: Restricts actions based on specific resource tags.


* `aws:MultiFactorAuthPresent`: Forces the presence of Multi-Factor Authentication (MFA) to allow the action.



#### IAM for Amazon S3 and Resource Policies

When defining S3 permissions, note the difference in resource levels:

* `s3:ListBucket` is a bucket-level permission and applies to the bucket ARN itself (e.g., `arn:aws:s3:::test`).


* `s3:GetObject`, `s3:PutObject`, and `s3:DeleteObject` are object-level permissions and apply to the contents inside the bucket (e.g., `arn:aws:s3:::test/*`).



To restrict access strictly to accounts within your organization, you can use the `aws:PrincipalOrgID` condition inside any resource policy. For instance, adding a condition string equals checking `aws:PrincipalOrgID` to the value of your Organization ID (`o-yyyyyyyyyy`) will block users outside the organization from accessing the S3 bucket.

#### IAM Roles vs. Resource-Based Policies

To grant cross-account access, you have two primary methods, which we will compare below without using a table:

**Using an IAM Role as a Proxy:**

* When a user, application, or service assumes a role, they must give up their original permissions and temporarily take on the permissions assigned to that assumed role.



**Using a Resource-Based Policy:**

* When using a resource-based policy, the principal does not have to give up their original permissions.


* This is highly beneficial for workflows. For example, if a user in Account A needs to scan a DynamoDB table in Account A and dump the data into an S3 bucket in Account B, a resource-based policy on the S3 bucket allows them to do both simultaneously.


* Resource policies are supported by services like Amazon S3 buckets, SNS topics, SQS queues, and more.



```text
[ Method 1: Assume Role ]
User (Account A) ----assumes----> Role (Account B) ----accesses----> Amazon S3

[ Method 2: Resource Policy ]
User (Account A) ----granted access by----> S3 Bucket Policy ----accesses----> Amazon S3

```

#### Amazon EventBridge Security

When an EventBridge rule runs, it requires specific permissions on its target.

* **Targets using Resource-based policies:** AWS Lambda, Amazon SNS, Amazon SQS, S3 buckets, API Gateway.


* **Targets using IAM roles:** EC2 Auto Scaling, Systems Manager Run Command, ECS tasks.



#### IAM Permission Boundaries

IAM Permission Boundaries are an advanced feature used to set the *maximum* permissions an IAM entity can have, using a managed policy.

* They are supported for IAM users and roles, but *not* for IAM groups.


* If your IAM policy grants full access to S3, EC2, and CloudWatch, but your Permission Boundary only allows `iam:CreateUser`, your effective permission is **No Permissions** for S3/EC2, because the boundaries and IAM policies must overlap.


* **Effective Permissions:** These are the overlapping intersection of AWS Organizations SCPs, Permission boundaries, and Identity-based policies.


* **Use Cases:**
* Delegating responsibilities to non-administrators within safe boundaries (e.g., allowing someone to create new IAM users without letting them grant admin rights).


* Allowing developers to self-assign policies while ensuring they cannot escalate their own privileges to become administrators.


* Restricting one specific user, as opposed to restricting a whole account which would require an SCP.





#### IAM Policy Evaluation Logic Flow

The evaluation of an IAM policy is strict. The decision always starts with a default Deny.

1. **Evaluate all applicable policies:** The system checks for any explicit Deny.


2. **Explicit Deny:** If there is an explicit Deny anywhere, the final decision is Deny.


3. **Organizations SCP:** If the principal's account is a member of an organization, it checks the SCP. If there is no Allow in the SCP, the final decision is implicitly Denied.


4. **Resource/Identity Policies:** It then evaluates if the resource has a resource-based policy or if the principal has an identity-based policy. If neither has an Allow, it results in an implicit Deny.


5. **Permissions Boundary:** If a boundary is present, there must be an Allow there as well; otherwise, implicit Deny.


6. **Session Policies:** Finally, for session principals, a session policy must also yield an Allow.


7. **Final Allow:** Only if all applicable layers have an Allow (and no explicit Denys exist) is the final decision Allow.



---

### 4. AWS IAM Identity Center and Active Directory

AWS IAM Identity Center is the successor to AWS Single Sign-On (SSO). It provides one single login (single sign-on) for:

* All your AWS accounts within AWS Organizations.


* Business cloud applications such as Salesforce, Box, and Microsoft 365.


* Custom SAML 2.0-enabled applications.


* EC2 Windows Instances.



#### Identity Providers and Setup

IAM Identity Center utilizes a built-in identity store but can also integrate with third-party providers like Active Directory (AD), OneLogin, and Okta.

* **Permission Sets:** This service utilizes Permission Sets, which are collections of one or more IAM Policies assigned to users and groups to define their AWS access.


* **Attribute-Based Access Control (ABAC):** You can define fine-grained permissions based on user attributes (like cost center, title, or locale) stored in the identity store. The primary use case is to define permissions just once, then seamlessly modify a user's AWS access simply by changing their attributes in the directory.



#### Microsoft Active Directory (AD) Concepts

Microsoft Active Directory is a database of objects (User Accounts, Computers, Printers, File Shares, Security Groups) found on Windows Servers running AD Domain Services.

* It provides centralized security management for creating accounts and assigning permissions.


* Objects are organized hierarchically into trees, and a group of trees is referred to as a forest.



#### AWS Directory Services Integration Comparisons

When linking your directories to AWS, you have three primary deployment options (compared below):

**AWS Managed Microsoft AD:**

* Allows you to create your own Active Directory natively in AWS and manage users locally.


* It supports MFA.


* It allows you to establish "trust" connections with your existing on-premises AD.



**AD Connector:**

* Acts as a Directory Gateway (or proxy) to redirect authentication requests to your on-premises AD.


* Users are managed entirely on the on-premises AD, not in AWS.


* It supports MFA.



**Simple AD:**

* Provides an AD-compatible managed directory running on AWS.


* It *cannot* be joined with an on-premises AD.



To connect IAM Identity Center to your directory:

* **AWS Managed Microsoft AD:** Integration is provided out of the box.


* **Self-Managed Directory:** You must either create a Two-way Trust Relationship using AWS Managed Microsoft AD, or create an AD Connector proxy.



---

### 5. Multi-Account Governance with AWS Control Tower

AWS Control Tower provides an easy way to set up and govern a secure, compliant, multi-account AWS environment based on well-architected best practices. Control Tower functions by utilizing AWS Organizations to provision and create accounts under the hood.

**Key Benefits of Control Tower:**

* Automates the setup of your AWS environment in just a few clicks.


* Automates ongoing policy management using mechanisms called guardrails.


* Automatically detects policy violations and can remediate them.


* Allows you to monitor organizational compliance through a centralized, interactive dashboard.



#### Guardrails in Control Tower

Guardrails provide ongoing governance for your Control Tower environment and AWS accounts. They are split into two categories:

* **Preventive Guardrails:** These leverage Service Control Policies (SCPs) to fundamentally block actions. An example would be restricting resource deployment to specific AWS Regions across all your accounts.


* **Detective Guardrails:** These leverage AWS Config to identify non-compliant resources after the fact. An example is identifying untagged resources.



```text
[ Detective Guardrail Workflow ]

AWS Config (Monitors untagged resources in Member Accounts)
       |
       +--- triggers (NON_COMPLIANT) ---> [ Amazon SNS ]
                                                |
                                                +--- notifies ---> Admin
                                                |
                                                +--- invokes ----> [ AWS Lambda ]
                                                                       |
                                                                  remediates
                                                                 (adds tags)

```

*An ASCII flow illustrating how AWS Config identifies an untagged resource and triggers an automated remediation pipeline.*
