# AWS-Cloud-Solutions-Architect
My AWS Cloud Solutions Architect study notes

## Services

**Primary ones to focus on:  VPC, EC2, S3, EBS, ECS, LAMBDA, API, AutoScaling**

## EC2
Timeout isses => security groups configuration?  
Security groups can reference other security groups instead of ip ranges  
Public IP (obvious)  
* elastic IP (same as public IP, but consistent IP between stop and starts)
Private IP (obvious)  
   
* EC2 launch modes  
on demand  
reserved  
spot instances  
dedicated  
   
* Instance types:  
R = Ram  
C = CPU  
M = Medium  
I = IO  
G = GPU  
T2/T3 = Burst (or budget)  

## Amazon Machine Image (AMI)  
Create AMI to pre-install software on for EC2 => faster boot/deployment of instances  
AMI can be copeid across regions and accounts.  (default within region only)  
  
* Instance placement groups  
Cluster - close together, high performance (faster network), less redundancy  
Spread - spread among multi-AZ's, more redundancy  

**CLB / ALB / NLB**  
* CLB - may still have questions regarding security groups and stickiness  
  
* ALB  
Support routing based on hostname (users.example.com & payments.example.com)  
Support routing based on path (example.com/users & example.com/payments)  


Support redirets (HTTP to HTTPS as example)  
Support dynamic host port mapping with ECS  

* NLB (Layer 4) gets a static IP per AZ  
Public facing: must attach Elastic IP - can help whitelist by clients  
Private facing: will get random private IP base on free ones at time of creation  
Has cross zone balancing  
Has SSL termination (Jan 2019)  
  
*Restrict access to the EC2 instance by limiting security group source from the load balancer:*
![alt text](https://github.com/rlam13/AWS-Cloud-Solutions-Architect/blob/master/screenshots/load_balancer_security_groups.png)

* SSL Certificates on load balancer  
The load balancer uses a X.509 certificate (SSL/TLS server certificate)  
Manage certificates using AWS Certificate Manager (ACM)  
Alternatively create/upload own certificates
  
HTTPS listener:  
Must specify default certificate  
Optional - add list of certs to support multiple domains
Clients can use Server Name Indication (SNI) to specify the hostname they reach  
Able to specify security policy to support older versions of SSL/TLS (for legacy clients)  

  
**ASG**  
ASG Default Termination Policy:  
1. Find AZ with most instances
2. If multiple instances in AZ, delete the one with oldest launch configuration  
  
* Scaling cooldown  
Ensures that ASG doesn't launch/terminate instances before previous scaling activity  
In addition to default cooldown -> create cooldowns that apply to specific scaling policy  
Default cooldown period is 300 seconds, can reduce down to reduce scale-in policy  
If multiple scale out/in during an hours, modifty ASG cool-down timer and CloudWatch alarm period that triggers scale in  

**EBS**  
Local storage, can only be attached to one instance at a time (like a USB stick)
Locked at the AZ level  
* if you need to attach to another AZ, then backup (snapshot) and recreate in another AZ  
EBS backups use IO, don't run while application is handling lots of traffic  
Root EBS volumes get terminated by default if EC2 instance is terminated (can be disabled, if needed)  
Increase EBS volume size if disk I/O is high

**EFS**  
Network drive (file system) essentially
Can mount across lots of instances  
EFS share website files
EBS gp2, optimize cost

**Instance Store**
The volume on the instance.  Ephemeral (higher performance, but not portable/flexible like EBS/EFS)
Custom AMI for faster deploy  
  
**RDS**  
Managed PostgreSQL/MySQL/Oracle/SQL Server
Need to provision EC2 instance & EBS Volume type and size
Relational DB  
Backups are enabled automatically  
  
DB Snapshots  
* manually triggered by user  
* reetention of backup as long as you need  
  
Read replicas can only SELECT (read-only), can't INSERT/DELETE/UPDATE  
Supports Transparent Data Encryption (TDE) for DB enryption  
* Oracle or SQL server DB instance only  
* TDE can be used on top of KMS - may affect performance  

  + Operations:
    + short downtime during failover, during maintenance, scaling in reads
      + EC2 or EBS restore implies manual intervention

  + Security:
    + IAM Authentication OR userid/password can be used  
    + Works for MySQL, PostgreSQL  
    + Lifespan of authentication token is 15 minutes (short-lived)  
    + Tokens are generated by AWS credentials   
    + SSL must be used when connecting to DB  
    + Easy to use EC2 Instance roles to connect to the RDS database  
    + Monitoring through CloudWatch
  
  + Reliability:
    + Multi AZ feature, failover in case of failure
    
  + Performance:
    + Depends on EC2 instance type, EBS volume type, ability to add read replicas.  Does not auto-scale
    
  + Cost:
    + Pay per hour for provisioned EC2 and EBS
    

  + Uses: RDBMS / OLTP, perform SQL queries, transaction inserts / update / delete is available
  
**Aurora**  
Proprietary from AWS  - Postgres and MySQL are supported with API  
Cloud optimzed 5x improvement over MySQL on RDS, and 3x of Postgres on RDS  
Grows in increments of 10GB, up to 64TB  
Up to 15 replicas.  (MySQL can have 5), faster replication <10ms replica lag
  + can be global read replica
Failover in Aurora is instantaneous - HA native  
Costs ~20% more - but more efficient  
Auto healing capability
Auto fail-over  
Backup & Recovery  
Isolation & security  
Industry Compliance  
Push-button scaling  
Automated patching w/ zero downtime  
Advanced monitoring  
Routine maintenance  
Backtrack: restore data at any point with using backups  
No SSH (managed service)  
No need to choose instance size / helpful with unknown workload / DB scales automatically based on CPU & connections  
Can migrate from Aurora Cluster to Aurora Serverless and vice versa  
  
Aurora Serverless usage measured in Aurora Capacity Units (ACU), billed in 5 minute increment of ACU  
  
IAM Authentication for MySQL and PostgreSQL  
Aurora Global DB span multipl regions and enable DR  
* one primary region  
* one DR region  
* DR region can be used for lower latency reads  
* < 1 second replica lag on average  
If not use Global DB, can create cross-region read replics  
* FAQ recommends Global DB  

Use: same as RDS, but less maintenance, more flexibility, more performance

* Operations:
  + less operations, auto scaling storage
* Security:
  + AWS responsible for OS security, owner responsible for setting up KMS, security groups, IAM policies, authorizing users in DB use SSL
* Reliability:
  + Multi AZ, highly available, possibly more than RDS, serverless option available
* Performance:
  + 5x performance (according to AWS) due to artchitectural optimizations.  Up to 15 Read Replicas (only five for RDS)
* Cost:
  + Pay per hour based on EC2 and storage usage.  Possibly lower costs compared to Enterprise grade databases such as Oracle
  
  
**Elasticache**  
Manaaged Redis or Memcached  (similar to RDS, but for cache)
Very high performance, sub-milisecond latency  
Must provision an EC2 instance type
Support for Clustering (Redis) and Multi AZ, Read Replicas (sharding)
Write scaling using sharding  
Read scaling useing read replicas  
Multi AZ with failover capability  
AWS managed. (OS maintenance/patching,optimizations,setup,config,monitoring, failure recovery and backups)  
SASL authentication (no IAM)  

Use Case: Key/Value store, Frequent reads, less writes, cache results for DB queries, store session data for websites, cannot use SQL.

Operations:
  + same as RDS
* Security:
  + AWS responsible for OS security, owner responsible for setting up KMS, security groups, IAM policies, users (Redis Auth), use SSL
* Reliability:
  + Clustering, Multi AZ
* Performance:
  + sub-milisecond performance, in memory, read replicas for sharding, very popular cache option
* Cost:
  + Pay per hour based on EC2 and storage usage.

**Redis**  
RedisAUTH (username/password) (no IAM)  
SSL in-flight encryption must be enabled and used  
  
**Route 53**  
Most common recoto know:  
* A: URL to IPv4
* AAAA: URL to IPv6
* CNAME: URL to URL
* Alias: URL to AWS resource
    
TTL (Time to Live)  
High TTL (24hr)  
* Less traffic on DNS
* Possibly outdated records  
  
Low TTL (60s)
* more traffic on DNS  
* records are outdated for less time
* easy to change records
  
TTL is mandatory for each DNS record  
  
* AWS Resources (load balancers, cloudfront, etc) expose AWS URL:  
lbl-us-west-1.elb.amazonaws.com but you want it to appear as mycoolapp.cooldomain.com  
  
* CNAME:  
Points URL to another URL (mycoolapp.cooldomain.com >> somethingelse.com)  
Works only for NON ROOT DOMAIN ( mycoolapp.cooldomain.com)
  
* Alias:
Points a URL to AWS resource (mycoolapp.cooldomain.com >> somethingelse.amazonaws.com)
Works for ROOT DOMAIN and NON ROOT DOMAIN.  (aka cooldomain.com)
  
* Simple Routing Policy  
Maps a domain to one URL  
Used to redirect single resource  
Cannot attach health checks to Simple Routing Policy  
If multiple values are returned, random one chosen by client  
  
* Weighted Routing Policy  
Route traffic to multiple resources in proportions you specify
Helpful to test percentage of traffic on a new app version, if needed  
Can split traffic between two regions  
Can be associated with health checks  
  
* Latency Routing Policy
Redirect to the server that has the least latency  
Latency determined by user to designated AWS Region
  - Germany could be directed to the US, if that's the lower latency  

* Health Checks  
Have X health checks failed - unhealthy (default 3)  
After X health checks passed - healthy (default 3)  
Default health check interval: 30s (can set to 10s - costs more)  
Approximately 15 health checkers will check the endpoint health  
Can use HTTP/TCP and HTTPS health checks (no SSL verification)  
Possibility of integrating health check with CloudWatch 
  
Health checks can be linked to Route53 DNS queries  
  
* Failover Routing Policy  
For active/passive failover  
  
* Geo Location Routing Policy  
Routing based on user location (different from latency based)  
  
* Multivalue Answer Routing Policy  
Deploy when you want Route 53 to respond to DNS queries with up to eight healthy records selected at random 
  
* Route 53 can be a Registar  
Domain registar is an organization that manages reservation of Internet domain names

* Third Party Registrar with Route 53  
Route 53 can be used if domain is bought from third party website
  - Create hosted zone in Route 53
  - Update NS records on third party website to use Route 53 name servers  
  
**S3**  
Objects(files) have a key.  The key is the full path:
<example_bucket_name>/example.txt  
<example_bucket_name>/sample_folder/another_folder/example.txt  
No actual directories - the UI will make it look like directories  
Object values are the content of the body:
  + max size 5TB
  + if upload is larger than 5GB, must use multi-part upload  
Metatdata (list of text key / value pairs - system or user metadata)  
Tags (unicode key/value pair - up to ten) - useful for security / lifecycle
Versioning can be enabled at the bucket level
  + same key overwrite will increment the version. IE: 1, 2, 3 etc....  
  + best practice to version buckets  
  
Files not versioned prior to enabling versioning will have version "null"  
  
* Encryption for Objects  
  + SSE-S3: AWS handles and manages keys  
    + Object is encrypted server side
    + AWS-256 encryption type
    + Must set header: "x-amz-server-side-encryption":"AES256"
  + SSE-KMS: AWS Key Management Service to manage encryption keys  
    + KMS Advantages: user control + audit trail
    + Object is encrypted server side
    + Must set header: "x-amz-server-side-encryption":"aws:kms"
  + SSE-C: self manage encryption keys  
    + Server-side encryption using data keys fully managed by the customer side
    + S3 doesn't store the encryption key
    + HTTPS must be used
    + Encryption key must provided in HTTP headers, for every HTTP request made
  + Client side encryption is an option  
    + Client library such as Amazon S3 Encryption Client
    + Client must encrypt data before sending to S3
    + Client must decrypt data after retrieving from S3
    + Encryption cycle fully managed by customer  
  
* Security
  + User based - IAM policies 
    + Set which API calls are permitted by specific user via IAM console
  + Resource based  
    + Bucket Policies - bucket wide rules from the S3 console - allows cross account
    + Object Access Control List (ACL) - finer granularity
    + Bucket Access Control List (ACL) - not as common  
    
* Bucket Policies
  + JSON based 
    + Resources buckets and objects
    + Actions: Set of API to Allow or Deny
    + Effect: Allow / Deny
    + Principal: The account or user to apply the policy to
  + Use S3 bucket for policy to:
    + Grant public access to the bucket
    + Force objects to be encrypted at upload
    + Grant access to another account (cross account)  
  
* Other
  + Networking:
    + Supports VPC Endpoints (for instance in VPC without www internet)
  + Logging and Audit
    + S3 access logs can be stored in other S3 bucket
    + API calls can be logged in AWS CloudTrail
  + User Security:
    + MFA can be required in versioned buckets to delete objects
    + Signed URL's: URL's that are valid only for a limited time (IE: premium video service for logged in users)
    
* Websites
  + S3 can host static websites 
  + The website URL would be:
    + <bucket_name>.s3-website-<AWS-region>.amazonaws.com
              OR
    + <bucket_name>.s3-website.<AWS-region>.amazonaws.com
  + If 403 error, confirm bucket policy allows public read
   
* As Database
  + Key / value store for objects
  + Great for big objects, not as good for small objects
  + Serverless, scales infinitely, max object size is 5TB
  + Eventually consistency for overwrites and deletes
  + Tiers: S3 Standard, S3 IA, S3 One Zone IA, Glacier for backups
  + Features: versioning, encryption, cross regions replication
  + Security: IAM, bucket policies, ACL
  + Encryption: SSE-S3, SSE-KMS, SSE-C, client side encryption, SSL in transit
  + Uses: static files, key value store for big files, webiste hosting
  
* Operations:
  + no operations needed
* Security:
  + IAM, Bucket Policies, ACL, Encryption (server/client), SSL
* Reliability:
  + 99.999999999% durability, 99.99% availability, Multi AZ, CRR
* Performance:
  + scales to thousands of reads / writes per second, transfer acceleration / multi-part for big files
* Cost:
  + pay per storage usage, network cost, request number
   
* Cross Origin Resource Sharing (CORS)
  + limits the number of webiste that request your files in S3 (saves money)
  
* Consistency Model
  + Read after write consistency for PUTS of new objects
    + as soon as object is written we can retrieve it (PUT 200 -> GET 200)
    + except if GET was done before (GET 404 -> PUT 200 -> GET 404) -eventually consistent
  + Eventual consistency for DELETES and PUTS of existing objects
    + If read object after updating, may get the older version (PUT 200 -> PUT 200 -> GET 200)
    + If delete object, may be able to retrieve it for a short time after (DELETE 200 -> GET 200)
    
* MFA Delete
    + Require MFA to permanently delete object version
    + Require MFA to suspend versioning on the bucket
    + MFA not required for enabling version
    + MFA not required for listing deleted versions
    + Only the bucket owner (root account) can enable/disable MFA delete
    + MFA delete currently enabled using the CLI

* Default Encryption vs Bucket Policies
    + old way: use bucket policy and refuse HTTP command without proper headers
    + new way: default encryption in S3 (bucket policies are evaulated before "default encryption"
  
* Access logs
    + optional: log all access to S3 buckets
    + optional: use Athena to analyze directly from a bucket
  
* Cross Region Replication
    + must enable versioing (source & destination)
    + buckets must be in different AWS regions
    + can be in different accounts
    + copying is asynchronous
    + must give proper IAM permissions to S3
    + use cases: compliance, lower latency access, replication across accounts
    
* Presigned URLs
  + Grant temporary access to users with pre-signed URL to have them inherit permissions of person that generated the URL (GET/PUT)
  + Default 3600 secs
  + Downloads (easy, can use the CLI)
  + Uploads (harder, must use the SDK)
  + Applications:  
    + allow only logged in users to download premium video on S3 bucket
    + allow ever changing list of users to download files by generating URL's dynamically
    + allow temporarily a user to upload a file to a precise location within a bucket
  
* CloudFront, aka Content Delivery Network (CDN)
  + improved read performance, content cached at edge locations, 136 points of presence globally
  + popular with S3, but works with EC2 and load balancing
  + can help against network attacks
  + can provide SSL encryption (HTTPS) at edge using ACM
  + Cloudfront can use HTTPS to communicate with your applications
  + supports RTMP Protocol (video/media)
  
* Cloudfront vs S3 Cross Region Replication  
  + Cloudfront
    + Global edge network
    + Files are cached for a TTL (approximately a day)
    + Perfect for static content that must be available everywhere
  + S3 Cross Region Replication
    + Required to be setup for each region where replication are needed
    + Files are updated in near real-time
    + Read only
    + Perfect for dynamic content that requires low-latency availability in few regions
  
* S3 Standard  
  + High durability 99.99999999999% across multiple AZ
  + 99.99% availability over a given year
  + Can sustain two concurrent facility failurs
  + Uses: Big data analytics, mobile & gaming applicatoins, content distribution
  
* S3 Standard - Infrequent Access (IA)
  + For data less frequently accessed, but requires rapid access
  + High durability 99.99999999999% across multiple AZ
  + 99.99% availability over a given year
  + Can sustain two concurrent facility failures
  + Uses: data store for disaster recovery and backups
  
* S3 One Zone - Infrequent Access (IA)
  + Same as IA but data in a single AZ
  + High durability 99.99999999999% in one AZ
  + 99.95% availability over a given year
  + Low cost compared to IA (20% cheaper)
  + Uses: secondary backup copies of on-premise data or storing data that can be recreated easily
  
* S3 Glacier
  + Low cost object storage for archiving
  + Meant for longer term storage
  + Alternative to on-premise magnetic tape
  + High durability 99.99999999999%
  + Cost per storage per month ($0.004/ GB) + retrieval cost
  + Each item in Glacier is called "Archive" (up to 40TB)
  + Archives are stored in "Vaults"
  + Retrieval options:
    + Expedited (1 to 5 minutes retrieval) - $0.03 per GB and $0.01 per request
    + Standard (3 to 5 hours retrieval) - $0.01 per GB and $0.05 per 1000 request
    + Bulk (5 to 12 hours retrieval) - $0.0025 per GB and $0.025 per 1000 request
    
* S3 Lifecycle Rules
  + Rules to trigger when to move data to lower tiers to save storage cost
  + IE: General Purpose > IA > Glacier
  + Transition actions: Defines when objects transition to another storage class
  + Expiration actions: Assists in configuring object to expire after certain period
  
* Snowball
  + Physical transport, secure, tamper resistant, KMS 256, tracking
  + If it takes more than a week to transfer over the network, use Snowball
  
* Snowball Edge
  + Same as Snowball but adds computational capability.
  + 100TB capacity with either:
    + Storage optimized - 24 vCPU
    + Compute optimized - 52 vCPU & option GPU
  + Supports a custom EC2 AMI, perform processing on the go
  + Supports custom Lambda functions
  + Practical to pre-process data in transit
  
* AWS Storage Gateway
  + Bridge between on-prem data and S3
    + File Gateway
      + S3 buckets accessible via NFS and SMB
      + supports S3 standard, S3 IA, S3 One Zone IA
      + Bucket access using IAM roles for each File Gateway  
    + Volume Gateway
      + Block Storage / iSCSI
      + Backed by S3 with EBS snapshots
    + Tape Gateway
      + Virtual Tape Library (VTL) Solution / Backup with iSCSI
      + Backed by S3 and Glacier
      
    
**ATHENA**
  + Serverless service to perform analytics directly in S3 with SQL capability
  + Has JDBC / ODBC driver
  + Charged per query and amount of data scanned
  + Supports CSV, JSON, ORC, Avro, and Parquet (built on Presto)
  + Uses: BI/analytics/reporting/analyze & query.  VPC flow logs, ELB logs, Cloudtrail etc etc
  + Pay per query
  + Output results back to S3
  
* Operations:
  + no operations needed, serverless 
* Security:
  + IAM + S3 security
* Reliability:
  + managed service, uses Presto engine, highly available
* Performance:
  + queries scale based on data size
* Cost:
  + Pay per per query / per TB of data scanned, serverless

**Redshift**
  + Based on PostgreSQL but not used for OLTP
  + OLAP - online analytical processing (analytics and data warehousing)
  + 10x better performance than other data warehouses, scale to PB's of data
  + Columnar storage of data (opposed to row based)
  + Massively parallel query execution (MPP), highly available
  + Pay as you go based on the instances provisioned
  + Has a SQL interface for performing the queries
  + BI tools such as AWS Quicksight or Tableau integrate with it
  **Redshift = Analytics / BI / Data Warehouse**
  
* Operations:
  + similar to RDS
* Security:
  + IAM, VPC, KMS, SSL (similar to RDS)
* Reliability:
  + highly available, auto healing features
* Performance:
  + 10x performance vs other data warehousing, compression
* Cost:
  + Pay per node provisioned, 1/10th cost of other warehouses
  
**Neptune**
  + Fully managed graph database
    + High relationship data
    + Social Networking: Users friends with users, replied to comment on post of user and likes other comments
    + knowledge graphics (wikipedia)
  + Highly available across three AZ's, with up to 15 read replicas
  + Point-in-time recovery, continuous backup to S3
  + Support for KMS encryption at rest + HTTPS
  
* Operations:
  + similar to RDS
* Security:
  + KMS, VPC, IAM policies, SSL (similar to RDS) + IAM Authentication
* Reliability:
  + Clustering, Multi AZ
* Performance:
  + best suited for graphs, clustering to improve performance
* Cost:
  + Pay per node provisioned (similar to RDS)
  
**ElasticSearch = Search / Indexing**
  + compared to DynamoDB, only find by primary key or indexes.
    + Elasticsearch can search any field, even partial matches
  + common to use ElasticSearch as complement to another database
  + some use for big data applications
  + provision cluster of instances
  + Built-in integrations: Kinesis Data Firehose, AWS IoT, and CloudWatch logs for data ingest
  + Security through Cognito & IAM, KMS encryption, SSL & VPC
 
* Operations:
  + similar to RDS
* Security:
  + Cognito, IAM, VPC, KMS, SSL
* Reliability:
  + Clustering, Multi AZ
* Performance:
  + based on ElasticSearch project (open-source), petabyte scale
* Cost:
  + Pay per node provisioned (similar to RDS)
   
**SQS**
  + Fully managed
  + Scales from one message/second to 10k messages/second
  + Default retention of messsage: four days, maxiumum 14 days
  + no limit of messages in the queue
  + low latency (<10ms on publish and receive)
  + horizontal scaling in terms of number of consumers
  + can have duplicate messages (at least once delivery, occasionally)
  + can have out of order messages
  + limitation of 256kb per message
    + Delay Queue
      + Delay a message up to 15 minutes
      + Default is 0 seconds (message available right away)
      + Set default at queue level
      + Can override the default using the DelaySeconds parameter
    + Visibility Timeout
      + when consumer polls a message from a queue, the message is "invisible" to other consumers for a defined period
        + set between 0 seconds and 12 hours (default 30 secs)
        + If too high, 15 minutes and consumer fails to process the message, wait a long time before processing the message again
        + If too low, 30 seconds and consumer needs time to process the message (two minutes), another consumer will receive the message will be processed more than once
      + ChangeMessageVisiblity API to change the visbility while processing a message
      + DeleteMessage API to tell SQS the message was successfully processed
    + Dead Letter Queue
      + If consumer fails to process a message within the Visibility Timeout --> message goes back to the queue
      + Can set threshold of how many times message(s) go back to queue (aka redrive policy)
        + After threshold exceeded messages goes to dead letter queue (DLQ)
        + Need to create a DLQ first then designate it as DLQ
        + Ensure to process messages in DLQ before they expire
        
![alt text](https://github.com/rlam13/AWS-Cloud-Solutions-Architect/blob/master/screenshots/SQS.png)
      
    + FIFO Queue
      + First In / First Out (not available in all regions)
      + Name of queue must end in .fifo
      + lower throughput (3,000/s with batching, 300/s w/o)
      + Messages processed in order by the consumer
      + Messages are sent exactly once
      + No per message delay (only per queue delay)
      + Ability for conten base de-duplication
      + Five minute interval de-duplication using "Duplication ID"
      + Message groups:
        + Can group messages for FIFO using "Message GroupID"
        + Only one workder assigned per message group so messages are prcessed in order
        + Message group is just and extra tag on the message
        
**SNS**
  + Event producer only sends message to one SNS topic
  + Event receivers (subscriptions) want to listen to the SNS topic notificaions
  + Each subscirber to topic will get all messages (note: new feature to filter messages)
  + Up to 10,000,000 subscriptions per topic / 100,000 topics limit
  + Subcribers can be:
    + SQS, HTTP / HTTPS (with deilvery retries - how many times), Lambda, Emails, SMS messages, Mobile Notifications
    
    + Topic Publish (SDK)
      + Create a topic
      + Create a subscription (or many)
      + Publish to the topic
      
    + Direct Publish (mobile apps SDK)
      + Create platform application
      + Create platform endpoint
      + Publish to platform endpoint
      + works with Google GCM, Apple APNS, Amazon ADM
      
    + SNS + SQS: Fan Out
      + Push once in SNS, receive in many SQS
      + Fully decoupled
      + No data loss
      + Ability to add receivers of data later
      + SQS allows for delayed processing
      + SQS allows for retries of work
      + May have many workers on one queue and one worker on the other queue
      
 **KINESIS**
   + Kinesis is a managed alternative to Apache Kafka
   + Uses: application logs, metircs, IOT, clickstreams, real-time big data, processing frameworks (Spark, Nifi, etc)
   + Data auto replicated to three AZ's
     + Streams: low latency streaming ingest at scale
     + Analytics: perform real-time analytics on streams using SQL
     + Firehose: load streams into S3, Redshift, ElasticSearch
   
   + Security
     + ACL via IAM policy
     + Encryption in flight - HTTPS
     + Encryption at rest - KMS
     + Possible to encrypt/decrypt on client side (more difficult)
     +VPC Endpoints available for Kinesis to access within VPC
     

 **Amazon MQ**
   + To migrate on-premises open protcols MQTT, AMQP, STOMP, Openwire, WSS --> AMazon MQ
   
 **Lambda**
   + Virtual functions - no servers to manage
   + Limited by time - short executions
   + Run on-demand
   + Scaling is automated
   + Easy pricing
     + Pay per request and compute time
     + Free tier of 1,000,000 AWS Lambda requests and 400,000 GBs of compute time
   + Integrated with entire AWS stack - Primary Ones:
     + API Gateway, Kinesis, DynamoDB, S3, AWS IOT, CloudWatch Events/Logs, SNS, SQS, Cognito
   + Integerated with many programming languages
   + Easy monitoring through AWS CloudWatch
   + Easy to get more resources per functions (up to 3GB of RAM!)
   + Increasing RAM will also improve CPU and network
     + Allocated 128M to 3G (64MB increments)
     + Able to deploy within VPC + attach security group
     + IAM execution role must be attached to the Lambda function
   + Max execution 300 seconds (5 minutes)
   + Disk capacity in function container 512mb (/tmp)
   + concurrency limit: 1000
   + Deployment
     + Lambda function deployment size (compressed .zip): 50MB
     + Size of uncompressed deployment (code + dependencies): 250MB
     + can use /tmp directory to load other files at startup
     + size of environment variables: 4kb
     
**DynamoDB**
  + AWS proprietary technology, managed NoSQL database
  + Serverless, provisioned capacity, auto scaling, on demand capacity (Nov 2018)
  + Can replace Elasticache as a key/value store (storing session data for example)
  + Fully managed, highly available, replication across three AZ's
  + NoSQL DB
     + made of tables
     + each table has primary key
     + each table can have infinite number of items (rows)
     + each item has attributes (can be added over time, can be null)
     + max size of item 400KB
  + Scales to massive workloads, distributed database
  + Millions of request per seconds, trillions of row, 100s of TB of storage
  + Fast and consistent in performance (low latency on retrieval)
  + Integrated with IAM (security/admin/authorization)
  + Enables event driven programming with DynamoDB streams
  + Low cost and auto scaling capabilities
  + Data types supported are:
    + Scalar Types: string, number, binary, boolean, null
    + document types: list, map
    + set types: string set, number set, binary set
  + DynamoDB Accelerator - aka DAX
  + Integrates with AWS Lambda

* Operations:
  + no operations needed, auto scaling capability, serverless
* Security:
  + full security through IAM policies, KMS encryption, SSL in flight
* Reliability:
  + Backups, Multi AZ
* Performance:
  + single digit milisecond performance, DAX for caching reads, peformance doesn't degrade if application scales
* Cost:
  + Pay per provisioned capacity and storage usage (no need to guess in advance any capacity - can use auto scaling)
  
# API Gateway
   + Lambda + Gateway: no infrastructure to manage
   + Handle API versioning
   + Handle different environments (dev, test, prod)
   + Handle security (authentication and authorization)
   + Create API keys, handle request throttling
   + Swagger / Open API import to quickly to define API's
   + Transform and validate requests and responses
   + Generate SDK / API specification
   + cache API responses
   + Integrations
     + Outside of VPC
       + AWS Lambda
       + Endpoints of EC2
       + Load balancers
       + Any AWS Service
       + External and publicy accessible HTTP endpoints
     + Inside of VPC
       + AWS Lambda in your VPC
       + EC2 endpoints in your VPC
   + Security
     + IAM - Gateway verifies IAM permissions passed by the calling application
       + Good to provide access within your infrastructure
       + Leverages "Sig v4" capability where IAM credential are in headers
     + Custom Authorizer
       + Good for third party tokens
       + Very flexible for which IAM policy is returned
       + Handle Authentication and Authorization
       + Pay per Lambda invocation
     + Cognito User Pool:
       + You manage your own user pool (can be backed by FB, google etc logins)
       + No need to write custom code
       + Must implement authorization in the backend
  
# Cognito
    + Provide users an identity so that they can interact with apps
      + Cognito User Pools (CUP)
        + Sign in functionality for app users
        + Simple login (username/email and password)
        + Can be MFA, use email/phone number to verify
        + Can be federated identity (FB, Google, SAML)
        + Sends back JSON Web Token (JWT)
        + Can Integerate with API Gateway for authentication
      + Cognito Identity Pools (Federated Identity)
        + Provide AWS credentials to users to they can access AWS resources directly
        + Integrate with Cognito User Pools as an identity provider
      + Cognito Sync (deprecated - use AWS AppSync now)
        + Synchronize data from device to Cognito
        + May be deprecated and replace by AppSync
        + Requires federated identity pool in Cognito (NOT user pool)
        
 ## Cloudwatch
 ### Metrics
 
 + Provides metrics for *every* service in AWS
 + Metric is a variable to monitor (CPUUtilization, NetworkIn...)
 + Metrics belong to namespaces
 + Dimension is attribue of metric (instance ID, environment, etc...)
 + Up to 10 dimensions per metric
 + Metrics have timestamps
 + Can create CloudWatch dashboards of metrics
  
 ### Detailed Monitoring
 + EC2 instance have metrics "every five minutes"
 + Detailed monitoring (for a cost), get data "every minute"
 + Use detailed monitoring for faster prompt scale of ASG
 + AWS free tier allows ten detailed monitoring metrics
 + *Note:* EC2 Memory usage is default not pushed (must be pushed from inside the instance as custom metric)
   
 + Able to define and send your own custom metrics to CloudWatch
 + Ability to use dimensions (attributes) to segment metrics
   + Instance.id
   + Environment.name
 + Metric resolution
   + Standard: one minute
   + High resolution: up to one second (StorageResolution API parameter) - higher cost
 + Use API call PutMetricData
 + Use exponential back off in case of throttle errors
     
 ### Dashboards
 + Dashboards are global
 + Include graphs from different regions
 + Able to change time zone & time range
 + Able to set auto refresh (10s,1m,2m,5m,15m)
 
 ### Logs
 + Applications can send logs to CloudWatch via SDK
 + CloudWatch can collect logs from:
   + Elastic Beanstalk: logs from application
   + ECS: collection from containers
   + AWS Lambda: from function logs
   + VPC flow logs: VPC specific logs
   + API Gateway
   + CloudTrail based on filter
   + CloudWatch log agents: eg. EC2 instances
   + Route53: Log DNS queries
 + CloudWatch Logs 
   + Batch exporter to S3 for archival
   + Stream to ElastSearch cluster for further analytics
 + Logs storage architecture:
   + Log groupcs: arbitrary name, usually representing an application
   + Log stream: instances within application / log files / containers
 + Can define log expiration policies (never expire, 30 days, etc...)
 + Using AWS CLI able to tail CloudWatch logs
 + Send logs to CloudWatch, ensure IAM permissions are correct
 + Security: encryption of logs using KMS at the group level

### Alarms
+ Used to trigger notifications for any metric
+ notifications can be sent to Auto Scaling, EC2 Actions, SNS notifications
+ Various options (sampling,percentage,max,min,etc...)
+ Alarm States: Ok, Insufficient_data, Alarm
+ Period
  + Length of time in seconds to evaluate the metric
  + High resolution custom metrics: can only choose 10s or 30s
  
### Events
+ Schedule: cron jobs
+ Event Pattern: Event rules to react to a service doing something
  + CodePipeline state changes!
+ Triggers to Lambda functions, SQS/SNS/Kinesis Messages
+ CloudWatch Event creates a small JSON document to give information about the change

### CloudTrail
+ Provides governance, compliance and audit for your AWS Account
+ CloudTrail is enabled by default
+ Get history of events / API calls made within your AWS Account by:
  + console
  + SDK
  + CLI
  + AWS Services
+ Put logs from CloudTrail into CloudWatch logs
+ If resource is delelted in AWS, look into CloudTrail first!

## Encryption 
 
### Key Manamgent Service (KMS)
+ Use to share sensitive information
   + Database passwords
   + Credentials to external service
   + Private key of SSL certificates
+ Customer Master Keys (CMK) is use to encrypt KMS value and cannot be retrieved by user.  CMK can be rotated for extra security
+ Never store passwords or other sensitive info in plaintext, especially code
+ Encrypt secrets can be stored in code / environment variables
+ KMS can only help in encrypting 4kb of data per call
+ If data >4kb, use envelope encryption
+ Assign KMS access to one (or more) person(s)
  + Make sure Key Policy allows the user
  + Make sure the IAM Policy allows the API calls
+ Manage key & Policies
  + Create
  + Rotation policies
  + Disable
  + Enable
+ Audit key usage (via CloudTrail)
+ Three types of CMK
  + AWS Managed Service Default CMK: free
  + User keys created in KMS: $1 / month
  + User keys imported (must be 256-bit symmetric key): $1/month
+ pay for API call to KMS ($0.03 / 10000 calls)

## Parameter Store
+ Secure storage for configuration and secrets
+ Optional seamless encryption using KMS
+ Serverless, scalable, durable, easy SDK, free
+ version tracking of configurations/secrets
+ configuration management using path & IAM
+ notifications with CloudWatch Events
+ Integration with CloudFormation

## AWS STS - Security Token Service
+ Allows to grant limited and temporary access to AWS resources
+ Token is valid for up to one hour (must be refreshed)
+ Cross Account Access
  + Allows users from one AWS account access resources in another
+ Federation (Active Directory)
  + Provides a non-AWS user with temporary AWS acces by linking users Active Directory credentials
  + Uses SAML - Security Assertion Markup Language
  + Allows Single Sign On (SSO), enables users to login to AWS console without assigning IAM credentials
+ Federation with third party providers / Cognito
  + used primarily in web and mobile application
  + makes use of FB/Google/Amazon to federate them
  + allows users outside of AWS to assume temporary role to access AWS resource
  + users assume identity provided access role
  + Federation assumes third party authentication
    + LDAP
    + Microsoft Active Directory (~= SAML)
    + Single Sign on
    + Open ID
    + Cognito
  
## AWS Shared Responsibility Model

### AWS Responsibility - Security of the Cloud
  + Protecting infrastructure (hardware,software,facilities, and networking) that runs all of the AWS services)
  + Managed services like S3, DynamoDB, RDS etc
  
### Customer Responsibility - Security in the Cloud
  + For EC2 instance, customer is responsible for management of the guest OS (including security patches and updates), firewall & network configuration, IAM etc

### RDS Example
  + AWS Resposibility:
    + Manage the underlying EC2 instance, disable SSH
    + Automated DB patching
    + Automated OS patching
    + Audit underlying instance and disks & guarantee it funtions
  + Customer Responsiblity:
    + Check the ports / IP / security group inbound rules in DB's SG
    + In-database user creation and permissions
    + Creating a database with or without public access
    + Ensure parameter groups or DB is configured to only allow SSL connections
    + Database encryption setting
    
### S3 Example
  + AWS Responsibility
    + Guarantee unlimited storage
    + Guarantee encryption is available
    + ensure separation of data between different customers
    + ensure AWS employees can't access your data
  + Customer Responsibility
    + Bucket configuration
    + Bucket policy / public setting
    + IAM user and roles
    + Enabling encryption



