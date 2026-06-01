## Chapter 1: Comprehensive AWS Security and Encryption

Understanding how to secure your infrastructure, manage encryption keys, and protect against external threats is paramount when architecting on AWS. This chapter covers everything you need to know about AWS security and encryption services, ranging from foundational encryption concepts to advanced threat detection mechanisms.

---

### 1. The Fundamentals of Encryption

Before diving into AWS-specific services, it is essential to understand the three primary states and strategies for encrypting data.

#### Encryption in Flight (TLS / SSL)

Data transmitted across networks is vulnerable to interception.

* Data is encrypted before sending and decrypted after receiving.


* TLS certificates help facilitate this encryption, commonly seen as HTTPS.


* Encryption in flight ensures that no Man in the Middle (MITM) attack can happen.



**ASCII Illustration: Encryption in Flight**

```text
[ Client ] ---- (Plaintext) ----> [ TLS Encryption ]
                                         |
                                (Encrypted Data Stream)
                                         |
[ HTTPS Website Server ] <---- [ TLS Decryption ]

```

#### Server-Side Encryption at Rest

With server-side encryption, the responsibility of encrypting the data falls on the destination server.

* Data is encrypted after being received by the server.


* Data is decrypted before being sent back to a requester.


* It is stored in an encrypted form thanks to a key, which is usually a data key.


* The encryption and decryption keys must be managed somewhere, and the server must have access to them.



#### Client-Side Encryption

For scenarios requiring the highest level of data privacy from the cloud provider, client-side encryption is used.

* Data is encrypted by the client and is never decrypted by the server.


* Data will only be decrypted by a receiving client.


* The server should not be able to decrypt the data.


* This approach could leverage Envelope Encryption.



**ASCII Illustration: Client-Side Encryption**

```text
[ Client Application ]
       |
  (Plaintext Object) + (Client-Side Data Key)
       |
  [ ENCRYPTION ]
       |
  (Encrypted Object)
       |
[ Any Storage Service (e.g., S3, FTP) ] -- Server never sees the plaintext or key

```

---

### 2. AWS Key Management Service (KMS)

AWS Key Management Service (KMS) is the central service for encryption in AWS. Anytime you hear the word "encryption" associated with an AWS service, it is most likely referring to KMS.

#### Core Concepts of KMS

* AWS manages encryption keys for us.


* KMS is fully integrated with IAM for authorization, providing an easy way to control access to your data.


* You are able to audit KMS Key usage using AWS CloudTrail.


* It is seamlessly integrated into most AWS services, including EBS, S3, RDS, and SSM.


* A fundamental rule is to never ever store your secrets in plaintext, especially in your code.


* KMS Key Encryption is also available through API calls using the SDK or CLI.


* Encrypted secrets can be safely stored in the code or as environment variables.



#### KMS Key Types

KMS Keys is the new terminology for what was formerly known as KMS Customer Master Keys (CMK).

**Symmetric Keys**

* These are AES-256 keys.


* They consist of a single encryption key that is used to both Encrypt and Decrypt.


* AWS services that are integrated with KMS use Symmetric keys.


* You never get access to the KMS Key unencrypted; you must call the KMS API to use it.



**Asymmetric Keys**

* These utilize RSA and ECC key pairs.


* They consist of a Public Key for encryption and a Private Key for decryption.


* They are used for Encrypt/Decrypt, or Sign/Verify operations.


* The public key is downloadable, but you cannot access the Private Key unencrypted.


* A primary use case is encryption outside of AWS by users who cannot call the KMS API.



#### KMS Key Categories, Pricing, and Rotation

* **AWS Owned Keys:** These are free and include SSE-S3, SSE-SQS, and SSE-DDB (which is the default key).


* **AWS Managed Keys:** These are free and typically follow the naming convention `aws/service-name`, for example, `aws/rds` or `aws/ebs`. Automatic key rotation happens every 1 year.


* **Customer Managed Keys (Created in KMS):** These cost **$1 / month**. Automatic rotation must be enabled, but it supports both automatic and on-demand rotation.


* **Customer Managed Keys (Imported):** These cost **$1 / month**. Only manual rotation is possible using an alias.


* **API Calls:** You pay for API calls to KMS at a rate of **$0.03 / 10000 calls**.



#### KMS Key Policies

KMS Key Policies control access to KMS keys, similar to how S3 bucket policies work. However, the key difference is that you cannot control access to a KMS key without them.

* **Default KMS Key Policy:** This is created if you do not provide a specific KMS Key Policy. It grants complete access to the key to the root user, which encompasses the entire AWS account.


* **Custom KMS Key Policy:** This allows you to define the specific users and roles that can access the KMS key. You can define who can administer the key, which is especially useful for cross-account access of your KMS key.



---

### 3. Advanced KMS Operations

#### Copying Snapshots Across Regions

When copying an encrypted EBS Snapshot to a different region, the snapshot must be re-encrypted.

1. Start with an EBS Volume encrypted with KMS Key A in Region 1.


2. Take an EBS Snapshot, which remains encrypted with KMS Key A.


3. When copying to Region 2, use KMS ReEncrypt to encrypt the new snapshot with KMS Key B (a key located in the target region).


4. Restore the new EBS Volume in Region 2, now encrypted with KMS Key B.



#### Copying Snapshots Across Accounts

To share encrypted snapshots across AWS accounts, follow this strict procedure:

1. Create a Snapshot, encrypted with your own KMS Key (Customer Managed Key).


2. Attach a KMS Key Policy to authorize cross-account access.


3. Share the encrypted snapshot.


4. In the target account, create a copy of the Snapshot, and encrypt it with a CMK located in the target account.


5. Create a volume from the newly copied snapshot.



> The IAM Role/User in the target account must have permissions to DescribeKey, ReEncrypt*, CreateGrant, and Decrypt.
> 
> 

#### KMS Multi-Region Keys

Multi-Region Keys are identical KMS keys located in different AWS Regions that can be used interchangeably.

* Multi-Region keys have the exact same key ID, key material, and automatic rotation configuration.


* You can encrypt data in one Region and decrypt it in other Regions.


* There is no need to re-encrypt data or make cross-Region API calls.


* KMS Multi-Region keys are NOT global; they consist of a Primary key and Replicas that are synced.


* Each Multi-Region key is managed independently.


* Use cases include global client-side encryption, encryption on Global DynamoDB, and Global Aurora.



#### Cross-Region Implementations with Multi-Region Keys

**DynamoDB Global Tables:**

* You can encrypt specific attributes client-side in your DynamoDB table using the Amazon DynamoDB Encryption Client.


* Combined with Global Tables, the client-side encrypted data is replicated to other regions.


* By using a multi-region key replicated in the same region as the DynamoDB Global table, clients in these regions can use low-latency API calls to KMS in their local region to decrypt the data client-side.


* This protects specific fields and guarantees decryption only occurs if the client has access to an API key.



**Global Aurora:**

* Specific attributes can be encrypted client-side in an Aurora table using the AWS Encryption SDK.


* Combined with Aurora Global Tables, the client-side encrypted data is replicated to other regions.


* Using a multi-region key replicated in the same region as the Global Aurora DB allows clients to use low-latency, local KMS API calls to decrypt data client-side.


* This method protects specific fields, even from database admins, requiring an API key for decryption.



---

### 4. S3 Replication and AMI Sharing Encryption

#### S3 Replication Encryption Considerations

* Unencrypted objects and objects encrypted with SSE-S3 are replicated by default.


* Objects encrypted with SSE-C (a customer provided key) can be replicated.


* For objects encrypted with SSE-KMS, you must explicitly enable the option to specify which KMS Key to encrypt the objects with in the target bucket.


* You must adapt the KMS Key Policy for the target key.


* An IAM Role is required with `kms:Decrypt` permissions for the source KMS Key and `kms:Encrypt` for the target KMS Key.


* If you experience KMS throttling errors during this process, you can ask for a Service Quotas increase.


* While you can use multi-region AWS KMS Keys, they are currently treated as independent keys by Amazon S3, meaning the object will still be decrypted and then re-encrypted.



#### AMI Sharing Process Encrypted via KMS

To share an Amazon Machine Image (AMI) across accounts when it is encrypted:

1. The AMI in the Source Account is encrypted with a KMS Key from the Source Account.


2. You must modify the image attribute to add a Launch Permission which corresponds to the specified target AWS account.


3. You must share the KMS Keys used to encrypt the snapshot the AMI references with the target account or IAM Role.


4. When launching an EC2 instance from the shared AMI, the target account can optionally specify a new KMS key in its own account to re-encrypt the volumes.



---

### 5. Configuration and Secrets Management

#### AWS Systems Manager (SSM) Parameter Store

SSM Parameter Store provides secure storage for configuration and secrets.

* It offers optional seamless encryption using KMS.


* The service is serverless, scalable, durable, and provides an easy SDK.


* It supports version tracking of configurations and secrets.


* Security is managed through IAM.


* It integrates with Amazon EventBridge for notifications.


* It integrates natively with CloudFormation.



**SSM Parameter Store Hierarchy**
Parameters can be organized hierarchically, allowing fetching via `GetParameters` or `GetParametersByPath` API calls:

* `/my-department/my-app/dev/db-url`

* `/my-department/my-app/dev/db-password`

* `/my-department/my-app/prod/db-url`

* `/aws/reference/secretsmanager/secret_ID_in_Secrets_Manager`


**Standard vs. Advanced Parameter Tiers**

*Standard Tier:*

* **Total parameters allowed (per account/Region):** 10,000.


* **Maximum parameter size:** 4 KB.


* **Parameter policies available:** No.


* **Cost:** No additional charge.


* **Storage Pricing:** Free.



*Advanced Tier:*

* **Total parameters allowed (per account/Region):** 100,000.


* **Maximum parameter size:** 8 KB.


* **Parameter policies available:** Yes.


* **Cost:** Charges apply.


* **Storage Pricing:** **$0.05 per advanced parameter per month**.



**Parameter Policies (Advanced Tier Only)**

* Policies allow you to assign a TTL (expiration date) to a parameter to force updating or deleting sensitive data, such as passwords.


* You can assign multiple policies at a time, such as Expiration (to delete a parameter), Expiration Notification via EventBridge, and NoChange Notification via EventBridge.



#### AWS Secrets Manager

Secrets Manager is a newer service explicitly meant for storing secrets.

* It has the capability to force rotation of secrets every X days.


* It can automate the generation of secrets on rotation using AWS Lambda.


* It features direct integration with Amazon RDS (MySQL, PostgreSQL, Aurora).


* Secrets are encrypted using KMS.


* It is mostly meant for RDS integration.



**Multi-Region Secrets**

* You can replicate Secrets across multiple AWS Regions.


* Secrets Manager keeps read replicas in sync with the primary Secret.


* You have the ability to promote a read replica Secret to a standalone Secret.


* Use cases include multi-region applications, disaster recovery strategies, and multi-region databases.



---

### 6. AWS Certificate Manager (ACM)

AWS Certificate Manager (ACM) makes it easy to provision, manage, and deploy TLS Certificates.

* It provides in-flight encryption for websites using HTTPS.


* ACM supports both public and private TLS certificates.


* It is free of charge for public TLS certificates.


* It handles automatic TLS certificate renewal.


* **Integrations:** ACM loads TLS certificates directly onto Elastic Load Balancers (CLB, ALB, NLB), CloudFront Distributions, and APIs on API Gateway.


* **Restriction:** You cannot use ACM directly with EC2 because the certificates cannot be extracted.



#### Requesting vs. Importing Public Certificates

**Requesting Public Certificates via ACM**

1. List the domain names to be included in the certificate. This includes Fully Qualified Domain Names (FQDN) like `corp.example.com` or Wildcard Domains like `*.example.com`.


2. Select a Validation Method: DNS Validation or Email validation.


* DNS Validation is preferred for automation purposes and leverages a CNAME record added to DNS config (e.g., Route 53).


* Email validation sends emails to contact addresses in the WHOIS database.




3. Verification takes a few hours.


4. The Public Certificate will be enrolled for automatic renewal, which happens 60 days before expiry.



**Importing Public Certificates**

* You have the option to generate a certificate outside of ACM and then import it.


* There is no automatic renewal; you must manually import a new certificate before expiry.


* ACM sends daily expiration events to EventBridge starting 45 days prior to expiration (this number of days is configurable).


* AWS Config has a managed rule named `acm-certificate-expiration-check` to check for expiring certificates.



#### API Gateway and ACM Integrations

API Gateway supports different endpoint types, changing how ACM is integrated:

* **Edge-Optimized (Default):** Built for global clients, where requests are routed through CloudFront Edge locations to improve latency. The API Gateway itself still lives in only one region. The TLS Certificate must be provisioned in the same region as CloudFront (us-east-1). You then set up a CNAME or A-Alias record in Route 53.


* **Regional:** Designed for clients within the same region. The TLS Certificate must be imported on API Gateway in the exact same region as the API Stage. You then set up a CNAME or A-Alias record in Route 53.


* **Private:** Can only be accessed from your VPC using an interface VPC endpoint (ENI), controlled via resource policies.



---

### 7. AWS CloudHSM

While KMS relies on AWS to manage the software for encryption, CloudHSM is a service where AWS provisions dedicated encryption hardware for you.

* It utilizes Dedicated Hardware known as a Hardware Security Module (HSM).


* You manage your own encryption keys entirely, not AWS.


* The HSM device is tamper-resistant and meets FIPS 140-2 Level 3 compliance.


* It supports both symmetric and asymmetric encryption, including SSL/TLS keys.


* There is no free tier available for CloudHSM.


* You must use the designated CloudHSM Client Software.


* Amazon Redshift supports CloudHSM for database encryption and key management.


* It is a good option to use alongside SSE-C encryption.


* **High Availability:** CloudHSM clusters are spread across multiple Availability Zones, which is great for availability and durability.


* **AWS Service Integration:** CloudHSM can integrate with AWS services (like EBS, S3, RDS) by configuring a KMS Custom Key Store backed by CloudHSM.



#### AWS KMS vs. AWS CloudHSM

**AWS KMS:**

* **Tenancy:** Multi-Tenant.


* **Standard:** FIPS 140-2 Level 3.


* **Master Keys:** Can be AWS Owned CMK, AWS Managed CMK, or Customer Managed CMK.


* **Key Types:** Symmetric, Asymmetric, and Digital Signing.


* **Key Accessibility:** Accessible in multiple AWS regions, but you cannot access keys outside the region they were created in.


* **Cryptographic Acceleration:** None.


* **Access & Authentication:** Governed by AWS IAM.


* **High Availability:** AWS Managed Service.


* **Audit Capability:** Logged via CloudTrail and CloudWatch.


* **Free Tier:** Yes.



**AWS CloudHSM:**

* **Tenancy:** Single-Tenant.


* **Standard:** FIPS 140-2 Level 3.


* **Master Keys:** Exclusively Customer Managed CMK.


* **Key Types:** Symmetric, Asymmetric, Digital Signing & Hashing.


* **Key Accessibility:** Deployed and managed within a VPC, but can be shared across VPCs using VPC Peering.


* **Cryptographic Acceleration:** Features SSL/TLS Acceleration and Oracle TDE Acceleration.


* **Access & Authentication:** You manually create users and manage their permissions.


* **High Availability:** You must add multiple HSMs over different AZs to achieve HA.


* **Audit Capability:** Logged via CloudTrail, CloudWatch, and supports MFA.


* **Free Tier:** No.



---

### 8. Perimeter Protection: WAF, Shield, and Firewall Manager

#### AWS WAF (Web Application Firewall)

AWS WAF protects your web applications from common web exploits at Layer 7 (HTTP).

* **Deployment:** Can be deployed on Application Load Balancers, API Gateway, CloudFront, AppSync GraphQL API, and Cognito User Pools.


* **Web ACL Rules:** You define Web Access Control List (Web ACL) Rules.


* **IP Set:** Supports up to 10,000 IP addresses; use multiple rules for more IPs.


* **Payload Inspection:** Inspects HTTP headers, HTTP body, or URI strings to protect from common attacks like SQL injection and Cross-Site Scripting (XSS).


* **Constraints:** Includes size constraints and geo-matching to block specific countries.


* **Rate-based Rules:** Counts occurrences of events for DDoS protection.




* **Scope:** Web ACLs are Regional, except when deployed for CloudFront.


* **Rule Groups:** A rule group is a reusable set of rules that you can add to a web ACL.


* **Integration with Load Balancers:** WAF does not support Network Load Balancers (Layer 4). To get a fixed IP, you can use Global Accelerator in front of an ALB, and attach WAF to the ALB. The WebACL must be in the same AWS Region as the ALB.



#### AWS Shield

AWS Shield protects against Distributed Denial of Service (DDoS) attacks, which occur when many requests hit an application at the same time.

* **AWS Shield Standard:** A free service automatically activated for every AWS customer. It provides protection from SYN/UDP Floods, Reflection attacks, and other layer 3/layer 4 attacks.


* **AWS Shield Advanced:** An optional DDoS mitigation service costing **$3,000 per month per organization**.


* Protects against more sophisticated attacks on EC2, ELB, CloudFront, Global Accelerator, and Route 53.


* Provides 24/7 access to the AWS DDoS Response Team (DRP).


* Protects against higher fees incurred during usage spikes due to a DDoS attack.


* Its automatic application layer DDoS mitigation automatically creates, evaluates, and deploys AWS WAF rules to mitigate layer 7 attacks.





#### AWS Firewall Manager

Firewall Manager lets you manage rules across all accounts of an AWS Organization.

* It establishes a security policy, which is a common set of security rules.


* It can manage WAF rules across ALBs, API Gateways, and CloudFront.


* It handles AWS Shield Advanced configuration for ALBs, CLBs, NLBs, Elastic IPs, and CloudFront.


* It can manage Security Groups for EC2, ALBs, and ENI resources in a VPC.


* It supports AWS Network Firewall at the VPC level and Amazon Route 53 Resolver DNS Firewall.


* Policies are created at the region level.


* Crucially, rules are applied to new resources as they are created, which is excellent for ensuring compliance across all existing and future accounts in your Organization.



#### Best Practices for DDoS Resiliency

To build resilient architectures against DDoS attacks, employ a multi-layered defense strategy:

**1. Edge Location Mitigation:**

* **CloudFront:** Delivers web applications at the edge and protects from common DDoS attacks like SYN floods and UDP reflection.


* **Global Accelerator:** Accesses your application from the edge, integrates with Shield for DDoS protection, and is helpful if your backend isn't compatible with CloudFront.


* **Route 53:** Provides domain name resolution at the edge as a DDoS protection mechanism.



**2. Infrastructure Layer Defense:**

* Protect Amazon EC2 against high traffic by using Global Accelerator, Route 53, CloudFront, and Elastic Load Balancing.


* **Auto Scaling:** Helps scale in case of sudden traffic surges, including flash crowds or DDoS attacks.


* **Elastic Load Balancing:** Scales with traffic increases and distributes traffic to many EC2 instances.



**3. Application Layer Defense:**

* Use CloudFront to cache static content and serve it from edge locations, protecting the backend.


* AWS WAF sits on top of CloudFront or ALB to filter and block requests based on request signatures. WAF rate-based rules can automatically block the IPs of bad actors.


* Use managed WAF rules to block attacks based on IP reputation, anonymous IPs, or specific geographies.


* Shield Advanced automatically evaluates and deploys WAF rules to mitigate layer 7 attacks.



**4. Attack Surface Reduction:**

* **Obfuscating AWS resources:** Use CloudFront, API Gateway, and ELB to hide your backend resources like Lambda functions or EC2 instances.


* **Security Groups and Network ACLs:** Filter traffic based on specific IPs at the subnet or ENI-level.


* **Protecting API Endpoints:** Hide EC2/Lambda by using edge-optimized modes or CloudFront plus regional mode for more DDoS control. Combine WAF and API Gateway to enforce burst limits, header filtering, and API key requirements.



---

### 9. Intelligent Threat Detection and Assessment

#### Amazon GuardDuty

Amazon GuardDuty provides intelligent threat discovery to protect your AWS Account.

* It uses Machine Learning algorithms, anomaly detection, and 3rd party data.


* It requires just one click to enable, offers a 30-day trial, and requires no software installation.


* **Input Data Analyzed:**
* **CloudTrail Events Logs:** Detects unusual API calls and unauthorized deployments.


* **CloudTrail Management Events:** Tracks actions like creating VPC subnets or trails.


* **CloudTrail S3 Data Events:** Monitors actions like get object, list objects, and delete object.


* **VPC Flow Logs:** Detects unusual internal traffic or unusual IP addresses.


* **DNS Logs:** Flags compromised EC2 instances sending encoded data within DNS queries.




* **Optional Features:** Monitors EKS Audit Logs, RDS & Aurora, EBS, Lambda, and S3 Data Events.


* You can set up EventBridge rules to be notified of findings, targeting services like AWS Lambda or SNS.


* It includes a dedicated finding to protect against CryptoCurrency attacks.



#### Amazon Inspector

Amazon Inspector provides Automated Security Assessments specifically for EC2 instances, Container Images, and Lambda Functions.

* **For EC2 Instances:** Leverages the AWS System Manager (SSM) agent to analyze unintended network accessibility and check the running OS against known vulnerabilities.


* **For Container Images:** Assesses Container Images as they are pushed to Amazon ECR.


* **For Lambda Functions:** Identifies software vulnerabilities in function code and package dependencies as functions are deployed.


* It provides reporting, integrates directly with AWS Security Hub, and sends findings to Amazon EventBridge.


* It provides continuous scanning of the infrastructure, only when needed.


* It evaluates package vulnerabilities against a CVE database and checks network reachability for EC2.


* A risk score is associated with all vulnerabilities to aid prioritization.



#### AWS Macie

AWS Macie is a fully managed data security and data privacy service.

* It uses machine learning and pattern matching to discover and protect your sensitive data in AWS.


* Specifically, Macie helps identify and alert you to sensitive data, such as personally identifiable information (PII), residing in S3 Buckets.


* It integrates with Amazon EventBridge for notifications upon discovery.
