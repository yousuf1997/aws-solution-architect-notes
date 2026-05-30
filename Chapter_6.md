#  High Availability & Scalability in AWS

Understanding how to design systems that handle varying loads and survive outages is the cornerstone of AWS cloud architecture. This chapter dives deep into the core concepts of scalability and high availability, focusing heavily on Elastic Load Balancing (ELB) and Auto Scaling Groups (ASG).

---

## 1. The Foundations: Scalability vs. High Availability

While often discussed together, scalability and high availability are distinct but linked concepts.

### Scalability

Scalability means that an application or system can handle greater loads by adapting. There are two primary kinds of scalability:

* **Vertical Scalability:** This means increasing the size of your instances. For example, scaling an application vertically means stopping it on a t2.micro and running it on a larger t2.large instance. This is very common for non-distributed systems, such as databases. Services like Amazon RDS and ElastiCache can scale vertically. However, there is usually a limit to how much you can scale vertically due to hardware constraints.


* **Horizontal Scalability:** Also known as elasticity, this means increasing the number of instances or systems for your application. Horizontal scaling implies the use of distributed systems and is very common for modern web applications. Cloud offerings like Amazon EC2 make it easy to horizontally scale.



```text
       VERTICAL SCALING                   HORIZONTAL SCALING
       (Scaling Up/Down)                  (Scaling Out/In)

          +-----------+                    +---+   +---+   +---+
          |           |                    |   |   |   |   |   |
+---+     | t2.large  |          +---+     +---+   +---+   +---+
|   | --> |           |          |   | --> 
+---+     |           |          +---+     t2.micro instances
t2.micro  +-----------+          

```

### High Availability

High availability usually goes hand in hand with horizontal scaling.

* High availability means running your application or system in at least two data centers, which in AWS translates to Availability Zones.


* The primary goal is to survive a data center loss.


* High availability can be passive, such as an RDS Multi-AZ standby instance.


* High availability can also be active, functioning alongside horizontal scaling to distribute active traffic across multiple zones.



---

## 2. Introduction to Load Balancing

Load balancers are servers that forward traffic to multiple downstream servers, such as EC2 instances.

### Why Use a Load Balancer?

* Spread the load across multiple downstream instances.


* Expose a single point of access, or DNS, to your application.


* Seamlessly handle the failures of downstream instances.


* Perform regular health checks on your instances.


* Provide SSL termination (HTTPS) for your websites.


* Enforce session stickiness using cookies.


* Achieve high availability across different zones.


* Separate public traffic from private traffic.



An **Elastic Load Balancer (ELB)** is a managed load balancer provided by AWS. AWS guarantees that it will be working and handles upgrades, maintenance, and high availability. While setting up your own load balancer costs less, it requires a lot more effort on your end, whereas ELB integrates seamlessly with AWS services like EC2, ASG, ECS, ACM, Route 53, and WAF.

### Health Checks and Security Groups

Health checks are crucial because they enable the load balancer to know if the downstream instances are available to reply to requests. The health check is performed on a specific port and route, commonly `/health`. If the response is not a 200 OK, the instance is marked as unhealthy.

For secure traffic flow, you must configure Security Groups correctly:

* The **Load Balancer Security Group** should allow inbound HTTP/HTTPS traffic from anywhere (0.0.0.0/0).


* The **EC2 Application Security Group** should only allow inbound traffic sourced from the Load Balancer's Security Group, ensuring users cannot bypass the ELB to access instances directly.



---

## 3. The 4 Types of AWS Load Balancers

AWS offers four generations/types of managed load balancers, though it is generally recommended to use the newer generation ones for more features.

### Classic Load Balancer (CLB)

* Introduced in 2009 as the v1 old generation.


* Supports TCP (Layer 4) and HTTP/HTTPS (Layer 7) routing.


* Health checks are TCP or HTTP based.


* Provides a fixed hostname in the format `XXX.region.elb.amazonaws.com`.



### Application Load Balancer (ALB)

* Introduced in 2016 as a v2 new generation balancer.


* Operates solely at Layer 7 to load balance HTTP and HTTPS applications across machines or containers.


* Supports HTTP/2, WebSockets, and redirects (e.g., HTTP to HTTPS).


* Features advanced routing capabilities to different target groups based on URL path, hostname, query strings, and headers.


* A great fit for microservices and container-based applications (like Docker and Amazon ECS), featuring port mapping to redirect to dynamic ports.


* **Target Groups:** Can route to EC2 instances, ECS tasks, Lambda functions, or private IP addresses. Health checks are performed at the target group level.


* **Client IP Visibility:** The application servers do not see the client's direct IP. The true client IP is inserted into the `X-Forwarded-For` header, along with port and protocol in `X-Forwarded-Port` and `X-Forwarded-Proto`.



### Network Load Balancer (NLB)

* Introduced in 2017 as a v2 new generation balancer.


* Operates at Layer 4, forwarding raw TCP, TLS, and UDP traffic.


* Built for extreme performance, capable of handling millions of requests per second with ultra-low latency.


* Provides one static IP per Availability Zone and supports assigning Elastic IPs, which is helpful for whitelisting specific IPs.


* **Target Groups:** Can route to EC2 instances, private IP addresses, or even an Application Load Balancer.



### Gateway Load Balancer (GWLB)

* Introduced in 2020.


* Operates at Layer 3 (Network Layer) for IP Protocol packets.


* Used to deploy, scale, and manage fleets of 3rd party network virtual appliances in AWS, such as Firewalls, Intrusion Detection/Prevention Systems, and Deep Packet Inspection Systems.


* Combines a Transparent Network Gateway (single entry/exit for traffic) with a Load Balancer that distributes traffic to virtual appliances.


* Uses the GENEVE protocol on port 6081.


* **Target Groups:** EC2 instances or private IP addresses.



---

## 4. Advanced Load Balancer Features

### Sticky Sessions (Session Affinity)

Stickiness ensures that the same client is always redirected to the same instance behind a load balancer, which is critical to make sure the user doesn't lose session data. This works for CLB, ALB, and NLB. However, enabling stickiness may bring an imbalance to the load across backend EC2 instances.

Cookies control this stickiness and fall into two categories:

* **Application-based Cookies:** Can be custom cookies generated by the target (with any custom attributes required) or an application cookie generated by the load balancer named `AWSALBAPP`. Do not use reserved names like AWSALB, AWSALBAPP, or AWSALBTG for custom cookies.


* **Duration-based Cookies:** Generated by the load balancer with an expiration date you control. The name is `AWSALB` for an ALB and `AWSELB` for a CLB.



### Cross-Zone Load Balancing

With Cross-Zone Load Balancing, the load balancer distributes traffic evenly across all registered instances in *all* Availability Zones. Without it, requests are distributed only to instances within the node's specific AZ, which can cause imbalances.

| Load Balancer | Default Status | Inter-AZ Data Charges |
| --- | --- | --- |
| **Application Load Balancer** | Enabled by default | No charges for inter-AZ data 

 |
| **Network & Gateway Load Balancer** | Disabled by default | You pay charges for inter-AZ data if enabled 

 |
| **Classic Load Balancer** | Disabled by default | No charges for inter-AZ data if enabled 

 |

### SSL/TLS and Server Name Indication (SNI)

An SSL/TLS certificate allows traffic between clients and the load balancer to be encrypted in transit. Public certificates are issued by Certificate Authorities (CA) and have an expiration date that must be renewed. The load balancer uses an X.509 certificate, which you can manage using AWS Certificate Manager (ACM) or upload yourself.

**Server Name Indication (SNI)** solves the problem of loading multiple SSL certificates onto one web server to serve multiple websites. The client indicates the target hostname in the initial SSL handshake, allowing the server to find the correct certificate.

* ALB and NLB support multiple listeners with multiple SSL certificates using SNI.


* CLB supports only one SSL certificate, requiring multiple CLBs for multiple hostnames.



### Connection Draining / Deregistration Delay

This feature gives time to complete "in-flight requests" while an instance is de-registering or becomes unhealthy. It stops sending new requests to the instance while waiting for existing connections to complete.

* It is named **Connection Draining** for CLB and **Deregistration Delay** for ALB & NLB.


* The delay can be set between 1 and 3600 seconds, with a default of 300 seconds.


* It can be disabled by setting the value to 0, or set to a low value if your application requests are short.



---

## 5. Auto Scaling Groups (ASG)

In real-life, the load on your websites changes, and in the cloud, you can create and get rid of servers very quickly.

### ASG Goals & Capacity

The goal of an ASG is to scale out to match increased load, scale in to match decreased load, and ensure a minimum and maximum number of EC2 instances are running. ASGs also automatically register new instances to a load balancer and re-create instances if a previous one is terminated. ASGs themselves are free; you only pay for the underlying EC2 instances.

```text
    MINIMUM         DESIRED          MAXIMUM
   CAPACITY         CAPACITY         CAPACITY
    (e.g., 2)        (e.g., 4)        (e.g., 6)

   +-------+        +-------+        + - - - +
   | EC2   |        | EC2   |        | EC2   |
   +-------+        +-------+        + - - - +
   +-------+        +-------+        + - - - +
   | EC2   |        | EC2   |        | EC2   |
   +-------+        +-------+        + - - - +

```

### ASG Attributes

To configure an ASG, you define a **Launch Template** (note: older Launch Configurations are deprecated). The template includes:

* AMI and Instance Type
* EC2 User Data
* EBS Volumes
* Security Groups and SSH Key Pairs
* IAM Roles for your EC2 instances

You also specify Network and Subnet information, Load Balancer integrations, Min/Max/Initial capacities, and Scaling Policies.

### Auto Scaling with CloudWatch

It is possible to scale an ASG based on CloudWatch alarms, which monitor metrics computed for the overall ASG instances. Good metrics to scale on include:

* `CPUUtilization`: Average CPU utilization across instances.
* `RequestCountPerTarget`: Ensures the number of requests per EC2 instance is stable.
* Average Network In / Out (for network-bound applications).
* Any custom metric pushed to CloudWatch.

### Scaling Policies

Based on these alarms, you create policies:

* **Target Tracking Scaling:** Dynamic and simple to set up (e.g., "Keep average ASG CPU around 40%").


* **Step/Simple Scaling:** Dynamic scaling where triggers add or remove specific units (e.g., Add 2 units if CPU > 70%, remove 1 unit if CPU < 30%).


* **Scheduled Scaling:** Anticipate scaling based on known usage patterns (e.g., increase minimum capacity to 10 at 5 pm on Fridays).


* **Predictive Scaling:** Continuously analyzes historical load to forecast load and schedule scaling actions ahead of time.



### Scaling Cooldowns

After a scaling activity happens, the ASG enters a cooldown period, which defaults to 300 seconds. During this period, the ASG will not launch or terminate additional instances, allowing metrics time to stabilize.

> **Pro Tip:** Use a ready-to-use AMI to reduce your instance configuration time. This allows instances to serve requests faster and helps you reduce the necessary cooldown period.
> 
>
