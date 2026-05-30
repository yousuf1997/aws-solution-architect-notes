

#  AWS Compute Storage and Image Management

When working with EC2 instances in AWS, understanding how to manage storage and deploy custom environments is critical. This chapter covers everything you need to know about Elastic Block Store (EBS), EC2 Instance Store, Amazon Machine Images (AMIs), and the Elastic File System (EFS).

## 1. Amazon Elastic Block Store (EBS) Overview

An **EBS (Elastic Block Store) Volume** is essentially a network drive that you can attach to your EC2 instances while they are running.

### Core Characteristics of EBS

* **Persistence:** EBS allows your instances to persist data even after the EC2 instance is terminated.


* **Attachment Limits:** At the base level, an EBS volume can only be mounted to a single instance at a time. Think of it as a "network USB stick".


* **Availability Zone Locking:** EBS volumes are bound to a specific Availability Zone (AZ). For example, a volume created in `us-east-1a` cannot be directly attached to an instance in `us-east-1b`.


* **Network Dependency:** Because it is a network drive (not a physical drive directly inside the host hardware), it uses the network to communicate with the instance, which can introduce a small amount of latency.


* **Agility:** Volumes can be detached from one EC2 instance and quickly attached to another.


* **Provisioning & Billing:** You define a provisioned capacity (size in GBs and performance in IOPS) and are billed for all of the capacity you provision, regardless of how much you actively use. You can also increase the capacity of the drive over time.



### Delete on Termination Attribute

By default, AWS treats the root volume of an EC2 instance differently than additional attached volumes:

* 
**Root Volume:** The "Delete on Termination" attribute is enabled by default, meaning the root EBS volume is automatically deleted when the instance is terminated.


* 
**Attached Volumes:** Any additionally attached EBS volumes have this attribute disabled by default and will survive the instance termination.


* This behavior can be modified using the AWS console or the AWS CLI if, for example, you want to preserve the root volume after terminating the instance.



```text
+---------------------+
| Availability Zone A |
|                     |
|    [ EC2 Instance ] |
|           ^         |
|           | Attach  |
|           v         |
|    [ EBS Volume ]   |
|   (Network Drive)   |
+---------------------+

```

## 2. EBS Snapshots

To move a volume across Availability Zones or to simply create backups, you must use **EBS Snapshots**. A snapshot is a backup of your EBS volume taken at a specific point in time.

* **Best Practices:** While it is not strictly necessary to detach the volume to take a snapshot, it is highly recommended for data consistency.


* **Mobility:** Snapshots can be copied across different Availability Zones or even entirely different AWS Regions.



### Advanced Snapshot Features

* **EBS Snapshot Archive:** You can move snapshots to an "archive tier" which is 75% cheaper. However, restoring an archived snapshot takes between 24 to 72 hours.


* 
**Recycle Bin:** You can set up retention rules (from 1 day to 1 year) to retain deleted snapshots, allowing you to recover them if they were deleted accidentally.


* **Fast Snapshot Restore (FSR):** Normally, snapshots have a slight latency penalty upon first use as data is pulled from Amazon S3. FSR forces full initialization of the snapshot for zero latency on first use, but it is an expensive feature.



## 3. EC2 Instance Store

While EBS volumes offer good performance, they are limited by the network. If your application requires ultra-high-performance hardware disks, you should use the **EC2 Instance Store**.

* **Performance:** They provide vastly superior I/O performance compared to EBS. For example, large metal instances can provide up to 3.3 million read IOPS.


* **Ephemeral Nature:** Instance Store volumes lose all their data if the EC2 instance is stopped (they are ephemeral).


* **Use Cases:** Because of data volatility, they are best suited for buffers, caches, scratch data, and temporary content.


* **Responsibility:** If the underlying hardware fails, data is lost. Backups and data replication are entirely the user's responsibility.



## 4. Amazon Machine Images (AMI)

An **AMI (Amazon Machine Image)** is a complete customization of an EC2 instance. It allows you to pre-package your own software, configurations, operating systems, and monitoring tools.

* **Benefits:** Using an AMI results in faster boot and configuration times since the software is already installed and packaged.


* **Region Limits:** AMIs are built for a specific AWS region, though they can be copied across regions if needed.


* **Sources:** You can launch instances from AWS-provided Public AMIs, your own custom AMIs, or an AWS Marketplace AMI (made and potentially sold by third parties).



### The AMI Creation Process

1. Start an EC2 instance and customize it to your liking.


2. Stop the instance to ensure data integrity.


3. Build the AMI, a process that automatically creates EBS snapshots of the attached volumes.


4. Launch new instances from this custom AMI.



```text
[ Step 1: Customize ]     [ Step 2: Stop & Build ]      [ Step 3: Launch ]
   +-----------+               +-------------+            +-----------+
   | EC2 (AZ 1)|  --------->   | Custom AMI  | ---------> | EC2 (AZ 2)|
   +-----------+               | (Snapshots) |            +-----------+

```

## 5. EBS Volume Types Deep Dive

AWS offers 6 different types of EBS volumes, characterized by Size, Throughput, and IOPS (Input/Output Operations Per Second). **Crucially, only `gp2/gp3` and `io1/io2 Block Express` can be used as boot volumes**.

### General Purpose SSD (gp2 / gp3)

These balance price and performance for a wide variety of workloads, making them ideal for system boot volumes, virtual desktops, and dev/test environments. They range from 1 GiB to 16 TiB.

* **gp3:** Offers a baseline of 3,000 IOPS and 125 MiB/s throughput. You can independently scale IOPS up to 16,000 and throughput up to 1,000 MiB/s.


* **gp2:** The size of the volume and the IOPS are inherently linked. You get 3 IOPS per GiB, meaning the maximum IOPS of 16,000 is achieved at 5,334 GiB. Small volumes can temporarily burst to 3,000 IOPS.



### Provisioned IOPS SSD (io1 / io2 Block Express)

Designed for mission-critical, low-latency, or high-throughput workloads, such as heavily utilized databases requiring sustained IOPS performance.

* **io1 (4 GiB - 16 TiB):** Allows you to increase IOPS independently of storage size. The maximum is 64,000 IOPS for Nitro EC2 instances and 32,000 for others.


* **io2 Block Express (4 GiB - 64 TiB):** Delivers sub-millisecond latency. Supports a massive maximum of 256,000 IOPS with a 1,000:1 IOPS to GiB ratio. Supports EBS Multi-attach.



### Hard Disk Drives (HDD)

HDD volumes cannot be used as boot volumes and range from 125 GiB to 16 TiB.

* **Throughput Optimized HDD (st1):** Low cost HDD for frequently accessed, throughput-intensive workloads like Big Data, Data Warehouses, and Log Processing. Max throughput is 500 MiB/s with a max IOPS of 500.


* **Cold HDD (sc1):** The absolute lowest cost HDD. Used for data that is infrequently accessed where cost-savings are paramount. Max throughput is 250 MiB/s with a max IOPS of 250.



## 6. EBS Multi-Attach

Available only for the `io1` and `io2` families, EBS Multi-Attach allows you to attach the exact same EBS volume to multiple EC2 instances simultaneously, provided they are in the same Availability Zone.

* Each attached instance possesses full read and write permissions to the volume.


* **Limits:** You can attach up to 16 EC2 instances at a time.


* **Prerequisites:** Your application must manage concurrent write operations, and you must use a cluster-aware file system. Standard file systems like XFS or EXT4 will not work.


* **Use Case:** Highly available clustered Linux applications, like Teradata.



## 7. EBS Encryption

EBS Encryption leverages KMS (AES-256) keys to secure your data. It handles encryption and decryption entirely transparently with minimal impact on latency.

When an encrypted volume is created:

1. Data at rest inside the volume is encrypted.


2. Data in flight between the instance and the volume is encrypted.


3. All snapshots created from the volume, and volumes created from those snapshots, are automatically encrypted.



**How to encrypt an unencrypted volume:**
You cannot directly encrypt a running unencrypted volume. Instead:

1. Create a snapshot of the unencrypted volume.


2. Copy that snapshot, and select the option to encrypt the copy.


3. Create a new EBS volume from the encrypted snapshot.


4. Attach the new, encrypted volume to the instance.



## 8. Amazon Elastic File System (EFS)

**Amazon EFS** is a managed Network File System (NFS) that can be mounted simultaneously on hundreds of EC2 instances across multiple Availability Zones.

* **Characteristics:** It is highly available, scalable, but quite expensive (roughly 3 times the cost of gp2) and bills on a pay-per-use model. There is no capacity planning required because the file system scales automatically.


* **Compatibility:** EFS uses the NFSv4.1 protocol and is a POSIX file system with standard APIs. It is only compatible with Linux-based AMIs; it does not support Windows.


* **Security:** Access is controlled via standard Security Groups, and data at rest is encrypted using KMS.


* **Scale:** It can support thousands of concurrent NFS clients, throughput of 10+ GB/s, and grows to Petabyte-scale automatically.


* **Use Cases:** Content management, data sharing, web serving, and hosting WordPress sites across multiple instances.



### EFS Performance and Throughput Modes

* **Performance Modes (Set at creation):** * *General Purpose (Default):* Used for latency-sensitive workloads like web servers and CMS.


* *Max I/O:* Features higher latency but massive throughput, used for highly parallel workflows like big data or media processing.

* **Throughput Modes:**
* *Bursting:* Provides 50 MiB/s per 1 TB of storage, with burst capability up to 100 MiB/s.


* *Provisioned:* Allows you to set specific throughput regardless of the storage size (e.g., 1 GiB/s for a 1 TB drive).


* *Elastic:* Automatically scales throughput based on your workload, up to 3 GiB/s for reads and 1 GiB/s for writes. Ideal for unpredictable workloads.


### EFS Storage Classes

To optimize the high cost of EFS, it utilizes lifecycle policies to automatically move files between different storage tiers after a set number of days:

* **Standard:** For frequently accessed files.


* **Infrequent Access (EFS-IA):** Lower price to store data, but incurs a cost every time a file is retrieved.


* **Archive:** Extremely rarely accessed data (a few times a year), providing a 50% cost reduction.



**Durability Options:**

* *Standard:* Replicates data across multiple AZs; great for production.


* *One Zone:* Stores data in only one AZ. Yields over 90% in cost savings, has backup enabled by default, and is ideal for dev environments. It is also compatible with Infrequent Access (EFS One Zone-IA).



```text
EBS vs. EFS Architecture Model

       EBS Architecture                        EFS Architecture
+-----------------------------+       +-----------------------------+
|        AZ 1                 |       |      AZ 1          AZ 2     |
|   [EC2]          [EC2]      |       |     [EC2]         [EC2]     |
|     |              |        |       |       \             /       |
|  [EBS Vol]      [EBS Vol]   |       |        \           /        |
|                             |       |      [ EFS File System ]    |
+-----------------------------+       +-----------------------------+

```
