## Chapter: Amazon EC2 Networking, Placement Strategy, and Lifecycle Management

### Understanding IP Addressing in AWS

Networking fundamentals rely heavily on IP addressing, which generally comes in two formats: IPv4 and IPv6. While IPv6 is the newer standard designed to accommodate the vast number of devices in the Internet of Things (IoT) , IPv4 remains the most common format used online. For the scope of this AWS Associate level content, the focus is entirely on IPv4. The IPv4 format allows for approximately 3.7 billion distinct addresses in the public space. These addresses are structured as four sets of numbers ranging from 0 to 255, formatted as `[0-255].[0-255].[0-255].[0-255]`.

Within IPv4, there is a fundamental distinction between Public and Private IPs:

* **Public IP Addresses:** A machine with a public IP can be directly identified on the global internet (WWW). These addresses must be completely unique across the entire web, meaning no two machines can share the same public IP. Because of this global registry, public IPs can be easily geo-located.


* **Private IP Addresses:** A private IP means the machine can only be identified within its specific private network. While the IP must be unique within that specific private network , two completely different private networks (like two different companies) can safely use the same exact private IPs. To communicate with the outside internet, machines with private IPs must use a Network Address Translation (NAT) service alongside an internet gateway, acting as a proxy.



By default, an Amazon EC2 instance is launched with both a private IP for the internal AWS network and a public IP to access the world wide web. When you want to SSH into your EC2 instance from your local computer, you must use the public IP because you are not on the same internal AWS network.

**ASCII Illustration: Public vs Private Architecture**

```text
[The Internet (WWW)]
        |
        | (Public IP: 149.140.72.10)
+-----------------------+
|   Internet Gateway    | 
+-----------------------+
        | (NAT Translation)
        |
+-----------------------------------+
| Company A Private Network         |
| Range: 192.168.0.1/22             |
|                                   |
|   +-------------------+           |
|   | EC2 Instance      |           |
|   | Private IP:       |           |
|   | 192.168.0.31      |           |
|   +-------------------+           |
+-----------------------------------+

```

---

### Elastic IPs and IP Lifecycle Management

When you stop and subsequently restart an EC2 instance, its automatically assigned public IP can change. If your architectural requirements dictate that an instance must retain a fixed public IP, you must provision an Elastic IP.

An Elastic IP is a fixed public IPv4 address that you own until you explicitly choose to delete it. You can attach this IP to one specific EC2 instance at a time. One of the primary benefits of an Elastic IP is the ability to mask an instance or software failure by rapidly remapping the IP address to a healthy standby instance in your account.

However, AWS limits you to only 5 Elastic IPs per account by default, though you can request an increase. From an architectural standpoint, you should generally try to avoid using Elastic IPs. Relying on them often reflects poor architectural decisions. Instead of mapping fixed IPs, best practices recommend using a random public IP and registering a DNS name to it , or utilizing a Load Balancer so your underlying instances do not need public IPs at all.

---

### EC2 Placement Groups

When provisioning instances, you may require granular control over the physical placement strategy of your underlying hardware. AWS provides this control through Placement Groups, which offer three distinct strategies:

**1. Cluster Placement Group**
A Cluster strategy tightly groups instances together in a single Availability Zone (AZ).

* **Pros:** It provides a fantastic network configuration, offering 10 Gbps bandwidth between instances when Enhanced Networking is enabled, resulting in extremely low latency.


* **Cons:** Because they are clustered in one AZ, if that AZ experiences an outage, all instances fail simultaneously.


* **Use Cases:** Ideal for Big Data jobs that must complete quickly or any application requiring extremely low latency and high network throughput.



**2. Spread Placement Group**
The Spread strategy strictly separates your instances, ensuring they are placed on entirely different underlying physical hardware.

* **Pros:** It can span across multiple Availability Zones, vastly reducing the risk of simultaneous hardware failures.


* **Cons:** You are strictly limited to a maximum of 7 instances per Availability Zone per placement group.


* **Use Cases:** Perfect for applications designed to maximize high availability and critical applications where instances must be strictly isolated from one another's failures.



**3. Partition Placement Group**
The Partition strategy divides instances into distinct logical segments called partitions. Instances in one partition do not share hardware racks with instances in another partition.

* **Scale:** You can have up to 7 partitions per AZ , it can span multiple AZs in the same region , and it easily scales to hundreds of EC2 instances per group.


* **Resiliency:** A hardware failure in one partition will not affect the instances in other partitions. EC2 instances can also access their specific partition information via instance metadata.


* **Use Cases:** Specifically designed for large distributed systems like Hadoop (HDFS), HBase, Cassandra, and Kafka.



**ASCII Illustration: Placement Group Strategies**

```text
[CLUSTER] - Single AZ, High Speed
+-------------------------+
| AZ: us-east-1a          |
|  [EC2]--10Gbps--[EC2]   |
|    |              |     |
|  [EC2]--10Gbps--[EC2]   |
+-------------------------+

[SPREAD] - Max 7 per AZ, Isolated Hardware
+-----------------+  +-----------------+
| AZ: us-east-1a  |  | AZ: us-east-1b  |
| [Rack 1: EC2]   |  | [Rack 4: EC2]   |
| [Rack 2: EC2]   |  | [Rack 5: EC2]   |
| [Rack 3: EC2]   |  | [Rack 6: EC2]   |
+-----------------+  +-----------------+

[PARTITION] - Hundreds of instances, isolated by partition
+-------------------------------------------------+
| AZ: us-east-1a                                  |
| +-------------+ +-------------+ +-------------+ |
| | Partition 1 | | Partition 2 | | Partition 3 | |
| | [EC2] [EC2] | | [EC2] [EC2] | | [EC2] [EC2] | |
| | [EC2] [EC2] | | [EC2] [EC2] | | [EC2] [EC2] | |
| +-------------+ +-------------+ +-------------+ |
+-------------------------------------------------+

```

---

### Elastic Network Interfaces (ENI)

An Elastic Network Interface (ENI) is a logical component within a VPC that represents a virtual network card. ENIs are bound to a specific Availability Zone.

An ENI can contain several attributes:

* A primary private IPv4 address, and optionally one or more secondary private IPv4 addresses.


* One Elastic IP per private IPv4 address.


* One Public IPv4 address.


* One or more security groups attached to it.


* A unique MAC address.



The primary advantage of ENIs is flexibility; you can create an ENI independently and attach, detach, and move it on the fly between EC2 instances. This allows for robust network failover strategies.

**ASCII Illustration: ENI Failover**

```text
     [EC2 Instance A - FAILING]
      |
      |-- (Eth0) Primary ENI: 192.168.0.31
      |
      X-- (Eth1) Secondary ENI: 192.168.0.42  <-- DETACHING

              |
              | (Moving ENI on the fly)
              V

     [EC2 Instance B - STANDBY]
      |
      |-- (Eth0) Primary ENI: 192.168.0.55
      |
      |-- (Eth1) Secondary ENI: 192.168.0.42  <-- ATTACHING

```

---

### Instance State and EC2 Hibernate

Understanding how instance state impacts data and boot times is critical for the exam.

* **Stop:** When an instance is stopped, the data on the underlying Elastic Block Store (EBS) disk is kept completely intact for the next start.


* **Terminate:** When an instance is terminated, any EBS volumes (particularly the root volume) that are configured to be destroyed upon termination are permanently lost.



When you normally start an instance, the OS must boot up. On the very first start, the EC2 User Data script is also executed. Following starts skip the User Data, but the OS still must boot. After the OS boots, your specific applications start and your caches must warm up, which can be a time-consuming process.

To solve this delay, AWS introduced **EC2 Hibernate**. With hibernation, the in-memory state (RAM) of the machine is perfectly preserved. When the machine is started again, the boot process is significantly faster because the operating system does not need to completely restart. Under the hood, AWS achieves this by freezing the instance and writing the active RAM state to a file located in the root EBS volume.

Hibernate is excellent for long-running processes, workloads that require saving the RAM state, and services that take an extraordinarily long time to initialize.

**Rules and Limitations for EC2 Hibernate:**

* **Root Volume:** The root volume must be EBS, it must be encrypted, it cannot be an instance store, and it must be large enough to hold the RAM dump.


* **RAM Limits:** The instance RAM size must be less than 150 GB.


* **Families & Sizes:** It is supported on instance families like C3, C4, C5, I3, M3, M4, R3, R4, T2, T3, etc., but it is absolutely not supported for bare metal instances.


* **Pricing Models:** Hibernation is available for On-Demand, Reserved, and Spot Instances.


* **Time Limit:** An instance cannot be left in a hibernated state for more than 60 days.


* **Supported OS:** It supports Amazon Linux 2, Linux AMI, Ubuntu, RHEL, CentOS, and Windows.



**ASCII Illustration: The Hibernation Lifecycle**

```text
[ RUNNING ] 
  |-- Active RAM contains application state and warmed caches.
  V
[ HIBERNATING TRIGGERED ]
  |-- RAM data is aggressively written to a file on the...
  V
[ ROOT EBS VOLUME (Must be Encrypted) ]
  |
  V
[ STOPPED STATE ]
  |-- Instance is inactive. You only pay for EBS storage.
  |-- (Maximum 60 Days allowed)
  V
[ START TRIGGERED ]
  |-- RAM state is read directly from the EBS root volume.
  |-- OS boot is bypassed.
  V
[ RUNNING ]
  |-- Instance resumes exactly where it left off!

```
