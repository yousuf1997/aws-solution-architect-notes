
# The Comprehensive Guide to Amazon EC2

## Chapter 1: Introduction to Amazon EC2 Basics

Amazon Elastic Compute Cloud (Amazon EC2) is one of the most popular and foundational offerings within AWS. At its core, EC2 represents Infrastructure as a Service (IaaS), providing secure, resizable compute capacity in the cloud.

Understanding EC2 requires familiarizing yourself with its primary capabilities:

* **EC2:** Renting virtual machines for compute power.

* **EBS (Elastic Block Store):** Storing data on virtual drives attached to these instances.

* **ELB (Elastic Load Balancing):** Distributing incoming traffic load across multiple machines.

* **ASG (Auto Scaling Groups):** Automatically scaling your services by adding or removing instances based on demand.



### Sizing & Configuration Options

When launching an EC2 instance, you must configure several parameters to suit your workload:

* **Operating System (OS):** You can choose between Linux, Windows, or Mac OS.

* **Compute Power & Memory:** You decide how many virtual CPUs (cores) and how much Random-Access Memory (RAM) the instance needs.

* **Storage:** You can attach network-based storage (EBS & EFS) or utilize hardware-based, temporary storage (EC2 Instance Store).

* **Networking:** You configure the network card speed and determine if the instance requires a Public IP address.


* **Firewall Rules:** Security groups define the allowed inbound and outbound traffic.



### EC2 User Data (Bootstrapping)

EC2 User Data is a script that runs automatically when a machine starts up for the very first time. This process is known as "bootstrapping," which allows you to automate boot tasks using root user privileges. Common bootstrapping tasks include installing software or updates, downloading common files from the internet, and executing any initial configuration commands.

---

## EC2 Instance Types

AWS provides a variety of EC2 instance types, each optimized for different use cases. AWS uses a specific naming convention to help you identify an instance's capabilities. For example, in the instance type `m5.2xlarge`:

* **m:** Represents the instance class.


* **5:** Indicates the generation of the instance, as AWS improves hardware over time.


* **2xlarge:** Specifies the size of the instance within its class.


### Instance Categories

* **General Purpose:** Ideal for workloads like web servers or code repositories, providing a balanced ratio of compute, memory, and networking resources. Instances in this family include the T2 (e.g., `t2.micro`), T3, M4, and M5 series.

* **Compute Optimized:** Designed for compute-bound applications that require high-performance processors. Typical use cases include batch processing workloads, media transcoding, High-Performance Computing (HPC), dedicated gaming servers, and scientific modeling/machine learning. (Examples: C4, C5, C6g) .

* **Memory Optimized:** Built for fast performance in workloads that process massive datasets in memory. Use cases include high-performance relational/non-relational databases, distributed web-scale cache stores, and real-time processing of big unstructured data. (Examples: R4, R5, X1) .

* **Storage Optimized:** Engineered for storage-intensive tasks requiring high, sequential read and write access to massive datasets on local storage (delivering tens of thousands of low-latency IOPS). Excellent for High Frequency Online Transaction Processing (OLTP), NoSQL databases, data warehousing, and distributed file systems. (Examples: I3, D2, H1) .

---

## Security Groups and Network Security

Security Groups act as the fundamental network "firewall" for your EC2 instances. They regulate access to ports, authorized IP ranges (IPv4 and IPv6), and control both inbound (traffic entering the instance) and outbound (traffic leaving the instance) networking.

### Important Rules and Behaviors

* Security groups contain **allow rules only**; you cannot create "deny" rules.

* By default, **all inbound traffic is blocked**, and **all outbound traffic is authorized**.

* They live "outside" the EC2 instance; if a security group blocks traffic, the EC2 instance will never even see it.

* They can be attached to multiple instances, but they are locked to a specific Region / VPC combination.

* **Troubleshooting Tip:** If your application is inaccessible and "times out," it is a security group issue. If the application gives a "connection refused" error, the traffic made it past the firewall, but the application itself has an error or isn't running.

### Referencing Other Security Groups

You do not have to rely solely on IP addresses; security group rules can also reference other security groups. This is highly useful for tiered architectures (e.g., allowing your database instance to only accept traffic from instances attached to your web-tier security group).

```text
+-------------------------------------------------------------+
|                      ASCII Illustration                     |
|                   Security Group Filtering                  |
+-------------------------------------------------------------+
 
 [Your Computer] (IP: XX.XX.XX.XX)
       |
       | (Requests Port 22 - SSH)
       v
 +-----------------------------------+
 | SECURITY GROUP (Inbound Rules)    |
 | [ALLOW] Port 22 from XX.XX.XX.XX  | ---> (Traffic Allowed)
 | [DENY]  All other inbound traffic |      |
 +-----------------------------------+      |
       ^                                    v
       | (Requests Port 22)           +--------------+
 [Other Unknown Computer] - - - - - X | EC2 INSTANCE |
      (Traffic Blocked / Timeout)     +--------------+
                                            |
                                            | (Initiates Outbound Request)
                                            v
 +-----------------------------------+
 | SECURITY GROUP (Outbound Rules)   |
 | [ALLOW] Any IP, Any Port (Default)| ---> (Traffic Allowed to WWW)
 +-----------------------------------+

```

### Classic Ports to Know for the Exam

You must memorize these common ports:

| Port | Protocol / Service | Use Case |
| --- | --- | --- |
| **22** | SSH (Secure Shell) | Log into a Linux instance.

 |
| **22** | SFTP (Secure File Transfer) | Upload files using SSH.

 |
| **21** | FTP (File Transfer Protocol) | Upload files into a file share.

 |
| **80** | HTTP | Access unsecured websites.

 |
| **443** | HTTPS | Access secured websites.

 |
| **3389** | RDP (Remote Desktop) | Log into a Windows instance.

 |

---

## Connecting to your EC2 Instance

There are multiple ways to securely connect to the command line of your EC2 instance.

* **Linux / Mac OS X:** Use the native SSH command line utility.


* **Windows (< 10):** Use a free tool called Putty.


* **Windows (>= 10):** Modern Windows includes native SSH support.


* **EC2 Instance Connect:** A browser-based solution that connects to your instance without needing to manage key files locally. AWS temporarily uploads a key to the instance to establish the connection. However, this only works out-of-the-box with Amazon Linux 2, and Port 22 must still be opened in the security group for it to function.



---

##  EC2 Purchasing Options

AWS provides diverse purchasing models to suit different technical requirements and budgets.

### 1. On-Demand Instances

You pay for what you use by the second (for Linux/Windows instances after the first minute) or by the hour (for other OSs). This model carries the highest cost but requires zero upfront payment and no long-term commitment.

* **Use case:** Short-term, unpredictable, and un-interrupted workloads.

* **Analogy:** Booking a hotel resort whenever you like, staying as long as you want, and paying the full rack rate.


### 2. Reserved Instances (RIs)

You commit to specific instance attributes (Instance Type, Region, Tenancy, and OS) for 1 or 3 years. You can receive up to a 72% discount compared to On-Demand. Payment options include No Upfront, Partial Upfront, or All Upfront (which provides the largest discount). RIs can be scoped regionally or to a specific AZ. You can also buy and sell them on the Reserved Instance Marketplace.

* **Convertible Reserved Instances:** Allow you to change the instance type, family, OS, scope, and tenancy later, offering up to a 66% discount.


* **Use case:** Steady-state usage applications like databases.

* **Analogy:** Planning ahead to stay at the resort for a long time to secure a bulk discount.



### 3. Savings Plans

Similar to RIs (1 or 3-year commitments, up to 72% discounts), but instead of committing to an instance *type*, you commit to a specific dollar amount of usage (e.g., $10/hour). Any usage beyond the plan is billed at On-Demand rates. Savings Plans are locked to a specific instance family and region (e.g., M5 in us-east-1), but are extremely flexible across instance size (m5.large vs m5.xlarge), OS, and tenancy.

* **Analogy:** Paying a flat rate to stay at the resort for a certain period, allowing you to freely switch between a Suite, Sea View, or King room.



### 4. Dedicated Hosts and Instances

* **Dedicated Hosts:** You book an entire physical server fully dedicated to your use. This is the most expensive option. You get visibility into sockets and cores.


* **Use case:** Addressing strict compliance requirements or utilizing existing server-bound software licenses (Bring Your Own License - BYOL).

* **Dedicated Instances:** Instances run on hardware dedicated to you, but you may share the hardware with your other instances in the same AWS account. Unlike Dedicated Hosts, you have no control over instance placement, and hardware placement can shift if you stop and start the instance.



### 5. Capacity Reservations

You reserve On-Demand capacity in a specific Availability Zone for any duration. There is no billing discount, and you are charged the On-Demand rate whether you are actively running instances in the reservation or not. However, you can combine this with Regional RIs or Savings Plans to gain discounts.

* **Use case:** Short-term, uninterrupted workloads that absolutely must run in a specific AZ.



---

##  Deep Dive into Spot Instances and Spot Fleets

Spot Instances provide the most cost-efficient compute in AWS, offering up to a 90% discount compared to On-Demand.

### How Spot Pricing Works

The hourly spot price fluctuates based on supply and demand capacity. You define a "maximum spot price" you are willing to pay. As long as the current market spot price remains below your max price, you keep the instance. If the market price exceeds your max price, your instance will be interrupted (stopped or terminated) after a 2-minute grace period.

* **Spot Block:** If you need uninterrupted time, you can "block" a spot instance for a specified timeframe (1 to 6 hours). In rare situations, the instance may still be reclaimed, but it is generally immune to price interruptions during the block.


* **Use cases:** Workloads highly resilient to failure, such as batch jobs, data analysis, image processing, distributed workloads, or workloads with flexible start/end times. They are *not* suitable for critical jobs or databases.


* **Analogy:** Bidding on empty hotel rooms. The highest bidder gets the room, but you can get kicked out if someone bids higher.



### Managing Spot Requests

To launch a Spot Instance, you create a **Spot Request**, specifying your max price, desired instances, launch specifications, and the request type (one-time or persistent).

* **Important Lifecycle Rule:** Canceling a Spot Request does *not* automatically terminate the running instances. If you want to shut everything down completely, you must **first cancel the Spot Request** (so it doesn't spin up a new replacement), and **then terminate the associated instances**. You can only cancel requests that are open, active, or disabled.



```text
+-------------------------------------------------------------+
|                      ASCII Illustration                     |
|           Proper Spot Instance Termination Flow             |
+-------------------------------------------------------------+

 [ ACTIVE SPOT REQUEST ] -----> (Maintains) -----> [ RUNNING EC2 INSTANCES ]
         |                                                 |
  STEP 1 | (Cancel Request)                         STEP 2 | (Terminate)
         v                                                 v
 [ CANCELLED REQUEST ] - - - - -(No new launches)- - [ TERMINATED INSTANCES ]

* If you only do Step 2, the active request will immediately 
  launch new instances to replace the terminated ones.

```

### Spot Fleets

A Spot Fleet is a configured set of Spot Instances, with the option to include On-Demand instances, designed to automatically meet your target capacity while adhering to specific price constraints.

You define multiple "launch pools" (combinations of instance types like `m5.large`, operating systems, and AZs) so the fleet has flexible choices to fulfill capacity. The fleet automatically stops launching instances once capacity or maximum budget limits are reached.

**Spot Fleet Allocation Strategies:**

* **lowestPrice:** Selects instances from the pool with the lowest price. Great for short workloads and strict cost optimization.


* **diversified:** Distributes instances across all defined pools. Excellent for high availability and long workloads.


* **capacityOptimized:** Selects the pool with the optimal overall capacity for the number of instances requested.


* **priceCapacityOptimized:** First looks for pools with the highest available capacity, then selects the one with the lowest price among them. This is the **recommended best choice** for most workloads.
