## Chapter 1: Mastering Amazon Virtual Private Cloud (VPC)

### Introduction to CIDR and IP Addressing

To understand networking in AWS, you must first master Classless Inter-Domain Routing (CIDR). CIDR is a method for allocating IP addresses and is heavily used in AWS networking and Security Group rules. A CIDR consists of two components: a Base IP and a Subnet Mask.

* The Base IP represents an IP contained in the range, such as 10.0.0.0 or 192.168.0.0.


* The Subnet Mask defines how many bits can change in the IP address.


* Subnet masks can take different forms, such as `/8` which corresponds to 255.0.0.0.


* A `/16` mask corresponds to 255.255.0.0, and a `/24` mask corresponds to 255.255.255.0.


* A `/32` mask means no octet can change, allowing for exactly 1 IP address.


* A `/24` mask means the last octet can change, allowing for 256 IP addresses.


* A `/0` mask means all octets can change, representing all IPs everywhere.



The Internet Assigned Numbers Authority (IANA) established specific blocks of IPv4 addresses solely for private Local Area Networks (LANs). Private IP addresses only allow certain values.

* Big networks often use the 10.0.0.0/8 range.


* AWS default VPCs operate in the 172.16.0.0/12 range.


* Home networks typically use the 192.168.0.0/16 range.


* Any IP address outside of these private blocks is considered a public IP on the internet.



---

### Core VPC Components and Subnets

A Virtual Private Cloud (VPC) lets you define a private network in the cloud. You can have multiple VPCs within an AWS region, with a soft limit of up to 5 per region.

* All new AWS accounts come with a Default VPC.


* New EC2 instances are launched into this Default VPC if no subnet is specified.


* The Default VPC has internet connectivity, and all EC2 instances inside it receive public IPv4 addresses and DNS names.


* The maximum number of CIDRs per VPC is 5.


* The minimum allowable size for a VPC CIDR is `/28` (16 IP addresses).


* The maximum allowable size for a VPC CIDR is `/16` (65,536 IP addresses).


* Because VPCs are private, only the authorized private IPv4 ranges are allowed.


* Your VPC CIDR should never overlap with your other networks, such as your corporate data center.



AWS reserves exactly 5 IP addresses in every single subnet. These reserved IPs are the first 4 and the last 1 in the range, and they cannot be assigned to an EC2 instance. For example, in a `10.0.0.0/24` subnet, AWS reserves the Network Address, the VPC router, the Amazon-provided DNS, a slot for future use, and the Network Broadcast Address (since AWS does not support broadcast in a VPC).

#### ASCII Illustration: Basic VPC Architecture

```text
[Internet]
    |
    v
[Internet Gateway]
    |
+---VPC (e.g., 10.0.0.0/16)--------------------------+
|                                                    |
|  +--Public Subnet-------------------------------+  |
|  | [EC2 Instance + Public IP]                   |  |
|  +----------------------------------------------+  |
|                                                    |
|  +--Private Subnet------------------------------+  |
|  | [EC2 Instance + Private IP]                  |  |
|  +----------------------------------------------+  |
+----------------------------------------------------+

```

---

### Internet Gateways and Bastion Hosts

An Internet Gateway (IGW) allows resources within a VPC to connect to the Internet.

* An IGW scales horizontally and is highly available and redundant.


* It must be created separately from a VPC.


* One VPC can only be attached to one IGW, and vice versa.


* Creating an IGW does not automatically provide internet access; route tables must also be edited to point to it.



A Bastion Host is a specialized EC2 instance used to securely SSH into private EC2 instances.

* The Bastion Host is placed in a public subnet.


* The Bastion Host's security group must allow inbound internet traffic on port 22 from a restricted CIDR, like a corporate network.


* The private EC2 instances must configure their security groups to allow SSH traffic specifically from the Bastion Host's security group or private IP.



---

### Network Address Translation: NAT Instances vs. NAT Gateways

When EC2 instances in a private subnet need to reach the internet (e.g., for software updates), they require a Network Address Translation (NAT) mechanism.

**NAT Instances**
NAT Instances are an older, outdated solution, though they still appear on exams.

* A NAT Instance must be launched in a public subnet.


* You must manually disable the "Source / Destination Check" setting on the EC2 instance.


* It must have an Elastic IP attached to it.


* Route Tables must be configured to route traffic from private subnets to the NAT Instance.


* You must manage the software, OS patches, and security groups yourself.


* Internet traffic bandwidth is limited by the specific EC2 instance type chosen.


* It is not highly available out of the box; you must script failover mechanisms.



**NAT Gateways**
NAT Gateways are the modern, AWS-managed alternative.

* A NAT Gateway is managed entirely by AWS, offering higher bandwidth and no administration.


* It provides 5 Gbps of bandwidth, automatically scaling up to 100 Gbps.


* It is billed per hour for usage and the amount of data transferred.


* It is created in a specific Availability Zone and uses an Elastic IP.


* A NAT Gateway cannot be used by an EC2 instance in the exact same subnet.


* It requires an Internet Gateway to function.


* There are no security groups to manage.


* For fault-tolerance across multiple Availability Zones, you must create multiple NAT Gateways (one per AZ) because there is no cross-AZ failover.



#### ASCII Illustration: NAT Gateway Traffic Flow

```text
[Private Subnet EC2] ---> [NAT Gateway (in Public Subnet)] ---> [Internet Gateway] ---> [Internet]

```

---

### Network Security: NACLs and Security Groups

AWS provides two primary firewalls to control traffic: Network Access Control Lists (NACLs) and Security Groups.

**Network Access Control Lists (NACLs)**
NACLs act as a firewall that controls traffic strictly from and to subnets.

* There is one NACL per subnet, and new subnets automatically receive the Default NACL.


* The Default NACL accepts all inbound and outbound traffic.


* Newly created custom NACLs will deny everything by default.


* NACL rules have a number ranging from 1 to 32766.


* Rules are evaluated in order, meaning lower numbers have higher precedence, and the first match determines the outcome.


* The final rule is an asterisk (*) which denies a request if no preceding rules matched.


* AWS recommends adding rules in increments of 100.


* NACLs are stateless, meaning return traffic must be explicitly allowed by rules.


* They support both "allow" and "deny" rules.


* NACLs are excellent for blocking specific IP addresses at the subnet level.



Because NACLs are stateless, they must account for **Ephemeral Ports**. When two endpoints establish a connection, the client uses a defined port (like 443) and expects a response on a random ephemeral port. Different operating systems use different ephemeral port ranges, such as 49152-65535 for Windows 10 or 32768-60999 for Linux. NACL outbound rules must explicitly allow these port ranges for the return traffic.

**Security Groups**
Security Groups operate independently of NACLs and protect at the instance level.

* Security groups are applied to an EC2 instance when specified by a user.


* They support "allow" rules only.


* They are stateful, meaning return traffic is automatically allowed regardless of outbound rules.


* All rules are evaluated simultaneously before a decision is made to allow traffic.



---

### VPC Connectivity: Peering and Endpoints

**VPC Peering**
VPC Peering privately connects two VPCs using the AWS backbone network, making them behave as if they were on the same network.

* Peered VPCs must not have overlapping CIDR blocks.


* VPC Peering is completely non-transitive.


* If VPC A is peered with VPC B, and VPC B is peered with VPC C, VPC A still cannot talk to VPC C unless a direct peering connection is made between A and C.


* You must manually update route tables in each VPC's subnets to enable communication.


* Peering connections can be created across different AWS accounts and different AWS regions.


* You can reference a security group from a peered VPC if both VPCs are in the same region.



**VPC Endpoints (AWS PrivateLink)**
Every AWS service (like S3 or DynamoDB) is normally exposed via a public URL. VPC Endpoints, powered by AWS PrivateLink, allow you to connect to these AWS services using a private network rather than routing traffic over the public internet.

* VPC Endpoints scale horizontally and are completely redundant.


* They eliminate the need for Internet Gateways or NAT Gateways when accessing AWS services.


* If you experience connectivity issues, you should check your VPC DNS Resolution settings and your Route Tables.



There are two distinct types of VPC Endpoints:

* **Interface Endpoints:** These provision an Elastic Network Interface (ENI) with a private IP address, require a Security Group, support most AWS services, and incur hourly costs plus data processing fees.


* **Gateway Endpoints:** These provision a gateway, must be used as a target in a route table, do not use security groups, are totally free, and strictly support Amazon S3 and Amazon DynamoDB.


* For Amazon S3, Gateway Endpoints are highly preferred due to being free, unless access is required from on-premises over VPN or Direct Connect, in which case an Interface Endpoint is required.



---

### Advanced Connectivity: VPNs, Direct Connect, and Transit Gateways

**AWS Site-to-Site VPN**
A Site-to-Site VPN connects your corporate data center securely to AWS over the public internet.

* It requires a Virtual Private Gateway (VGW) attached to your AWS VPC.


* You can customize the Autonomous System Number (ASN) on the VGW.


* It requires a Customer Gateway (CGW), which is a software application or physical device on the customer's side.


* The Customer Gateway uses a public internet-routable IP address.


* You must enable Route Propagation for the Virtual Private Gateway in your route tables.


* If you need to ping your instances, ICMP must be allowed in the security groups.



**AWS VPN CloudHub**
AWS VPN CloudHub provides secure communication between multiple on-premises sites if you have multiple VPN connections.

* It operates as a low-cost hub-and-spoke model for network connectivity.


* Traffic routes over the public internet.


* It is configured by connecting multiple VPNs to the exact same Virtual Private Gateway (VGW) and setting up dynamic routing.



**AWS Direct Connect (DX)**
Direct Connect provides a dedicated, private connection from a remote network directly to your VPC, entirely bypassing the public internet.

* A physical connection must be established between your data center and an AWS Direct Connect Location.


* You must set up a Virtual Private Gateway on your VPC.


* It allows access to both public resources (like S3) and private resources (like EC2) on the same connection.


* It increases bandwidth throughput, lowers costs for large data sets, and provides a more consistent network experience.


* It supports both IPv4 and IPv6.


* Direct Connect Gateway allows a single Direct Connect to route to multiple VPCs across different AWS regions.


* **Dedicated Connections:** Provide 1 Gbps, 10 Gbps, and 100 Gbps capacity via a physical ethernet port dedicated to the customer.


* **Hosted Connections:** Provide 50 Mbps up to 10 Gbps, established via AWS Direct Connect Partners with on-demand capacity.


* Data in transit over Direct Connect is private, but it is not automatically encrypted.


* You can combine AWS Direct Connect with a VPN to create an IPsec-encrypted private connection.


* To achieve maximum resiliency for critical workloads, you should use separate connections terminating on separate devices in more than one location.


* A cheaper backup alternative to a secondary Direct Connect is a Site-to-Site VPN connection.



**Transit Gateway**
Network topologies can become incredibly complicated when managing VPC peering, VPNs, and Direct Connects. Transit Gateway resolves this.

* Transit Gateway enables transitive peering between thousands of VPCs and on-premises environments using a hub-and-spoke model.


* It is a regional resource but can peer with other Transit Gateways across different regions.


* It can be shared across multiple AWS accounts using AWS Resource Access Manager (RAM).


* It supports Equal-cost multi-path routing (ECMP), allowing you to route packets over multiple best paths to increase VPN bandwidth.


* It is the only AWS service that supports IP Multicast.



#### ASCII Illustration: Transit Gateway Topology

```text
[VPC 1] \
         \
[VPC 2] ---- [ AWS Transit Gateway ] ---- [ Direct Connect Gateway ]
         /           |
[VPC 3] /            |
               [ Site-to-Site VPN ]

```

---

### Monitoring, IPv6, and Network Costs

**VPC Flow Logs**
VPC Flow Logs capture information about IP traffic going in and out of your interfaces.

* They can be applied at the VPC level, Subnet level, or specific Elastic Network Interface (ENI) level.


* The data can be sent to Amazon S3, CloudWatch Logs, and Kinesis Data Firehose.


* They capture network information from AWS managed interfaces as well, such as ELB, RDS, ElastiCache, NAT Gateway, and Transit Gateway.


* The log format includes the `action` field, which displays either ACCEPT or REJECT.


* If an inbound request shows REJECT, it is due to either a NACL or a Security Group.


* If an inbound request shows ACCEPT but the outbound shows REJECT, the issue is explicitly tied to a stateless NACL.


* You can query flow logs using Amazon Athena on S3 or CloudWatch Logs Insights.


* To publish logs to CloudWatch, the IAM Service Role must have permissions like `logs:CreateLogGroup` and `logs:PutLogEvents`.



**VPC Traffic Mirroring**
Traffic Mirroring allows you to capture and inspect actual network traffic in your VPC.

* Traffic is sourced from an ENI and routed to target security appliances or a Network Load Balancer.


* You can capture all packets, filter them, or truncate them.


* It is utilized for deep content inspection, threat monitoring, and troubleshooting.



**IPv6 in AWS**
Because IPv4 addresses are being exhausted, IPv6 was created to provide $3.4 \times 10^{38}$ unique IP addresses.

* Every single IPv6 address in AWS is public and Internet-routable; there are no private IPv6 ranges in AWS.


* IPv4 cannot be disabled for your VPC or your subnets.


* If you cannot launch an EC2 instance, it is because there are no available IPv4 addresses left in the subnet, and the solution is to create a new IPv4 CIDR.


* You can enable IPv6 to operate in dual-stack mode alongside IPv4.


* **Egress-only Internet Gateways** are used exclusively for IPv6.


* An Egress-only Internet Gateway allows outbound IPv6 connections while strictly preventing the internet from initiating an IPv6 connection back to your instances.



**Networking Costs and Protection**
Network costs heavily depend on routing.

* Ingress traffic (inbound to AWS) is typically free.


* Egress traffic (outbound from AWS) carries high costs.


* To minimize costs, you should keep as much traffic within AWS as possible.


* Traffic within the exact same Availability Zone using a private IP is entirely free.


* Using public IPs or Elastic IPs within an AZ costs money.


* Data transfer out of S3 to the internet incurs fees, but data transfer out to CloudFront is free.


* S3 Transfer Acceleration adds an additional cost on top of normal data transfer pricing.


* To protect this network infrastructure globally, you can utilize AWS Network Firewall.


* AWS Network Firewall protects your entire VPC from Layer 3 to Layer 7 and is powered internally by the AWS Gateway Load Balancer.


* It supports thousands of rules, stateful domain list filtering, regex pattern matching, and active flow intrusion prevention.
