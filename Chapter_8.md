
# Amazon Route 53 and the Domain Name System

## 1. The Backbone of the Internet: What is DNS?

The Domain Name System (DNS) is often referred to as the backbone of the Internet. Its primary function is to act as a directory, translating human-friendly hostnames (like `www.google.com`) into the machine IP addresses (like `172.217.18.36`) that computers use to communicate.

DNS relies on a hierarchical naming structure. To understand this, let's break down a Fully Qualified Domain Name (FQDN):

```text
       URL Breakdown
       ========================================================
       http://api.www.example.com.
       |____| |__________________| |_|
         |             |            |
      Protocol        FQDN         Root 

       FQDN Breakdown
       ========================================================
       api  .  www.example . com
       |_|     |_________|   |_|
        |           |         |
     Sub Domain    SLD       TLD 

```

* **Root:** The hidden dot at the very end of a domain.


* **TLD (Top Level Domain):** Extensions like `.com`, `.us`, `.gov`, or `.org`.


* **SLD (Second Level Domain):** The primary domain name, such as `amazon.com` or `google.com`.


* **Sub Domain:** Prefixes added to the SLD, such as `api.` or `www.`.



### Key DNS Terminology

* **Domain Registrar:** A company that allows you to purchase and register domain names (e.g., Amazon Route 53, GoDaddy).


* **DNS Records:** The actual mapping instructions, such as A, AAAA, CNAME, or NS records.


* **Zone File:** The file that contains all your DNS records.


* **Name Server:** The server responsible for resolving DNS queries, which can be Authoritative or Non-Authoritative.



---

## 2. How DNS Works

When a user tries to access a website, a multi-step resolution process occurs to find the correct IP address:

```text
[Web Browser] ---> Wants to visit "example.com"
      |
      v
[Local DNS Server] (Managed by ISP or Company)
      |  (If not in cache, queries the Root Server)
      v
[Root DNS Server] (Managed by ICANN)
      |  (Directs to the .com TLD Server)
      v
[TLD DNS Server] (Managed by IANA)
      |  (Directs to the Name Server for example.com)
      v
[SLD DNS Server] (Managed by Domain Registrar)
      |  (Returns the IP Address: 9.10.11.12)
      v
[Web Browser] ---> Connects to [Web Server] at 9.10.11.12

```

Note: The Local DNS Server and your Web Browser utilize a Cache governed by a Time To Live (TTL) to speed up future requests.

---

## 3. Introduction to Amazon Route 53

Amazon Route 53 is a highly available, scalable, and fully managed Authoritative DNS service. Being "Authoritative" means you, the customer, have the power to update the DNS records yourself.

**Key Features:**

* It acts as both a DNS service and a Domain Registrar.


* It has the ability to check the health of your resources.


* It is the **only** AWS service that provides a 100% availability SLA.


* *Fun Fact:* The "53" in its name is a reference to port 53, the traditional port used for DNS traffic.



---

## 4. Route 53 Records and Types

A DNS record dictates exactly how you want to route traffic for a specific domain. Each record contains the domain/subdomain name, the record type, the mapped value (e.g., an IP address), a routing policy, and a TTL.

For the exam, you must intimately understand these four record types:

* **A Record:** Maps a hostname directly to an IPv4 address.


* **AAAA Record:** Maps a hostname directly to an IPv6 address.


* **CNAME:** Maps a hostname to another hostname. The target domain must ultimately have an A or AAAA record. *Crucial limitation:* You cannot create a CNAME record for the top node of a DNS namespace, also known as the Zone Apex (e.g., you can't use it for `example.com`, but you can for `www.example.com`).


* **NS:** Designates the Name Servers for your Hosted Zone, controlling how traffic is routed for a domain.



*(Advanced record types include CAA, DS, MX, NAPTR, PTR, SOA, TXT, SPF, and SRV)*.

---

## 5. Hosted Zones

A Hosted Zone is simply a container for records that define how to route traffic to a domain and its subdomains. You pay $0.50 per month per hosted zone.

There are two types:

1. **Public Hosted Zones:** Contain records that specify how to route traffic publicly on the Internet (e.g., routing to a public EC2 instance, Application Load Balancer, or CloudFront distribution).


2. **Private Hosted Zones:** Contain records that specify how you route traffic within one or more Virtual Private Clouds (VPCs). This is used for internal applications (e.g., an internal web app routing to a private Amazon RDS instance).



---

## 6. Time To Live (TTL)

TTL determines how long a DNS record is cached at DNS Resolvers and client browsers.

* **High TTL (e.g., 24 hours):** Results in less traffic to Route 53 (saving money), but records might be outdated if you make a change, as clients will rely on their cache.


* **Low TTL (e.g., 60 seconds):** Makes it easy to change records quickly with minimal downtime, but results in more traffic and queries hitting Route 53 (which costs more money).


* *Note:* TTL is mandatory for every DNS record type **except** Alias records.



---

## 7. The Crucial Exam Topic: CNAME vs. Alias Records

AWS resources (like Load Balancers or CloudFront distributions) expose AWS-managed hostnames (e.g., `lb1-1234.us-east-2.elb.amazonaws.com`). You generally want to map your own custom domain to these resources.

**CNAME Record:**

* Points a hostname to any other hostname.


* **Exam Rule:** It can ONLY be used for non-root domains (e.g., `app.mydomain.com`).



**Alias Record:**

* An Amazon-specific extension to DNS functionality that maps a hostname directly to an AWS resource.


* Automatically recognizes changes in the target resource's IP addresses.


* **Exam Rule:** It WORKS for both root domains (Zone Apex like `mydomain.com`) and non-root domains.


* They are free of charge and offer native health checking.


* Alias records are always of type A or AAAA (IPv4/IPv6).


* You cannot set the TTL on an Alias record.



**Valid Alias Targets:** 

* Elastic Load Balancers
* CloudFront Distributions
* API Gateway
* Elastic Beanstalk environments
* S3 Websites
* VPC Interface Endpoints
* Global Accelerator
* Another Route 53 record in the same hosted zone
* 
*Exam Trap:* You **cannot** set an ALIAS record for an EC2 DNS name.



---

## 8. Routing Policies

Routing policies define how Route 53 responds to DNS queries. (Do not confuse this with Load Balancer routing, which routes actual network traffic; DNS simply responds to queries with an IP address) .

### Simple Routing

* Typically routes traffic to a single resource.


* You can specify multiple values in the same record. If you do, the client chooses one at random.


* When Alias is enabled, you can specify only one AWS resource.


* Cannot be associated with Health Checks.



### Weighted Routing

* Controls the percentage of requests that go to specific resources by assigning relative weights.


* Traffic % = `(Weight for a specific record) / (Sum of all weights)`.


* Weights do not need to sum up to 100.


* Useful for load balancing between regions or testing new application versions.


* Assigning a weight of `0` stops sending traffic to that resource. If all records have a weight of `0`, traffic is returned equally among them.


* Can be associated with Health Checks.



### Latency-based Routing

* Redirects users to the resource that provides the least latency.


* Based on traffic patterns between users and AWS Regions. (For example, a user in Germany might be routed to the US if that happens to provide the lowest latency at that moment) .


* Can be associated with Health Checks for failover capabilities.



### Failover Routing (Active-Passive)

* Routes traffic to a Primary resource as long as it is healthy.


* If the Primary fails its mandatory Health Check, Route 53 automatically fails over to the Secondary (Disaster Recovery) resource.



### Geolocation Routing

* Routes traffic based on the actual geographic location of the user (Continent, Country, or US State).


* *Crucial distinction:* This is different from Latency-based routing. It doesn't care about network speed, only physical location.


* If locations overlap, the most precise location is selected.


* You should always create a "Default" record in case the user's location doesn't match any of your rules.


* Can be associated with Health Checks.



### Geoproximity Routing

* Routes traffic based on the geographic location of users *and* your resources.


* You can shift traffic boundaries using a "Bias" value.


* To expand a region's traffic reach: specify a bias of 1 to 99.


* To shrink a region's traffic reach: specify a bias of -1 to -99.




* Works for AWS resources (specify region) and Non-AWS resources (specify Latitude/Longitude).


* **Exam Rule:** You *must* use the Route 53 Traffic Flow feature to use Geoproximity.



### IP-based Routing

* Routing is determined by the client's IP address.


* You provide a list of CIDR blocks (user-IP-to-endpoint mappings).


* For example, you can route end-users from a specific Internet Service Provider (ISP) to a particular endpoint to optimize network costs.



### Multi-Value Answer Routing

* Used when routing traffic to multiple resources.


* Route 53 returns multiple values (up to 8 healthy records) to the client.


* It is associated with Health Checks, meaning Route 53 will *only* return IP addresses for healthy resources.


* *Note:* It is not a substitute for an actual Elastic Load Balancer.



---

## 9. Route 53 Health Checks

Health Checks enable Automated DNS Failover. There are three main types of health checks:

1. Monitoring an Endpoint (Public Resources only): 

* Around 15 global health checkers continually monitor your endpoint via HTTP, HTTPS, or TCP.


* The default Healthy/Unhealthy threshold is 3.


* The default interval is 30 seconds (can be lowered to 10 seconds for a higher cost).


* *Exam Rule:* An endpoint is considered healthy if more than 18% of health checkers report it as healthy.


* Passes based on 2xx and 3xx HTTP status codes, or by checking text in the first 5120 bytes of the response.


* You must configure your firewall/router to allow incoming requests from Route 53 Health Checker IP ranges.



2. Calculated Health Checks: 

* Combine the results of multiple child health checks into a single parent check.


* Uses logic operators (OR, AND, NOT).


* Can monitor up to 256 Child Health Checks.


* Great for website maintenance—you can ensure the parent stays "healthy" even if a subset of servers are taken down.



3. Monitoring CloudWatch Alarms (For Private Resources): 

* Route 53 health checkers live *outside* your VPC and cannot directly access private endpoints (like a database).


* To check a private resource, create a CloudWatch Metric and Alarm for that resource, then instruct Route 53 to monitor that specific CloudWatch Alarm.



---

## 10. Domain Registrars vs. DNS Services

A Domain Registrar and a DNS Service are distinct entities, though they are often bundled.

You might purchase a domain from a 3rd party registrar (like GoDaddy) but decide to use Amazon Route 53 as your DNS service. To do this:

1. Create a Public Hosted Zone in Route 53.


2. Route 53 will generate Name Server (NS) records for that zone (e.g., `ns-1083.awsdns-07.org`).


3. Log into your 3rd party registrar and replace their default Name Servers with the ones provided by Route 53.



---

## 11. Route 53 Hybrid DNS and Resolver Endpoints

By default, the Route 53 Resolver automatically answers queries for local VPC domain names, records in Private Hosted Zones, and public names.

However, enterprises often need Hybrid DNS—resolving queries between an AWS VPC and an On-Premises network connected via Direct Connect (DX) or a VPN. This requires Route 53 Resolver Endpoints:

* **Inbound Endpoint:** Allows your On-Premises DNS Resolvers to forward queries into AWS. It allows the on-premises network to resolve domain names for AWS resources (like EC2 instances) and records inside your Private Hosted Zones.


* **Outbound Endpoint:** Allows the Route 53 Resolver inside your VPC to forward DNS queries out to your On-Premises DNS Resolvers, enabling your cloud resources to resolve local, on-premises hostnames.
