# Chapter 1: AWS Monitoring, Audit, Performance, and Advanced Identity

Welcome to the definitive guide on monitoring, auditing, and securing your AWS environments. As your AI assistant, I have compiled everything you need to know about these critical domains into a comprehensive chapter. We will dive deep into how you can keep a watchful eye on your architecture, maintain rigorous compliance, and master advanced identity management across multiple accounts.

---

## 1. Amazon CloudWatch: The Eyes and Ears of AWS

Amazon CloudWatch is the foundational monitoring and observability service in AWS. CloudWatch provides metrics for every service in AWS.

### Metrics and Dimensions

A metric is a variable that CloudWatch monitors, such as `CPUUtilization` or `NetworkIn`.

* Metrics are organized into containers called **namespaces**.


* A **dimension** is an attribute of a metric, such as an environment or instance ID.


* You can assign up to 30 dimensions per metric.


* All metrics have timestamps attached to them.


* You can create custom CloudWatch dashboards to visualize these metrics.


* If a default metric doesn't exist, you can create CloudWatch Custom Metrics (for example, to track EC2 RAM usage).



### CloudWatch Metric Streams

Metric Streams allow you to continually stream CloudWatch metrics to a destination of your choice.

* Delivery is near-real-time with low latency.


* You can stream to Amazon Kinesis Data Firehose, which can then route metrics to Amazon S3, Amazon Redshift, or Amazon OpenSearch.


* You can also stream to third-party service providers like Datadog, Dynatrace, New Relic, Splunk, and Sumo Logic.


* You have the option to filter metrics so you only stream a specific subset.



### CloudWatch Logs

Logs are essential for troubleshooting and auditing.

* **Log groups:** These have arbitrary names and usually represent a specific application.


* **Log streams:** These represent instances within an application, specific log files, or containers.


* **Expiration:** You can define log expiration policies ranging from 1 day to 10 years, or set them to never expire.


* **Destinations:** CloudWatch Logs can be sent to Amazon S3 (via exports), Kinesis Data Streams, Kinesis Data Firehose, AWS Lambda, or OpenSearch.


* **Security:** Logs are encrypted by default, and you can set up KMS-based encryption using your own keys.



CloudWatch can collect logs from numerous sources, including the SDK, CloudWatch Logs Agent, CloudWatch Unified Agent, Elastic Beanstalk (application logs), ECS (container logs), AWS Lambda (function logs), VPC Flow Logs, API Gateway, CloudTrail (based on filters), and Route53 (DNS queries).

#### S3 Export vs. Subscriptions

If you want to send logs to Amazon S3, you can use an export task. However, log data can take up to 12 hours to become available for export. The API call used is `CreateExportTask`, and it is not a real-time or near-real-time process.

For real-time log processing and analysis, you should use **CloudWatch Logs Subscriptions**.

* Subscriptions deliver real-time log events to Kinesis Data Streams, Kinesis Data Firehose, or Lambda.


* A **Subscription Filter** determines which log events are delivered to your destination.


* **Cross-Account Subscriptions:** You can send log events to resources (like Kinesis) in a entirely different AWS account.



**ASCII Illustration: Multi-Account Log Aggregation**

```text
  ACCOUNT A                       ACCOUNT B (Central Logging)
 +-----------+                   +--------------------------+
 | CW Logs   |---(Sub Filter)---+|                          |
 +-----------+                   |                          |
                                 |                          |
  ACCOUNT C                      |      [Kinesis Stream] -----> [Amazon S3]
 +-----------+                   |                          |
 | CW Logs   |---(Sub Filter)---+|                          |
 +-----------+                   +--------------------------+

```

#### CloudWatch Logs Insights

CloudWatch Logs Insights is a powerful querying engine (not a real-time engine) to search and analyze log data stored in CloudWatch Logs.

* It provides a purpose-built query language to fetch event fields, filter conditions, calculate aggregate statistics, sort events, and limit the number of events.


* It automatically discovers fields from AWS services and JSON log events.


* You can save queries, add them to CloudWatch Dashboards, and even query multiple Log Groups across different AWS accounts.


* Use cases include finding a specific IP inside a log or counting the occurrences of "ERROR".



### CloudWatch Agents

By default, EC2 instances do not send logs to CloudWatch; you must install an agent. Ensure the EC2 instance has the correct IAM permissions to push data. These agents can also be installed on on-premises servers.

* **CloudWatch Logs Agent:** The older version of the agent, which can only send logs to CloudWatch Logs.


* **CloudWatch Unified Agent:** The newer version that collects logs AND additional system-level metrics. It collects detailed metrics such as CPU (active, guest, idle, system, user, steal), Disk metrics (free, used, total, IO), RAM (free, inactive, used, total, cached), Netstat (TCP/UDP connections), Processes (running, dead, idle), and Swap Space. It features centralized configuration using SSM Parameter Store. *(Note: Out-of-the-box EC2 metrics only cover high-level disk, CPU, and network stats).*



### CloudWatch Alarms

Alarms trigger notifications based on metric thresholds using various evaluation options (sampling, percentages, max, min, etc.).

* **States:** OK, INSUFFICIENT_DATA, ALARM.


* **Period:** The length of time in seconds to evaluate the metric. High-resolution custom metrics can be evaluated in 10 seconds, 30 seconds, or multiples of 60 seconds.


* **Targets:** An alarm can stop, terminate, reboot, or recover an EC2 Instance. It can trigger an Auto Scaling Action or send a notification to Amazon SNS.


* **Testing:** You can test alarms and notifications by forcefully setting the state via the CLI: `aws cloudwatch set-alarm-state --alarm-name "myalarm" --state-value ALARM --state-reason "testing purposes"`.



**Composite Alarms**
While standard CloudWatch Alarms evaluate a single metric, **Composite Alarms** monitor the states of multiple other alarms using AND and OR conditions. This is highly helpful to reduce "alarm noise" by creating complex rule sets before an alert is actually sent to SNS.

**EC2 Instance Recovery**
Alarms can trigger EC2 recovery based on status checks.

* *Instance status* checks the EC2 virtual machine itself.


* *System status* checks the underlying hardware.


* *Attached EBS status* checks attached volumes.
If an alarm triggers recovery (e.g., `StatusCheckFailed_System`), the instance is recovered on new hardware but retains its same Private IP, Public IP, Elastic IP, metadata, and placement group.



### Advanced CloudWatch Insights & Monitoring Tools

* **CloudWatch Network Synthetic Monitor:** Detects network performance degradation (packet loss, latency, jitter) between apps hosted on AWS and on-premises data centers. It tests ICMP or TCP traffic over Direct Connect or VPN. It publishes data to CloudWatch Metrics and requires no agents to be installed.


* **CloudWatch Container Insights:** Collects, aggregates, and summarizes metrics and logs from containers. Available for Amazon ECS, Amazon EKS, Fargate, and Kubernetes on EC2. *Note: EKS and Kubernetes use a containerized version of the CloudWatch Agent to discover containers*.


* **CloudWatch Lambda Insights:** A troubleshooting solution for serverless applications. Provided as a Lambda Layer, it collects system-level metrics (CPU time, memory, disk) and diagnostic information (cold starts, worker shutdowns).


* **CloudWatch Contributor Insights:** Analyzes log data to display time-series metrics about the top-N contributors (e.g., top IP addresses, top talkers, heaviest network users, or URLs generating the most errors). It works for any AWS-generated logs like VPC Flow Logs and DNS logs.


* **CloudWatch Application Insights:** Provides automated dashboards showing potential problems with supported applications (Java, .NET, databases) and associated AWS resources (EBS, RDS, ELB, ASG, Lambda, etc.). Powered by SageMaker, it reduces troubleshooting time and sends alerts to EventBridge and SSM OpsCenter.



---

## 2. Amazon EventBridge (Formerly CloudWatch Events)

Amazon EventBridge is the serverless event bus that connects application data from your apps, SaaS, and AWS services.

* **Triggers:** You can trigger events on a schedule using Cron jobs, or define Event Patterns to react to an AWS service doing something (e.g., an IAM Root User sign-in event).


* **Destinations:** EventBridge can trigger Lambda functions, send SQS/SNS messages, integrate with AWS Batch, ECS Tasks, CodePipeline, CodeBuild, Systems Manager (SSM), and execute EC2 actions.


* **Event Buses:**
* *Default Event Bus:* For AWS Services.


* *Partner Event Bus:* For AWS SaaS Partners like Zendesk or Datadog.


* *Custom Event Bus:* For your custom applications.




* **Archiving and Replay:** You can archive all events or filtered subsets sent to a bus indefinitely or for a set period, and you have the ability to replay these archived events later.



### Schema Registry

EventBridge can analyze the events flowing through your bus and infer their schema. The Schema Registry allows you to generate code for your application so it knows in advance how data is structured. Schemas can be versioned.

### Security & Cross-Account Access

EventBridge relies on strictly defined permissions.

* **Resource-based Policies:** Event buses can be accessed by other AWS accounts using Resource-based Policies. For example, you can allow or deny `events:PutEvents` from another AWS Account or Region to aggregate all events from an AWS Organization into a single central event bus. Furthermore, when an EventBridge Rule targets a destination like Lambda, SNS, SQS, S3, or API Gateway, those targets must have a Resource-based Policy allowing EventBridge to invoke them.


* **IAM Roles:** When an EventBridge rule triggers targets like EC2 Auto Scaling, Systems Manager Run Command, or an ECS task, EventBridge assumes an IAM Role to gain permission.



### Intercepting API Calls

EventBridge can intercept API calls seamlessly by integrating with CloudTrail. If a user makes a `DeleteTable` API call in DynamoDB, CloudTrail logs it, sends an event to EventBridge, which can then immediately trigger an SNS alert.

---

## 3. AWS CloudTrail: The Auditor

AWS CloudTrail provides governance, compliance, and auditing for your AWS Account. It provides a history of events and API calls made by the AWS Management Console, SDKs, CLI, and AWS Services.

* CloudTrail is enabled by default.


* If a resource is unexpectedly deleted in AWS, you should investigate CloudTrail first.


* A trail can be applied to all regions (default) or a single region.


* Logs can be put into CloudWatch Logs or an S3 bucket.



### Types of Events

1. **Management Events:** Operations that are performed on resources in your AWS account, such as configuring IAM security, routing rules (CreateSubnet), or setting up logging. By default, trails are configured to log management events. You can separate Read Events (don't modify resources) from Write Events (may modify resources).


2. **Data Events:** High-volume operations that are NOT logged by default. Examples include Amazon S3 object-level activity (`GetObject`, `DeleteObject`) or AWS Lambda function execution activity (`Invoke` API).


3. **CloudTrail Insights Events:** A feature you must actively enable to detect unusual activity, such as inaccurate resource provisioning, hitting service limits, bursts of IAM actions, or gaps in maintenance. Insights analyzes normal management events to establish a baseline, then continuously monitors write events for anomalies. Anomalies appear in the console, are sent to S3, and generate an EventBridge event.



**Event Retention**
Events are stored for 90 days in CloudTrail natively. To keep events for long-term retention beyond this period, log them to Amazon S3 and analyze them using Amazon Athena.

---

## 4. AWS Config: The Configuration Tracker

AWS Config helps with auditing and recording the compliance of your AWS resources by keeping a timeline of configurations and changes. It can answer questions like: *Are there unrestricted SSH access rules? Do buckets have public access? How has my ALB changed over time?*.

* AWS Config is a per-region service, but data can be aggregated across regions and accounts.


* Configuration data can be stored in S3 and analyzed by Athena.



### Config Rules

You can use over 75 AWS managed config rules or create your own custom config rules defined via AWS Lambda. Examples include evaluating if every EBS volume is `gp2` or if every EC2 instance is `t2.micro`.

* Rules are evaluated either for each configuration change or at regular time intervals.


* **Crucial Note:** AWS Config Rules *do not* prevent actions from happening; there is no "deny" enforcement, only a compliance evaluation.


* Pricing does not have a free tier; you are charged $0.003 per configuration item recorded and $0.001 per config rule evaluation per region.



### Remediations and Notifications

* **Remediations:** You can automate the remediation of non-compliant resources using SSM Automation Documents. You can use AWS-Managed Documents or create custom ones (which can also invoke a Lambda function). You can configure Remediation Retries if the resource fails to become compliant after the first auto-remediation.


* **Notifications:** EventBridge can monitor AWS Config and trigger notifications (via SNS, SQS, or Lambda) when resources become `NON_COMPLIANT`. You can also send all configuration change and compliance state events to SNS, using SNS filtering or client-side filtering.



---

## 5. Comparing CloudWatch, CloudTrail, and AWS Config

When distinguishing between these three foundational services, look closely at their core purpose:

**Amazon CloudWatch**

* **Focus:** Performance monitoring, metrics, dashboards, events, alerting, log aggregation, and analysis.


* **Key Action:** Monitoring "How is my system performing?".



**AWS CloudTrail**

* **Focus:** Recording API calls made within your account by users, services, or roles. Trails can be defined for specific resources, and it functions as a global service tracker.


* **Key Action:** Auditing "Who did what, and when?".



**AWS Config**

* **Focus:** Recording configuration state changes and evaluating resources against defined compliance rules to build a timeline of changes.


* **Key Action:** Compliance "What did my system look like, and is it compliant?".



**Applied Example: An Elastic Load Balancer (ELB)**

* **CloudWatch:** Monitoring incoming connection metrics, visualizing error codes as a percentage over time, and feeding a dashboard to gauge ELB performance.


* **CloudTrail:** Tracking the identity of who made modifications to the Load Balancer via API calls.


* **AWS Config:** Tracking the timeline of security group rules and config changes on the ELB, and ensuring an SSL certificate is always assigned to meet compliance.



---

## 6. Advanced Identity and Governance

As your AWS footprint grows, managing access becomes increasingly complex. AWS offers robust tools to centralize identity, billing, and permission boundaries.

### AWS Organizations

AWS Organizations is a global service that manages multiple AWS accounts.

* It consists of one main **Management Account** and multiple **Member Accounts**.


* A member account can only belong to one organization.


* **Billing:** Provides Consolidated Billing across all accounts via a single payment method, and offers pricing benefits from aggregated usage (volume discounts for EC2, S3, and shared Reserved Instances/Savings Plans).


* **Automation:** An API is available to automate new AWS account creation.



**Organizational Units (OUs)**
Accounts can be grouped into OUs based on Business Unit (Sales, Retail, Finance), Environmental Lifecycle (Prod, Dev, Test), or Project-Based needs.

**Service Control Policies (SCPs)**
SCPs are IAM policies applied to OUs or individual Accounts to restrict what Users and Roles can do.

* SCPs do *not* apply to the Management Account; the Management Account retains full administrative power.


* By default, an SCP does not allow anything. A permission must have an explicit allow from the root down through each OU in the direct path to the target account.


* You can utilize both Blocklist and Allowlist strategies.



**ASCII Illustration: SCP Evaluation Hierarchy**

```text
 [Root OU - FullAWSAccess]
        |
        +-- [Management Account] (SCPs do not apply here)
        |
        +-- [Sandbox OU - FullAWSAccess + Deny S3]
        |      |
        |      +-- Account A (Can do anything EXCEPT S3)
        |
        +-- [Workloads OU - Allow EC2 Only]
               |
               +-- Account D (Can ONLY access EC2)

```

**Tag Policies**
Tag policies help standardize tags across resources within an Organization. They ensure consistent tag keys and allowed values, audit tagged resources, and prevent non-compliant tagging operations on specified services. They do not affect resources that have no tags. You can generate compliance reports and use EventBridge to monitor non-compliant tags.

### Advanced IAM Concepts

#### IAM Conditions

You can make policies highly granular using conditions:

* `aws:SourceIp`: Restricts the client IP from which the API calls are made.


* `aws:RequestedRegion`: Restricts the AWS region the API calls are made to.


* `ec2:ResourceTag`: Restricts actions based on resource tags.


* `aws:MultiFactorAuthPresent`: Forces MFA verification.



#### IAM for S3

Permissions in S3 are split between the bucket and the objects within it.

* `s3:ListBucket` is a bucket-level permission and must be applied to the bucket ARN: `arn:aws:s3:::test`.


* `s3:PutObject`, `s3:GetObject`, and `s3:DeleteObject` are object-level permissions and must be applied to the contents: `arn:aws:s3:::test/*`.



#### IAM Roles vs. Resource-Based Policies

For cross-account access, you can either attach a resource-based policy to a resource (like an S3 bucket) or use a role as a proxy.

* When a principal assumes a role, they *give up* their original permissions and take on the permissions of the role.


* When using a resource-based policy, the principal does *not* have to give up their original permissions. This is supported by S3, SNS, SQS, etc..


* **aws:PrincipalOrgID:** This condition key can be used in resource policies to restrict access solely to accounts that are members of your specific AWS Organization.



#### IAM Permission Boundaries

Permission boundaries are advanced features supported for IAM users and roles (but not groups). They use a managed policy to set the *maximum* permissions an IAM entity can obtain.

* If an IAM policy grants `iam:CreateUser` but the Permission Boundary only allows `s3:*`, `cloudwatch:*`, and `ec2:*`, the effective permission is *No Permissions* for creating users.


* This is highly useful for delegating responsibilities (like letting developers self-assign policies) while ensuring they cannot escalate their own privileges to become administrators.


* Effective permissions are the intersection of Organizations SCPs, Permission Boundaries, and Identity-Based policies.



#### IAM Policy Evaluation Logic

When AWS evaluates an API request, the decision always starts with a default Deny.

1. It checks for any explicit Deny across all applicable policies. If an explicit Deny is found, the final decision is Deny.


2. It then evaluates Organizations SCPs, Resource-based policies, Identity-based policies, IAM permissions boundaries, and Session policies.


3. If an explicit Allow is found and no explicit Deny overrides it, the action is Allowed.


4. If no Allow is found anywhere, it falls back to an implicit Deny.



### AWS IAM Identity Center (Formerly AWS SSO)

IAM Identity Center provides a single login (single sign-on) for all your AWS accounts in an Organization, business cloud applications (Salesforce, Microsoft 365, Box), custom SAML 2.0-enabled apps, and EC2 Windows instances.

* Identity providers can be the built-in IAM Identity Center Identity Store or a 3rd party like Active Directory, OneLogin, or Okta.


* **Permission Sets:** Collections of one or more IAM policies assigned to users and groups to define AWS access across the organization.


* **Attribute-Based Access Control (ABAC):** Provides fine-grained permissions based on user attributes (e.g., cost center, title) stored in the Identity Store. You define permissions once, then modify access simply by changing user attributes.



#### Microsoft Active Directory (AD) Integration

Microsoft AD is a database of objects (users, computers, security groups) organized into trees and forests that provides centralized security management. AWS Directory Services offers three integration models for IAM Identity Center:

1. **AWS Managed Microsoft AD:** Create your own AD managed natively in AWS. It supports MFA and can establish "trust" connections with your on-premises AD.


2. **AD Connector:** Acts as a directory gateway (proxy) redirecting authentication requests to your on-premises AD. Users are managed purely on-premises, and it supports MFA.


3. **Simple AD:** An AD-compatible managed directory on AWS. It *cannot* be joined with an on-premises AD.



### AWS Control Tower

AWS Control Tower provides an easy way to set up and govern a secure, compliant multi-account AWS environment based on best practices. It utilizes AWS Organizations to provision accounts in a few clicks.

* It automates ongoing policy management using **Guardrails**.


* **Preventive Guardrails:** Use SCPs to strictly prevent actions, such as restricting regions across all accounts.


* **Detective Guardrails:** Use AWS Config to identify policy violations (like untagged resources) after the fact. These can trigger SNS notifications to administrators and invoke Lambda functions to automatically remediate the issue.


* Control Tower provides an interactive dashboard to monitor overall compliance.
