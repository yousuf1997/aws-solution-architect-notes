Here is a comprehensive, book-like chapter based on the provided slides, complete with ASCII illustrations and formatted strictly without tables.

---

# Chapter: AWS Integration & Messaging – SQS, SNS, & Kinesis

When deploying multiple applications in a cloud environment, they will inevitably need to communicate with one another. Understanding how to integrate these applications effectively is crucial for building scalable, resilient, and highly available systems.

## 1. Patterns of Application Communication

There are two primary patterns of application communication:

1. **Synchronous Communications (Application-to-Application):** In this model, applications speak directly to one another. For example, a Buying Service communicates directly with a Shipping Service.


2. **Asynchronous / Event-Based Communications (Application-to-Queue-to-Application):** Applications communicate via an intermediary, such as a queue. The Buying Service sends a message to a queue, which is then picked up by the Shipping Service.



Synchronous communication can become highly problematic if there are sudden spikes in traffic. For instance, if an application usually encodes 10 videos but suddenly receives a request to encode 1,000, the synchronous connection might become overwhelmed.

To solve this, it is better to **decouple** your applications using asynchronous services. These services can scale completely independently from your core application. The three main models for decoupling are:

* **SQS:** The queue model.


* **SNS:** The publish/subscribe (pub/sub) model.


* **Kinesis:** The real-time streaming model.



---

## 2. Amazon SQS (Simple Queue Service)

Amazon SQS is the oldest AWS offering, having been available for over 10 years. It is a fully managed service explicitly used to decouple applications.

### 2.1 SQS Standard Queues

* **Throughput & Capacity:** It offers unlimited throughput and can hold an unlimited number of messages in the queue.


* **Retention:** Messages are retained for a default of 4 days, with a maximum retention period of 14 days.


* **Latency:** SQS offers low latency, typically less than 10 milliseconds for both publishing and receiving messages.


* **Message Size:** There is a limitation of 1,024 KB per message sent to the queue.


* **Delivery Guarantees:** It provides "at least once delivery," meaning duplicate messages can occasionally occur.


* **Ordering:** SQS Standard utilizes "best effort ordering," meaning messages can sometimes arrive out of order.



```text
[ ASCII Illustration: SQS Queue Model ]

 +----------+       +-------------------+       +----------+
 | Producer | ----> |                   | ----> | Consumer |
 +----------+       |  ||| SQS Queue ||||       +----------+
                    |                   |
 +----------+ ----> |                   | ----> +----------+
 | Producer |       +-------------------+       | Consumer |
 +----------+                                   +----------+

```

### 2.2 Producing and Consuming Messages

**Producing:** Messages are sent to SQS using the AWS SDK via the `SendMessage` API. Messages are persisted in SQS until a consumer deletes them. A typical use case is sending an order to be processed, where the message payload might include an Order ID, a Customer ID, and any other relevant attributes.

**Consuming:** Consumers can be EC2 instances, on-premises servers, or AWS Lambda functions.

* Consumers poll SQS for messages, receiving up to 10 messages at a time.


* They process these messages (for example, by inserting the data into an Amazon RDS database).


* Once successfully processed, consumers must delete the messages from the queue using the `DeleteMessage` API so they are not processed again.



Because SQS Standard offers unlimited throughput, multiple consumers can receive and process messages in parallel. You can scale consumers horizontally to improve the processing throughput.

### 2.3 SQS with Auto Scaling

SQS is highly effective when used to decouple application tiers (like a front-end web app and a back-end processing application). An Auto Scaling Group (ASG) of EC2 consumers can automatically scale based on the queue length. This is achieved using a CloudWatch Alarm triggered by the `ApproximateNumberOfMessages` CloudWatch metric. SQS also acts as an excellent buffer for database writes; if the load is too high, the queue absorbs the incoming requests, preventing transactions from being lost before they can be inserted into RDS, Aurora, or DynamoDB.

### 2.4 SQS Security

* **Encryption:** In-flight encryption is handled via the HTTPS API. At-rest encryption is provided using KMS keys. Client-side encryption is also an option if the client wishes to encrypt and decrypt data themselves.


* **Access Controls:** IAM policies regulate access to the SQS API.


* **SQS Access Policies:** Similar to S3 bucket policies, these are useful for cross-account access and for allowing other AWS services (like SNS or S3) to write to an SQS queue.



### 2.5 Message Visibility Timeout

After a consumer polls a message, it becomes invisible to other consumers.

* By default, this "message visibility timeout" is 30 seconds.


* This means the consumer has 30 seconds to process and delete the message.


* If the message is not processed within this window, it becomes visible again in SQS, resulting in the message being processed twice.



If a consumer needs more time, it can call the `ChangeMessageVisibility` API. Selecting the right timeout is a balancing act: if it is too high (hours) and a consumer crashes, re-processing takes a very long time. If it is too low (seconds), duplicate processing is highly likely.

### 2.6 Long Polling

When a consumer requests messages, it can optionally "wait" for messages to arrive if the queue is empty. This is known as Long Polling.

* It decreases the number of API calls made to SQS.


* It increases application efficiency and reduces latency.


* The wait time can be set between 1 and 20 seconds, with 20 seconds being the preferable duration.


* Long Polling is always preferable to Short Polling.


* It can be enabled at the queue level or at the API level via the `WaitTimeSeconds` parameter.



### 2.7 Amazon SQS – FIFO Queues

FIFO stands for First In First Out, ensuring strict ordering of messages.

* **Throughput:** Throughput is limited compared to Standard SQS: 300 messages per second without batching, and 3,000 messages per second with batching.


* **Deduplication:** It guarantees "exactly-once" delivery by removing duplicates using a Deduplication ID.


* **Ordering:** Messages are processed in exact order by the consumer. Ordering is managed by a mandatory parameter called the Message Group ID, which ensures all messages in the same group are strictly ordered.



---

## 3. Amazon SNS (Simple Notification Service)

When an application needs to send a single message to multiple receivers, Amazon SNS is the solution. Instead of integrating directly with multiple services, the event producer sends a message to a single SNS Topic.

### 3.1 Pub/Sub Model

In the Pub/Sub (Publish/Subscribe) model, any "event receiver" (subscription) listening to the SNS topic will receive notifications.

* Each subscriber receives all the messages sent to the topic.


* SNS is highly scalable, supporting up to 12,500,000 subscriptions per topic.


* AWS accounts have a limit of 100,000 SNS topics.



```text
[ ASCII Illustration: SNS Pub/Sub ]

                      /--> [ SQS Queue ] -----> [ Fraud Service ]
                     /
[ Buying Service ] ----->  [[ SNS Topic ]]
                     \
                      \--> [ Email Notif ] ---> [ Admin Email ]

```

### 3.2 Publishers and Subscribers

**Publishers:** Many AWS services can send data directly to SNS for notifications. These include CloudWatch Alarms, AWS Budgets, Auto Scaling Groups, S3 Bucket events, CloudFormation, AWS DMS, DynamoDB, and RDS Events.
**Subscribers:** Subscribers can include SQS, Lambda, Kinesis Data Firehose, Emails, SMS & Mobile Notifications, and HTTP/HTTPS Endpoints.

### 3.3 Publishing Methods

* **Topic Publish (using the SDK):** You create a topic, create one or many subscriptions, and publish directly to the topic.


* **Direct Publish (Mobile Apps SDK):** Used for mobile push notifications. You create a platform application, create a platform endpoint, and publish to that endpoint. This works with services like Google GCM, Apple APNS, and Amazon ADM.



### 3.4 SNS Security

SNS shares the same security profile as SQS:

* In-flight HTTPS encryption, at-rest KMS encryption, and client-side encryption.


* IAM policies to regulate API access.


* SNS Access Policies for cross-account access and allowing services like S3 to write to an SNS topic.



### 3.5 SNS Advanced Features

**SNS + SQS: Fan-Out Architecture**
You can push data once into SNS, and it is instantly received by multiple subscribed SQS queues. This is fully decoupled with no data loss. The SQS queues then provide data persistence, delayed processing, and work retries. You can add more SQS subscribers over time. This model works across AWS Regions, provided the SQS queue access policies allow SNS to write to them.

* *Use Case:* S3 Events. S3 limits you to one event rule per combination of event type and prefix. If you need to send the same S3 event to multiple SQS queues, you must use SNS Fan-Out.



**SNS to Amazon S3**
SNS can send messages directly to Kinesis Data Firehose. Because Firehose supports S3 as a destination, you can easily route SNS notifications directly into Amazon S3 storage or any other supported Kinesis Data Firehose destination.

**SNS FIFO Topics**
Just like SQS FIFO, SNS offers FIFO topics that maintain strict First In First Out ordering.

* It supports ordering by Message Group ID and deduplication (via Deduplication ID or Content-Based Deduplication).


* It shares the same limited throughput restrictions as SQS FIFO.


* Subscribers to SNS FIFO topics can be both Standard and FIFO SQS queues.


* **Fan-Out + Ordering:** By combining SNS FIFO with SQS FIFO, you achieve a fan-out architecture that strictly maintains ordering and deduplication across all consumer queues.



**SNS Message Filtering**
A JSON policy can be used to filter the messages sent to an SNS topic's subscriptions. If a subscription does not have a filter policy attached, it will receive every single message published to the topic. This is highly useful for routing specific payloads (e.g., separating "Cancelled" orders from "Placed" orders into different SQS queues).

---

## 4. Amazon Kinesis

While SQS and SNS are for messaging, Amazon Kinesis is designed to collect and store massive streams of data in real-time.

### 4.1 Kinesis Data Streams

Kinesis Data Streams collects real-time data from sources like click streams, IoT devices, metrics, and logs.

* **Producers:** Applications, Kinesis Agent.


* **Consumers:** Lambda functions, Applications, Amazon Data Firehose, Managed Service for Apache Flink.



**Key Features:**

* Data retention ranges from a minimum up to 365 days.


* Because data is retained, consumers have the ability to reprocess or replay data.


* Data is immutable; it cannot be deleted from Kinesis until it expires naturally.


* It handles data up to 10MiB, typically consisting of many small real-time data packets.


* Data ordering is guaranteed for data sharing the same "Partition ID".


* Security includes at-rest KMS encryption and in-flight HTTPS encryption.


* AWS provides the Kinesis Producer Library (KPL) to write optimized producers and the Kinesis Client Library (KCL) for optimized consumers.



**Capacity Modes:**

* **Provisioned Mode:** You manually choose the number of shards. Each shard provides 1 MB/s (or 1000 records per second) in, and 2 MB/s out. You scale manually to increase or decrease shards, and you pay per provisioned shard per hour.


* **On-Demand Mode:** No manual capacity management is required. It defaults to a provisioned capacity of 4 MB/s in (or 4000 records per second). It automatically scales based on the highest observed throughput peak from the last 30 days. Pricing is based on a per-stream-per-hour fee plus the data transferred in/out per GB.



### 4.2 Amazon Data Firehose

Formerly known as "Kinesis Data Firehose," this is a fully managed, serverless service with automatic scaling. You only pay for what you use. It takes streaming data (from Kinesis Streams, CloudWatch, AWS IoT, SDKs, etc.) and loads it into external destinations in near real-time.

**Destinations include:**

* **AWS Destinations:** Amazon Redshift, Amazon S3, Amazon OpenSearch Service.


* **3rd Party Partners:** Splunk, MongoDB, Datadog, New Relic.


* **Custom Destinations:** Any custom HTTP endpoint.



**Key Features:**

* It uses buffering capabilities based on size and time.


* It supports various formats: CSV, JSON, Parquet, Avro, Raw Text, and Binary data.


* It can convert data formats (e.g., to Parquet or ORC) and compress data using gzip or snappy.


* It allows for custom data transformations (such as converting CSV to JSON) on the fly using AWS Lambda.



---

## 5. Service Comparisons

Understanding when to use which service is essential for cloud architecture design.

### Kinesis Data Streams vs. Amazon Data Firehose

* **Purpose:** Kinesis Data Streams handles streaming data collection using custom producer and consumer code, while Firehose simply loads streaming data into S3, Redshift, OpenSearch, 3rd party apps, or custom HTTP endpoints.


* **Management:** Streams require you to choose between Provisioned or On-Demand modes, whereas Firehose is entirely fully managed with automatic scaling.


* **Latency:** Streams are true real-time, whereas Firehose operates in near real-time due to its buffering capability.


* **Storage & Replay:** Streams store data for up to 365 days and support replay capabilities. Firehose does not store data and does not support replay.



### SQS vs. SNS vs. Kinesis

* **SQS:**
* Consumers "pull" data from the queue.


* Data is permanently deleted after being consumed.


* You can attach as many worker consumers as you want without provisioning throughput.


* Ordering guarantees are only available on FIFO queues.


* Allows for individual message delay capabilities.




* **SNS:**
* "Pushes" data to many subscribers in a Pub/Sub model.


* Supports up to 12,500,000 subscribers per topic and 100,000 topics.


* Data is not persisted; if it is not delivered to an endpoint, it is lost.


* No throughput provisioning is needed.


* Commonly integrates with SQS for the fan-out architecture pattern.




* **Kinesis:**
* Standard mode utilizes data pulling (2 MB per shard), while Enhanced Fan-out pushes data (2 MB per shard per consumer).


* Possibility to replay data.


* Built for real-time big data, analytics, and ETL workflows.


* Maintains ordering at the shard level.


* Data expires automatically after X days.


* Requires choosing between Provisioned or On-Demand capacity modes.





---

## 6. Amazon MQ

While SQS and SNS are highly scalable "cloud-native" services using proprietary AWS protocols, many traditional on-premises applications rely on open messaging protocols. These protocols include MQTT, AMQP, STOMP, Openwire, and WSS.

When migrating these applications to the cloud, it can be costly and time-consuming to re-engineer them to use SQS and SNS. Instead, AWS provides Amazon MQ.

* Amazon MQ is a managed message broker service specifically designed for ActiveMQ and RabbitMQ.


* It supports both queue features (similar to SQS) and topic features (similar to SNS).


* Unlike SQS and SNS, Amazon MQ runs on dedicated servers and does not scale infinitely.


* To ensure High Availability, it can run in a Multi-AZ configuration with automatic failover.



```text
[ ASCII Illustration: Amazon MQ High Availability ]

      Region (us-east-1)
      
                 Active Broker (us-east-1a) <------> [ Amazon EFS ]
               /                                        (Storage)
[ Client ] <---                                             ^
               \ (failover)                                 |
                 Standby Broker (us-east-1b) <--------------+

```

In a Multi-AZ setup, a client connects to an Active Broker in one Availability Zone. The broker uses an Amazon EFS drive as its shared storage. If the Active Broker fails, the client experiences a failover and connects to the Standby Broker in a different Availability Zone, which mounts the same EFS storage to resume operations.
