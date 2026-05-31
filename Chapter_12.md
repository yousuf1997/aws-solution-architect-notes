Here is a comprehensive, book-like chapter based on the provided material, designed to cover everything you need for your exam.

---

# Chapter: CloudFront & Global Accelerator

When deploying applications to a global audience, minimizing latency and delivering content securely and efficiently is paramount. This chapter explores two core AWS networking services designed to optimize global content delivery and application routing: Amazon CloudFront and AWS Global Accelerator.

## 1. Amazon CloudFront Basics

Amazon CloudFront is a Content Delivery Network (CDN). Its primary purpose is to improve read performance by ensuring content is cached at the edge of the AWS network, which significantly improves the end user's experience.

CloudFront achieves this global reach through hundreds of Points of Presence (also known as edge locations and regional edge caches) distributed worldwide. Because CloudFront operates globally, it inherently provides DDoS protection and integrates seamlessly with AWS Shield and AWS Web Application Firewall (WAF).

### High-Level Architecture

When a user requests a file, the request is routed to the nearest CloudFront Edge Location.

```text
+----------+      GET /image.jpg      +------------------------+
|          | -----------------------> |                        |
|  Client  |                          | CloudFront Edge Cache  |
|          | <----------------------- |                        |
+----------+      Cached Content      +------------------------+
                                         |                  ^
                                   Forward Request      Fetch Data
                                   (If Cache Miss)     (From Origin)
                                         v                  |
                                      +------------------------+
                                      |    Origin Server       |
                                      |   (S3 or HTTP / ALB)   |
                                      +------------------------+

```

## 2. CloudFront Origins

CloudFront cannot generate content on its own; it must pull data from an "Origin." CloudFront supports several types of origins:

* **S3 Bucket:** This is used for distributing files and caching them at the edge. It can also be used for uploading files to Amazon S3 directly through CloudFront. Access to the S3 bucket is securely restricted using Origin Access Control (OAC) along with an S3 bucket policy.


* **VPC Origin:** This allows you to deliver content from applications hosted in your VPC private subnets without exposing them to the Internet. It can deliver traffic to a private Application Load Balancer (ALB), a Network Load Balancer (NLB), or private EC2 instances.


* **Custom Origin (HTTP):** This can be an S3 website (the bucket must first be enabled as a static S3 website) or any public HTTP backend you desire, such as a Public ALB.



### Public Origins vs. VPC Origins Security

When routing to origins, the security posture changes based on the network configuration:

* **Using a Public Network:** If your origin is a Public ALB or public EC2 instances, the associated security groups must explicitly allow the public IP addresses of the CloudFront Edge Locations. EC2 instances can remain private only if they sit behind a Public Application Load Balancer.


* **Using VPC Origins:** Traffic is delivered directly to private subnets. The target (ALB, NLB, or EC2 instance) does not need to be exposed to the public internet.



## 3. Data Distribution Strategy: CloudFront vs. S3 Cross-Region Replication

If you need to distribute data globally, you might wonder whether to use CloudFront or S3 Cross-Region Replication (CRR). Here is how they compare:

| Feature | Amazon CloudFront | S3 Cross-Region Replication |
| --- | --- | --- |
| **Network** | Global Edge network.

 | Must be explicitly set up for each region you want replication to happen.

 |
| **Data Freshness** | Files are cached for a Time To Live (TTL), such as a day.

 | Files are updated in near real-time.

 |
| **Access Type** | Caches data for fast reads.

 | Read-only in the replicated regions.

 |
| **Best Use Case** | Great for static content that must be available everywhere.

 | Great for dynamic content that needs to be available at low latency in a few specific regions.

 |

## 4. Cache Management and Access Control

### Cache Invalidations

Because CloudFront caches content at the edge, if you update a file on your backend origin, CloudFront will not automatically know about the change. It will continue serving the old file until the TTL expires.

To fix this, you can force an entire or partial cache refresh, effectively bypassing the TTL, by performing a **CloudFront Invalidation**. You can invalidate all files using a wildcard (`*`) or target a specific path, like `/images/*`.

### Geo Restriction

CloudFront allows you to restrict who can access your distribution based on geography:

* **Allowlist:** Allows users to access your content only if they reside in an approved list of countries.


* **Blocklist:** Prevents users from accessing your content if they reside in a banned list of countries.


* **Mechanism:** The user's country is determined using a 3rd-party Geo-IP database.


* **Use Case:** This is commonly used to enforce Copyright Laws and control access to region-locked content.



---

## 5. Network Routing Concepts

To understand AWS Global Accelerator, we must first understand the problem it solves.

When you deploy a global application, users connecting over the public internet suffer from high latency because their traffic must traverse many individual routing "hops" across different internet service providers. To minimize latency, the goal is to route user traffic onto the high-speed AWS internal network as fast as possible.

This relies on understanding two routing mechanisms:

* **Unicast IP:** One server holds one specific IP address.


* **Anycast IP:** All servers hold the *same* IP address, and the client is automatically routed to the nearest server on the network.



```text
Anycast IP Illustration:

                 +--> [Edge Location US] ---> [App Server]
                 |     (IP: 12.34.56.78)
[US Client] -----+
(Target: 12.34.56.78)

[EU Client] -----+
(Target: 12.34.56.78)
                 |
                 +--> [Edge Location EU] ---> [App Server]
                       (IP: 12.34.56.78)

```

## 6. AWS Global Accelerator

AWS Global Accelerator leverages the AWS internal network to route users to your application efficiently.

When you set up Global Accelerator, 2 Anycast IPs are created for your application. These Anycast IPs send user traffic directly to the nearest AWS Edge Location. From that edge location, the traffic is routed over the private AWS global network directly to your application.

### Key Features of Global Accelerator

* **Target Compatibility:** It works with Elastic IPs, EC2 instances, Application Load Balancers (ALB), and Network Load Balancers (NLB), whether they are public or private.


* **Consistent Performance:** It provides intelligent routing to the lowest latency path and enables fast regional failover. There are no issues with client-side caching because the Anycast IP never changes.


* **Health Checks:** Global Accelerator performs continuous health checks of your applications. If an application becomes unhealthy, failover happens in less than 1 minute. This makes it an excellent choice for making applications global and handling disaster recovery.


* **Security:** Only 2 external Anycast IPs need to be whitelisted by your users, and the service provides built-in DDoS protection via AWS Shield.



## 7. Service Comparison: CloudFront vs. Global Accelerator

While both services use the AWS global network, its edge locations, and integrate with AWS Shield for DDoS protection, they serve different primary purposes.

**Use CloudFront when:**

* You want to improve performance for cacheable static content, like images and videos.


* You need dynamic content delivery and API acceleration, where content is served directly at the edge.



**Use Global Accelerator when:**

* You need to improve performance for a wide range of applications over TCP or UDP protocols.


* You need to proxy packets at the edge directly to applications running in one or more AWS Regions.


* You have non-HTTP use cases, such as gaming (UDP), IoT (MQTT), or Voice over IP.


* You have HTTP use cases that strictly require static IP addresses.


* You have HTTP use cases that require deterministic, incredibly fast regional failover.
