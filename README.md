# AWS Certified Developer - Associate Study notes

These are my study notes for the AWS Certified Developer - Associate certification. I have already passed the Solutions Architect - Associate exam so the notes might not cover topics if i feel i already know it well enough.

Table of Contents
=================

* [AWS Services 10,000 foot overview](#AWS-Services)

* [IAM](#iam)

* [Security and Encryption](#security-and-encryption)

* [EC2](#EC2)

* [EBS & EFS](#ebs--efs)

* [RDS](#RDS)

* [S3](#S3)

* [ECS](#ECS)

* [DynamoDB](#DynamoDB)
  
* [Messaging](#messaging)

* [Elastic Beanstalk](#Elastic-Beanstalk)

* [CI/CD](#cicd)

* [CloudFormation](#CloudFormation)

* [Monitoring](#monitoring)

* [AWS Shared Responsibility](#AWS-Shared-Responsibility)

* [Route 53](#Route-53)

* [VPC](#VPC)

* [CloudFront](#CloudFront)

* [Lambda](#Lambda)

* [API Gateway](#api-gateway)

* [SAM](#sam)

[Exam Blueprint](http://awstrainingandcertification.s3.amazonaws.com/production/AWS_certified_developer_associate_blueprint.pdf)

# Cheatsheet

[https://digitalcloud.training/certification-training/aws-developer-associate/](https://digitalcloud.training/certification-training/aws-developer-associate/)

# IAM

[IAM FAQ](https://aws.amazon.com/iam/faqs/)
* Users: A physical person. Users have the choice of being given access to the management console and/or programmatic access. Access via the management console enables a password for the account. Enabling programmatic access enables an access key ID and secret access key. 
* Groups: Funtions and teams. Groups allow you to apply policies to multiple users. Recommended to apply policies to groups even if it is for one user.
  * Users can be members of multiple groups
  * Groups cannot be nested
* Roles: Internal usage within AWS resources.
* Policies: Policies are JSON documents that contain permissions to AWS services. ie Roles and secrets
  * Policy types:
    * Identity-based policies – Attach managed and inline policies to IAM identities (users, groups to which users belong, or roles). Identity-based policies grant permissions to an identity.
    * Resource-based policies – Attach inline policies to resources. The most common examples of resource-based policies are Amazon S3 bucket policies and IAM role trust policies.
      * IAM service supports only one type of resource-based policy called a role trust policy, which is attached to an IAM role. An IAM role is both an identity and a resource that supports resource-based policies. 
    * Permissions boundaries – Use a managed policy as the permissions boundary for an IAM entity (user or role). That policy defines the maximum permissions that the identity-based policies can grant to an entity. **Do not grant permissions**.
    * Organizations SCPs – Use an AWS Organizations service control policy (SCP) to define the maximum permissions for account members of an organization or organizational unit (OU). **Do not grant permission**. 
      * A deny list – actions are allowed by default, and you specify what services and actions are prohibited
      * An allow list – actions are prohibited by default, and you specify what services and actions are allowed
    * Access control lists (ACLs) – Use ACLs to control which principals in other accounts can access the resource to which the ACL is attached. 
    * Session policies – Pass advanced session policies when you use the AWS CLI or AWS API to assume a role or a federated user. 
  * IAM policy variables: Instead of creating individual policies for each user, you can use policy variables and create a single policy that applies to multiple users (a group policy). 
* To configure many AWS services, must ***pass*** an IAM role to the service during setup. For this, need action `iam:PassRole`.
* Billing and Cost: By default, IAM users do not have access to the AWS Billing and Cost Management console. Can grant access by activating IAM user access to the Billing and Cost Management console and attaching an IAM policy to your users.
* Tools:
  * IAM Access Analyzer: help identify the resources in organization and accounts, such as Amazon S3 buckets or IAM roles, that are shared with an external entity. This lets you identify unintended access to your resources and data, which is a security risk.
  * Access Advisor: To help identify the unused roles, IAM reports the last-used timestamp that represents when a role was last used to make an AWS request.

### Security Token Service (STS)

* Assume a role: define an IAM role with or cross-account and which principles can acess this role, use AssumeRole API to impersonate this role with temporary credentials fro **15 mins to 1 h**. *Example: The development team at a retail organization wants to allow a Lambda function in its AWS Account A to access a DynamoDB table in another AWS Account B. Create an IAM role in Account B with access to DynamoDB. Modify the trust policy of the role in Account B to allow the execution role of Lambda to assume this role. Update the Lambda function code to add the AssumeRole API call.* See [here](https://aws.amazon.com/premiumsupport/knowledge-center/lambda-function-assume-iam-role/)
  * MFA: Use GetSessionToken API to get a session token after MFA, need to set appropriate IAM policy with IAM conditions, set `aws::MultiFactorAuth::true` 

### Active Directory Federation
* AD: Database of objects, centralized security management on Window Servers with AD Domain Services.
* AWS Directory Services:
  * AWS Managed Microsoft AD: create own AD on AWS, establish "trust" connections with on-premise AD
  * AD Connector: Directory Gateway (proxy) to redirect to on-premise AD, users are managed solely on-premise
  * Simple AD: AD-compatible managed by AWS, can't be joined with on-premise AD
  
## Cognito
Comparing to IAM, Cognito is for "hundreds of users", "mobile users", "Social Identity Provider" like Google or "SAML users"
* Cognito User Pools: "Manage user and password". create a serverless database for application users. Integrate with API Gateway and ALB
  * Can invoke Lambda function on some **triggers** like authentication events, sign-up, messages or token creation
* Cognito Identity Pools (Federated Identities): "Access AWS services". get identitier for "users" so they obtain temporary AWS credentials, users then can access AWS services directly or through API Gateway. IAM credentials are obtained via STS. You can partition user access using ***policy variables***.
  
# Security and Encryption
### KMS
* Key Management Service managed by AWS. Keys are bound to region. 
  * Cusomter Master Keys: AES-256 or RSA & ECC. Three types: AWS default, user keys created in KMS and user keys imported
* Envelope Encryption: KMS Encrypt API has a limit for 4K, to encrypt > 4K need to use Envelope Encryption `GenerateDataKey` API
* All cryptographic operations shall the same quota. Over the quota will get `ThrottlingException`
* CloudWatch logs can be encrypted with KMS keys: `associate-kms-key` and `create-log-group` API to associate a CMK with a log group

### Secrets Store
* SSM Parameter Store: secure storage for configuration and secrets. Can create secure string parameters, which are parameters that have a plaintext parameter name and an encrypted parameter value. KMS is optional. 
* Secret Manager: store secrets, capability to force totation of secrects every X days. Integration with RDS. KMS is mandatory.

### CloudHSM
* A Hardware Security Module (HSM) provides secure key storage and cryptographic operations within a tamper-resistant hardware device

# EC2

[EC2 FAQ](https://aws.amazon.com/ec2/faqs/)

Access instance meta data at http://169.254.169.254/latest/meta-data/

* Instance profile: a container for an IAM role that you can use to pass role information to an EC2 instance when the instance starts
* Scipts can be run from the user data section when creating an instance
* Instance purchasing options
  * On-Demand Instances – Pay, by the second, for the instances that you launch.
  * Savings Plans – Reduce your Amazon EC2 costs by making a commitment to a consistent amount of usage, in USD per hour, for a term of 1 or 3 years.
  * Reserved Instances – Reduce your Amazon EC2 costs by making a commitment to a consistent instance configuration, including instance type and Region, for a term of 1 or 3 years.
    * The offering class of a Reserved Instance is either Standard or Convertible. A Standard Reserved Instance provides a more significant discount than a Convertible Reserved Instance, but you can't exchange a Standard Reserved Instance. 
    * Only Zonal Reserved Instances provide capacity reservations, Regional Reserved Instances don't
  * Spot Instances – Request unused EC2 instances, which can reduce your Amazon EC2 costs significantly. Can specify that Amazon EC2 should do one of the following when it interrupts a Spot Instance:
    * Stop the Spot Instance
    * Hibernate the Spot Instance
    * Terminate the Spot Instance
  * Dedicated Hosts – Pay for a physical host that is fully dedicated to running your instances, and bring your existing per-socket, per-core, or per-VM software licenses to reduce costs.
  * Dedicated Instances – Pay, by the hour, for instances that run on single-tenant hardware.
  * Capacity Reservations – Reserve capacity for your EC2 instances in a specific Availability Zone for any duration.
* Monitoring: By default, your instance is enabled for basic monitoring with **5-minute**. You can optionally enable detailed monitoring. After you enable detailed monitoring, the Amazon EC2 console displays monitoring graphs with a **1-minute** period for the instance. `aws ec2 monitor-instances --instance-ids i-1234567890abcdef0`
  * [Available CloudWatch metrics](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/viewing_metrics_with_cloudwatch.html): CPU, Disk, Network 

## Elastic Load Balancers

* Multi AZ, but not cross region. Cross-zone load balancing mechanism, see [here](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html)
* There are 3 types of load balancer; Application, Network and Classic. Application is used to route HTTP/HTTPS (L7) traffic. Network and Classic are used to route TCP (L4) traffic.
    1. Application - Layer 7, target groups (based on path, hostname, query string), TLS termination 
          1. Target Groups: EC2, ECS, Lambda, IP
          2. True client IP address is in `X-Forwarded-For`
    2. Network - Extermeme performance and static IP
    3. Classic (also refered to as Elastic Load Balancer ELB) - Layer 4 & 7, Fixed hostname
      *  By default, the health check configuration of your Auto Scaling group is set as an EC2 type that performs a status check of EC2 instances. To automate the replacement of unhealthy EC2 instances, you must change the health check type of your instance's Auto Scaling group from EC2 to ELB by using a configuration file. See [here](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environmentconfig-autoscaling-healthchecktype.html)

* Sticky sessions are a mechanism to route requests to the same target in a target group. This is useful for servers that maintain state information in order to provide a continuous experience to clients.
* Cross zone load balancing: always on for ALB, disabled by default for NLB
* SNI to load multiple SSL certs to servce multple websites on one server on ALB and NLB
* HTTPS listener: to offload the work of encryption and decryption to your load balancer so that your applications can focus on their business logic. Must deploy at least one SSL server certificate on the listener. Application Load Balancer can be used to securely authenticate users for accessing applications via HTTP listener and Cognito User Pools, see [here](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-authenticate-users.html).
* Access Logs: Elastic Load Balancing provides access logs that capture detailed information about requests sent to load balancer
* Lambda: register Lambda functions as targets and configure a listener rule to forward requests to the target group for your Lambda function


## Auto Scaling Groups
* Cross AZ, but regional
* [Scaling policies: target tracking, simple/step, schedule action](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html#as-scaling-types)
* [Cooldowns](https://docs.aws.amazon.com/autoscaling/ec2/userguide/Cooldown.html)

# EBS & EFS

## EBS

* Network drive, can be attached to only on instance, locked to AZ
* EBS has four types: 
    * SSD (can be used as boot volumes): 
      * GP2: 3 IOPS/GB, between a minimum of 100 IOPS (at 33.33 GiB and below) and a maximum of 16,000 IOPS (at 5,334 GiB and above)
      * IOI: high performance, provisioned IOPS (max 64000 Nitro or 32000). The maximum ratio of provisioned IOPS to requested volume size (in GiB) is 50:1 for io1 volumes, and 500:1 for io2 volumes.
    * HDD: 
      * STI: throughput optimized, streaming workload, e.g. Apache Kafka
      * SCI: lowest cost
* Encryption: Encryption by default is a Region-specific setting. If you enable it for a Region, you cannot disable it for individual volumes or snapshots in that Region
  * types:
    1. Data at rest inside the volume
    2. All data moving between the volume and the instance
    3. All snapshots created from the volume
    4. All volumes created from those snapshots
  * A volume restored from an encrypted snapshot, or a copy of an encrypted snapshot, is always encrypted

## Instance Store

* Physically attached disk, very high IOPS (100k ~ millions), can't be increased in size, ***data lost on stop or termination***. e.g. High-performance cache.

## EFS

* Managed NFS, multi AZ
* Usage case: content management, web serving, data sharing, Wordpress

# RDS

## RDS

* Supports Postgres, MySQL, MariaDB, Oracle, Microsoft SQL Server
* Backup automatically, 7 - 35 days retention. Multi-AZ, single region 
* Snapshots triggered manually

### Read replicas: 
* up to **5**
* within AZ, cross AZ or cross region
* ASYNC
* network cost when data goes over AZs, so extra cost if read replicas are in different AZs
* can be setup as multi AZ for disaster recovery
* Multi AZ: SYNC, one DNS name, disaster recovery

### Encryption

* At rest encryption has to be defined at launch time. If master is not encrypted, read replicas can't be encrypted.
    * Whether a snapshot is encrypted is defined by whether the RDS is encrypted or not. ***If a user copies an encrypted snapshot, the copy of the snapshot must also be encrypted.*** If a user copies an encrypted snapshot across Regions, users cannot use the same AWS KMS encryption key for the copy as used for the source snapshot, because KMS keys are Region-specific. Instead, users must specify a KMS key that is valid in the destination Region.  
    * How to encrypt an unencrypted RDS
    * TDE: Microsoft and Oracle
* In-flight encryption: SSL certificates 

### IAM Authentication

* IAM database authentication works with RDS MySQL and PostgresSQL 

## Aurora

* Support MySQL and PostgresSQL 
* From 10G Up to 64TB
* 15 replicas
* ***Infrequent, intermittent or unpredicatable workloads***, pay per second

### HA

* 6 copies over 3 AZ, one instance is master
* Writer endpoint pointed to master, reader endpoint pointed to connection load balancing that connects to all read replicas
* Cross region or global (1 primary region, up to 5 secondary region, up to 16 read replica per secondary region)

## ElastiCache
* Significantly improve latency and throughput for many **read-heavy** application workloads (such as social networking, gaming, media sharing, and Q&A portals) or **compute-intensive** workloads (such as a recommendation engine) 
* Managed memcached and Redis
    * Redis: Multi AZ, Read replicas, Data Durability with AOF, backup and restore
    * memcached: sharding, non persistence, no backup or restore, multi-threaded
* Multi AZ
* [Strategies](https://aws.amazon.com/caching/best-practices/): 
  * Lazy loading: loads data into the cache only when necessary
  * Write through: adds data or updates data in the cache whenever data is written to the database
  * Time-to-live

# S3

[S3 FAQ](https://aws.amazon.com/s3/faqs/)

* Max object size is 5TB, min object size is 0 bytes. Largest upload in a single PUT is 5GB. (Objects larger than 100MB should be uploaded with multipart uploader)
* No limit on number of objects in a bucket
* Versioning needs to be enabled at **bucket** level. Same key overwrite will increase the version. If it's not enabled, file will be versioned "null". 
* CLI: e.g. `aws s3api list-objects`
  * `--page-size`: full dataset is retrieved but each API call will request less data (help avoid timeout)
  * `--max-times` and`--starting-token`: pagination

### Encryption

* Server-Side
  1. SSE-KMS: KMS-Managed Encryption keys. Give user control and audit trail. Must set header `"x-amz-server-side-encryption":"aws:kms"`
  2. SSE-S3: Amazon S3-Managed Encryption keys. Must set header `"x-amz-server-side-encryption":"AES-256"`
  3. SSE-C: Customer-Provided Encryption keys. S3 does not store the key. Must use HTTPS. Must provide key in header.

* Client-Side
  1. AWS KMS-managed customer master key
  2. Client-side master key

### Security

* If user IAM allows or resource policy ALLOWs, an IAM principal can access an S3 object. Unless there's an explicit DENY in IAM policy. `Total Policy = IAM policy + S3 Bucket Policy`
* Bucket policy: JSON based. ALLOW / DENY.
  * To force SSL, create a bucket policy with DENY on `aws:SecureTransport=false`
  * To force SSE-KMS encryption: DENY incorrect encryption header, make sure it includes `asw:kms` and DENY no encryption header
* Bucket settings for blocking public access to prevent data leaks
* Managing access examples:
  * [Bucket owner granting its users bucket permissions](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-walkthroughs-managing-access-example1.html)
  * [Bucket owner granting cross-account bucket permissions](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-walkthroughs-managing-access-example2.html)
  * [Bucket owner granting its users permissions to objects it does not own](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-walkthroughs-managing-access-example3.html)
  * [Bucket owner granting cross-account permission to objects it does not own](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-walkthroughs-managing-access-example4.html)


### Static Websites

* S3 cab host static websites `<bucket-name>.s3-website.<AWS-region>.com`. Neet to make bucket policy allow public reads otherwise will get 403, create a bucket policy that grants everyone the `s3:GetObject` permission.
* CORS (Cross-Origin resource sharing) enables a way for client web applications loaded in one domain to interact with resources in a different domain. If a client does a cross-origin request on S3 bucket, we need to enable correct CORS headers `"AllowOrigins":<First-bucket-name>` on the cross origin bucket.

### Advanced

* Consistency: S3 provides read-after-write consistency for PUTS of new objects. S3 offers eventual consistency for overwrite PUTS and DELETES.
* When versioning is on, S3 MFA-Delete can be enabled. It can only be done by bucket owner via AWS CLI.
* Access logs for auditing. Don't set logging bucket to be the monitored bucket. 
* CRR and SRR (Cross and Same region replication) requires proper IAM permission to S3 for copying data
* Presigned-URL is valid for 3600s, can be changed with `--expires-in` arg.
* Lifecyle rules can be created for a specific prefix
    * Transition actions
    * Expiration actions
* Event notifications can sent in response to actions such as PUTs, POSTs, COPYs or DELETEs, Messages can be sent through SNS, SQS or Lambda. 
  * If two writes are made to a single non-versioned object at the same time, it is possible that only a single event notification will be sent 

#### Storage classes
1. S3 Standard - General Purpose. E.g. big data analytics, mobile & gaming app, content distribution
2. S3 Standard - Infrequent Access - for data that is less frequently accessed but requires rapid access when needed. Availability drops to 99.9% and there is a data retrieval charge of $0.01 / GB. E.g. data store for disaster recovery and backups.
3. S3 One Zone - Infrequent Access - Offers similar performance as other S3 classes but stores data redundantly within an Availability Zone not across Availability Zones. E.g. secondary backup for on-premise data or storing data that can be recreated.
4. S3 Intelligent Tiering - Automatically moves objects between two tiers based on changing access patterns. 
5. Glacier - used for archiving data, each item up to 40TB. Minimum storage duration of ***90 days***. 3 retrieval options:
    * expedited (1 to 5min)
    * standard (3 to 5 hours)
    * bulk (5 to 12 hours)
6. Glacier Deep Archive. Minimum storage duration of ***180 days***.
    * standard (12 hours)
    * bulk (48 hours)

### Performance
* At least 3500 PUT/COPY/POST/DELETE and 5500 GET/HEAD request per second per prefix in a bucket. No limits for the number of prefixes in a bucket. 
* Note S3 KMS limiation
* Multipart upload must be used for file > 5GB, recommended for file > 100MB
* S3 Byte-Range fetches parallize GETs, can be used to speed up downloads
* S3 Select enables applications to retrieve only a subset of data from an object by using simple SQL expressions. 

### Athena
* Serverless service to perform analytics directly against S3 files

# ECS
* ECS is used to run Docker containers and has 3 flavours:
  * Classic: provision EC2 instances to run containers onto.
      * ***Must configure `/etc/ecs/ecs.config` with the cluster name***
      * EC2 instance must run an ECS agent through EC2 instance profile
      * ECS tasks can have IAM roles to execute actions against AWS
  * Fargate: Serverless
  * EKS: managed Kubernetes
* Task definition: can define multiple containers in a task definition. Use IAMTaskRoles for ECS tasks. 
  * Task placement strategies: 
    * binpack: place tasks based on the least available amount of CPU or memory. cost-efficient.
    * random
    * spread: place tasks evenly based on the specific value
   * Task Placement Constraints: a rule that is considered during task placement
   * Cluster query language: expressions that enable you to group objects
* Load balancing: When launching a task, don't specify host port but only the container port, and ALB dynamic forwarding will root the traffic to random port, which allows multiple containers of the same type to launch on the same instance
* Deregistering: If terminate a container instance in the RUNNING state, that container instance is automatically removed from the cluster. However if terminate a container instance in the STOPPED state, that container instance isn't automatically removed from the cluster.
* Port mapping: allow containers to access ports on the host container instance to send or receive traffic
  * container port: bound to the user-specified or automatically assigned host port
  * host port: reserve for your container. set the host port number to **0** and ECS will automatically assign an available port

### ECR
ECR is used to store Docker images

# DynamoDB

[DynamoDB FAQ](https://aws.amazon.com/dynamodb/faqs/)

* NoSQL database, stored on SSD. HA with replication over 3 AZs.
* Optimistic locking powered by conditional update / delete
* Primary Keys: 
  1. Partition Key only (HASH). Partition key must be unique. E.g. user id.
  2. Partition Key + Sort Key. The combination must be unique. Data is grouped by partition key.
* [DynamoDB APIs](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/Welcome.html)
  * Query - a query find items in a table using only the primary key (sort key is optional). Query is more efficient than scan. 
  * Scan - a scan operation examines every item in the table. By default a scan returns all of the data attributes for every item. You can use the  `ProjectionExpression + FilterExpression` parameter so that Scan only returns some of the attributes. 
    * For faster performance,use parallel scans. 
    * Use the Limit parameter on table's throughput so that it doesn’t interfere with more important operations
    * To minimum the impact of scan on provisioned throughput, reduce page size.
* TTL: automatically delete an item after expiry time
* CLI:
  * `--projection-expression`: to get only some of the attributes rather than all of them
  * `--filter-expression`: filter results
  * `--page-size`: full dataset is retrieved but each API call will request less data (help avoid timeout)
  * `--max-times` and`--starting-token`: pagination
* Backup to S3: DynamoDB offers two built-in backup methods, on-demand and point-in-time recovery, but **don't have access to the S3 buckets that are used for these backups**. To create backups that can be downloaded, see [here](https://aws.amazon.com/premiumsupport/knowledge-center/back-up-dynamodb-s3/):
  * Data Pipeline
  * Amazon EMR
  * AWS Glue
  
### Consistency
* Read consistency:
  1. Eventual consistent reads (default). Offers best read performance. Consistency across all copies of data is usually reached within a second.
  2. Strongly consistent reads. Returns a result that reflects all writes that received a successful response prior to the read. By setting `ConsistentRead = true`.
* Transactions: use transactional read and write APIs to manage to achieve ACID

### Provisioned Throughput
* DynamoDB tables must be have provisioned RCUs and WCUs. They are specific to one table. If you exceed your provisioned throughput you will get a HTTP status code 400, `ProvisionedThroughputExceededException`.
* WCU (Write Capacity Units): one write per second for an item up to 1KB. If an item is bigger, more WCUs are consumed.
* RCUs: one strongly consistent read per second, or two eventually consistent reads per second, for an item up to 4KB.
* [HA resolutions](https://aws.amazon.com/premiumsupport/knowledge-center/dynamodb-high-latency/):
  * DAX: cache, solves the hot key problems (too many reads)
  * Global tables: reduce the distance between the client and the DynamoDB endpoint
  * Use eventually consistent reads
  * Reduce the request timeout settings: `requestTimeOut` and `clientExecutionTimeout`
  * Send constant traffic or reuse connections

### Indexes
* Local Secondary Index: same partition key but different sort key (local to the hash key). Must be defined when creating a table.
* Global Secondary Index: Different partition key and different sort key (whole new different table). Can be created at table creation or added later. If writes are throttled on GSI, the main table will be throttled.  

### Streams
* Changes in DynamoDB (CREATE, UPDATE, DELETE). Max 24 hour retention. Can be read by EC2 or Lambda. Could be used to implement cross-region replication.
* Four options for streams only 1 can be selected
  1. Keys Only - Only the key attributes of the modified item
  2. New image the entire item, as it appears after it was modified
  3. Old image - the entire item, as it appeared before it was modified
  4. New and old images - both the new and the old images of the item
* Records are not retroactively populated in a stream after enabling it

# Messaging

## Simple Queue Service SQS

[SQS FAQ](https://aws.amazon.com/sqs/faqs/)

[SQS tutorial](https://aws.amazon.com/getting-started/tutorials/send-messages-distributed-applications/)
* Scales automatically. Low latency.
* Queueing works as a buffer to decouple of components of an application. 
* Unlimited number of messages in the queue. The retention period can be configred to be anywhere betweeen **1 minute and 14 days**. The default is 4 days. Once the message retention limit is reached, your messages are automatically deleted.
* The max number of messages to retrieve at one time is **10**
* SQS messages must be between 1 and ***256 KB*** in size. To send big message by using SQS Extended Client: send large message to S3 and small metadata message to the queue. 
* Can have duplicated messages (at least once delivery)
* Can have out-of-order messages (best offer ordering)
* Consumer: Poll SQS for messages. When a consumer receives a message from the SQS queue, it stays in the SQS queue. The message must be deleted by the consumer via DeleteMessageAPI once the message has been fully processed. ASG can be used for scaling consumers horizontally.
* Message Visibility Timeout: To prevent other conumers from receiving the message, SQS sets a Visibility Timeout, which is the period of time when SQS prevents other consumers from receiving and processing the message by making the message invisible to other consumers. By default it's ***30s***. Can be set via `ChangeMessageVisibility`.
* Long polling: When a consumer polls messages from a queue but it is empty, it can wait ***1-20s*** for messages to arrive to descrease the number of API calls and latency. It is preferable to short polling. Can be enabled at queue level of using `WaitTimeSeconds`

### Types
* Dead Letter Queue: Can set a threshold of how many times a message can go back to the queue. After the `MaximumReceives` threshold is exceeded, the message goes into a dead letter queue.
* Delay Queue: delay a message up to **15mins**, default is 0s.Can override default value on sending by using `DelaySeconds` param.
* FIFO queue: exactly-once send capability, maintain ordering. Limited throughput: 300 ms/s without batch, 3000 with.
  * Deduplication interval is 5mins. Two methods: content-based or explicitly provide a `MessageDeduplicationID`.
  * **MessageGroupID**: Each Group ID have a different consumer. Messages in the same group sharing the same `MessageGroupID` will be in order, but across-group ordering is not guaranteed. 
  * **MessageDeduplicationId**: If a message with a particular message deduplication ID is sent successfully, any messages sent with the same message deduplication ID are accepted successfully but aren't delivered during the 5-minute deduplication interval.

## Simple Notification Service (SNS)

[SNS FAQ](https://aws.amazon.com/sns/faqs/)

[SNS tutorial](https://aws.amazon.com/getting-started/tutorials/filter-messages-published-to-topics/)
After a message has been published to a topic it cant be deleted (recalled)

* SNS is a messaging service that 'pushes' messages to clients. Up to 10 million subscriptions per topic, up to 100,000 topics.
* Messages subscribers:
  * Application
  * SMS text message
  * Email
  * Email-JSON
  * AWS SQS
  * HTTP
  * HTTPS
* Fan Out: SNS can be used with SQS to fan messages out to multiple queues. SQS allows for data persistance, delayed processing and retries. Make sure SQS access policy allows SNS to write. Can't be FIFO queue.

## Kinesis
* A managed alternative to Apache Kafka. 
  * Kinesis Streams: low latency streaming ingest at scale
  * Kinesis Analytics: real-time analysis on streams using SQL
  * Kinesis Firehose: load streams into S3, Redshift, ElasticSearch, ...
* Streams are devided into shards. ***Records are ordered per shard***. 1MB/s.
* Data retention is 1 day by default, up to **7 days**
* Data can't deleted once inserted into Kinesis
* Put Records: PutRecordAPI + Partition key that gets hashed, same key goes to the same partition
* ProvisionedThroughputExceededExceptions: sending too much data for one shard. Solution:
    * Retries with backoff
    * Increas shards (scaling)
    * Ensure partition key is a good one
* KCL(Kinesis Client Library): Each KCL consumer application instance uses "workers" to process data in Kinesis shards. At any given time, __each shard of data records is bound to a particular worker__ via a lease.
  
## Simple Workflow Service (SWF)

[SWF FAQ](https://aws.amazon.com/swf/faqs/)

* Workers are programs that interact with SWF to get tasks, process received tasks and return the results.
* Decider is a program that controls the coordination of tasks.
* Domains - workflow and activity types and the workflow execution itself are all scoped to a domain. Domains isolate a set of types, executions, and task lists from other within the same account. You can register a domain by using the console or SWF API. Using JSON.
* Tasks assigned only once and never duplicated.

### SWF vs SQS

* SWF presents task oriented API whereas SQS offers message oriented API.
* SWF ensures that a task is assigned only once. With SQS you need to handle duplicated messages and may also need to ensure that a message is processed only once.
* SWF keeps track of all tasks and events in an application. With SQS you need to implement your own application-level tracking.


# Elastic Beanstalk

[Elastic Beanstalk FAQ](https://aws.amazon.com/elasticbeanstalk/faqs/)

* Three components: Application, Application version and environment name
* Deployment Mode: Single instance, HA with LB
* Deployment options for updates: 
    * All at once: fastest, but with downtime
    * rolling: update a few instances at a time, and move onto the next bucket when the previous one is healthy
    * rolling with additional batches: spin up new instances to move the batch
    * immutable: spin up new instances in a new ASG, deploy new version to it, and swap all the instances when everything is healthy. longest deployment, rollback quickly.
* Can have at most 1000 versions. Lifecycle policy: based on time (old versions are removed) or space 
* Cloning with exactly the same configuration, good for testing. **Migration** cross-account: Create a saved configuration in Team A's account and download it to local machine. Make the account-specific parameter changes and upload to the S3 bucket in Team B's account. From Elastic Beanstalk console, create an application from the saved Configurations.
* Extensions: Any resources created as part of `.ebextensions` is part of EB template and will get deleted if the environment is terminated
* Platforms: see [here](https://docs.aws.amazon.com/elasticbeanstalk/latest/platforms/platforms-supported.html)
  * Docker: Can run the application as a single docker, doesn't use ECS; ECS can run multiple dockers per EC2 instance in EB, requires `Dockerrun.aws.json(v2)` at the root of source code to generate the ECS task definition 
  * MultiDocker
* Environments: Can create and manage separate environments for development, testing, and production use
  * Worker environment: define a cron.yaml file and offload long-to-complete tasks to a dedicated worker environment

### Languages

* Apache Tomcat for Java
* Apache HTTP server for PHP
* Apache HTTP Server for Python
* Nginx or Apache HTTP for Node.js
* Passenger or Puma for Ruby
* Microsoft IIS 7.5, 8.0 and 8.5 for .NET
* Java SE
* Docker
* Go

# CI/CD

### CodeCommit
* Notifications: 
  * SNS/Lambda E.g. Deletion of branches, trigger for push in master, notify external build system, trigger Lambda function for code analysis
  * CloudWatch events. E.g. trigger for pull requests, commit comment events, event rules go to SNS topic
* Encryption at rest
* Security. IAM supports CodeCommit with three types of credentials:
  * Git credentials:  an IAM -generated user name and password pair you can use to communicate with CodeCommit repositories over HTTPS.
  * SSH keys: a locally generated public-private key pair that you can associate with your IAM user to communicate with CodeCommit repositories over SSH.
  * AWS access keys: which you can use with the credential helper included with the AWS CLI to communicate with CodeCommit repositories over HTTPS.

### CodePipeline
Troubleshooting: 
* state changes happen in CloudWatch events, which can in return create SNS notifications
* If it fails at a stage, there is info in console
* CloudTrail can be used to audit API calls
* If pipleline can't perform an action, make sure the attached IAM Service Role has correct permission

### CodeBuild
* fully managed, scale automatically
* `buildspec.yaml` must be at the root of the source code
* Environment variables can be plaintext or secure store
* Phases: Install -> Pre build -> Build -> Post builds
* Artifacts uploaded to S3. Can cache files to S3 to increase performance for future builds
* Specify VPC configuration (VPC ID, Subnet ID, Security Group ID) so build can access resources in VPC
* CodeBuild Agent: can use the AWS CodeBuild agent to test and debug builds on a local machine.
* Timeout: the build process will automatically terminate post the expiry of the configured timeout

### CodeDeploy
* `appspec.yml` file in the root directory to map resources and specify permissions
* Deployment groups: in an EC2/On-Premises deployment, a deployment group is a set of individual instances targeted for deployment. 
* Deployment: 
  * In-place deployment: The application on each instance in the deployment group is stopped, the latest application revision is installed, and the new version of the application is started and validated
  * Blue/green deployment: Povision a new set of instances to install the latest version of the application. CodeDeploy then re-routes load balancer traffic from an existing set of instances running the previous version of the application to the new set of instances running the latest version and then terminate the old instances
* AppSpec:
  * File section
  * Hooks: set of instructions to manage deployment
    * EC2: ApplicationStop > DownloadBundle > BeforeInstall > AfterInstall > ApplicationStart > **ValidateService***
    * Lambda: BeforeAllowTraffic -> AfterAllowTraffic
    * ECS: BeforeInstall > AfterInstall > AfterAllowTestTraffic > BeforeAllowTraffic > AfterAllowTraffic
      
* Deployment:
  * In-place Deployment: The application on each instance in the deployment group is stopped, the latest application revision is installed, and the new version of the application is started and validated. Only for EC2/On-Premises.
  * Blue/Green deployment: CodeDeploy provisions your new application version alongside the old version before rerouting your production traffic.
    * AWS Lambda: Traffic is shifted from one version of a Lambda function to a new version of the same Lambda function.
    * Amazon ECS: Traffic is shifted from a task set in your Amazon ECS service to an updated, replacement task set in the same Amazon ECS service.
    * EC2/On-Premises: Traffic is shifted from one set of instances in the original environment to a replacement set of instances.
* [Rollback](https://docs.aws.amazon.com/codedeploy/latest/userguide/deployments-rollback-and-redeploy.html): roll back deployments by redeploying a previously deployed revision of an application as a new deployment on the failed instances

# CloudFormation

[CloudFormation FAQ](https://aws.amazon.com/cloudformation/faqs/)

Infrastructure as code. 

* A CloudFormation is made up of the following sections:
    * __Resources__ (required) - specify the stack resources and their properties such as an EC2 instance or a S3 bucket. You can refer to resources in the Resources and Outputs sections of the template. Resource identifier form:`AWS::aws-product-name::data-type-name`. !GetAtt returns the value of an attribute from a resource in the template e.g. `!GetAtt logicalNameOfResource.attributeName`
    * __Parameters__ (optional) - specifies values that you can pass in to your template at runtime (when you create or update a stack). You can refer to parameters anywhere in the template by leveraging `Fn::Ref`, shorthands is `!Ref` . Useful if resource config is likely to change in the future. Supported types:
        ```
        String – A literal string
        Number – An integer or float
        List<Number> – An array of integers or floats
        CommaDelimitedList – An array of literal strings that are separated by commas
        AWS::EC2::KeyPair::KeyName – An Amazon EC2 key pair name
        AWS::EC2::SecurityGroup::Id – A security group ID
        AWS::EC2::Subnet::Id – A subnet ID
        AWS::EC2::VPC::Id – A VPC ID
        List<AWS::EC2::VPC::Id> – An array of VPC IDs
        List<AWS::EC2::SecurityGroup::Id> – An array of security group IDs
        List<AWS::EC2::Subnet::Id> – An array of subnet IDs
        ```
    * __Mappings__ (optional) - hardcoded. a mapping of keys and associated values that you can use to specify conditional parameter values, similar to a lookup table. You can match a key to a corresponding value by using the `Fn::FindInMap`, shorthands `!FindInMap [MapName, TopLevelKey, SecondLevelKey]` in the Resources and Outputs section.
    * __Outputs__ (optional) - describes the values that are returned whenever you view your stack’s properties. For example, you can declare an output for an S3 bucket name and then call the Cloudformation describe-stacks AWS CLI command to view the name. A stack can't be deleted if its outputs is referenced by another stack. To create a cross-stack reference, use the `Export` output field to flag the value of a resource output for export. Then, use `Fn::ImportValue`to import the value. 
    * __Conditions__ (optional) - defines conditions that control whether certain resources are created or whether certain resource properties are assigned a value during stack creation or update. For example, you could conditionally create a resource that depends on whether the stack is for a production or test environment.
    * __Metadata__ (optional) - objects that provide additional information about the template.
* To create a cross-stack reference, use the **Export** output field to flag the value of a resource output for export. Then, use the `Fn::ImportValue` intrinsic function to import the value. For each AWS account, Export names must be unique within a region.
* Stack creation rollback (get deleted) on error is enabled by default. Stack update rollback will reset the stack to the previous state.
* Change sets: allow you to preview how proposed changes to a stack might impact your running resources
* StackSets: enable to create, update, or delete stacks across **multiple accounts and regions** with a single operation
* Use function Fn:GetAtt to output data
* CLI commands: 
  * `cloudformation package`: packages the local artifacts (local paths) that your AWS CloudFormation template references. The command will upload local artifacts, such as your source code for your AWS Lambda function.
  * `cloudformation deploy`: command deploys the specified AWS CloudFormation template by creating and then executing a changeset

# Monitoring

## CloudWatch
* Metrics: a variable to monitor, belong to namespaces. EC2 details monitoring get metrics every 1 minute instead of 5. Customer high resolution metrics standard is 1 min, can be up to 1 second. To create custom metrics, use API PutMetricData.
  * Namespace: is a container for CloudWatch metrics. Metrics in different namespaces are isolated from each other
* Alarms: used to trigger notification for metrics. High resolution metric alarm can be trigger at 10 or 30 s.
* Logs: can go to batch exporter to **S3** for archival or stream to **ElasticSearch** for analysis. Never expire by default, but can define expiration policy at log groups level. Make sure IAM permission correct to send logs to CloudWatch. 
  * EC2 needs to install a CloudWatch agent to push logs. 
  * Logs can use filter expressions. Filters only publish metrics data for events after the filter is created.
* Events: scheduled cron jobs.

### EventBridge
* Next evolution of CloudWatch Events. Can analyze events in buses and infer the schema via SchemaRegistry.

## X-Ray
* Visual analysis of application
* Concepts:
  * Segment: break down the data about the work done into subsegments. 
  * Subsegments provide more granular timing information and details about downstream calls.
  * Annotations: for adding system or user-defined data， used for indexing traces and use with filters
  * Metadata: not indexed, additional data
* Leverages tracing, end-to-end way of following a request
* Application needs to import the SDK and the instance needs to install the X-Ray daemon or need to enable X-Ray integration.
   * To make X-Ray work on EC2, ensure IAM role is correct and the daemon is running
   * To make X-Ray work on Lambda, ensure it has IAM execution role with proper policy (AWSX-RayWriteOnlyAccess) and X-Ray SDK is imported in the code
* Sampling rule: by default X-Ray records the first request every second (reservoir) and 5% (rate) of any addtional requests. Customer rules can change the reservoir and the rate. By customizing sampling rules, you can control the amount of data that you record.
* Elastic Beanstalk includes X-Ray daemon. Run by setting `.ebextensions/xray-daemon.config`. Make sure instance profile has the right IAM role and applicaiton code imports X-Ray SDK.
* X-Ray on ECS, three patterns: 
  1. As daemon: X-Ray container as a daemon on each instances
  2. As side-car: X-Ray daemon container alongside each application container
  3. Fargate claster: side-car

## CloudTrail
* Provide governance, compliance and audit. Enabled by default
* Organization trail: If you have created an organization in AWS Organizations, you can create a trail that will log all events for all AWS accounts in that organization.

# AWS Shared Responsibility

IaaS - Customer manages OS and above including security and patches. AWS manages hypervisor and below including physical infrastructure.

SaaS – AWS manages everything except user credentials.

# Route-53

[Route53 FAQ](https://aws.amazon.com/route53/faqs/)

* Limit of 50 domain names - speak to AWS support to adjust.
* TTL: DNS records have a TTL (Time to Live) in order for clients to know for how long to caches these values and not overload the DNS with DNS requests.

* CNAME (Canonical Name) can be used to resolve one domain another
* A Record (Address Record) for resolving a domain name to an IP address
* Alias records are an AWS / Route 53 specific term, similar to CNAME with the key distinction that CNAMEs can't be used on the zone apex (root domain i.e. cnames could be used against sofa.furniture.com, but not against furniture.com - for this you'd need to use either an A Record or Alias record)

* In the exam always choose Alias over CNAME. Amazon dont charge to resolve Alias records and they can be used to map naked domain apex to an ELB.

## Routing Policies

* Simple - default policy, used when there is only one resource.
* Weighted - send a specified amount of traffic to certain resources. ie for every 10 requests send 70% to us-east-1 and 30% to eu-west-1.
* Latency - used to send traffic to lowest latency region. This requires you to create an A record for each region you want the latency to be evaluated against.
* Failover - failover allows you to have an active/passive design. Using health checks to assess whether to send traffic to the primary or secondary resource. A health check can use Cloud watch alarms, other health checks or simply use a TCP connection to an IP or domain name.
* Geolocation - used to send traffic to a particular region based on source location. ie Customers in a Eurozone country always get routed to a server with prices in Euros.
* Multivalue - distribute DNS responses across multiple resources.Use when associate routing records with a Route 53 health check

# VPC

[VPC FAQ](https://aws.amazon.com/vpc/faqs/)

* VPC: Region 

* Subnet: AZ
    * public and private
    * ***Route tables***: to define access between internet and subnets
    * By default all traffic between subnets is allowed

* Internet Gateway helps VPC instances connect with the internet. One IGW per VPC. A public subnet always has a route to IGW.

* NAT allows instances in ***private*** subnets to access the internet. Deployed in the public subnet, connected to IGW, then create a route from private subnet to NAT.

* /16 is the largest CIDR block available. You cannot change the ip range of a VPC

* Network ACL: a firewall that controls traffic in/out from a ***subnet*** (DENY/ALLOW) <---> Differ from security groups (at ENI/EC2 level, ALLOW only)

* VPC Flow logs: information about IP traffic going into interfaces. For troubleshooting.

* VPC Peering: connect two VPCs, privately using AWS network, not transitive. The peered VPCs must not have overlapping CIDR.

* **VPC Endpoints**: connect to AWS services from with a VPC using a private network. Gateway for S3 and DynamoDB, ENI for the rest.

* From on-premise to AWS: VPN goes over the public internet, Direct Connect goes over a private network by establishing a physical connection.

# CloudFront

* Content Delivery Network (CDN). Provides content quicker to customers by caching it in edge locations. ie customer 1 watches a video from s3. s3 bucket is in Ireland but user is in Sydney. The content flows Ireland -----> Sydney. CloudFront caches it locally near Sydney so the second time its accessed the content flows, CloudFront Sydney -> Sydney.

* Objects are cached for ther life of their TTL. TTL can be 0 seconds to 365 days. Default is 24 hours.

* Edge locations can be used for write as well as read.

* Origin can be S3 or Customer Origin (HTTP) such as EC2, ALB, Route53 and non AWS server i.e. on-premise
    * S3 origin enhanced security with OAI (Origin Access Identify) + S3 bucket policy

* Security
    * HTTPS
        * Viewer Protocol Policy: redirect HTTP to HTTPS or HTTPS only. Between Viewer (client) and CloudFront
        * Origin Protocol Policy: HTTPS only or Match Viewer. Between CloudFront and origin.
    * Restrict viewer access by signed URL (single file) or Signed Cookies (multiple files) for distributing paid content
    * Restrict content based on geo location (whitelist and blacklist)

# Lambda
* Compute service allows you to run code without provisioning and managing servers. Under the hood are EC2 Instances managed by AWS. Lambda is stateless and event driven. Up to 15mins excecution time.
* Lambda needs correct Excecution Roles (IAM Roles) to access AWS services. When using event source mapping, Lambda uses the execution role to read event data. An IAM principal can access Lambda if the IAM policy attached to the principal authorizes it OR if the resource-based policy authorizes.
* Lambda environmental variables: helpful to store secrets. Lambda always encrypts environment variables at rest. To customize how environment variables are encrypted:
   * Key configuration
   * Encryption helpers: encrypt environment variable values client side, before sending them to Lambda.
* Loggin and Monitoring:
  * CloudWatch Logs: make sure Lambda has an execution role with IAM policy that authorizes writes to CloudWatch logs
  * X-Ray: Enable `Active Tracing` in config. Make sure Lambda has an execution role with correct IAM policy and env var to communicate with X-Ray
* External dependencies need to be install with the code and zip together. If zip file is less than 50M then directly to Lambda, else to S3
* Alias can be use to manage different versions for Lambda. They are mutable.
* **How to configure a Lambda function to assume a role from another AWS account** See [here](#security-token-service-sts)

### Performance
* RAM: from 128M to 3008M in 64M increment. The more RAM added, the more vCPU credits to get. If application is CPU-bound (computational heave), add more RAM.
* Timeout: default 3s, max is 15min.
* Execution context: temporary runtime environment that initializes dependencies for Lambda. **So intilization code should be outside the function handler and reuse it across executions.** It includes `/tmp` directory that can be used to write heavy files, max 512M. For permanent file storage, use S3 or even DynamoDB.

### Concurrency
* Up to 1000 concurrent excecutions per account. Set a **reserved concurrency** to limit this number. Each invocation over the concurrency will trigger a throttle.
* Set **provisioned concurrency** to enable your function to scale without fluctuations in latency
* If one application invokes too many concurrent Lambda functions, it may throttle other applications' Lambda functions.
* Asychronous Invocations: if Lambda doesn't have enough concurrency available, additional requests are throttled, events are returned to the internal event queue.
* Cold start: when lauching a new instance, the initialization stage when code is loaded and code outside the handler run. First request has longer latency than the rest. Provisioned concurrency can be allocated before the function is invoked.
* Can configure Application Auto Scaling to manage provisioned concurrency on a schedule or based on utilization

### Invocations
* Three ways of processing events:
  * Synchronouse Invocation: result is returned right away, error handling happens at client side `--invocation-type RequestResponse`
    * With ALB: register Lambda function in a target group. Conversion between ALB <-> Lambda: HTTP <-> JSON. When enabling ALB multi-header values, HTTP headers and query parameters that are sent with multiple values are show as arrays within Lambda event and response objects.
    * With Lambda@Edge: deploy Lambda alongside CloudFront CDN. Use case: website security and privacy, SEO, etc
  * Asynchronouse Invocation: Events are placed in an event queue. Lambda retries after error, 3 retries in total. Make sure the process is idempotent. Can define a DLQ for failed processing. `--invocation-type Event`
    * CloudWatch events: CRON / Rate EventBridge Rule (Trigger periodlically) OR CodePipeline / EventBridge Rule (Trigger on state change)
    * S3 Event notification
    * Can define destination for successful or failed events
  * Event Source Mapping: records need to be polled from the source, e.g Kinesis, SQS, DynamoDB. Lambda function in invoked synchronously.
    * Streams: Kinesis or DynamoDB. Streams for event source mapping. Up to 10 batches per shard.
    * Queues: SQS & SQS FIFO. Long polling. 
    * Can define destination for discarded event batches

### In VPC
* By default Lambda is launched outside your own VPC. To access your own VPC, Lambda will create an ENI in the subnets 
* Deploy Lambda in a public subnet doesn't give it internet access. Deploy Lambda in private subnet with NAT gives it internet access or use VPC endpoints to access AWS services. 

### Limits
* Exceutions:
  * Memory 128M to 3008M in 64M increment
  * Max execution time: 15min
  * Environmental Variables: 4K
  * Disk capacity in `/tmp`: 512M
  * Concurrency: up to 1000
* Deployment: 
  * compressed.zip size: 50M
  * uncompressed (code + dependencies): 250M
  * Can use /tmp directory to load other files at startup
  * Environmental Variables: 4K
  
  
# API Gateway
* Must deploy it to make it callable by your users
* CORS can be enabled at API Gateway
* Endpoint types: 
  * Edge-optimized: for global clients
  * Regional
  * Private
* Stage variables: like environmental variables for API stages. Use them to manage often changing config values. They are passed to context object in Lambda. E.g. stage variables to indicate Lambda alias.
* Canary depployment: choose % of the traffic the canary channel receives
* Integration types:
  * MOCK: return a response without sending request to backend
  * HTTP/AWS: must configure both integration request and response. Setup data mapping using mapping templates.
    * Mapping templates: can be used to modify request/response. Like query strings, body content, headers. E.g. JSON to XML with SOAP
  * AWS_PROXY: incoming request from the client is the input to Lambda. No mapping templates, header, ... they are passed as arguments
  * HTTP_PROXY: HTTP request is passed to backend, no mapping template
* Caching: default TTL 300s. 0 - 3600s. Can be defined per stage. To invalidate cache, client sends header `Cache-Control: max-age=0` with proper IAM authorization
* CloudWatch metrics: 
  * `4XXError` and `5XXError`: client-side  and server-side errors
  * `CacheHitCount` and `CacheMissCount`: efficiency of the cache
  * `Count`: total number of API requests
  * `IntegrationLatency`: between API Gateway relays a request to the backend and backend response  
  * `Latency`: between API Gateway receives request from client to it returns response to client 
* Limits:
  * Max time to perform request: 29s
  * Throttles at 10000 request/s, returns 429 error. Can set Stage Limit & Method Limit to improve performance
* Security:
  * IAM Permission: Sig V4 header. Authentication = IAM | Authorization = IAM Policy. Resource Policies can be used for cross-account access (combined with IAM policy). 
    * How: Create a resource policy for the APIs that allows access for each IAM user. Create an IAM permission policy and attach it to each IAM user. Set the APIs method authorization type to AWS_IAM. Use Signature Version 4 to sign the API requests.
  * Cognito user tools: pass token. Cognito user tools. Authentication = Cognito User Pools | Authorization = API Gateway Methods
  * Lambda authorizer: request parameter-based token authorizer. Authentication = External | Authorization = Lambda function. 
    * A Lambda authorizer is useful if you want to implement a **custom authorization scheme** that uses a bearer token authentication strategy such as OAuth or SAML, or that uses request parameters to determine the caller's identity.
* API Gateway Usage Plans: specify who can access one or more deployed API stages and methods—and also how much and how fast they can access them 

# SAM
* Serverless Application model, generate CloudFormation
* Transform header indicates it's SAM: `Transform: 'AWS::Severless-2017-01-01'`
* Package and Deploy: 
  * `aws cloudformation package / sam package`
  * `aws cloudformation deploy / sam deploy`
* Policy Templates: S3ReadPolicy, SQSPollerPolicy, DynamoDBCrudPolicy, etc
