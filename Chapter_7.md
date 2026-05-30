

#  AWS Databases – RDS, Aurora, & ElastiCache

## Section 1: Amazon Relational Database Service (RDS)

### Introduction to RDS

Amazon RDS stands for Relational Database Service. It is a fully managed service that allows you to create and operate relational databases in the cloud, utilizing SQL as the primary query language.

AWS supports a wide variety of database engines under the RDS umbrella:

* PostgreSQL 


* MySQL 


* MariaDB 


* Oracle 


* Microsoft SQL Server 


* IBM DB2 


* Amazon Aurora (an AWS proprietary database) 



### Why Choose RDS Over Deploying on EC2?

Deploying a database manually on an EC2 instance leaves the administrative burden entirely on you. RDS, as a managed service, provides significant advantages:

* **Automation:** AWS handles automated provisioning and OS patching.


* **Resilience & Recovery:** You get continuous backups and the ability to restore your database to a specific timestamp (Point in Time Restore).


* **Visibility:** Built-in monitoring dashboards track database performance.


* **Performance & Reliability:** Access to Read Replicas for improved read capabilities and Multi-AZ setups for Disaster Recovery (DR).


* **Management:** Scheduled maintenance windows for database upgrades.


* **Flexibility:** Easily scale your databases vertically or horizontally. Data storage is backed by Elastic Block Store (EBS).


* *Note:* Because it is a managed service, you cannot SSH directly into your RDS underlying instances.



### RDS Storage Auto Scaling

RDS can dynamically increase the storage of your database instance. When RDS detects that you are running out of free space, it scales the storage automatically, helping you avoid manual interventions. To control costs and runaway scaling, you must configure a **Maximum Storage Threshold**.

Auto-scaling triggers when the following conditions are met:

* Free storage drops below 10% of your allocated storage.


* The low-storage state persists for at least 5 minutes.


* At least 6 hours have passed since the last storage modification.



This feature supports all RDS database engines and is especially useful for applications with unpredictable workloads.

### Scaling Reads: RDS Read Replicas

To handle high read traffic, RDS allows you to create up to 15 Read Replicas.

* **Placement:** Replicas can reside in the same Availability Zone (AZ), across different AZs, or even across different AWS Regions.


* **Replication Method:** Data replication is **ASYNC** (asynchronous), meaning reads from the replicas are eventually consistent.


* **Promotion:** A read replica can be promoted to become its own standalone database if needed.


* **Application Logic:** Your applications must be updated with the specific connection string of the Read Replica to leverage the scaled read performance.


* 
**Use Cases:** Replicas are strictly used for `SELECT` (read-only) statements, not `INSERT`, `UPDATE`, or `DELETE`. A common use case is running an analytics or reporting application against a Read Replica so that the primary production database remains unaffected.



**Network Costs for Read Replicas:**
Generally, AWS charges a network fee when data moves between different Availability Zones. However, for RDS Read Replicas placed within the *same* Region (even if in different AZs), this network transfer is entirely free. If you deploy a Read Replica in a *different* Region (Cross-Region), you will incur network costs.

```text
+----------------------+
| Reporting App        |
| (Queries Analytics)  |
+---------+------------+
          | (Reads)
          v
+----------------------+     ASYNC       +-----------------------+
|  RDS Read Replica    | <-------------  |   RDS Master DB       |
|  (Read-only queries) |   Replication   |   (Handles Writes)    |
+----------------------+                 +---------^-------------+
                                                   | (Writes)
                                         +-----------------------+
                                         | Production App        |
                                         +-----------------------+

```

### High Availability: RDS Multi-AZ

While Read Replicas are designed for scaling, **Multi-AZ deployments are designed for Disaster Recovery (DR)**.

* **Replication Method:** Multi-AZ uses **SYNC** (synchronous) replication from the Master database to a Standby database in another AZ.


* **Failover:** If the primary AZ experiences a failure, network loss, or instance/storage issues, RDS automatically fails over to the Standby.


* **App Transparency:** Because Multi-AZ provides a single DNS name, the failover happens automatically with no manual intervention required in your application code.


* **Zero Downtime Migration:** You can transition a Single-AZ database to a Multi-AZ deployment without stopping the database. By clicking "modify", RDS internally takes a snapshot, restores it to a new AZ, and establishes synchronization.



```text
               +-------------------+
               |    Application    |
               +---------+---------+
                         | (One DNS Name - Auto Failover)
                         v
+------------------+           +-------------------+
|  Master DB (AZ A)| --SYNC--> |  Standby DB (AZ B)|
|  (Active)        |           |  (Passive)        |
+------------------+           +-------------------+

```

### RDS Custom

For most users, standard RDS is ideal because AWS manages the entire OS and database layer. However, if you require underlying access, **RDS Custom** provides a managed Oracle and Microsoft SQL Server environment where you retain full admin access to the underlying OS and database.

With RDS Custom, you can:

* Configure custom settings, install patches, and enable native database features.


* Access the underlying EC2 instance via SSH or AWS SSM Session Manager.


* *Note:* You must temporarily disable Automation Mode to apply customizations, and it is highly recommended to take a snapshot before doing so.



### RDS Backups, Restores, & Security

**Backups:**

* **Automated Backups:** RDS automatically takes daily full backups during a defined backup window, and backs up transaction logs every 5 minutes. This provides the ability to restore to any point in time (from the oldest backup up to 5 minutes ago). Retention can be set from 1 to 35 days (setting it to 0 disables automated backups).


* **Manual Snapshots:** Users can trigger snapshots manually. These are retained indefinitely (as long as you want).


* **Cost-Saving Trick:** If you stop an RDS instance, you still pay for the allocated storage. If you plan to stop it for an extended period, it is cheaper to take a snapshot, delete the instance, and restore the snapshot later.


* **Restoring:** Restoring an RDS or Aurora backup/snapshot always creates a completely *new* database instance. You can also backup on-premises MySQL databases to Amazon S3 and restore them directly onto a new RDS or Aurora cluster.



**Security:**

* **At-Rest Encryption:** Handled by AWS KMS. Encryption must be defined at launch time. If a master DB is unencrypted, its Read Replicas cannot be encrypted. To encrypt an existing unencrypted DB, you must take a snapshot and restore that snapshot as an encrypted instance.


* **In-Flight Encryption:** Databases are TLS-ready by default; applications must use AWS TLS root certificates.


* **Access Control:** Network access is governed by Security Groups. You can also enable IAM Authentication, allowing applications to connect using IAM roles instead of traditional usernames and passwords.


* **Auditing:** Audit logs can be enabled and pushed to CloudWatch Logs for long-term retention.


* *Note:* There is no SSH access to standard RDS instances (only available on RDS Custom).



### Amazon RDS Proxy

Opening and closing database connections frequently can exhaust your database's memory and CPU. **Amazon RDS Proxy** is a fully managed, serverless, highly available database proxy that allows applications to pool and share established database connections.

* **Benefits:** It minimizes open connections and timeouts, reducing stress on database resources. It also reduces Aurora and RDS failover times by up to 66%.


* **Compatibility:** Supports RDS (MySQL, PostgreSQL, MariaDB, MS SQL Server) and Aurora (MySQL, PostgreSQL). It requires no code changes for most applications.


* **Security:** RDS Proxy enforces IAM Authentication for databases and integrates with AWS Secrets Manager to store credentials securely. It is strictly private and can only be accessed from within a VPC.



---

## Section 2: Amazon Aurora

### Aurora Overview & Architecture

Amazon Aurora is AWS’s proprietary database technology (not open source). However, it is fully compatible with PostgreSQL and MySQL drivers. As a cloud-native database, Aurora boasts massive performance gains: up to 5x the performance of standard MySQL on RDS, and 3x the performance of standard PostgreSQL on RDS.

**Aurora Cluster Architecture:**
Unlike traditional RDS where storage and compute are bound together, Aurora utilizes a shared, auto-expanding storage volume.

* **Storage:** Data is striped across hundreds of volumes and automatically grows in 10GB increments up to a massive 256 TB.


* **Resiliency:** Aurora maintains 6 copies of your data across 3 Availability Zones. To perform a write, 4 out of 6 copies must acknowledge it; to perform a read, 3 out of 6 are required. It features peer-to-peer self-healing.


* **Endpoints:** Client applications connect to Aurora via endpoints.


* **Writer Endpoint:** Points directly to the single Master instance that handles all writes.


* **Reader Endpoint:** Provides connection load balancing across all available Read Replicas to serve read traffic.





```text
                            +----------+
                            |  Client  |
                            +----+-----+
                                 |
           +---------------------+---------------------+
           |                                           |
           v                                           v
+---------------------+                      +---------------------+
|   Writer Endpoint   |                      |   Reader Endpoint   |
| (Points to Master)  |                      |  (Load Balancing)   |
+---------+-----------+                      +---------+-----------+
          |                                            |
          v                                            v
     [ Master ] <==============================> [ Replicas ]
    (Takes Writes)          (Shared Auto-Expanding Storage Volume)

```

### High Availability & Advanced Features

Aurora is natively Highly Available (HA). Failovers happen automatically in less than 30 seconds. While Aurora costs roughly 20% more than standard RDS, its efficiency and features often justify the price.

**Key Features:**

* Up to 15 Read Replicas with sub-10ms replica lag (faster replication than standard MySQL).


* Support for Cross-Region Replication.


* Automated patching with zero downtime, advanced monitoring, and push-button scaling.


* **Backtrack:** An Aurora-specific feature allowing you to instantly rewind the database state to any point in time without needing to restore a backup.


* 
**Replica Auto Scaling:** Aurora can automatically add more Read Replicas if CPU usage spikes from high read traffic, extending the Reader Endpoint seamlessly.



**Custom Endpoints:**
You can group a subset of Aurora instances (for example, instances with high memory specifically provisioned for analytics) into a "Custom Endpoint". Once you define a Custom Endpoint, you generally stop using the default Reader Endpoint for those specific analytical queries.

### Aurora Serverless & Global Aurora

**Aurora Serverless:**
If your application has unpredictable, intermittent, or infrequent workloads, you can use Aurora Serverless. It completely automates database instantiation and scales based on actual usage. Because there is no capacity planning required and you pay per second, it can be highly cost-effective.

**Global Aurora:**
For cross-region disaster recovery and low-latency global reads, Global Aurora is the recommended architecture.

* You designate one Primary Region for read/write operations.


* You can attach up to 10 secondary regions (read-only).


* Each secondary region can host up to 16 Read Replicas, and the cross-region replication lag is typically less than 1 second.


* In the event of a disaster, promoting a secondary region to become the primary has a Recovery Time Objective (RTO) of less than 1 minute.



### Specialized Aurora Capabilities

**Aurora Machine Learning:**
Aurora allows you to embed Machine Learning predictions directly into your SQL queries without needing any prior ML experience. It securely integrates with:

* **Amazon SageMaker** for custom ML models.


* 
**Amazon Comprehend** for sentiment analysis.


* *Use Cases:* Fraud detection, ads targeting, sentiment analysis, product recommendations.



**Babelfish for Aurora PostgreSQL:**
Migrating from Microsoft SQL Server can be difficult due to proprietary code. Babelfish allows Aurora PostgreSQL to understand MS SQL Server commands (like T-SQL). Consequently, legacy MS SQL Server applications can run on Aurora PostgreSQL with little to no code changes, utilizing the same database client drivers.

**Aurora Database Cloning:**
Creating a staging database from a production database usually requires a time-consuming snapshot and restore process. Aurora Database Cloning creates a new cluster from an existing one almost instantly. It utilizes a "copy-on-write" protocol. Initially, the clone shares the exact same underlying storage volume as the original (making it fast and cheap). It only allocates new storage when updates are made to the new cluster.

---

## Section 3: Amazon ElastiCache

### ElastiCache Overview

Just as Amazon RDS provides managed relational databases, Amazon ElastiCache provides managed **Redis** or **Memcached** environments. These are in-memory database engines that deliver ultra-high performance and exceptionally low latency.

ElastiCache is primarily used to:

1. Reduce the load on traditional databases for read-intensive workloads.


2. Store temporary session data to make applications stateless.



AWS manages the underlying infrastructure, including OS maintenance, patching, setup, monitoring, failure recovery, and backups. However, unlike RDS, integrating ElastiCache requires significant application code changes because your application must be programmed to interact with the cache.

### ElastiCache Architecture Patterns

**1. DB Cache Pattern:**
In this architecture, the application queries ElastiCache first.

* **Cache Hit:** If the data exists in the cache, it is returned instantly.


* **Cache Miss:** If the data is absent, the application reads it from the primary database (like RDS) and then explicitly writes it back to ElastiCache for future requests.


* This significantly relieves load on RDS, but requires an active "cache invalidation" strategy to ensure applications do not serve outdated (stale) data.



```text
               (1. Cache Hit / Cache Miss)
           +---------------------------------+
           |                                 v
+-------------------+                 +-------------------+
|    Application    |                 | Amazon ElastiCache|
+-------------------+                 +-------------------+
           |                                 ^
           | (2. Read from DB on miss)       | (3. Write to cache)
           v                                 |
+-------------------+                        |
|    Amazon RDS     |------------------------+
+-------------------+

```

**2. User Session Store Pattern:**
When a user logs into an application, the application writes the user's session data into ElastiCache. If the user's subsequent requests are routed to a completely different instance of the application, that instance queries ElastiCache, retrieves the session data, and the user remains seamlessly logged in.

### Redis vs. Memcached

When provisioning ElastiCache, you must choose your engine based on application needs:

| Feature | Redis 

 | Memcached 

 |
| --- | --- | --- |
| **Availability** | Multi-AZ with Auto-Failover 

 | No high availability / No replication 

 |
| **Scaling** | Read Replicas to scale reads 

 | Multi-node data partitioning (Sharding) 

 |
| **Data Persistence** | Data Durability via AOF persistence 

 | Non-persistent 

 |
| **Backup & Restore** | Supported natively 

 | Serverless (no native backup/restore) 

 |
| **Data Types** | Advanced (Sets, Sorted Sets) 

 | Simple objects (Multi-threaded architecture) 

 |

### ElastiCache Security

* **IAM Policies:** IAM policies are strictly used for AWS API-level security (e.g., who can create or delete a cluster). ElastiCache does support IAM Authentication for Redis.


* **Redis AUTH:** When launching a Redis cluster, you can establish a password/token. This acts as an extra layer of security on top of standard VPC Security Groups.


* **In-flight Encryption:** ElastiCache supports SSL encryption in flight.


* **Memcached Authentication:** Memcached utilizes SASL-based authentication.



### Advanced Caching Strategies & Use Cases

**Caching Patterns:**

* **Lazy Loading:** Data is only cached when it is actively read. The downside is that data can become stale.


* **Write Through:** Data is added or updated in the cache at the exact same moment it is written to the database. This guarantees no stale data, but adds slight write latency.


* **Session Store:** Leveraging TTL (Time-To-Live) features to automatically expire temporary session data.



**Real-World Use Case: Gaming Leaderboards**
Calculating leaderboards in real-time on a traditional relational database is computationally complex and slow. ElastiCache for Redis solves this using its native **Sorted Sets** data type. Sorted sets inherently guarantee element uniqueness and automatically order the data. Each time a player's score changes, it is updated and ranked in real-time, making Redis the perfect engine for live leaderboards.
