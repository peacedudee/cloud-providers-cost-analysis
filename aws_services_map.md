# AWS — Services Master Map

> **Objective:** Map every AWS service to establish a comprehensive tracking list for our cloud cost-optimization research.
>
> **How to use this file:** Each service has a `Research Status` column. As we deep-dive into individual services, we'll update the status and link to the dedicated research file for that service under the `aws/` directory.

---

## Legend

| Symbol | Meaning |
|--------|---------|
| ⬜ | Not started |
| 🟡 | In progress |
| ✅ | Research complete |

---

## 1. Compute

Core virtual servers, serverless compute, and container orchestration platforms.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 1 | **[EC2 (Elastic Compute Cloud)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/ec2.md)** | Virtual servers (instances) in the cloud with highly configurable OS, CPU, memory, and storage. | ✅ | Core VM compute platform; eligible for Savings Plans and Spot instances. |
| 2 | **[Lambda](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/lambda.md)** | Serverless, event-driven compute service that runs code in response to events and triggers. | ✅ | Billed per invocation and duration (GB-seconds); scales to zero. |
| 3 | **[ECS (Elastic Container Service)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/ecs.md)** | Highly secure, reliable, and scalable container management service supporting Docker containers. | ✅ | Can run on EC2 or serverless via Fargate. |
| 4 | **[EKS (Elastic Kubernetes Service)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/eks.md)** | Managed Kubernetes service to run and scale Kubernetes applications on AWS. | ✅ | Pays a flat rate cluster management fee ($0.10/hr) + underlying compute resources. |
| 5 | **[Fargate](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/fargate.md)** | Serverless compute engine for containers that works with both ECS and EKS. | ✅ | Charges based on vCPU and memory resources requested per second. |
| 6 | **[App Runner](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/app_runner.md)** | Fully managed containerized web application service that builds and deploys applications. | ✅ | Focuses on simplicity; charges for provisioned container instances and active request CPU. |
| 7 | **[Lightsail](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/lightsail.md)** | Easy-to-use virtual private servers, databases, and networking for simple workloads. | ✅ | Uses flat monthly pricing bundles (VPS package style). |
| 8 | **[Batch](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/batch.md)** | Managed batch job scheduler that dynamically provisions compute resources based on volume. | ✅ | No additional charge; pay only for underlying resources (EC2, Fargate) consumed. |
| 9 | **[Elastic Beanstalk](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/elastic_beanstalk.md)** | Platform-as-a-service (PaaS) to deploy and scale web applications and services. | ✅ | No additional charge; pay only for the resources (EC2, ELB, S3) created. |
| 10 | **[Outposts](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/outposts.md)** | Run native AWS services on-premises with physical rack hardware managed by AWS. | ⬜ | Involves upfront/monthly hardware lease contracts. |
| 11 | **[Wavelength](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/wavelength.md)** | Infrastructure offering ultra-low latency mobile edge computing within 5G networks. | ⬜ | Pricing maps to EC2 instance rates in Wavelength zones. |
| 12 | **[Local Zones](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/local_zones.md)** | Run latency-sensitive applications closer to end-users in metropolitan centers. | ⬜ | Billing maps to EC2 and local resources in specific Local Zones (rates vary from parent region). |
| 13 | **[SimSpace Weaver](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/simspace_weaver.md)** | Run massive, spatial simulations in the cloud, orchestrating multiple EC2 instances. | ⬜ | Charged per simulator hour and underlying instance sizes. |
| 14 | **[App 2 Container](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/app2container.md)** | Command-line tool to containerize existing .NET and Java applications. | ⬜ | Developer utility; no extra charge. |

---

## 2. Storage

Object, block, file, and hybrid cloud storage services.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 15 | **[S3 (Simple Storage Service)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/s3.md)** | Object storage service offering data availability, security, and performance. | ✅ | Highly tiered (Standard, IA, Intelligent-Tiering); billing driven by storage and requests. |
| 16 | **[S3 Glacier](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/s3_glacier.md)** | Secure, durable, and low-cost storage classes for data archiving and long-term backup. | ✅ | Very cheap storage (Glacier Flexible & Deep Archive) but has retrieval and early delete fees. |
| 17 | **[EBS (Elastic Block Store)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/ebs.md)** | High-performance block storage volumes designed for use with EC2 instances. | ✅ | Charged per GB-month provisioned (gp3, io2, etc.) and provisioned IOPS/throughput. |
| 18 | **[EFS (Elastic File System)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/efs.md)** | Serverless, fully managed elastic NFS file system for EC2, ECS, EKS, and Lambda. | ✅ | Charges for storage consumed (Standard, Infrequent Access) and read/write throughput. |
| 19 | **[FSx for Lustre](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/fsx_lustre.md)** | High-performance file system integrated with S3 for compute-intensive workloads. | ✅ | Charged for provisioned capacity and SSD throughput. |
| 20 | **[FSx for Windows File Server](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/fsx_windows.md)** | Fully managed Microsoft Windows file system with native SMB support. | ✅ | Charged for capacity, throughput, and backups. |
| 21 | **[FSx for NetApp ONTAP](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/fsx_netapp.md)** | Fully managed NetApp ONTAP file systems supporting multi-protocol access. | ✅ | Charged for SSD storage, SSD IOPS, throughput capacity, and capacity pool usage. |
| 22 | **[FSx for OpenZFS](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/fsx_openzfs.md)** | Fully managed OpenZFS file system for high-performance workloads. | ✅ | Charged for SSD storage, provisioned IOPS, and throughput capacity. |
| 23 | **[Storage Gateway](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/storage_gateway.md)** | Hybrid cloud storage service providing on-premises apps access to AWS storage. | ✅ | Charges based on gateway type (File, Volume, Tape), storage, and data written. |
| 24 | **[Backup](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/backup.md)** | Centralized backup service to automate and manage backups across AWS services. | ✅ | Charges for backup storage (warm/cold) per GB-month. |
| 25 | **[Snowball Edge](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/snowball.md)** | Physical data transfer and edge computing device for petabyte-scale migrations. | ✅ | Charged per-job fee plus daily usage fees. |
| 26 | **[Snowcone](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/snowcone.md)** | Ultra-portable, rugged, and secure edge computing and data transfer device. | ✅ | Small-scale transfer device; fixed per-job charge plus daily rates. |
| 27 | **[Elastic Disaster Recovery (DRS)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/drs.md)** | Minimize downtime and data loss with fast, reliable replication and recovery of servers. | ✅ | Charges per server replicated plus replication resources in AWS. |

---

## 3. Databases

Managed relational, NoSQL, in-memory, graph, and ledger databases.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 28 | **[RDS (Relational Database Service)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/rds.md)** | Managed relational database engine (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server). | ✅ | Billed by database instance hours, storage class, backups, and data egress. |
| 29 | **[Aurora](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/aurora.md)** | High-performance, MySQL and PostgreSQL-compatible relational database. | ✅ | Charges for DB instances (or Serverless v2 ACUs), storage, and I/O requests. |
| 30 | **[DynamoDB](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/dynamodb.md)** | Serverless NoSQL key-value database delivering single-digit millisecond performance. | ✅ | Provisioned Capacity (WCU/RCU) or On-Demand (RRU/WRU) billing, plus storage. |
| 31 | **[ElastiCache](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/elasticache.md)** | Managed in-memory caching and data store service supporting Redis and Memcached. | ✅ | Billed by cache node hours, node sizes, and serverless data storage. |
| 32 | **[MemoryDB for Redis](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/memorydb.md)** | Redis-compatible, durable, in-memory database service for fast performance. | ✅ | Charges for database node hours, data written to transaction log (per GB), and storage. |
| 33 | **[Redshift](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/redshift.md)** | Fast, fully managed petabyte-scale data warehouse for business intelligence. | ✅ | Billed by node hours (RA3/dense compute), serverless compute (RPU-hours), and storage. |
| 34 | **[DocumentDB](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/documentdb.md)** | Fully managed, MongoDB-compatible JSON document database. | ✅ | Billed by instance hours, storage, I/O operations, and backup storage. |
| 35 | **[Keyspaces](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/keyspaces.md)** | Serverless, Apache Cassandra-compatible database service. | ✅ | Billed based on read/write throughput (on-demand or provisioned), storage, and transfer. |
| 36 | **[Neptune](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/neptune.md)** | High-performance, managed graph database service supporting Property Graph and RDF. | ✅ | Billed by instance hours, storage, I/O requests, and backup storage. |
| 37 | **[Timestream](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/timestream.md)** | Serverless, fast, and scalable time-series database for IoT and operational apps. | ✅ | Charges for queries, write volume (per GB), and data storage (memory vs magnetic). |

---

## 4. Networking & Content Delivery

VPCs, load balancing, DNS, routing, content delivery, and secure connectivity.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 38 | **[VPC (Virtual Private Cloud)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/vpc.md)** | Provision logical isolated sections of AWS network for resources. | ✅ | Free basic routing; charges apply to egress, VPC Flow Logs, and IP addresses. |
| 39 | **[Route 53](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/route53.md)** | Scalable, highly available Domain Name System (DNS) and domain registration. | ✅ | Charged per hosted zone monthly and per-million DNS queries. |
| 40 | **[CloudFront](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/cloudfront.md)** | Global Content Delivery Network (CDN) for low-latency distribution of data. | ✅ | Charges for data transfer out (egress) and HTTP/HTTPS requests. |
| 41 | **[ELB (Elastic Load Balancing)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/elb.md)** | Distribute traffic across targets via Application (ALB), Network (NLB), and Gateway (GLB). | ✅ | Charged per load balancer hour and Capacity Units (LCU/NPU/GBCU). |
| 42 | **[API Gateway](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/api_gateway.md)** | Create, publish, maintain, monitor, and secure APIs (REST, HTTP, WebSockets) at scale. | ✅ | Charged per-million API calls, plus caching and data transfer costs. |
| 43 | **[Transit Gateway](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/transit_gateway.md)** | Central hub to connect VPCs, VPNs, and on-premises networks. | ✅ | Charged per attachment hour and per GB of data processed. |
| 44 | **[Direct Connect](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/direct_connect.md)** | Dedicated network connection from on-premises datacenter/office to AWS. | ✅ | Charged per port-hour (capacity) and data egress. |
| 45 | **[NAT Gateway](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/nat_gateway.md)** | Managed Network Address Translation to let private subnet resources access the internet. | ✅ | Charged per gateway hour and per GB of data processed; high cost hotspot. |
| 46 | **[PrivateLink](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/privatelink.md)** | Connect VPCs to AWS/partner services privately without public IPs or NAT. | ✅ | Charged per VPC endpoint hour and per GB of data processed. |
| 47 | **[Global Accelerator](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/global_accelerator.md)** | Improve global app availability and performance using AWS's private network path. | ✅ | Charged per accelerator hour and Data Transfer-Premium fee. |
| 48 | **[Route 53 Resolver](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/route53_resolver.md)** | Hybrid DNS resolution between VPCs and on-premises DNS servers. | ✅ | Charged per network interface endpoint hour and per-million recursive queries. |
| 49 | **[Client VPN](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/client_vpn.md)** | Managed client-to-site VPN connection for secure employee access to AWS/on-prem. | ✅ | Charged per active client connection hour and per endpoint association hour. |
| 50 | **[Site-to-Site VPN](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/site_to_site_vpn.md)** | Secure tunnel connecting VPC to on-premises office or data center. | ✅ | Charged per VPN connection hour. |
| 51 | **[Network Firewall](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/network_firewall.md)** | Managed stateful network inspection firewall for VPC protection. | ✅ | Charged per firewall endpoint hour and per GB of traffic processed. |
| 52 | **[Cloud Map](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/cloud_map.md)** | Cloud resource registry and discovery service for microservices. | ✅ | Charged per registered resource and per-million discovery API queries. |
| 53 | **[Verified Access](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/verified_access.md)** | Zero-trust application access for corporate workforces using identity/device states. | ✅ | Charged per application endpoint hour and per GB of data processed. |
| 54 | **[VPC Lattice](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/vpc_lattice.md)** | Serverless, service-to-service connectivity, security, and monitoring fabric. | ✅ | Charged per service hour, data processed, and requests. |
| 55 | **[Cloud WAN](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/cloud_wan.md)** | Build, manage, and monitor global networks using a central dashboard. | ✅ | Charged per core network edge hour and attachment hour. |

---

## 5. Data Analytics

Big data processing, data warehousing, serverless querying, and data cataloging.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 56 | **[EMR (Elastic MapReduce)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/emr.md)** | Managed cluster platform for big data frameworks like Apache Spark, Hadoop, Hive. | ✅ | Charges an EMR management premium hourly on top of underlying EC2/EKS resources. |
| 57 | **[Athena](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/athena.md)** | Serverless interactive query service to analyze S3 data using standard SQL. | ✅ | Charged per TB of data scanned during queries (min 10MB per query). |
| 58 | **[Glue](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/glue.md)** | Serverless data integration, ETL, and data catalog service. | ✅ | Charged per Data Processing Unit (DPU) hour for jobs, crawlers, and interactive sessions. |
| 59 | **[Kinesis Data Streams](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/kinesis_data_streams.md)** | Collect and process large streams of data records in real time. | ✅ | Billed by shard hours (provisioned) or per-GB (on-demand), plus PUT payloads. |
| 60 | **[Kinesis Data Firehose](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/kinesis_data_firehose.md)** | Capture, transform, and load streaming data into S3, Redshift, OpenSearch, etc. | ✅ | Charged based on the volume of data ingested (per GB). |
| 61 | **[MSK (Managed Streaming for Apache Kafka)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/msk.md)** | Fully managed Apache Kafka clusters for streaming real-time event data. | ✅ | Charged for broker instance hours, broker storage, and serverless partitions/storage. |
| 62 | **[OpenSearch Service](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/opensearch.md)** | Deploy, operate, and scale OpenSearch/Elasticsearch clusters for search and analytics. | ✅ | Charged per instance hour, storage volumes (EBS), and serverless OCU-hours. |
| 63 | **[QuickSight](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/quicksight.md)** | Serverless business intelligence (BI) service to create interactive dashboards. | ✅ | Tiered licensing (Enterprise/Enterprise+Q) per author and per active reader session. |
| 64 | **[Lake Formation](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/lake_formation.md)** | Orchestrates creation, permissions, and security policies of AWS data lakes. | ✅ | Free service; metadata access and underling data query costs apply. |
| 65 | **[Data Exchange](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/data_exchange.md)** | Find, subscribe to, and use third-party data datasets in the AWS cloud. | ✅ | Paid data subscription fees set by providers. |
| 66 | **[Clean Rooms](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/clean_rooms.md)** | Secure data collaboration workspace to analyze collective datasets without sharing raw data. | ✅ | Charged based on Clean Room CRU (Compute Resource Unit) hours during queries. |
| 67 | **[Data Pipeline](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/data_pipeline.md)** | Orchestrate scheduled, data-driven workflows for data movement and transformation. | ✅ | Charged per active pipeline based on frequency (high/low) and running location. |
| 68 | **[Entity Resolution](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/entity_resolution.md)** | Match, link, and deduplicate related records across different data sources using rule-based/ML matching. | ✅ | Charged based on number of records processed. |
| 69 | **[DataZone](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/datazone.md)** | Catalog, discover, share, and govern data at scale across organizational boundaries. | ✅ | Billed based on user profiles and metadata storage. |

---

## 6. AI & Machine Learning

Machine learning platforms, foundation model access, and specialized cognitive APIs.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 70 | **[SageMaker](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/sagemaker.md)** | Complete platform to build, train, and deploy machine learning models. | ✅ | Charges for notebooks, training instances, and real-time inference hosting hours. |
| 71 | **[Bedrock](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/bedrock.md)** | Serverless access to foundation models (FMs) via a single unified API. | ✅ | Billed per-token (input/output) or via provisioned throughput (model hours). |
| 72 | **[Q Developer (formerly CodeWhisperer)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/q_developer.md)** | AI-powered coding and troubleshooting companion for IDEs and CLI. | ✅ | Charges a fixed subscription fee per-user-month for the Professional tier. |
| 73 | **[Rekognition](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/rekognition.md)** | Deep learning computer vision API to analyze images, videos, faces, and text. | ✅ | Charged per image analyzed and per minute of video processed. |
| 74 | **[Polly](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/polly.md)** | Text-to-speech engine using deep learning to synthesize realistic human voices. | ✅ | Charged per million characters converted (Standard vs Neural/Long-Form voices). |
| 75 | **[Lex](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/lex.md)** | Build conversational interfaces (chatbots) using voice and text. | ✅ | Charged per speech request or text request processed. |
| 76 | **[Transcribe](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/transcribe.md)** | High-quality, automated speech-to-text transcription service. | ✅ | Charged per second of audio transcribed (Standard, Call Analytics, Medical). |
| 77 | **[Translate](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/translate.md)** | Fast, high-quality neural machine translation service. | ✅ | Charged per million characters of text translated. |
| 78 | **[Comprehend](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/comprehend.md)** | Natural language processing (NLP) to discover insights and relationships in text. | ✅ | Charged based on unit volume of text analyzed (in 100-character units). |
| 79 | **[Textract](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/textract.md)** | Automatically extract text, handwriting, tables, and structured layout from documents. | ✅ | Charged per page processed depending on analyzer type (Tables, Forms, Queries). |
| 80 | **[Personalize](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/personalize.md)** | Real-time personalization and recommendations API powered by machine learning. | ✅ | Charged per GB data ingested, training hours, and recommendation requests. |
| 81 | **[Forecast](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/forecast.md)** | High-accuracy time-series forecasting engine based on machine learning. | ✅ | Charged per forecast data point generated, training hours, and data metadata. |
| 82 | **[Kendra](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/kendra.md)** | Highly accurate enterprise search service powered by natural language search. | ✅ | Billed as a flat hourly platform cost per index depending on tier (Developer vs Enterprise). |
| 83 | **[DevOps Guru](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/devops_guru.md)** | ML-powered operations engine to detect anomalies and suggest application fixes. | ✅ | Charged per resource analyzed per month (e.g., active Lambda functions, EC2s). |
| 84 | **[Comprehend Medical](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/comprehend_medical.md)** | HIPAA-eligible NLP service specialized to extract medical insights from unstructured clinical text. | ✅ | Billed based on characters of text analyzed. |
| 85 | **[Transcribe Medical](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/transcribe_medical.md)** | HIPAA-eligible automatic speech recognition to convert clinician-patient dialog into text. | ✅ | Billed based on seconds of audio transcribed. |

---

## 7. Security, Identity & Compliance

Identity management, encryption, key storage, firewalls, and active threat analysis.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 86 | **[IAM (Identity and Access Management)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/iam.md)** | Securely manage access to AWS services, users, groups, and roles. | ✅ | Core foundation; free service. |
| 87 | **[KMS (Key Management Service)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/kms.md)** | Create, manage, and audit cryptographic keys to protect data. | ✅ | Charged per active key version per month and per 10,000 API operations. |
| 88 | **[Secrets Manager](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/secrets_manager.md)** | Rotate, manage, and retrieve credentials, keys, and API tokens. | ✅ | Charged per secret stored per month and per 10,000 API access requests. |
| 89 | **[Security Hub](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/security_hub.md)** | Automated security compliance checks and security finding aggregator. | ✅ | Charged per security check evaluated and per 10,000 ingestion events. |
| 90 | **[GuardDuty](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/guardduty.md)** | Continuous, intelligent threat detection and security monitoring. | ✅ | Charged based on volume of logs analyzed (VPC, CloudTrail, DNS, S3, RDS). |
| 91 | **[WAF (Web Application Firewall)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/waf.md)** | Protect web applications and APIs from common exploits and bot attacks. | ✅ | Charged per Web ACL, rule per Web ACL, and per-million requests evaluated. |
| 92 | **[Shield](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/shield.md)** | Managed DDoS protection service. Standard is free; Advanced offers higher tiers. | ✅ | Shield Standard is free; Shield Advanced costs a flat $3,000/month per organization. |
| 93 | **[Inspector](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/inspector.md)** | Automated vulnerability assessment for EC2 instances, ECR images, and Lambdas. | ✅ | Charged per agent/resource scanned per month. |
| 94 | **[Cognito](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/cognito.md)** | Customer identity management, authentication, user directories, and login providers. | ✅ | Charged based on Monthly Active Users (MAUs) and verification messages (SMS). |
| 95 | **[ACM (Certificate Manager)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/acm.md)** | Provision, manage, and renew public and private SSL/TLS certificates. | ✅ | Public certificates are free; private CA certificates have a monthly fee. |
| 96 | **[Directory Service](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/directory_service.md)** | Host and manage Microsoft Active Directory in the AWS cloud. | ✅ | Charged per active directory hour depending on size (Small/Large) and type. |
| 97 | **[IAM Identity Center](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/iam_identity_center.md)** | Successor to AWS Single Sign-On (SSO); centralize portal access to AWS/apps. | ✅ | Free service. |
| 98 | **[Macie](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/macie.md)** | Automated sensitive data discovery using ML to scan S3 buckets. | ✅ | Charged per GB of S3 data evaluated for sensitive content plus job costs. |
| 99 | **[Detective](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/detective.md)** | Investigate security events and identify root causes using graph models. | ✅ | Charged based on volume of security findings data ingested (per GB). |
| 100 | **[Audit Manager](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/audit_manager.md)** | Continuously audit AWS usage to simplify risk assessment and compliance. | ✅ | Charged based on number of resource assessments/evidence collected. |
| 101 | **[CloudHSM](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/cloudhsm.md)** | Dedicated hardware security module (HSM) instances inside your VPC. | ✅ | Charged per hour per HSM instance configured (high cost driver). |
| 102 | **[Private CA](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/private_ca.md)** | Create private certificate authority hierarchies to sign internal certificates. | ✅ | High cost driver ($400/CA/month flat fee + certificate fees). |
| 103 | **[Security Lake](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/security_lake.md)** | Automatically centralize security data from cloud, on-premises, and custom sources into a data lake. | ✅ | Charged based on GB of security data ingested and query runtimes. |
| 104 | **[Payment Cryptography](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/payment_cryptography.md)** | Import/export cryptographic keys and perform encryption operations on payment card data. | ✅ | Billed based on active key storage and cryptographic API calls. |

---

## 8. Management & Governance

Resource monitoring, automated ops, template orchestration, and cost tools.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 105 | **[CloudWatch](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/cloudwatch.md)** | Complete monitoring and observability service for resources and applications. | ✅ | Driven by custom metrics, dashboard count, alarms, log ingestion, and Synthetics. |
| 106 | **[Systems Manager (SSM)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/systems_manager.md)** | Operational cockpit for configuration management, patch execution, and terminal access. | ✅ | Basic features free; advanced agents, parameter store throughput, and tasks charge. |
| 107 | **[CloudFormation](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/cloudformation.md)** | Declarative infrastructure-as-code (IaC) templating engine. | ✅ | Free to create resources; charges apply for custom providers or handlers. |
| 108 | **[Organizations](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/organizations.md)** | Consolidate billing, policies, and account setups in multi-account organizations. | ✅ | Free governance service. |
| 109 | **[Config](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/config.md)** | Track and audit historical configurations and compliance changes of resources. | ✅ | Charged per configuration item recorded and per active rule evaluation. |
| 110 | **[Auto Scaling](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/autoscaling.md)** | Dynamically scales compute fleets (EC2, ECS, DynamoDB) based on rules. | ✅ | Free tool; user pays for the resources added. |
| 111 | **[Service Catalog](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/service_catalog.md)** | Create, manage, and govern portfolios of pre-approved cloud services. | ✅ | Charged per portfolio metadata API call. |
| 112 | **[Trusted Advisor](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/trusted_advisor.md)** | Scans AWS accounts to recommend cost-saving, security, and performance fixes. | ✅ | Basic checks free; full recommendation suite requires Business/Enterprise Support. |
| 113 | **[Well-Architected Tool](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/well_architected.md)** | Review architectures against five operational pillars using templates. | ✅ | Free tool. |
| 114 | **[Compute Optimizer](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/compute_optimizer.md)** | ML-powered sizing recommendations for EC2 instances, EBS, Lambda, and ECS. | ✅ | Free basic recommendations; opt-in metrics charge. |
| 115 | **[Budgets](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/budgets.md)** | Set custom budgets for costs, usage, or reservations with custom alerts. | ✅ | First two budgets free, then $0.02 daily per active budget. |
| 106 | **[Cost Explorer](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/cost_explorer.md)** | Graph and analyze AWS costs, usage patterns, and savings potentials. | ✅ | Free basic UI; API requests charged at $0.01 per query. |
| 117 | **[Control Tower](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/control_tower.md)** | Easily set up and govern secure, compliant, multi-account environments. | ✅ | Free orchestrator; underling Config rules, Service Catalog, and accounts apply. |
| 118 | **[License Manager](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/license_manager.md)** | Track and manage software licenses (Microsoft, Oracle, IBM, SAP) on AWS. | ✅ | Free tool. |
| 119 | **[Health Dashboard](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/health_dashboard.md)** | View active service performance alerts, scheduled maintenance, and global status. | ✅ | Free tracking dashboard. |
| 120 | **[Fault Injection Service (FIS)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/fis.md)** | Managed chaos engineering service to run controlled experiments on AWS resources. | ✅ | Billed based on the total minutes that each action runs. |
| 121 | **[Resilience Hub](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/resilience_hub.md)** | Assess and improve the resilience of application workloads against outages. | ✅ | Billed per described application per month. |
| 122 | **[Billing Conductor](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/billing_conductor.md)** | Customize billing data, structure showbacks, and simulate pricing models for chargebacks. | ✅ | Billed per active billing group per month. |

---

## 9. Developer Tools & CI/CD

Hosted compilation, continuous integration pipelines, package indexing, and distributed tracing.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 123 | **[CodeBuild](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/codebuild.md)** | Managed compilation and testing service that runs build containers. | ✅ | Charged per build-minute based on build machine type. |
| 124 | **[CodeDeploy](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/codedeploy.md)** | Automated application deployments to EC2, ECS, Lambda, or on-prem. | ✅ | Free for deployments to EC2/Fargate/Lambda; charges apply for on-premises targets. |
| 125 | **[CodePipeline](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/codepipeline.md)** | Fully managed CI/CD pipeline automation service. | ✅ | Charged per active pipeline per month (first one free). |
| 126 | **[CodeArtifact](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/codeartifact.md)** | Managed artifact repository service for Maven, Gradle, npm, pip, NuGet, etc. | ✅ | Charged for storage (GB-month) and data requests (reads/writes). |
| 127 | **[Cloud9](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/cloud9.md)** | Cloud-based IDE for writing, running, and debugging code in a browser. | ✅ | Free IDE tool; underlying EC2 instance and EBS volumes are billed. |
| 128 | **[X-Ray](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/xray.md)** | Trace and analyze distributed transactions across serverless or container microservices. | ✅ | Charged per-million trace segments stored and retrieved. |
| 129 | **[AppConfig](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/appconfig.md)** | Create, manage, and deploy application configurations and feature flags safely. | ✅ | Charged based on configuration requests and configurations deployed. |
| 130 | **[Proton](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/proton.md)** | Automated delivery service for containerized and serverless environments. | ✅ | Free tool. |

---

## 10. Application Integration

Decoupled message queues, publish-subscribe notification channels, and workflow engines.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 131 | **[SQS (Simple Queue Service)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/sqs.md)** | Managed message queuing to decouple and scale microservices. | ✅ | Billed per-million API requests; free tier has 1M requests/month. |
| 132 | **[SNS (Simple Notification Service)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/sns.md)** | Publish/subscribe notification messaging for mobile, SMS, email, and endpoints. | ✅ | Billed per-million notifications; costs vary by delivery type (SMS vs HTTP/Lambda). |
| 133 | **[EventBridge](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/eventbridge.md)** | Serverless event bus that connects app data from custom or SaaS sources. | ✅ | Charged per-million events processed, plus custom schema registries. |
| 134 | **[Step Functions](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/step_functions.md)** | Visual serverless workflow state machine orchestrator. | ✅ | Billed per state transition (Standard) or per execution duration (Express). |
| 135 | **[AppSync](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/appsync.md)** | Serverless GraphQL and real-time Pub/Sub interface engine. | ✅ | Charged per query/mutation request and subscription connection minutes. |
| 136 | **[MQ](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/mq.md)** | Managed message broker service for ActiveMQ and RabbitMQ. | ✅ | Billed by active broker instance hours, broker storage, and data transfer. |
| 137 | **[MWAA (Managed Workflows for Apache Airflow)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/mwaa.md)** | Managed execution environment for Apache Airflow data orchestration. | ✅ | Billed by environment class (Small/Medium/Large) hour and worker autoscaling. |
| 138 | **[B2B Data Interchange](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/b2b_data_interchange.md)** | Fully managed Electronic Data Interchange (EDI) transaction engine to map EDI files to JSON/XML. | ✅ | Billed per EDI transaction processed. |

---

## 11. Migration & Transfer

Heterogeneous database conversion, data-sync agents, and physical transfer hardware.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 139 | **[DMS (Database Migration Service)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/dms.md)** | Managed replication engine to migrate databases to AWS with low downtime. | ✅ | Billed by active replication instance hours, storage, and transfer. |
| 140 | **[MGN (Application Migration Service)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/mgn.md)** | Lift-and-shift physical, virtual, and cloud servers to AWS. | ✅ | Free for the first 90 days per replication server; user pays for replication resources. |
| 141 | **[DataSync](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/datasync.md)** | Accelerate online data transfers between on-prem storage/clouds and AWS. | ✅ | Billed per GB of data copied. |
| 142 | **[Transfer Family](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/transfer_family.md)** | Managed endpoint for secure SFTP, FTPS, and FTP transfers directly to S3 or EFS. | ✅ | Billed per-protocol endpoint hour and per GB of data transferred. |
| 143 | **[Migration Evaluator](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/migration_evaluator.md)** | Discovery tool to build data-driven business cases for migration. | ✅ | Free service. |
| 144 | **[Schema Conversion Tool (SCT)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/sct.md)** | Local desktop tool to convert database schemas for heterogeneous moves. | ✅ | Free software. |
| 145 | **[Discovery Service](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/discovery_service.md)** | Identify, collect, and map on-premises server inventory for migration. | ✅ | Free agentless/agent collection; agent data stored in Application Discovery. |
| 146 | **[Mainframe Modernization](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/mainframe_modernization.md)** | Migrate, run, and modernise mainframe applications in a managed runtime. | ✅ | Billed per runtime environment hour and migration tools. |

---

## 12. Frontend Web & Mobile

Hosting frameworks, customer push messaging, device testing, and maps.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 147 | **[Amplify](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/amplify.md)** | Full-stack platform to build, deploy, and host mobile and web apps. | ✅ | Billed per build minute and per GB data served/stored. |
| 148 | **[Device Farm](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/device_farm.md)** | App testing service to run automated tests on hundreds of physical mobile devices. | ✅ | Billed per device minute. |
| 149 | **[Pinpoint](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/pinpoint.md)** | Customer engagement service to send push, SMS, email, and voice campaigns. | ✅ | Billed per monthly targeted user (MTU) and per delivery event. |
| 150 | **[Location Service](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/location_service.md)** | Integrate maps, geofences, tracking, and routing indicators securely. | ✅ | Charged per map tile request, address search, or routing request. |

---

## 13. Customer Engagement & Business Applications

Cloud contact centers, outbound email dispatchers, and virtual desktops.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 151 | **[Connect](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/connect.md)** | Managed omni-channel cloud contact center service. | ✅ | Pay-per-minute of voice/chat interaction plus phone number leases and sub-features. |
| 152 | **[SES (Simple Email Service)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/ses.md)** | Bulk and transactional outbound email service. | ✅ | Charged per 1,000 emails sent/received, plus data payloads. |
| 153 | **[WorkSpaces](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/workspaces.md)** | Fully managed secure desktop-as-a-service (DaaS) for Windows/Linux users. | ✅ | Billed per workspace (monthly flat rate or hourly rate + storage). |
| 154 | **[AppStream 2.0](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/appstream.md)** | Stream desktop applications to a web browser on any device. | ✅ | Charged hourly per running instance and user licensing fees. |
| 155 | **[Chime](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/chime.md)** | Video conferencing, chat, and business call services. | ✅ | Billed per user per day. |
| 156 | **[WorkMail](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/workmail.md)** | Secure, managed business email and calendar service. | ✅ | Billed per user-month ($4/user). |
| 157 | **[WorkSpaces Secure Browser](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/workspaces_secure_browser.md)** | Secure virtual browser access to internal corporate websites and SaaS apps (formerly WorkSpaces Web). | ✅ | Billed per user-month. |
| 158 | **[WorkSpaces Core](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/workspaces_core.md)** | API-based virtual desktop infrastructure (VDI) to integrate with third-party software like VMware/Citrix. | ✅ | Billed based on monthly or hourly virtual desktop hours. |
| 159 | **[WorkSpaces Thin Client](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/workspaces_thin_client.md)** | Hardware utility and managed console to securely connect employees to WorkSpaces environments. | ✅ | Billed per device hardware purchase plus monthly device management fee. |
| 160 | **[Chime SDK](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/chime_sdk.md)** | Real-time audio, video, and screen-sharing media session infrastructure for developers. | ✅ | Billed per media connection minute. |

---

## 14. Internet of Things (IoT)

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 161 | **[IoT Core](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/iot_core.md)** | Managed connection framework for IoT devices to securely stream data. | ✅ | Charged per-million messages sent, plus connection minutes. |
| 162 | **[IoT Device Management](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/iot_device_management.md)** | Remotely register, track, and monitor IoT devices at scale. | ✅ | Billed per device registered and per bulk execution action. |
| 163 | **[IoT Greengrass](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/iot_greengrass.md)** | Run local compute, messaging, data management, and ML inference on IoT devices. | ✅ | Charged per active device running Greengrass software per month. |
| 164 | **[IoT Analytics](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/iot_analytics.md)** | Clean, process, and analyze unstructured data from IoT channels. | ✅ | Billed based on processed data volume (per GB) and query execution. |
| 165 | **[IoT SiteWise](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/iot_sitewise.md)** | Collect, store, organize, and monitor industrial sensor data. | ✅ | Billed based on data ingested (per GB), stored, and processed. |

---

## 15. Media Services

Broadcast-grade video encoders, live packagers, and interactive streaming tools.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 166 | **[Elemental MediaLive](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/medialive.md)** | Real-time live video encoding service to output streams. | ✅ | Billed based on input channels and output stream resolutions/hours. |
| 167 | **[Elemental MediaConvert](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/mediaconvert.md)** | File-based video transcoding service. | ✅ | Billed per minute of video output depending on resolution/codecs. |
| 168 | **[Elemental MediaPackage](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/mediapackage.md)** | Video prep and packager for adaptive bitrate delivery. | ✅ | Billed per GB of video processed and delivered. |
| 169 | **[Elemental MediaStore](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/mediastore.md)** | High-performance, low-latency video media storage. | ✅ | Billed based on stored volume (GB-month) and HTTP requests. |
| 170 | **[Elemental MediaTailor](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/mediatailor.md)** | Server-side ad insertion and personalization engine. | ✅ | Billed based on ad insertion requests and stream delivery. |
| 171 | **[IVS (Interactive Video Service)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/ivs.md)** | Managed live video solution built on Twitch infrastructure. | ✅ | Billed per hour of video input and per hour of video delivered to viewers. |

---

## 16. Satellite / Quantum / Game Tech

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 172 | **[Ground Station](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/ground_station.md)** | Managed control of satellite communications and data reception. | ✅ | Billed per minute of antenna downlink/uplink connection. |
| 173 | **[Braket](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/braket.md)** | Design, test, and run quantum algorithms on physical QPUs and simulators. | ✅ | Billed per simulator hour or per physical QPU task and shot request. |
| 174 | **[GameLift](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/gamelift.md)** | Deploy, scale, and manage dedicated server fleets for multiplayer games. | ✅ | Billed based on underlying EC2 instance hours consumed plus data transfer. |

---

## 17. Niche & Specialized Industry Services

Broadcast, supply chain management, encrypted collaboration, health tech, and specialized genomics tools.

| # | Service | Description | Research Status | Notes |
|---|---------|-------------|----------------|-------|
| 175 | **[Managed Blockchain](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/managed_blockchain.md)** | Easily create and manage scalable blockchain networks using Hyperledger Fabric or Ethereum. | ✅ | Billed based on peer node hours, node storage, and data processed. |
| 176 | **[Telco Network Builder (TNB)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/tnb.md)** | AWS service designed to deploy, orchestrate, and manage cellular network functions. | ✅ | Charged per active network function instance hour. |
| 177 | **[Supply Chain](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/supply_chain.md)** | ML-powered supply chain platform that connects data, provides insights, and helps plan inventory. | ✅ | Billed per-user per month plus data storage and transfer. |
| 178 | **[Wickr](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/wickr.md)** | End-to-end encrypted messaging, voice/video calls, and file sharing for businesses. | ✅ | Billed based on monthly active users. |
| 179 | **[AppFabric](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/appfabric.md)** | Connect and standardise logs, security states, and access across SaaS platforms. | ✅ | Billed based on active user profiles per SaaS application per month. |
| 180 | **[Omics](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/omics.md)** | Dedicated health and science data service to store, query, and run analysis workflows on genomic datasets. | ✅ | Billed based on genomic sequence storage, database storage, and workflow CPU/GPU run times. |
| 181 | **[HealthLake](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/aws/healthlake.md)** | HIPAA-eligible data store to search, query, and transform healthcare data into standard formats. | ✅ | Charged per active data store hour and per GB storage. |

---

## Research Tracking Summary

| Category | Total Services | ⬜ Not Started | 🟡 In Progress | ✅ Complete |
|----------|---------------|---------------|----------------|-------------|
| Compute | 14 | 0 | 0 | 14 |
| Storage | 13 | 0 | 0 | 13 |
| Databases | 10 | 0 | 0 | 10 |
| Networking & Content Delivery | 18 | 0 | 0 | 18 |
| Data Analytics | 14 | 0 | 0 | 14 |
| AI & Machine Learning | 16 | 0 | 0 | 16 |
| Security, Identity & Compliance | 19 | 0 | 0 | 19 |
| Management & Governance | 18 | 0 | 0 | 18 |
| Developer Tools & CI/CD | 8 | 0 | 0 | 8 |
| Application Integration | 8 | 0 | 0 | 8 |
| Migration & Transfer | 8 | 0 | 0 | 8 |
| Frontend Web & Mobile | 4 | 0 | 0 | 4 |
| Customer Engagement & Business Apps | 10 | 0 | 0 | 10 |
| Internet of Things (IoT) | 5 | 0 | 0 | 5 |
| Media Services | 6 | 0 | 0 | 6 |
| Satellite / Quantum / Game Tech | 3 | 0 | 0 | 3 |
| Niche & Specialized Industry Services | 7 | 0 | 0 | 7 |
| **TOTAL** | **181** | **0** | **0** | **181 🎉** |

---

> **💡 Next Steps:** Choose a service category or individual high-spend service to begin deep-dive research. We will create dedicated `<Service_Name>_Research.md` files under the `aws/` directory details with pricing structures, cost levers, and optimization recommendations.
