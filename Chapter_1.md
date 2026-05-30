

# Chapter 1: Getting Started with Amazon Web Services (AWS)

## 1.1 The Evolution and Dominance of AWS

Amazon Web Services (AWS) is the undisputed pioneer in cloud computing, but its origins are rooted in Amazon's own necessity. In 2003, Amazon realized that its robust internal infrastructure was actually one of its core strengths, sparking the idea to bring it to market.

The timeline of AWS's early growth reveals a rapid evolution:

**2002:** AWS was initially launched internally.

**2004:** The platform launched publicly, featuring the Simple Queue Service (SQS).

**2006:** AWS officially re-launched to the public, offering SQS alongside Amazon S3 (storage) and Amazon EC2 (compute).

**2007:** AWS expanded its footprint by launching in Europe.



Today, AWS dominates the strategic cloud platform services market. According to the Gartner Magic Quadrant (as of October 2023), AWS is positioned prominently in the "Leaders" quadrant, sitting ahead of competitors like Microsoft and Google.

**AWS By the Numbers:**
To understand the scale of AWS, consider its market performance:

**Revenue:** In 2023, AWS generated $90 billion in annual revenue.

**Market Share:** As of Q1 2024, AWS held 31% of the cloud market, with Microsoft trailing in second place at 25%.


**Leadership:** AWS has maintained its position as the Pioneer and Leader of the Cloud Market for 13 consecutive years.


**User Base:** The platform supports over 1,000,000 active users.



---

## 1.2 Why AWS? Use Cases and Global Adoption

AWS enables organizations to build sophisticated, highly scalable applications. Because of its immense flexibility, it is applicable to a wildly diverse set of industries.

Primary use cases include:

* Enterprise IT, Backup & Storage, and Big Data analytics.


* Website hosting, as well as Mobile & Social Applications.


* Gaming.



This versatility has attracted some of the world's most recognizable brands. Major entities such as Netflix, NASA, Airbnb, Dropbox, 21st Century Fox, and Activision all rely on AWS infrastructure to power their operations.

---

## 1.3 Demystifying AWS Global Infrastructure

For the exam, understanding the physical and logical layout of AWS's global network is critical. The AWS Global Infrastructure is broken down into Regions, Availability Zones, Data Centers, and Edge Locations (Points of Presence).

### AWS Regions

An AWS Region is a geographical area that contains a cluster of data centers. Regions are given specific naming conventions, such as `us-east-1` (N. Virginia) or `eu-west-3` (Paris). Importantly, most AWS services are **region-scoped**, meaning the resources you deploy are tied to the specific region you select.

**How to Choose a Region:**
When launching a new application, you must strategically select a region based on four key factors:

1. **Compliance:** Data governance and legal requirements often dictate that data must not leave a specific country or region without explicit permission.
2. **Proximity to Customers:** Choosing a region close to your user base significantly reduces network latency.
3. **Available Services:** Not every service or new feature is immediately available in every single region.
4. **Pricing:** The cost of running services varies from region to region.

### Availability Zones (AZs)

Inside every Region are Availability Zones. A region will typically have 3 AZs, with a minimum of 3 and a maximum of 6.

Each AZ consists of one or more discrete data centers equipped with redundant power, networking, and connectivity. These AZs are geographically separated from one another within the region so they remain isolated from local disasters. Despite this physical separation, all AZs within a region are connected via high-bandwidth, ultra-low latency networking.

**Textual Illustration: Region and AZ Architecture**

```text
[ AWS Region: Sydney (ap-southeast-2) ]
   |
   |-- [ AZ 1: ap-southeast-2a ] ---> (1 or more discrete Data Centers)
   |       ^
   |       | (Ultra-low latency, high-bandwidth connection)
   |       v
   |-- [ AZ 2: ap-southeast-2b ] ---> (1 or more discrete Data Centers)
   |       ^
   |       | (Ultra-low latency, high-bandwidth connection)
   |       v
   |-- [ AZ 3: ap-southeast-2c ] ---> (1 or more discrete Data Centers)

```

Note: AZ names are always the Region name followed by a letter.

### Points of Presence (Edge Locations)

To deliver content to end-users with the lowest possible latency, AWS uses Points of Presence. Amazon operates over 400 Points of Presence globally, which includes more than 400 Edge Locations and over 10 Regional Edge Caches. These are distributed across 90+ cities in 40+ countries.

---

## 1.4 Navigating the AWS Ecosystem: Global vs. Regional Services

When interacting with the AWS Console, it is vital to know which services operate on a global scale and which are restricted to specific regions.

**Global Services** (These operate worldwide, not tied to a single region):

**IAM (Identity and Access Management):** For managing user access and credentials.
 
**Route 53:** AWS's highly available DNS service.

**CloudFront:** The Content Delivery Network (CDN) that utilizes the global Edge Locations.

**WAF:** The Web Application Firewall.

**Region-Scoped Services** (These must be deployed within a specific Region):

**Amazon EC2:** Infrastructure as a Service (IaaS) for virtual servers.

**Elastic Beanstalk:** Platform as a Service (PaaS) for deploying applications.

**Lambda:** Function as a Service (FaaS) for serverless computing.
 
**Rekognition:** Software as a Service (SaaS) for image and video analysis.
