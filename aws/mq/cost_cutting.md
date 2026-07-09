# Cost-Cutting Playbook: Amazon MQ
> **Companion File:** [mq.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/mq/mq.md)
> **Last Updated:** July 2026
---
## Executive Summary
This playbook outlines 20 actionable strategies to optimize and reduce the costs of Amazon MQ. Amazon MQ is a managed message broker service for Apache ActiveMQ and RabbitMQ that relies on provisioned instance-hours and storage. Unlike serverless alternatives, it incurs 24/7 costs regardless of idle time. The primary drivers for cost reduction involve rightsizing instance families (adopting Graviton), restricting Multi-AZ deployments to production, optimizing storage types (EBS GP3 over EFS), and refactoring cloud-native workloads to serverless messaging services like Amazon SQS or SNS.

## Strategy Categories
### 1. Waste Elimination
### 2. Rightsizing
### 3. Commitment Discounts
### 4. Architecture Changes
### 5. Scheduling & Auto-Scaling
### 6. Pricing Model Optimization
### 7. Network & Data Transfer Optimization
---
## Cross-Service Synergies
- **Amazon SQS / SNS:** Serverless messaging alternatives that can replace Amazon MQ for brand-new cloud-native architectures, avoiding fixed 24/7 broker costs.
- **Amazon EC2 / ECS / EKS (Consumers):** Optimizing auto-scaling of message consumers can reduce the time messages sit in the broker, marginally reducing storage volume costs.
- **AWS Cost Explorer & CUR:** Essential for identifying underutilized brokers and tracking cross-AZ/data transfer costs associated with message routing.
---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Hourly billing records for `AmazonMQ` usage.
- Data Transfer and NAT Gateway costs correlated with MQ traffic.
- Storage billing metrics (EBS vs EFS) for ActiveMQ.

### B. CloudWatch Metrics
- `CPUUtilization`: To identify idle or over-provisioned brokers.
- `NetworkIn` / `NetworkOut`: To assess throughput and size instances accordingly.
- `QueueSize` / `MessageCount`: To evaluate dead-letter queues or backlog inflation.

### C. AWS Config / Trusted Advisor
- Identification of Multi-AZ vs. Single-AZ broker configurations across environments.
- Public accessibility flags on brokers.

### D. Company Policies
- Environment-specific high availability (HA) requirements (e.g., Multi-AZ mandates).
- Data retention and dead-letter queue (DLQ) compliance policies.

### E. IaC (Optional)
- Terraform/CloudFormation templates to automate the teardown/recreation of non-prod brokers.
---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "MQ-RGT-001",
  "resource_id": "arn:aws:mq:us-east-1:123456789012:broker:dev-broker:b-1234a567-89bc-012d-3456-e78f90a123bc",
  "strategy_used": "Downgrade Non-Prod to Single-AZ",
  "current_state": "Multi-AZ Active/Standby (mq.m5.large)",
  "recommended_state": "Single-AZ (mq.m7g.medium)",
  "monthly_savings_usd": 315.36,
  "effort_level": "Medium"
}
```
### Summary Report Table
| Finding ID | Strategy Name | Resource ID | Estimated Savings/Mo | Risk Level |
|------------|---------------|-------------|----------------------|------------|
| MQ-RGT-001 | Downgrade to Single-AZ | `dev-broker-01` | $315.36 | Low |
| MQ-ARC-001 | Migrate to SQS | `new-app-broker` | $105.12 | High |

---

#### 1. Delete Idle and Unused Brokers
- **What:** Identify and terminate Amazon MQ brokers that have zero active connections or zero message throughput over a 30-day period.
- **Why It Saves Money:** Amazon MQ charges an hourly fee ($105.12+/mo for `mq.m7g.medium`) just for the instance running, even if completely idle. 
- **Implementation Steps:** 
  1. Use CloudWatch to track `NetworkIn`, `NetworkOut`, and `TotalMessageCount`.
  2. Flag brokers showing near-zero activity for 30 days.
  3. Verify with application owners and terminate the brokers.
- **Estimated Savings:** 100% per terminated broker.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Visibility into CloudWatch metrics.

#### 2. Purge Stale Dead Letter Queues (DLQs)
- **What:** Implement Time-To-Live (TTL) on messages and actively monitor/purge dead letter queues so they do not grow indefinitely.
- **Why It Saves Money:** MQ charges for storage ($0.30/GB for EFS, $0.10/GB for EBS). A bloated DLQ inflates your storage footprint.
- **Implementation Steps:** 
  1. Inspect RabbitMQ or ActiveMQ management consoles for large DLQs.
  2. Configure TTL policies on queues.
  3. Establish automated jobs to archive or drop old DLQ messages.
- **Estimated Savings:** 10-30% on MQ storage costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Agreed-upon message retention policies.

#### 3. Disable Unnecessary CloudWatch Audit Logging
- **What:** Turn off verbose general/audit logging for Amazon MQ in environments where it is not strictly required.
- **Why It Saves Money:** CloudWatch Logs ingestion costs $0.50 per GB. Heavy messaging traffic with verbose logging can generate massive log volumes.
- **Implementation Steps:** 
  1. Edit the broker configuration in the AWS Console.
  2. Uncheck general and audit logging for non-production brokers.
- **Estimated Savings:** 50-80% on associated CloudWatch log ingestion costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compliance approval to disable audit logs in dev/test.

#### 4. Consolidate Shadow IT Developer Brokers
- **What:** Prevent individual developers from spinning up dedicated MQ brokers in their personal AWS sandbox accounts.
- **Why It Saves Money:** Eliminates duplicated base instance costs ($105+/month per developer).
- **Implementation Steps:** 
  1. Identify fragmented brokers via AWS Cost Explorer.
  2. Provide a shared, multi-tenant broker in a central development account.
- **Estimated Savings:** $105.12+ per month per consolidated broker.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Network access to the shared development broker.

#### 5. Migrate to Graviton Instance Families (`mq.m7g`)
- **What:** Upgrade existing `m5` class message brokers to AWS Graviton `m7g` instances.
- **Why It Saves Money:** Graviton instances offer better price-performance. For example, replacing a single-AZ `mq.m5.large` ($0.288/hr) with `mq.m7g.medium` ($0.144/hr) saves 50%.
- **Implementation Steps:** 
  1. Check application compatibility (AMQP/JMS are inherently agnostic to underlying CPU architecture).
  2. Modify the broker instance type during a maintenance window.
- **Estimated Savings:** Up to 50% on broker instance compute.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Maintenance window for broker restart.

#### 6. Downgrade Non-Prod to Single-AZ
- **What:** Use Single-AZ brokers for development and staging instead of Multi-AZ Active/Standby clusters.
- **Why It Saves Money:** Multi-AZ `mq.m5.large` costs $420.48/mo. Downgrading to a Single-AZ `mq.m7g.medium` costs $105.12/mo.
- **Implementation Steps:** 
  1. Identify Multi-AZ deployments labeled as dev/test.
  2. Recreate the broker as Single-AZ (Amazon MQ does not support in-place Multi-AZ to Single-AZ downgrade).
  3. Update application connection strings.
- **Estimated Savings:** 75% on compute per non-prod broker.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Acceptable downtime in non-prod during migration.

#### 7. Downsize Underutilized Production Brokers
- **What:** Reduce the instance size of brokers (e.g., `mq.m5.xlarge` down to `mq.m5.large`) if CPU and Network metrics are consistently low.
- **Why It Saves Money:** Broker hourly rates double with every step up in instance size.
- **Implementation Steps:** 
  1. Analyze CloudWatch `CPUUtilization` (e.g., consistently < 20%).
  2. Schedule a maintenance window to modify the broker instance type.
- **Estimated Savings:** 50% per downsized tier.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Thorough load testing to ensure smaller instance can handle peak traffic.

#### 8. Migrate ActiveMQ Storage to EBS GP3
- **What:** Shift Apache ActiveMQ storage backends from EFS to EBS GP3.
- **Why It Saves Money:** EFS storage costs $0.30 per GB-month, while EBS GP3 costs just $0.10 per GB-month.
- **Implementation Steps:** 
  1. Assess if Multi-AZ active/standby is strictly reliant on the EFS shared filesystem for your specific architecture.
  2. Provision a new broker with EBS GP3 and migrate messages/traffic.
- **Estimated Savings:** 66% on storage costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Validation of durability requirements.

#### 9. Leverage AWS Enterprise Discount Program (EDP)
- **What:** Include Amazon MQ spend in the organization's overarching EDP volume commitment.
- **Why It Saves Money:** Amazon MQ does not offer native Reserved Instances or Savings Plans. EDP is the primary vehicle for discounting the raw hourly rate.
- **Implementation Steps:** 
  1. Forecast Amazon MQ annual spend.
  2. Provide forecast to the FinOps/Procurement team negotiating the EDP.
- **Estimated Savings:** 5-15% across all MQ spend.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Overall AWS spend high enough to qualify for EDP.

#### 10. Refactor New Services to Serverless Messaging (SQS/SNS)
- **What:** Use Amazon SQS and SNS for new microservices instead of provisioning new Amazon MQ brokers.
- **Why It Saves Money:** SQS/SNS charge per request ($0.40-$0.50 per million) and scale to $0.00 when idle. MQ has a fixed base cost of ~$1,200/year minimum per environment.
- **Implementation Steps:** 
  1. Enforce architectural standards preventing MQ use for brand new cloud-native apps.
  2. Implement SQS/SNS using AWS SDKs instead of JMS/AMQP.
- **Estimated Savings:** Up to 99% for low-to-medium throughput, bursty workloads.
- **Risk Level:** High (Requires code changes)
- **Implementation Scope:** Engineer/DevOps | Architecture
- **Prerequisites:** Workload does not rely on legacy JMS features like distributed transactions.

#### 11. Refactor Event Routing to Amazon EventBridge
- **What:** Replace topic-based routing on Amazon MQ with Amazon EventBridge for Pub/Sub architectures.
- **Why It Saves Money:** EventBridge charges per published event ($1.00/million) with no hourly broker fees, saving money for sparse, event-driven integrations.
- **Implementation Steps:** 
  1. Identify MQ topics used solely for internal event broadcasting.
  2. Rewrite publisher/consumer logic to use EventBridge APIs.
- **Estimated Savings:** Eliminates fixed compute base costs entirely for event routing.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Architecture supports EventBridge latency SLAs.

#### 12. Consolidate Multi-Tenant Brokers
- **What:** Combine multiple small, dedicated Amazon MQ brokers into a single, slightly larger shared broker.
- **Why It Saves Money:** Running five `mq.m7g.medium` brokers ($720/mo) is more expensive than running one `mq.m5.large` broker ($210/mo) heavily utilized with logical segregation.
- **Implementation Steps:** 
  1. Use RabbitMQ Virtual Hosts (vhosts) or ActiveMQ authorization plugins to isolate applications.
  2. Migrate applications to connect to the shared broker.
- **Estimated Savings:** 50-70% depending on consolidation ratio.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Centralized management of the broker and strict access controls.

#### 13. Implement Producer-Side Payload Compression
- **What:** Compress message payloads (e.g., GZIP, Snappy) in the producer application before sending to Amazon MQ.
- **Why It Saves Money:** Reduces the GBs of data transferred across AZs and the GB-hours of storage accrued on the broker.
- **Implementation Steps:** 
  1. Update producer SDK to compress the payload.
  2. Update consumer SDK to decompress the payload.
- **Estimated Savings:** 10-50% on storage and data transfer costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Applications can tolerate minimal CPU overhead for compression.

#### 14. Tear Down Non-Prod Brokers via IaC Nightly
- **What:** Automate the destruction of dev/test MQ brokers at night and recreate them in the morning.
- **Why It Saves Money:** Because MQ brokers cannot be "paused" or "stopped" like EC2 instances, they must be destroyed to stop hourly billing (saving ~65% if off 16 hours a day + weekends).
- **Implementation Steps:** 
  1. Define broker infrastructure strictly in Terraform/CDK.
  2. Use CI/CD schedules (e.g., GitHub Actions) to run `terraform destroy` at 7 PM and `terraform apply` at 7 AM.
- **Estimated Savings:** 65-70% on non-prod broker compute.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ephemeral non-prod environments where message persistence overnight is not required.

#### 15. Optimize Consumer Auto-Scaling to Reduce Storage
- **What:** Aggressively scale consumer applications based on queue depth to process messages as fast as possible.
- **Why It Saves Money:** MQ bills for storage per GB-month. If messages pile up, storage footprint increases. Faster consumption keeps the footprint near zero.
- **Implementation Steps:** 
  1. Configure Target Tracking scaling on ECS/EKS using the MQ `QueueSize` metric.
  2. Ensure consumers drain messages quickly during traffic spikes.
- **Estimated Savings:** 5-15% on storage costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Consumer application is highly scalable and stateless.

#### 16. Maximize AWS Free Tier for R&D
- **What:** Ensure R&D and sandbox accounts leverage the Amazon MQ free tier for prototyping.
- **Why It Saves Money:** New AWS accounts include 750 free hours of micro broker usage and 1 GB storage per month for a year.
- **Implementation Steps:** 
  1. Spin up sandbox projects in fresh AWS Organization member accounts.
  2. Use the free tier eligible instance class (historically `mq.t3.micro`, verify current eligible type post-deprecation).
- **Estimated Savings:** Up to $105/month during R&D phases.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Applicable only for the first 12 months of a new AWS account.

#### 17. Implement Strict Chargeback Tagging
- **What:** Tag every Amazon MQ broker with a `CostCenter` or `ApplicationId`.
- **Why It Saves Money:** Exposes the cost of messaging infrastructure to the application teams using it, driving natural rightsizing and architecture optimization behaviors.
- **Implementation Steps:** 
  1. Enforce a Tag Policy in AWS Organizations for the `mq:broker` resource type.
  2. Activate tags in Billing and Cost Management for Cost Allocation.
- **Estimated Savings:** 5-10% through behavioral changes.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Leadership
- **Prerequisites:** Established tagging taxonomy.

#### 18. Keep Traffic Intra-AZ for Single-AZ Brokers
- **What:** Ensure that EC2/ECS/EKS instances acting as message consumers reside in the same Availability Zone as your Single-AZ Amazon MQ broker.
- **Why It Saves Money:** AWS charges $0.01 per GB for cross-AZ data transfer. Keeping producers, brokers, and consumers in one AZ eliminates this fee.
- **Implementation Steps:** 
  1. Determine the AZ of the active MQ broker node.
  2. Pin consumer autoscaling groups or subnets to that specific AZ for dev/test environments.
- **Estimated Savings:** $0.01 per GB of processed messages.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Only recommended for non-production (Single-AZ) deployments.

#### 19. Deploy VPC Endpoints / VPC Peering over NAT
- **What:** Route cross-VPC traffic to Amazon MQ via VPC Peering or Transit Gateway rather than NAT Gateways.
- **Why It Saves Money:** Routing traffic from a private subnet in VPC A out through a NAT Gateway to hit VPC B incurs a $0.045/GB NAT data processing fee.
- **Implementation Steps:** 
  1. Audit network routes using VPC Flow Logs.
  2. Establish VPC Peering between the consumer VPC and the MQ VPC.
- **Estimated Savings:** $0.045 per GB of inter-VPC traffic.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps (Network)
- **Prerequisites:** Non-overlapping CIDR blocks.

#### 20. Disable Public Accessibility
- **What:** Do not expose Amazon MQ brokers to the public internet unless absolutely required.
- **Why It Saves Money:** Publicly accessible brokers incur standard Internet Egress fees ($0.09/GB) when serving external consumers, and are susceptible to bot traffic/DDoS which inflates data processing and compute costs.
- **Implementation Steps:** 
  1. Set `PubliclyAccessible` to `false` in the broker configuration.
  2. Use AWS Client VPN or Site-to-Site VPN for external administrative access.
- **Estimated Savings:** Eliminates unquantifiable risk of excessive egress billing from malicious traffic.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | Security
- **Prerequisites:** Internal networking capabilities configured.
