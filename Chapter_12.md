
# **Chapter: Content Delivery and Global Routing in AWS**

## **AWS CloudFront & Global Accelerator**

When deploying applications to a global audience, minimizing latency and optimizing the user experience are paramount. Sending user traffic across the public internet often involves multiple network hops, which can significantly degrade performance. Amazon Web Services (AWS) provides two powerful solutions to solve this: **Amazon CloudFront** and **AWS Global Accelerator**.

---

### **1. Amazon CloudFront: The Content Delivery Network (CDN)**

Amazon CloudFront is a globally distributed Content Delivery Network (CDN). Its primary function is to deliver data, videos, applications, and APIs to customers globally with low latency and high transfer speeds.

**Core Benefits:**

* **Performance:** It drastically improves read performance by caching content directly at the network's edge, closer to your users. This directly translates to an improved overall user experience.


* **Global Reach:** CloudFront operates across hundreds of Points of Presence globally, which are comprised of Edge Locations and Regional Edge Caches.


* **Security:** Because it is a worldwide, edge-level service, CloudFront provides inherent DDoS (Distributed Denial of Service) protection and integrates seamlessly with AWS Shield and the AWS Web Application Firewall (WAF).



#### **High-Level Architecture**

When a client requests a file (e.g., `GET /beach.jpg HTTP/1.1`), the request is routed to the nearest CloudFront Edge Location. The Edge Location checks its local cache. If the file is not present, CloudFront forwards the request to your backend Origin, retrieves the file, returns it to the client, and caches it for future requests.

```text
    +--------+       Request       +-----------------+     Cache Miss    +----------------+
    | Client | ------------------> |  Edge Location  | ----------------> | Backend Origin |
    +--------+                     |     (Cache)     |                   |  (S3 or HTTP)  |
                                   +-----------------+ <---------------- +----------------+
                                                         Fetch & Cache

```

#### **CloudFront Origins**

CloudFront must know where to fetch the original content from. You can configure multiple types of origins:

1. **S3 Buckets:** Used for distributing files and caching them at the edge. You can also use CloudFront to securely upload files back to S3. To keep the S3 bucket secure and prevent direct internet access, you utilize an **Origin Access Control (OAC)** combined with an S3 bucket policy.


2. **VPC Origins:** Used for applications hosted securely in VPC private subnets. This allows CloudFront to securely route traffic to internal Application Load Balancers (ALB), Network Load Balancers (NLB), or EC2 Instances.


3. **Custom Origins (HTTP):** CloudFront can front any public HTTP backend. This includes an S3 bucket configured as a static website (note: you must enable static website hosting on the bucket first) or a Public ALB.



---

### **2. CloudFront Architectures and Comparisons**

#### **S3 Origin Deep Dive**

When users across the globe (e.g., Los Angeles, São Paulo, Mumbai, Melbourne) request data, they access it via the public internet routing to their nearest Edge Location. From the Edge Location, the request travels securely over the private AWS network to the centralized S3 Origin.

#### **CloudFront vs. S3 Cross-Region Replication (CRR)**

Both services can improve global data access, but they serve different use cases:

| Feature | Amazon CloudFront | S3 Cross-Region Replication (CRR) |
| --- | --- | --- |
| **Network** | Global Edge network.

 | Dedicated S3 buckets in specific regions.

 |
| **Data Freshness** | Files are cached for a specified Time to Live (TTL), such as a day.

 | Files are updated in near real-time as they are written.

 |
| **Access Type** | Highly optimized for delivery. | Read-only in the replicated region.

 |
| **Best Use Case** | Great for **static content** that must be available everywhere globally.

 | Great for **dynamic content** needing low-latency availability in just a few specific regions.

 |

#### **ALB and EC2 Origins: Private vs. Public Networks**

When routing traffic to an Application Load Balancer or EC2 instances, you have two architectural choices:

* **Using VPC Origins (Private Network):** This is the most secure method. It allows you to deliver content from applications hosted entirely within private subnets. There is no need to expose your Load Balancers or EC2 instances to the internet; CloudFront connects to them securely using the VPC Origin feature.


* **Using Public Networks:** If you are not using VPC Origins, your edge locations must communicate over the public network.
* **EC2 Instances:** The instances must be public. Their Security Groups must be configured to explicitly allow the public IPs of CloudFront Edge Locations.


* **Application Load Balancers (ALB):** The ALB must be public, but the backend EC2 instances can remain private. The ALB's Security Group must allow CloudFront's public IPs, and the EC2 instances' Security Group must allow traffic from the ALB's Security Group.





---

### **3. Advanced CloudFront Features**

#### **Geo Restriction**

CloudFront allows you to restrict who can access your distribution. The country of origin is determined using a 3rd party Geo-IP database.

* **Allowlist:** Users can access content *only* if they are in an approved country.


* **Blocklist:** Users are prevented from accessing content if they are in a banned country.


* **Use Case:** This is heavily utilized to enforce Copyright Laws and control digital distribution rights.



#### **Cache Invalidations**

Because CloudFront caches content based on a Time to Live (TTL), updating a file on your backend origin does not immediately update the cache. CloudFront will not fetch the new content until the TTL expires.

To force an immediate refresh and bypass the TTL, you must perform a **CloudFront Invalidation**. You can invalidate all files using a wildcard (`*`) or target specific paths (e.g., `/images/*` or `/index.html`).

---

### **4. AWS Global Accelerator**

When deploying a globally accessed application directly via a Public ALB, user traffic travels over the public internet. This involves many network hops, leading to unpredictable and high latency. AWS Global Accelerator solves this by routing your users through the internal AWS network as quickly as possible to minimize latency.

#### **How Global Accelerator Works: Unicast vs. Anycast IP**

* **Unicast IP:** The traditional internet model where one server holds one unique IP address.


* **Anycast IP:** A routing topology where all servers hold the *same* IP address, and the client network is automatically routed to the nearest physical server.



Global Accelerator creates **two Anycast IPs** for your application. When a user sends a request, the Anycast IP routes traffic directly to the nearest AWS Edge Location. From there, the Edge Location sends the traffic across the highly optimized, private AWS global network directly to your application.

```text
                                  [AWS Private Global Network]
                                 /                            \
[User in America] ---> [Edge Location]                        [Public ALB / App] 
                                                              [Location: India ]
[User in Europe]  ---> [Edge Location]                        /
                                 \                           /
                                  [AWS Private Global Network]

```

#### **Key Features of Global Accelerator**

* **Compatibility:** Works seamlessly with Elastic IPs, EC2 instances, ALBs, and NLBs (both public and private).


* **Consistent Performance:** Provides intelligent routing to the lowest latency endpoints and ensures no issues with client-side caching because the Anycast IPs never change.


* **Disaster Recovery & Health Checks:** Global Accelerator continuously performs health checks on your applications. If an endpoint becomes unhealthy, it triggers a fast regional failover in less than 1 minute, making it an excellent tool for Disaster Recovery.


* **Security:** It significantly reduces your attack surface because only the 2 external Anycast IPs need to be whitelisted. Furthermore, it includes robust DDoS protection via AWS Shield.



---

### **5. Summary: Global Accelerator vs. CloudFront**

While both services utilize the AWS global network, Edge Locations, and AWS Shield for DDoS protection, they are engineered for different architectural needs.

* **Use CloudFront** when you need to improve performance for cacheable content (like images and videos) or deliver dynamic site content and APIs directly from the edge.


* **Use Global Accelerator** when you need to improve performance for a wide range of applications over TCP or UDP by proxying packets at the edge to your AWS Regions. It is the ideal fit for non-HTTP use cases like gaming (UDP), IoT (MQTT), or Voice over IP. It is also highly recommended for HTTP use cases that require static IP addresses or deterministic, fast regional failover.
