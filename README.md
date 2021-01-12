# AWS Certified Developer - Associate Study notes

These are my study notes for the AWS Certified Developer - Associate certification. I have already passed the Solutions Architect - Associate exam so the notes might not cover topics if i feel i already know it well enough.

Table of Contents
=================

* [AWS Services 10,000 foot overview](#AWS-Services)

* [IAM](#iam)

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

[Exam Blueprint](http://awstrainingandcertification.s3.amazonaws.com/production/AWS_certified_developer_associate_blueprint.pdf)

# AWS-Services

EC2 - Elastic Compute Cloud virtual machines

Lightsail - Provisiong service - Very hands off

Elastic Container Service - running containers such as docker at scale

Lambda - Serverless functions

Elastic Beanstalk - Easier route for developers to get up and running with their cloud

ElastiCache - Cache common searches in front of DB servers

S3 - Key pair object storage kept in buckets

EFS - NFS can be mounted on multiple instances

Glacier - Archival storage

Snowball - Hardware appliance to transfer data between on-prem and AWS

Storage gateway - Virtual appliances that live on-prem and replicate to AWS

RDS - MySQL, MSSQL, Aurora, PostGreSQL

DynamoDB - NoSQL

RedShitft - Data warehousing

AWS Migration Hub - Dashboard that lets you track your application migration

Application Discovery Service - tracks your applications dependencies

Database Migration Service - Migrate DBs to AWS

Server Migration Service

VPC Virtual Private Cloud

Cloudfront - Content Delivery Network (CDN) caches content to make it available quicker to the end user.

Route53 - Amazon's DNS service

API Gateway - Creating API's for your own services

Direct Connect - Network peering between yourself and AWS

Codestar - Project managing code. Collorbation tool

Codecommit - Source control service

Codebuild - Complies and test your code

Codedeploy - Automates your application deployment

Codepiple - CDS

X-Ray - Used to debug your serverless application

Cloud9 - Online IDE

CloudWatch - Monitoring

CloudFormation - Infrastructure as Code

CloudTrail - API logging

Config - Monitor AWS account config

OpsWorks - Config management using Chef or Puppet

Service Catalog - Managing IT services approved for use

Systems Manager - Patch maintenance
Trusted Advisor - Gives advice on security, cost

Elastic Transcoder - Video transcoding. Sizing videos for various devices


Lex - Powers Alexa


Polly - Text to speech

Rekognition - Analyse images and video

Amazon translate - Language translate

Amazon Transcribe - Automatic speech regonition

Athena - run SQL queries against S3 buckets

Elastic Map Reduce - managed software framework used to process large data sets in a distributed computing environment. Used for data analysis, web indexing, data warehousing, machine learning, financial analysis, scientific simulation etc. EMR supports workloads based on Hadoop, Apache Spark, Presto and Apache HBase.

CloudSearch

ElasticSearch Service

Kinesis - Ingesting large amounts of data

Kinesis Video Streams - Ingesting lots of video streams

Data Pipeline - Moving data between AWS services

IAM - Identity and Management access

Cognito - Mobile Device authentication using federated accounts Facebook etc

Guard Duty -

Inspector - Anaylse instance security using agent

Macie - Scans S3 buskets for personal iD numbers

Certificate manager - Free SSL certs

CloudHSM - Hardware security module which store keys

Directory services - Connect AWS to onside AD

WAF Web application firewall - L7 firewall

Shield - DDOS mitigation

Artifcat - AWS compliance reports

SNS - Simple Notification Service

SQS - Simple Queue Service -  is a web service that gives you access to message queues that store messages waiting to be processed. With Amazon SQS, you can quickly build message queuing applications that can run on any computer. Amazon SQS can help you build a distributed application with decoupled components, working closely with the Amazon Elastic Compute Cloud (Amazon EC2) and other AWS infrastructure web services.

SWF - Simple Workflow Service

Simple Email Service - Sending emails to customers

WorkMail - Office365

Workspaces - VDI

# IAM

[IAM FAQ](https://aws.amazon.com/iam/faqs/)

Users
Users have the choice of being given access to the management console and/or programmatic access. Access via the management console enables a password for the account. Enabling programmatic access enables an access key ID and secret access key. This can be used to access t

### Groups

Groups allow you to apply policies to multiple users. Recommended to apply policies to groups even if it is for one user.

* Users can be members of multiple groups
* Groups cannot be nested

Policies
Policies are JSON documents that contain permissions to AWS services. ie

Roles

Secret

## Security Token Service (STS)

* Grants users limited and temporary access to AWS resources.
* 3 sources:
    1. Federation (often Active Directory)
        * Uses SAML
        * SSO allows users to log in to AWS Console without assigning IAM credentials
    2. Federation with mobile app
        * Use Facebook/Amazon/Google or other openID provider
    3. Cross account access
        * Lets users from one AWS account access resources in another

### Active Directory Federation

# EC2

[EC2 FAQ](https://aws.amazon.com/ec2/faqs/)

Access instance meta data at http://169.254.169.254/latest/meta-data/

* Scipts can be run from the user data section when creating an instance

## Load Balancers

* There are 3 types of load balancer; Application, Network and Classic. Application is used to route HTTP/HTTPS (L7) traffic. Network and Classic are used to route TCP (L4) traffic.

    1. Application - Layer 7, target groups (based on path, hostname, query string), TLS termination 
          1. Target Groups: EC2, ECS, Lambda, IP
          2. True client IP address is in X-Forwarded-For
    2. Network - Extermeme performance and static IP
    3. Classic (also refered to as Elastic Load Balancer ELB) - Layer 4 & 7, Fixed hostname

* Sticky sessions are a mechanism to route requests to the same target in a target group. This is useful for servers that maintain state information in order to provide a continuous experience to clients.
* Cross zone load balancing: always on for ALB, disabled by default for NLB
* SNI to load multiple SSL certs to servce multple websites on one server on ALB and NLB

## Auto Scaling Groups

* [Scaling policies: target tracking, simple/step, schedule action](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html#as-scaling-types)
* [Cooldowns](https://docs.aws.amazon.com/autoscaling/ec2/userguide/Cooldown.html)

# EBS & EFS

## EBS

* Network drive, can be attached to only on instance, locked to AZ
* EBS has four types: 
    * SSD (can be used as boot volumes): 
      * GP2: 3 IOPS/GB, max IOPS is 16000
      * IOI: high performance, provisioned IOPS (max 64000 Nitro or 32000)
    * HDD: 
      * STI: throughput optimized, streaming workload, e.g. Apache Kafka
      * SCI: lowest cost
      
## Instance Store

* Physically attached disk, very high IOPS (100k ~ millions), can't be increased in size, ***data lost on stop or termination***. e.g. High-performance cache.

## EFS

* Managed NFS, multi AZ
* Usage case: content management, web serving, data sharing, Wordpress

# RDS

## RDS

* Supports Postgres, MySQL, MariaDB, Oracle, Microsoft SQL Server
* Backup automatically, 7 - 35 days retention
* Snapshots triggered manually

### Read replicas: 
* up to 5
* within AZ, cross AZ or cross region
* ASYNC
* network cost when data goes over AZs, so extra cost if read replicas are in different AZs
* can be setup as multi AZ for disaster recovery
* Multi AZ: SYNC, one DNS name, disaster recovery

### Encryption

* At rest encryption has to be defined at launch time. If master is not encrypted, read replicas can't be encrypted.
    * Whether a snapshot is encrypted is defined by whether the RDS is encrypted or not
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

* Managed memcached and Redis
    * Redis: Multi AZ, Read replicas, Data Durability with AOF, backup and restore
    * memcached: sharding, non persistence, no backup or restore, multi-threaded
* Multi AZ
* [Strategies](https://aws.amazon.com/caching/best-practices/): Lazy loading, Write through, Time-to-live, Evictions

# S3

[S3 FAQ](https://aws.amazon.com/s3/faqs/)

* Max object size is 5TB, min object size is 0 bytes. Largest upload in a single PUT is 5GB. (Objects larger than 100MB should be uploaded with multipart uploader)
* No limit on number of objects in a bucket
* Versioning needs to be enabled at bucket level. Same key overwrite will increase the version. If it's not enabled, file will be versioned "null". 

### Encryption

* Server-Side
  1. SSE-KMS: KMS-Managed Encryption keys. Give user control and audit trail. Must set header `"x-amz-server-side-encryption":"aws:kms"`
  2. SSE-S3: Amazon S3-Managed Encryption keys. Must set header `"x-amz-server-side-encryption":"AES-256"`
  3. SSE-C: Customer-Provided Encryption keys. S3 does not store the key. Must use HTTPS. Must provide key in header.

* Client-Side
  1. AWS KMS-managed customer master key
  2. Client-side master key

### Security

* If user IAM allows or resource policy ALLOWs, an IAM principal can access an S3 object. Unless there's an explicit DENY in IAM policy.
* Bucket policy: JSON based. ALLOW / DENY.
* Bucket settings for blocking public access to prevent data leaks

### Websites

* S3 cab host static websites `<bucket-name>.s3-website.<AWS-region>.com`. Neet to make bucket policy allow public reads otherwise will get 403.
* CORS (Cross-Origin resource sharing) enables a way for client web applications loaded in one domain to interact with resources in a different domain. If a client does a cross-origin request on S3 bucket, we need to enable correct CORS headers `"AllowOrigins":<First-bucket-name>` on the cross origin bucket.

### Consistency Model

* S3 provides read-after-write consistency for PUTS of new objects. S3 offers eventual consistency for overwrite PUTS and DELETES.

### Advanced

* When versioning is on, S3 MFA-Delete can be enabled. It can only be done by bucket owner via AWS CLI.
* Access logs for auditing
* CRR and SRR (Cross and Same region replication) requires proper IAM permission to S3 for copying data
* Presigned-URL is valid for 3600s, can be changed with `--expires-in` arg.
* Lifecyle rules can be created for a specific prefix
    * Transition actions
    * Expiration actions

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

### Event Notifications
* Event notifications can sent in response to actions such as PUTs, POSTs, COPYs or DELETEs, Messages can be sent through SNS, SQS or Lambda.

### Athena
* Serverless service to perform analytics directly against S3 files

# ECS

ECS is used to run Docker containers and has 3 flavours:
  * Classic: provision EC2 instances to run containers onto.
      * Must configure `/etc/ecs/ecs.config` with the cluster name
      * EC2 instance must run an ECS agent through EC2 instance profile
      * ECS tasks can have IAM roles to execute actions against AWS
  * Fargate: Serverless
  * EKS: managed Kubernetes
  
Use IAMTaskRoles for ECS tasks. Task management strategies: 
   * binpack: place tasks based on the least available amount of CPU or memory. cost-efficient.
   * random
   * spread: place tasks evenly based on the specific value
   
Load balancing: When launching a task, don't specify host port but only the container port, and ALB dynamic forwarding will root the traffic to random port, which allows multiple containers of the same type to launch on the same instance
  
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
  * Scan - a scan operation examines every item in the table. By default a scan returns all of the data attributes for every item. You can use the  `ProjectionExpression + FilterExpression` parameter so that Scan only returns some of the attributes. For faster performance,use parallel scans.
* TTL: automatically delete an item after expiry time
* CLI:
  * `--projection-expression`: attributes to retrieve
  * `--filter-expression`: filter results
  * `--page-size`: full dataset is retrieved but each API call will request less data (help avoid timeout)
  * `--max-times` and`--starting-token`: pagination

### Provisioned Throughput
* DynamoDB tables must be have provisioned RCUs and WCUs. If you exceed your provisioned throughput you will get a HTTP status code 400, `ProvisionedThroughputExceededException`.
* WCU (Write Capacity Units): one write per second for an item up to 1KB. If an item is bigger, more WCUs are consumed.
* Reads: 
  * Read consistency:
    1. Eventual consistent reads (default). Offers best read performance. Consistency across all copies of data is usually reached within a second.
    2. Strongly consistent reads. Returns a result that reflects all writes that received a successful response prior to the read.
  * RCUs: one strongly consistent read per second, or two eventually consistent reads per second, for an item up to 4KB.
* DAX: cache, solves the hot key problems (too many reads)

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

* Queueing works as a buffer to decouple of components of an application. 
* Unlimited number of messages in the queue. The maximum amount of time that a message can live in a SQS queue is 14 days. The retention period can be configred to be anywhere betweeen 1 minute and 14 days. The default is 4 days. Once the message retention limit is reached, your messages are automatically deleted.
* Low latency
* SQS messages must be between 1 and 256 KB in size. To send big message by using SQS Extended Client: send large message to S3 and small metadata message to the queue. 
* Can have duplicated messages (at least once delivery)
* Can have out-of-order messages (best offer ordering)
* Consumer: Poll SQS for messages. When a consumer receives a message from the SQS queue, it stays in the SQS queue. The message must be deleted by the consumer via DeleteMessageAPI once the message has been fully processed. ASG can be used for scaling consumers horizontally.
* Message Visibility Timeout: To prevent other conumers from receiving the message, SQS sets a Visibility Timeout, which is the period of time when SQS prevents other consumers from receiving and processing the message by making the message invisible to other consumers. By default it's 30s. Can be set via `ChangeMessageVisibility`.
* Long polling: When a consumer polls messages from a queue but it is empty, it can wait 1-20s for messages to arrive to descrease the number of API calls and latency. It is preferable to short polling. Can be enabled at queue level of using `WaitTimeSeconds`

### Types
* Dead Letter Queue: Can set a threshold of how many times a message can go back to the queue. After the `MaximumReceives` threshold is exceeded, the message goes into a dead letter queue.
* Delay Queue: delay a message up to 15mins, default is 0s.Can override default value on sending by using `DelaySeconds` param.
* FIFO queue: exactly-once send capability, maintain ordering. Limited throughput: 300 ms/s without batch, 3000 with.
  * Deduplication interval is 5mins. Two methods: content-based or explicitly provide a `MessageDeduplicationID`.
  * MessageGroupID: Each Group ID have a different consumer. Messages in the same group sharing the same `MessageGroupID` will be in order, but across-group ordering is not guaranteed. 

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
* Data retention is 1 day by default, up to 7 days
* Data can't deleted once inserted into Kinesis
* Put Records: PutRecordAPI + Partition key that gets hashed, same key goes to the same partition
* ProvisionedThroughputExceededExceptions: sending too much data for one shard. Solution:
    * Retries with backoff
    * Increas shards (scaling)
    * Ensure partition key is a good one
* KCL: Each shard is read by only one KCL
  

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
* Cloning with exactly the same configuration, good for testing
* Can run the application as a single docker, doesn't use ECS; ECS can run multiple dockers per EC2 instance in EB, requires `Dockerrun.aws.json(v2)` at the root of source code to generate the ECS task definition
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
Notifications: 
* SNS/Lambda E.g. Deletion of branches, trigger for push in master, notify external build system, trigger Lambda function for code analysis
* CloudWatch events. E.g. trigger for pull requests, commit comment events, event rules go to SNS topic

### CodePipeline
Troubleshooting: 
* state changes happen in CloudWatch events, which can in return create SNS notifications
* If it fails at a stage, there is info in console
* CloudTrail can be used to audit API calls
* If pipleline can't perform an action, make sure the attached IAM Service Role has correct permission

### CodeBuild
* `buildspec.yaml` must be at the root of the source code
* Environment variables can be plaintext or secure store
* Phases: Install -> Pre build -> Build -> Post builds
* Artifacts uploaded to S3
* Can cache files to S3 to increase performance for future builds
* Specify VPC configuration (VPC ID, Subnet ID, Security Group ID) so build can access resources in VPC

### CodeDeploy
AppSpec:
* File section
* Hooks: set of instructions
    * ApplicationStop
    * DownloadBundle
    * BeforeInstall
    * AfterInstall
    * ApplicationStart
    * ***ValidateService***

# CloudFormation

[CloudFormation FAQ](https://aws.amazon.com/cloudformation/faqs/)

Infrastructure as code. 

* A CloudFormation is made up of the following sections:
    * __Resources__ (required) - specify the stack resources and their properties such as an EC2 instance or a S3 bucket. You can refer to resources in the Resources and Outputs sections of the template. Resource identifier form:`AWS::aws-product-name::data-type-name`
    * __Parameters__ (optional) - specifies values that you can pass in to your template at runtime (when you create or update a stack). You can refer to parameters anywhere in the template by leveraging `Fn::Ref`, shorthands is `!Ref` . Useful if resource config is likely to change in the future. 
    * __Mappings__ (optional) - hardcoded. a mapping of keys and associated values that you can use to specify conditional parameter values, similar to a lookup table. You can match a key to a corresponding value by using the `Fn::FindInMap`, shorthands `!FindInMap [MapName, TopLevelKey, SecondLevelKey]` in the Resources and Outputs section.
    * __Outputs__ (optional) - describes the values that are returned whenever you view your stack’s properties. For example, you can declare an output for an S3 bucket name and then call the Cloudformation describe-stacks AWS CLI command to view the name. A stack can't be deleted if its outputs is referenced by another stack. To reference the output in another stack, leverage `Fn::ImportValue`.
    * __Conditions__ (optional) - defines conditions that control whether certain resources are created or whether certain resource properties are assigned a value during stack creation or update. For example, you could conditionally create a resource that depends on whether the stack is for a production or test environment.
    * __Metadata__ (optional) - objects that provide additional information about the template.

* Stack creation rollback (get deleted) on error is enabled by default. Stack update rollback will reset the stack to the previous state.
* Use function Fn:GetAtt to output data

# Monitoring

## CloudWatch
* Metrics: a variable to monitor, belong to namespaces. EC2 details monitoring get metrics every 1 minute instead of 5. Customer high resolution metrics standard is 1 min, can be up to 1 second. To create custom metrics, use API PutMetricData.
* Alarms: used to trigger notification for metrics. High resolution metric alarm can be trigger at 10 or 30 s.
* Logs: can go to batch exporter to S3 for archival or stream to ElasticSearch for analysis. Never expire by default, but can define expiration policy at log groups level. Make sure IAM permission correct to send logs to CloudWatch. 
  * EC2 needs to install a CloudWatch agent to push logs. 
  * Logs can use filter expressions. Filters only publish metrics data for events after the filter is created.
* Events: scheduled cron jobs.

### EventBridge
* Next evolution of CloudWatch Events. Can analyze events in buses and infer the schema via SchemaRegistry.

## X-Ray
* Visual analysis of application
* Annotations: used for indexing traces and use with filters
* Leverages tracing, end-to-end way of following a request
* Application needs to import the SDK and the instance needs to install the X-Ray daemon or need to enable X-Ray integration.
   * To make X-Ray work on EC2, ensure IAM role is correct and the daemon is running
   * To make X-Ray work on Lambda, ensure it has IAM execution role with proper policy (AWSX-RayWriteOnlyAccess) and X-Ray SDK is imported in the code
* Instrumentation.
* Sampling rule: by default X-Ray records the first request every second (reservoir) and 5% (rate) of any addtional requests. Customer rules can change the reservoir and the rate.
* Elastic Beanstalk includes X-Ray daemon. Run by setting `.ebextensions/xray-daemon.config`. Make sure instance profile has the right IAM role and applicaiton code imports X-Ray SDK.
* X-Ray on ECS, three patterns: 
  1. As daemon: X-Ray container as a daemon on each instances
  2. As side-car: X-Ray daemon container alongside each application container
  3. Fargate claster: side-car

## CloudTrail
* Provide governance, compliance and audit. Enabled by default

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

* VPC Endpoints: connect to AWS services from with a VPC using a private network. Gateway for S3 and DynamoDB, ENI for the rest.

* From on-premise to AWS: VPN goes over the public internet, Direct Connect goes over a private network by establishing a physical connection.

# CloudFront

* Content Delivery Network (CDN). Provides content quicker to customers by caching it in edge locations. ie customer 1 watches a video from s3. s3 bucket is in Ireland but user is in Sydney. The content flows Ireland -----> Sydney. CloudFront caches it locally near Sydney so the second time its accessed the content flows, CloudFront Sydney -> Sydney.

* Objects are cached for ther life of their TTL. TTL can be 0 seconds to 365 days. Default is 24 hours.

* Edge locations can be used for write as well as read.

* Origin can be S3 or Customer Origin (HTTP) such as EC2, ALB, Route53 and non AWS server i.e. on-premise
    * S3 origin enhanced security with OAI (Origin Access Identify) + S3 bucket policy

* Security
    * HTTPS
        * Viewer Protocol Policy: redirect HTTP to HTTPS or HTTPS only
        * Origin Protocol Policy: HTTPS only or Map Viewer
    * Restrict viewer access by signed URL (single file) or Signed Cookies (multiple files) for distributing paid content
    * Restrict content based on geo location (whitelist and blacklist)

# Lambda
* Compute service allows you to run code without provisioning and managing servers. Under the hood are EC2 Instances managed by AWS. Lambda is stateless and event driven. Up to 15mins excecution time.
* Lambda needs correct Excecution Roles (IAM Roles) to access AWS services. When using event source mapping, Lambda uses the execution role to read event data. An IAM principal can access Lambda if the IAM policy attached to the principal authorizes it OR if the resource-based policy authorizes.
* Lambda environmental variables: helpful to store secrets
* Loggin and Monitoring:
  * CloudWatch Logs: make sure Lambda has an execution role with IAM policy that authorizes writes to CloudWatch logs
  * X-Ray: Enable `Active Tracing` in config. Make sure Lambda has an execution role with correct IAM policy and env var to communicate with X-Ray
* External dependencies need to be install with the code and zip together. If zip file is less than 50M then directly to Lambda, else to S3

### Performance
* RAM: from 128M to 3008M in 64M increment. The more RAM added, the more vCPU credits to get. If application is CPU-bound (computational heave), add more RAM.
* Timeout: default 3s, max is 15min.
* Execution context: temporary runtime environment that initializes dependencis for Lambda. So intilization code should be outside the function handler and reuse it across executions. It includes `/tmp` directory that can be used to write heavy files, max 512M. For permanent file storage, use S3.

### Concurrency
* Up to 1000 concurrent excecutions per account. Set a reserved concurrency to limit this number. Each invocation over the concurrency will trigger a throttle.
* If one application invokes too many concurrent Lambda functions, it may throttle other applications' Lambda functions.
* Asychronous Invocations: if Lambda doesn't have enough concurrency available, additional requests are throttled, events are returned to the internal event queue.
* Cold start: when lauching a new instance, the initialization stage when code is loaded and code outside the handler run. First request has longer latency than the rest. Provisioned concurrency can be allocated before the function is invoked.
* Alias can be use to manage different versions for Lambda. They are mutable.
* Lambda Optimization Tips: Avoid using recursion, keep deployment size minimum, install only dependecies that is required, keep your function logic outside handler. (source: https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)

### Invocations
* Three ways of processing events:
  * Synchronouse Invocation: result is returned right away, error handling happens at client side
    * With ALB: register Lambda function in a target group. Conversion between ALB <-> Lambda: HTTP <-> JSON. When enabling ALB multi-header values, HTTP headers and query parameters that are sent with multiple values are show as arrays within Lambda event and response objects.
    * With Lambda@Edge: deploy Lambda alongside CloudFront CDN. Use case: website security and privacy, SEO, etc
  * Asynchronouse Invocation: Events are placed in an event queue. Lambda retries after error, 3 retries in total. Make sure the process is idempotent. Can define a DLQ for failed processing.
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
