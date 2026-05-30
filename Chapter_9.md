## Chapter: Mastering Amazon S3

Amazon S3 is widely considered one of the foundational building blocks of the AWS ecosystem. It is advertised as an "infinitely scaling" storage solution, acting as the backbone for countless websites and integrating deeply with many other AWS services. This chapter provides a comprehensive, step-by-step approach to understanding Amazon S3, covering everything required for a thorough mastery of the subject.

---

### Amazon S3 Use Cases

Organizations leverage Amazon S3 for a wide variety of business needs.

* Backup and storage.


* Disaster Recovery.


* Archive.


* Hybrid Cloud storage.


* Application hosting.


* Media hosting.


* Data lakes & big data analytics.


* Software delivery.


* Static website hosting.



> **Industry Examples:** Nasdaq stores 7 years of data into S3 Glacier. Sysco runs analytics on its data stored in S3 to gain actionable business insights.
> 
> 

---

### Core Concepts: Buckets and Objects

Amazon S3 operates on a simple premise: it allows users to store files, known as "objects," inside directories, known as "buckets".

```text
    _______
   /       \
  |  [ ] ^  |  <-- Objects (files)
   \_______/
   S3 Bucket

```

**Bucket Rules and Naming Conventions**

Buckets are defined at the region level. While Amazon S3 appears as a global service in the management console, the buckets themselves are always created within a specific AWS region. Furthermore, buckets must have a globally unique name across all regions and all AWS accounts.

When creating a bucket, you must adhere to strict naming conventions:

* No uppercase letters.


* No underscores.


* The name must be 3 to 63 characters long.


* The name cannot be formatted as an IP address.


* The name must start with a lowercase letter or a number.


* The name must NOT start with the prefix "xn--".


* The name must NOT end with the suffix "-s3alias".



**Understanding S3 Objects**

Objects are the files stored within your S3 buckets. Every object has a "Key," which represents the full path to the file.

```text
s3://my-bucket/my_folder1/another_folder/my_file.txt
|_____________||____________________________________|
    Bucket                     Key

```

The Key is composed of a prefix and an object name. There is actually no true concept of "directories" within S3 buckets, even though the user interface tricks you into thinking otherwise. S3 simply uses keys with very long names that contain slashes ("/") to simulate a folder structure.

The object values represent the content of the file body.

* The maximum object size in S3 is 5TB (5000GB).


* If you are uploading an object larger than 5GB, you must use a "multi-part upload".


* Objects can have Metadata, which is a list of text key/value pairs that can be either system-defined or user-defined.


* Objects can have Tags, which are Unicode key/value pairs (up to 10 per object) useful for security and lifecycle management.


* Objects will have a Version ID if versioning is enabled on the bucket.



---

### Security and Access Management

Securing your data in Amazon S3 involves several layers of policies and access control lists.

**Access Control Methods**

* **User-Based Security:** This relies on IAM Policies to dictate which API calls are allowed for a specific user from AWS IAM.


* **Resource-Based Security:** This involves Bucket Policies, which set bucket-wide rules from the S3 console and can allow cross-account access.


* **Object Access Control List (ACL):** This provides finer grain control at the object level, though it can be disabled.


* **Bucket Access Control List (ACL):** This is less commonly used and can also be disabled.



An IAM principal can access an S3 object based on specific evaluation logic. Access is granted if the user's IAM permissions ALLOW it, OR if the resource policy ALLOWS it, AND there is no explicit DENY present. Additionally, objects in Amazon S3 can be encrypted using encryption keys.

**Bucket Policies and Public Access Block**

S3 Bucket Policies are JSON-based documents. They define Resources (buckets and objects), the Effect (Allow or Deny), Actions (the set of APIs to allow or deny), and the Principal (the account or user the policy applies to).

You can use S3 bucket policies to grant public access to the bucket, force objects to be encrypted upon upload, or grant access to another AWS account (Cross Account).

Amazon S3 also provides settings to "Block Public Access" at both the bucket and account level. These settings include blocking public access granted through new or any ACLs, and blocking public and cross-account access through new or any public bucket/access point policies. These features were created specifically to prevent company data leaks, and it is recommended to leave them on if your bucket should never be public.

---

### Static Website Hosting

Amazon S3 can host static websites and make them accessible on the Internet.

Depending on the region, the website URL will take one of two formats:

* `[http://bucket-name.s3-website-aws-region.amazonaws.com](http://bucket-name.s3-website-aws-region.amazonaws.com)`.


* `[http://bucket-name.s3-website.aws-region.amazonaws.com](http://bucket-name.s3-website.aws-region.amazonaws.com)`.



If you attempt to access an S3 static website and receive a 403 Forbidden error, you must make sure that the bucket policy allows public reads.

---

### Versioning and Replication

**S3 Versioning**

You can version your files in Amazon S3 to maintain multiple variants of an object in the same bucket.

```text
       [ File.txt ]  <-- User uploads
            |
            v
     _______________
    |               |
    |  [File.txt]v3 | <-- Current
    |  [File.txt]v2 | <-- Older
    |  [File.txt]v1 | <-- Oldest
    |_______________|
        S3 Bucket

```

* Versioning is enabled at the bucket level.


* Overwriting an object with the same key will increment the "version" (e.g., 1, 2, 3).


* It is a best practice to version your buckets to protect against unintended deletes and to enable easy rollbacks to previous versions.


* Any file that is not versioned prior to enabling versioning will be assigned the version ID "null".


* Suspending versioning on a bucket does not delete the previously stored versions.



**S3 Replication**

Replication allows you to automatically copy objects across buckets.

* You must enable Versioning in both the source and destination buckets.


* **Cross-Region Replication (CRR):** Used for compliance, lower latency access, and replication across accounts.


* **Same-Region Replication (SRR):** Used for log aggregation and live replication between production and test accounts.


* Buckets involved in replication can reside in different AWS accounts.


* The copying process is asynchronous.


* You must give proper IAM permissions to S3 to perform the replication.



There are several important technical notes regarding replication:

* After you enable Replication, only new objects are replicated.


* You can optionally replicate existing objects, or objects that previously failed replication, using S3 Batch Replication.


* For DELETE operations, you have the optional setting to replicate delete markers from the source to the target.


* Deletions performed with a specific version ID are not replicated to avoid malicious deletes.


* There is no "chaining" of replication. If bucket 1 replicates to bucket 2, and bucket 2 replicates to bucket 3, objects created in bucket 1 will not automatically replicate to bucket 3.



---

### Storage Classes, Durability, and Availability

Amazon S3 offers a variety of storage classes designed for different use cases. Understanding the difference between Durability and Availability is critical.

* **Durability:** Represents the probability that an object will remain intact and not be lost. Amazon S3 provides high durability of 99.999999999% (11 9's) across multiple Availability Zones (AZ) for all storage classes. Statistically, if you store 10,000,000 objects with S3, you can expect to lose a single object only once every 10,000 years.


* **Availability:** Measures how readily available a service is to be accessed. Availability varies depending on the chosen storage class. For example, S3 Standard has 99.99% availability, which equates to it not being available for about 53 minutes a year.



**Storage Class Categories**

* **S3 Standard - General Purpose:** Offers 99.99% Availability and is used for frequently accessed data. It features low latency, high throughput, and can sustain 2 concurrent facility failures. Use cases include Big Data analytics, mobile & gaming applications, and content distribution.


* **S3 Standard-Infrequent Access (IA):** Designed for data that is less frequently accessed but requires rapid access when needed, offering lower costs than S3 Standard with 99.9% Availability. Ideal for Disaster Recovery and backups.


* **S3 One Zone-Infrequent Access:** Provides high durability (11 9's) but in a single AZ. Data is lost if that specific AZ is destroyed. It has 99.5% Availability and is used for storing secondary backup copies of on-premises data or data you can recreate.


* **S3 Glacier Instant Retrieval:** Low-cost storage meant for archiving that offers millisecond retrieval. Great for data accessed once a quarter, requiring a minimum storage duration of 90 days.


* **S3 Glacier Flexible Retrieval (formerly S3 Glacier):** Offers retrieval times ranging from Expedited (1 to 5 minutes), Standard (3 to 5 hours), to Bulk (5 to 12 hours - free). Requires a minimum storage duration of 90 days.


* **S3 Glacier Deep Archive:** Designed for long term storage, offering Standard (12 hours) and Bulk (48 hours) retrieval times. Requires a minimum storage duration of 180 days. Glacier pricing is based on the price for storage plus the object retrieval cost.


* **S3 Intelligent-Tiering:** Automatically moves objects between Access Tiers based on usage for a small monthly monitoring and auto-tiering fee. There are no retrieval charges in this class. Automatic tiers include Frequent Access (default), Infrequent Access (objects not accessed for 30 days), and Archive Instant Access (objects not accessed for 90 days). Optional configurable tiers include Archive Access (90 to 700+ days) and Deep Archive Access (180 to 700+ days).



Users can move objects between these classes manually or automate the process using S3 Lifecycle configurations.

**S3 Storage Classes Attribute Comparison**

| Feature | Standard | Intelligent-Tiering | Standard-IA | One Zone-IA | Glacier Instant Retrieval | Glacier Flexible Retrieval | Glacier Deep Archive |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Durability** | 99.999999999% | 99.999999999% | 99.999999999% | 99.999999999% | 99.999999999% | 99.999999999% | 99.999999999% |
| **Availability** | 99.99% | 99.9% | 99.9% | 99.5% | 99.9% | 99.99% | 99.99% |
| **Availability SLA** | 99.9% | 99% | 99% | 99% | 99% | 99.9% | 99.9% |
| **Availability Zones** | >= 3 | >= 3 | >= 3 | 1 | >= 3 | >= 3 | >= 3 |
| **Min. Storage Duration** | None | None | 30 Days | 30 Days | 90 Days | 90 Days | 180 Days |
| **Min. Billable Object Size** | None | None | 128 KB | 128 KB | 128 KB | 40 KB | 40 KB |
| **Retrieval Fee** | None | None | Per GB retrieved | Per GB retrieved | Per GB retrieved | Per GB retrieved | Per GB retrieved |

**S3 Storage Classes Price Comparison (Example: us-east-1)**

| Feature | Standard | Intelligent-Tiering | Standard-IA | One Zone-IA | Glacier Instant Retrieval | Glacier Flexible Retrieval | Glacier Deep Archive |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Storage Cost (per GB/mo)** | $0.023 | $0.0025-$0.023 | $0.0125 | $0.01 | $0.004 | $0.0036 | $0.00099 |
| **Retrieval Cost (per 1000 requests)** | GET: $0.0004, POST: $0.005 | GET: $0.0004, POST: $0.005 | GET: $0.001, POST: $0.01 | GET: $0.001, POST: $0.01 | GET: $0.01, POST: $0.02 | GET: $0.0004, POST: $0.03 (Expedited: $10, Standard: $0.05, Bulk: free) | GET: $0.0004, POST: $0.05 (Standard: $0.10, Bulk: $0.025) |
| **Retrieval Time** | Instantaneous | Instantaneous | Instantaneous | Instantaneous | Instantaneous | Expedited (1-5 mins), Standard (3-5 hours), Bulk (5-12 hours) | Standard (12 hours), Bulk (48 hours) |
| **Monitoring Cost** | None | $0.0025 per 1000 objects | None | None | None | None | None |

---

### High Performance: S3 Express One Zone

For extreme workloads, Amazon offers S3 Express One Zone. This is a high-performance, single Availability Zone storage class.

* Objects in this class are stored in a Directory Bucket, which is a bucket confined to a single AZ.


* It can handle hundreds of thousands of requests per second.


* It provides single-digit millisecond latency.


* It boasts up to 10x better performance than S3 Standard while delivering 50% lower costs.


* It maintains High Durability (99.999999999%) and provides an Availability of 99.95%.


* It allows you to co-locate your storage and compute resources in the same AZ to significantly reduce latency.


* Use cases include latency-sensitive apps, data-intensive apps, AI & ML training, financial modeling, media processing, and High Performance Computing (HPC).


* It is best integrated with AWS services like SageMaker Model Training, Athena, EMR, and Glue.
