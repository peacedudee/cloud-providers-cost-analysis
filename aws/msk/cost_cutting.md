# Cost-Cutting Playbook: Amazon MSK (Managed Streaming for Apache Kafka)

> **Companion File:** [msk.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/msk/msk.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon MSK is a fully managed Apache Kafka service available in **Provisioned Clusters** (broker nodes + EBS storage + cross-AZ inter-broker replication traffic) or **MSK Serverless** (cluster base + partition hours + data volume).

Key billing traps include:
1. **MSK Serverless Idle Base Trap:** MSK Serverless charges a flat **$0.75 per cluster-hour ($547.50/month base)** regardless of traffic, making it 5x more expensive than provisioned `kafka.t3.small` nodes ($94.80/mo) for low-volume dev/test apps!
2. **EBS Over-Provisioning on Brokers:** Keeping historical Kafka messages on expensive broker EBS disks ($0.10/GB-mo) instead of offloading to S3 Tiered Storage ($0.060/GB-mo).
3. **Inter-Broker Replication Tax:** Kafka replicates messages across 3 Availability Zones, incurring standard cross-AZ data egress charges ($0.01/GB).

This playbook provides **18 actionable strategies** across six operational categories, delivering an estimated **30–65% reduction in total MSK spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Bypass MSK Serverless for Dev/Test (Use 3 `kafka.t3.small` Nodes):** Replaces the $547.50/mo flat Serverless base fee with a ~$95.00/mo provisioned cluster (saving $452/mo per dev cluster).
2. **Enable Tiered Storage to Offload Historical Data to S3:** Shifts cold message data from broker EBS ($0.10/GB-mo) to S3 ($0.060/GB-mo) for a **40% storage discount**.
3. **Enable Producer Message Compression (LZ4 / Snappy):** Reduces message sizes by 50–80%, directly cutting EBS disk storage, network egress, and cross-AZ replication fees.

---

## Strategy Categories

### 1. Waste Elimination & Serverless vs Provisioned Sizing

#### 1. Avoid MSK Serverless for Low-Traffic Dev/Test Environments
- **What:** Use 3-node Provisioned MSK clusters (`kafka.t3.small`) for non-production environments instead of MSK Serverless.
- **Why It Saves Money:** MSK Serverless charges a flat **$0.75/cluster-hour base ($547.50/month)** plus partition fees ($0.0015/part-hr). Running a 3-node `kafka.t3.small` provisioned cluster costs **$0.13/hr ($94.80/month)** total. MSK Serverless in dev wastes **$452.70/month per cluster**!
- **Detailed Implementation Steps:**
  1. Audit MSK Serverless clusters in dev/staging accounts.
  2. Create 3-node provisioned `kafka.t3.small` cluster via Terraform:
     ```hcl
     resource "aws_msk_cluster" "dev_kafka" {
       cluster_name           = "dev-kafka-cluster"
       kafka_version          = "3.6.0"
       number_of_broker_nodes = 3
       broker_node_group_info {
         instance_type = "kafka.t3.small"
         client_subnets = module.vpc.private_subnets
         storage_info {
           ebs_storage_info {
             volume_size = 50
           }
         }
       }
     }
     ```
  3. Terminate MSK Serverless cluster.
- **Estimated Savings:** **$452.70 per month saved** per non-prod cluster (82.7% reduction).
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod throughput assessment.

#### 2. Terminate Unused or Abandoned MSK Clusters
- **What:** Identify and delete MSK clusters with zero producer write activity over 14 days.
- **Why It Saves Money:** A 3-node `kafka.m5.large` provisioned cluster costs **$459.90/month** in compute alone.
- **Detailed Implementation Steps:**
  1. Check CloudWatch metric `BytesInPerSec = 0` over 14 days across clusters.
  2. Delete cluster:
     ```bash
     aws msk delete-cluster --cluster-arn arn:aws:kafka:us-east-1:123456789012:cluster/dev-cluster/xxx
     ```
- **Estimated Savings:** 100% of idle cluster costs ($459.90/mo per `m5.large` cluster).
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Producer/consumer activity audit.

---

### 2. Storage Tiering & Architecture Optimization

#### 3. Enable MSK Tiered Storage (Offload EBS to S3)
- **What:** Turn on **Tiered Storage** on Provisioned MSK clusters, setting short local broker retention (e.g. 4–12 hours) and offloading older message logs to S3 Tiered Storage.
- **Why It Saves Money:** Broker EBS storage costs **$0.10 per GB-month**. S3 Tiered Storage costs **$0.060 per GB-month** — a **40% direct storage discount**. Offloading 10 TB of historical messages saves **$400.00/month**.
- **Detailed Implementation Steps:**
  1. Enable Tiered Storage on cluster via AWS CLI:
     ```bash
     aws msk update-storage \
       --cluster-arn arn:aws:kafka:us-east-1:123456789012:cluster/prod-cluster/xxx \
       --current-version K1KSBERTV821XO \
       --storage-mode TIERED
     ```
  2. Update topic retention settings to tier to S3:
     ```bash
     kafka-configs.sh --bootstrap-server broker-url:9098 \
       --entity-type topics --entity-name telemetry-topic \
       --alter --add-config remote.storage.enable=true,local.retention.ms=14400000
     ```
- **Estimated Savings:** 40% reduction in historical message storage costs.
- **Risk Level:** Zero risk (transparent read retrieval for consumers).
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** MSK cluster running Kafka version 2.8.2+.

#### 4. Provision Minimum EBS Storage per Broker (Use Storage Auto-Scaling)
- **What:** Configure initial broker EBS volume size to minimum baseline (e.g. 100 GB) and enable **MSK Storage Auto-Scaling**.
- **Why It Saves Money:** Prevents static over-provisioning of 1 TB+ per broker upfront. Storage auto-scaling expands EBS volume sizes dynamically when broker storage reaches 80% capacity.
- **Detailed Implementation Steps:**
  1. Register MSK broker storage as scalable target in Application Auto Scaling.
  2. Set target tracking policy to 80% disk space utilization.
- **Estimated Savings:** 30–60% reduction in idle provisioned EBS storage fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** MSK cluster setup.

---

### 3. Compute Rightsizing & Graviton Migration

#### 5. Migrate Broker Instance Families to AWS Graviton (`kafka.m7g`)
- **What:** Update provisioned broker node instance types from x86 (`kafka.m5.large`) to Graviton-based families (`kafka.m7g.large`).
- **Why It Saves Money:** `kafka.m7g.large` ($0.1785/hr) is **15% cheaper** than `kafka.m5.large` ($0.2100/hr) while providing up to 25% higher message throughput per broker.
- **Detailed Implementation Steps:**
  1. Update broker instance type in AWS Console or CLI:
     ```bash
     aws msk update-broker-count \
       --cluster-arn arn-xxx \
       --target-number-of-broker-nodes 3
     ```
  2. Modify broker instance type to `kafka.m7g.large` (MSK performs a rolling, zero-downtime broker update).
- **Estimated Savings:** **15% direct compute savings** + performance boost.
- **Risk Level:** Zero risk (rolling update preserves cluster availability).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 6. Optimize Broker Node Sizing for Production Clusters
- **What:** Downsize broker node instance size (e.g. from `kafka.m5.2xlarge` to `kafka.m5.large`) where `CpuUser` is < 25% and network throughput is well within instance limits.
- **Why It Saves Money:** Halving broker node size cuts compute costs by 50% ($460/mo saved across a 3-broker cluster).
- **Detailed Implementation Steps:**
  1. Review CloudWatch metrics `CpuUser`, `MemoryUsed`, and `BytesInPerSec`.
  2. Trigger rolling broker modification.
- **Estimated Savings:** 50% compute node savings per step-down.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Throughput baseline verification.

---

### 4. Partition & Serverless Cost Control

#### 7. Prevent Un-Monitored Partition Sprawl on MSK Serverless
- **What:** Restrict the total partition count created on MSK Serverless clusters.
- **Why It Saves Money:** MSK Serverless bills **$0.0015 per partition-hour** ($1.095/mo per partition). Creating 1,000 unneeded partitions adds **$1,095.00/month** in pure partition maintenance fees!
- **Detailed Implementation Steps:**
  1. Audit active partitions via Kafka CLI:
     ```bash
     kafka-topics.sh --bootstrap-server serverless-url:9098 --list | xargs -n1 kafka-topics.sh --bootstrap-server serverless-url:9098 --describe --topic
     ```
  2. Keep partition counts aligned strictly with consumer group concurrency needs (e.g. 6–12 partitions per topic instead of 100).
- **Estimated Savings:** 50–80% reduction in MSK Serverless partition fees.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Topic partition schema review.

---

### 5. Compression & Data Transfer Optimization

#### 8. Enforce Producer Message Compression (LZ4 / Snappy / Gzip)
- **What:** Configure Kafka producers (`compression.type = lz4` or `snappy`) in application code.
- **Why It Saves Money:** Compresses message payloads by **50–80%** before transmission. Slashes:
  1. Broker EBS disk storage ($0.10/GB-mo)
  2. Inter-broker cross-AZ replication data transfer fees ($0.01/GB)
  3. MSK Serverless data ingestion fees ($0.10/GB).
- **Detailed Implementation Steps:**
  1. Update Java / Python / Go Kafka producer properties:
     ```java
     props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "lz4");
     ```
- **Estimated Savings:** 50–80% reduction in total data ingestion and storage fees.
- **Risk Level:** Zero risk (Kafka consumers handle decompression transparently).
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Producer code configuration update.

#### 9. Minimize Inter-Broker Cross-AZ Replication Egress
- **What:** Align high-volume producer and consumer workloads to connect to brokers in the same Availability Zone where possible.
- **Why It Saves Money:** Cross-AZ inter-broker message replication incurs standard data transfer egress fees (**$0.01/GB**). Passing 100 TB/month across AZs adds $1,000/mo in network taxes.
- **Detailed Implementation Steps:**
  1. Ensure client subnets match broker subnets across all 3 AZs.
  2. Enable `client.rack` aware consumer fetching (`rack.id` setting) in Kafka consumers (Kafka 2.2+ feature).
- **Estimated Savings:** 30–50% reduction in inter-broker data transfer fees.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Kafka 2.2+ client SDK.

---

### 6. Observability & Maintenance Optimization

#### 10. Disable PER_TOPIC_PER_BROKER CloudWatch Metrics
- **What:** Set MSK Enhanced Monitoring level to `PER_BROKER` or `DEFAULT` instead of `PER_TOPIC_PER_BROKER`.
- **Why It Saves Money:** `PER_TOPIC_PER_BROKER` monitoring publishes thousands of custom metrics to CloudWatch ($0.30/metric-mo), generating massive CloudWatch billing spikes on clusters with many topics.
- **Detailed Implementation Steps:**
  1. Update cluster monitoring level via CLI:
     ```bash
     aws msk update-cluster-configuration \
       --cluster-arn arn-xxx \
       --enhanced-monitoring PER_BROKER
     ```
- **Estimated Savings:** 70–90% reduction in MSK CloudWatch custom metric fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Confirm per-topic metric granularity is not required.

#### 11. Restrict Retention Windows on Non-Production Topics
- **What:** Configure default topic retention (`retention.ms`) in non-prod clusters to 4 hours.
- **Why It Saves Money:** Prevents test topics from consuming broker EBS storage.
- **Detailed Implementation Steps:**
  1. Set topic retention:
     ```bash
     kafka-configs.sh --bootstrap-server broker-url:9092 --entity-type topics --entity-name dev-topic --alter --add-config retention.ms=14400000
     ```
- **Estimated Savings:** 80% storage savings on test topics.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Non-prod retention agreement.

#### 12. Enable In-Place Apache Kafka Version Upgrades
- **What:** Perform in-place Kafka version upgrades (`aws msk update-cluster-kafka-version`) to access performance enhancements.
- **Why It Saves Money:** Newer Kafka releases include memory and replication efficiency improvements.
- **Detailed Implementation Steps:**
  1. Upgrade cluster to latest supported version (e.g. 3.6.0).
- **Estimated Savings:** Operational efficiency.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Version compatibility check.

#### 13. Optimize MSK Connect Connector Task Sizing
- **What:** Right-size worker task counts on MSK Connect connectors (`capacity.autoscaling` min/max MCU).
- **Why It Saves Money:** Prevents MSK Connect MCU over-provisioning ($0.11/MCU-hr).
- **Detailed Implementation Steps:**
  1. Configure MSK Connect auto-scaling policy (`minWorkerCount = 1`, `maxWorkerCount = 4`).
- **Estimated Savings:** 30–50% connector compute savings.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** MSK Connect usage.

#### 14. Deploy Free Gateway VPC Endpoints for S3 Tiered Storage Access
- **What:** Ensure MSK broker subnets route S3 traffic via Gateway VPC Endpoints.
- **Why It Saves Money:** Offloading Tiered Storage data to S3 via NAT Gateway incurs $0.045/GB processing fees. Gateway Endpoints make S3 transfer **100% FREE ($0.00/GB)**.
- **Detailed Implementation Steps:**
  1. Attach Gateway VPC Endpoint to MSK route table.
- **Estimated Savings:** Saves $45.00/TB on Tiered Storage offloads.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC Gateway Endpoint setup.

#### 15. Consolidate Low-Volume Topics into Shared Topic Partitions
- **What:** Combine single-message micro-topics into shared multi-tenant topics using message header routing.
- **Why It Saves Money:** Reduces cluster metadata and partition overhead.
- **Detailed Implementation Steps:**
  1. Update producer schema design.
- **Estimated Savings:** Partition maintenance savings.
- **Risk Level:** Medium.
- **Implementation Scope:** Data Architect
- **Prerequisites:** Schema refactoring.

#### 16. Monitor and Alert on Broker Disk Space Utilization
- **What:** Put CloudWatch alarm on `KafkaDataLogsDiskUsed` (> 80%).
- **Why It Saves Money:** Prevents broker disk full states that trigger emergency storage expansion.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm for `KafkaDataLogsDiskUsed`.
- **Estimated Savings:** Proactive risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch alarm setup.

#### 17. Audit and Remove Stale MSK Cluster Configurations
- **What:** Delete legacy custom configuration revisions in MSK console.
- **Why It Saves Money:** Administrative hygiene.
- **Detailed Implementation Steps:**
  1. Delete unused configuration versions via CLI.
- **Estimated Savings:** Administrative efficiency.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 18. Leverage Account-Wide Savings Plans for Underlying EC2 Brokers
- **What:** Ensure underlying EC2 broker node spend is covered by Compute Savings Plans.
- **Why It Saves Money:** Slashes node compute rates by up to 55%.
- **Detailed Implementation Steps:**
  1. Include MSK broker spend in Savings Plan commitment calculations.
- **Estimated Savings:** 20–55% compute discount.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Savings Plan commitment.

---

## Cross-Service Synergies

```
[ Amazon MSK Cluster ] 
        │
        ├──(Dev Cluster Mode)──> [ 3 x kafka.t3.small Nodes ] (Saves $452/mo vs Serverless $547 base)
        │
        ├──(Storage Offloading)─> [ S3 Tiered Storage ] (40% discount vs broker EBS $0.10/GB)
        │
        └──(Message Compression)─> [ Producer LZ4 Compression ] (Saves 50-80% on storage & egress)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Kafka:m5.large`, `Kafka:t3.small`, `Kafka:Serverless-ClusterHour`, `Kafka:Serverless-PartitionHour`, `Kafka:StorageUsage`.
- `line_item_resource_id`: MSK Cluster ARN.

### B. CloudWatch & Cluster Metrics
- `AWS/Kafka` Namespace: `BytesInPerSec`, `BytesOutPerSec`, `CpuUser`, `KafkaDataLogsDiskUsed`, `GlobalTopicCount`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "MSK-SRV-001",
  "service": "MSK",
  "category": "Waste Elimination & Serverless Sizing",
  "resource_id": "arn:aws:kafka:us-east-1:123456789012:cluster/dev-analytics-serverless/xxx",
  "resource_name": "dev-analytics-serverless",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "deployment_mode": "SERVERLESS",
    "cluster_base_cost_usd": 547.50,
    "partition_count": 150,
    "monthly_cost_usd": 711.75
  },
  "recommended_config": {
    "deployment_mode": "PROVISIONED",
    "instance_type": "kafka.t3.small",
    "broker_count": 3,
    "projected_monthly_cost_usd": 94.80
  },
  "financial_impact": {
    "monthly_savings_usd": 616.95,
    "annual_savings_usd": 7403.40,
    "savings_percentage": 86.7
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "Dev environment processes < 50 GB/mo; t3.small 3-node cluster provides ample capacity."
  },
  "implementation": {
    "scope": "Engineer/DevOps",
    "effort_estimate": "1-2 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Dev Serverless -> Provisioned t3.small**| 4 | $2,847.00 | $2,467.80 | 86.7% | Low |
| **Tiered Storage (S3 Offloading)** | 6 | $8,500.00 | $3,400.00 | 40.0% | Zero |
| **Producer LZ4 Compression** | 10 | $12,400.00 | $7,440.00 | 60.0% | Zero |
| **Broker Graviton Migration (m7g)** | 8 | $9,200.00 | $1,380.00 | 15.0% | Zero |
| **Total** | **28** | **$32,947.00** | **$14,687.80** | **44.5%** | -- |
