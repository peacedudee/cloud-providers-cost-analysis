# Cost-Cutting Playbook: Amazon DynamoDB

> **Companion File:** [dynamodb.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/dynamodb/dynamodb.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon DynamoDB is a fully managed, serverless NoSQL database service with single-digit millisecond latency. Billing is driven by Capacity Mode (On-Demand pay-per-request vs Provisioned pay-per-hour), Table Class (Standard vs Standard-IA), item storage volume, global table replication, and backup retention. A major price reduction cut On-Demand rates in half in late 2024, and Database Savings Plans introduced in late 2025 unlocked commitment discounts.

This playbook presents **18 actionable strategies** across six operational categories, delivering an estimated **30–70% reduction in total DynamoDB expenditure**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Switch Flat/Predictive Tables from On-Demand to Provisioned Auto-Scaling:** Consistently utilized tables (> 15% average utilization) are up to **5x cheaper** in Provisioned mode.
2. **Enforce Eventually Consistent Reads in Client Code:** Cuts read capacity unit (RCU/RRU) consumption by **50% instantly**.
3. **Enable Free Time-To-Live (TTL) Auto-Deletions:** Deletes expired session or audit items automatically for **FREE ($0.00 WCU)**, reclaiming storage capacity.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Audit & Remove Unnecessary Global Secondary Indexes (GSIs)
- **What:** Identify and delete Global Secondary Indexes that record zero or negligible read activity over 30 days.
- **Why It Saves Money:** GSIs act as cost multipliers. Every table write MUST be replicated to all GSIs, consuming write capacity (WCU/WRU) and storage for *every* index. Deleting 3 unused GSIs cuts table write costs by 75%!
- **Implementation Steps:**
  1. Check CloudWatch metric `ConsumedReadCapacityUnits` for each GSI over 30 days.
  2. Identify GSIs with 0 reads.
  3. Delete GSI: `aws dynamodb update-table --table-name name --global-secondary-index-updates '[{"Delete":{"IndexName":"gsi-name"}}]'`.
- **Estimated Savings:** 25–75% reduction in total table write and storage costs.
- **Risk Level:** Medium (confirm application code does not query index).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** GSI query metrics audit.

#### 2. Enable Free TTL (Time-To-Live) for Automatic Item Expiration
- **What:** Configure TTL on tables storing ephemeral data (session states, temporary tokens, audit logs, caching layers).
- **Why It Saves Money:** DynamoDB deletes expired TTL items as a background process for **100% FREE ($0.00 WCU/WRU)** without consuming any write capacity, automatically reclaiming storage ($0.25/GB-mo). Manual delete operations consume 1 WRU per KB.
- **Implementation Steps:**
  1. Add epoch timestamp attribute `ttl_timestamp` to table items.
  2. Enable TTL on table: `aws dynamodb update-time-to-live --table-name name --time-to-live-specification "Enabled=true, AttributeName=ttl_timestamp"`.
- **Estimated Savings:** 100% savings on deletion write capacity + storage recovery.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Timestamp attribute inclusion in application writes.

#### 3. Clean Up Abandoned Manual Table Backups
- **What:** Locate and delete old manual table snapshots that exceed compliance retention requirements.
- **Why It Saves Money:** Manual backups cost **$0.10 per GB-month** indefinitely until explicitly deleted.
- **Implementation Steps:**
  1. List backups: `aws dynamodb list-backups`.
  2. Delete stale manual backups: `aws dynamodb delete-backup --backup-arn arn-xxx`.
- **Estimated Savings:** 100% of abandoned backup storage fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Backup retention policy review.

---

### 2. Rightsizing & Capacity Mode Optimization

#### 4. Switch Steady-State Tables from On-Demand to Provisioned Mode with Auto-Scaling
- **What:** Evaluate table utilization patterns; convert tables with predictable, continuous traffic from On-Demand mode to Provisioned Capacity mode with Auto Scaling (target utilization 70%).
- **Why It Saves Money:** **The Break-Even Rule:** If average table utilization exceeds **15%**, Provisioned Mode is vastly cheaper. On-Demand writes cost $0.625/M WRUs; Provisioned WCU-hours cost ~$0.47/WCU-month ($0.00065/hr). For flat workloads, Provisioned mode is **up to 70–80% cheaper**.
- **Implementation Steps:**
  1. Analyze 30-day read/write request patterns in CloudWatch.
  2. If traffic is steady, change mode: `aws dynamodb update-table --table-name name --billing-mode PROVISIONED --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=10`.
  3. Attach Application Auto Scaling targets for RCU/WCU (Min 5, Max 500, Target 70%).
- **Estimated Savings:** 50–80% reduction in throughput billing for flat workloads.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Traffic predictability verification.

#### 5. Keep On-Demand Mode ONLY for Spiky, Low-Utilization, or Dev Tables
- **What:** Ensure development tables, staging environments, and highly erratic production tables (idle 95% of the day) remain in On-Demand Mode.
- **Why It Saves Money:** On-Demand mode scales compute costs to **$0.00 when idle**. Setting Provisioned mode on an idle dev table bills fixed hourly capacity charges 24/7 for zero traffic.
- **Implementation Steps:**
  1. Audit dev/test tables; switch any fixed Provisioned dev tables to On-Demand: `billing-mode PAY_PER_REQUEST`.
- **Estimated Savings:** 80–90% reduction in non-prod database throughput spend.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 6. Optimize Auto-Scaling Min/Max RCU/WCU Thresholds
- **What:** Review Application Auto Scaling targets on Provisioned tables; adjust minimum RCU/WCU thresholds to match actual off-peak baselines.
- **Why It Saves Money:** Setting `MinCapacity = 500` on a table that only needs 10 WCUs overnight wastes 490 WCU-hours continuously.
- **Implementation Steps:**
  1. Check off-peak RCU/WCU utilization in CloudWatch.
  2. Update auto-scaling minimum capacity via AWS CLI/Terraform.
- **Estimated Savings:** 20–50% savings on overnight provisioned capacity.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch capacity utilization analysis.

---

### 3. Commitment Discounts

#### 7. Apply Database Savings Plans to DynamoDB Throughput
- **What:** Purchase 1-year or 3-year Database Savings Plans to cover baseline DynamoDB throughput.
- **Why It Saves Money:** Database Savings Plans offer discounts up to **18% on On-Demand throughput** (WRU/RRU) and **12% on Provisioned capacity** (WCU/RCU).
- **Implementation Steps:**
  1. Evaluate minimum steady-state DynamoDB hourly spend in Cost Explorer.
  2. Purchase Database Savings Plan commitment.
- **Estimated Savings:** 12–18% direct discount on throughput billing.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** 1-year spend commitment approval.

---

### 4. Architecture Changes

#### 8. Enforce Eventually Consistent Reads by Default in Client Code
- **What:** Configure database SDK client query calls explicitly with `ConsistentRead = false` (eventual consistency).
- **Why It Saves Money:** 1 strongly consistent read (up to 4 KB) costs **1 RRU/RCU**. 2 eventually consistent reads (up to 4 KB) cost **1 RRU/RCU**. Eventual consistency is **50% CHEAPER**.
- **Implementation Steps:**
  1. Audit application codebase for `ConsistentRead: true` flags.
  2. Remove strong consistency requirement unless strict financial/state consistency is mandatory.
- **Estimated Savings:** 50% direct reduction in read throughput billing.
- **Risk Level:** Low (most web/mobile apps tolerate 100ms eventual consistency).
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Application code audit.

#### 9. Implement S3 "Pointer Pattern" for Large Payload Items (> 2 KB)
- **What:** Offload large JSON attributes, binary blobs, or text payloads (> 2 KB) from DynamoDB items to Amazon S3, storing only the **S3 S3-URI pointer** in the DynamoDB table.
- **Why It Saves Money:** DynamoDB meters writes in **1 KB blocks** ($0.625/M WRUs) and storage at **$0.25/GB-mo**. Writing a 50 KB item to DynamoDB consumes **50 WRUs** and costs $0.25/GB-mo. Storing the 50 KB file in S3 costs **1 WRU** (pointer write) and **$0.023/GB-mo** (S3 storage is 10x cheaper!).
- **Implementation Steps:**
  1. Add middleware to application data layer: If payload > 2 KB, upload payload to S3 bucket `s3://app-blobs/item-id` and write item containing `{id: "123", blob_ptr: "s3://app-blobs/item-id"}` to DynamoDB.
- **Estimated Savings:** 80–95% cost reduction for large item architectures.
- **Risk Level:** Medium (adds S3 read/write step to data layer).
- **Implementation Scope:** Software Engineer
- **Prerequisites:** S3 bucket provisioned.

#### 10. Cache High-Velocity Read Keys using Amazon DAX or ElastiCache
- **What:** Place Amazon DynamoDB Accelerator (DAX) or an ElastiCache Redis cluster in front of read-heavy DynamoDB tables.
- **Why It Saves Money:** Microsecond in-memory caching absorbs millions of repeated GET requests, preventing read capacity unit consumption on the table.
- **Implementation Steps:**
  1. Provision DAX cluster.
  2. Swap application DynamoDB client SDK with DAX SDK (drop-in API compatible).
- **Estimated Savings:** 70–95% reduction in read throughput costs for hot keys.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** DAX cluster setup.

#### 11. Adopt Single-Table Design Patterns to Reduce Table & GSI Overhead
- **What:** Structure NoSQL schemas using Single-Table Design patterns (combining related entities into one table via composite Partition/Sort keys `PK`/`SK`).
- **Why It Saves Money:** Eliminates the need for multiple independent tables each carrying minimum provisioned thresholds and redundant GSIs.
- **Implementation Steps:**
  1. Design composite PK/SK schema (e.g. `USER#123`, `ORDER#456`).
- **Estimated Savings:** 30–60% structural schema savings.
- **Risk Level:** High (requires application data model refactoring).
- **Implementation Scope:** Database Architect / Engineer
- **Prerequisites:** Schema redesign.

---

### 5. Storage & Data Tiering

#### 12. Migrate Cold Tables to DynamoDB Standard-IA Table Class
- **What:** Change table class from `DynamoDB Standard` to `DynamoDB Standard-IA` (Infrequent Access) for tables where storage makes up > 50% of the total monthly bill.
- **Why It Saves Money:** Standard-IA reduces data storage costs by **60%** ($0.10/GB-mo vs $0.25/GB-mo). However, throughput rates are 25% higher. If storage dominates the bill, net savings are substantial.
- **Implementation Steps:**
  1. Compare Storage cost vs Throughput cost in CUR.
  2. If Storage > 50% of total table bill, update table class: `aws dynamodb update-table --table-name name --table-class STANDARD_INFREQUENT_ACCESS`.
  3. Operation executes online with zero downtime.
- **Estimated Savings:** Up to 60% reduction in table storage spend.
- **Risk Level:** Zero risk (easily revertible online).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Financial cost distribution analysis.

#### 13. Compress Attribute Field Names and String Values
- **What:** Shorten JSON attribute key names in table items (e.g. rename `"user_transaction_history_id"` to `"tx_id"`).
- **Why It Saves Money:** DynamoDB includes attribute key string lengths when calculating 1 KB write boundaries and total GB storage. Shorter keys keep items below 1 KB / 4 KB billing thresholds.
- **Implementation Steps:**
  1. Update ORM / application attribute mappings.
- **Estimated Savings:** 10–30% capacity unit and storage reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Code attribute mapping update.

---

### 6. Pricing Model & Network Optimization

#### 14. Deploy Free Gateway VPC Endpoints for DynamoDB
- **What:** Create Gateway VPC Endpoints for DynamoDB in all VPC route tables.
- **Why It Saves Money:** Routes DynamoDB API requests from EC2/Lambda over the internal AWS network for **100% FREE ($0.00/GB)**, avoiding NAT Gateway processing taxes ($0.045/GB).
- **Implementation Steps:**
  1. Create Gateway Endpoint: `aws ec2 create-vpc-endpoint --vpc-id vpc-xxx --service-name com.amazonaws.us-east-1.dynamodb --route-table-ids rtb-xxx`.
- **Estimated Savings:** Saves $45.00 per 1 TB of DynamoDB traffic from private subnets.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 15. Audit and Optimize Global Tables Cross-Region Replication
- **What:** Review DynamoDB Global Tables replication; remove unused replica regions or restrict replicated attributes.
- **Why It Saves Money:** Global Tables bill replicated writes as **replicated WRUs** in every destination region PLUS inter-region data transfer fees ($0.01–$0.02/GB). Replicating to 3 regions quadruples write throughput costs!
- **Implementation Steps:**
  1. Audit Global Tables in console.
  2. Remove unnecessary region replicas where local read latency is not required.
- **Estimated Savings:** 50–75% reduction in replication throughput and network charges.
- **Risk Level:** Medium (impacts multi-region failover/read latency).
- **Implementation Scope:** Engineer/DevOps & Architect
- **Prerequisites:** Multi-region RTO/RPO review.

#### 16. Restrict Point-in-Time Recovery (PITR) to Production Tables Only
- **What:** Disable Point-in-Time Recovery (PITR) on development, staging, and transient test tables.
- **Why It Saves Money:** PITR charges **$0.228 per GB-month** for continuous backups. Enabling PITR on dev tables adds unnecessary storage fees equal to active table storage.
- **Implementation Steps:**
  1. Disable PITR on non-prod tables: `aws dynamodb update-continuous-backups --table-name name --point-in-time-recovery-specification PointInTimeRecoveryEnabled=false`.
- **Estimated Savings:** 100% of non-prod continuous backup fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod backup SLA confirmation.

#### 17. Batch Read & Write Operations (BatchGetItem / BatchWriteItem)
- **What:** Refactor application data layer to use `BatchGetItem` (up to 100 items / 16 MB) and `BatchWriteItem` (up to 25 items / 16 MB).
- **Why It Saves Money:** Consolidates HTTP connection overhead and optimizes throughput block rounding.
- **Implementation Steps:**
  1. Replace sequential single-item `GetItem` loops with `BatchGetItem`.
- **Estimated Savings:** 15–30% operational duration savings.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Code refactoring.

#### 18. Monitor & Eliminate Transactional Write Overuse (TransactWriteItems)
- **What:** Reserve transactional writes (`TransactWriteItems`) strictly for operations requiring ACID guarantees across multiple items.
- **Why It Saves Money:** Transactional writes consume **2 WRUs per 1 KB** (2x standard write cost) and perform 2 underlying reads/writes for concurrency checks.
- **Implementation Steps:**
  1. Audit codebase for `TransactWriteItems`.
  2. Replace non-critical transactional writes with standard `PutItem` or conditional writes (`ConditionExpression`).
- **Estimated Savings:** **50% savings** on audited write operations.
- **Risk Level:** Medium (requires transaction safety audit).
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Application transaction logic review.

---

## Cross-Service Synergies

```
[ Amazon DynamoDB ] 
        │
        ├──(Large Payloads > 2KB)──> [ Amazon S3 ] (S3 storage is 10x cheaper $0.023 vs $0.25/GB)
        │
        ├──(Read Offloading)───────> [ DAX / ElastiCache ] (Absorbs hot-key reads in RAM)
        │
        └──(Network Routing)───────> [ Gateway VPC Endpoint ] (100% FREE - Avoids NAT $0.045/GB)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `WriteCapacityUnit-Hrs`, `ReadCapacityUnit-Hrs`, `TimedStorage-ByteHrs`, `PayPerRequestThroughput`, `PITR-Storage-ByteHrs`.
- `line_item_resource_id`: DynamoDB Table Name / ARN.
- `line_item_operation`: `PutItem`, `GetItem`, `Query`, `Scan`, `BatchWriteItem`.

### B. CloudWatch Metrics
- `AWS/DynamoDB` Namespace: `ConsumedReadCapacityUnits`, `ConsumedWriteCapacityUnits`, `ProvisionedReadCapacityUnits`, `ProvisionedWriteCapacityUnits`, `ThrottledRequests`, `AccountProvisionedReadCapacityUtilization`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "DDB-CM-001",
  "service": "DynamoDB",
  "category": "Rightsizing & Capacity Mode",
  "resource_id": "arn:aws:dynamodb:us-east-1:123456789012:table/user-session-store",
  "resource_name": "user-session-store",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "billing_mode": "PAY_PER_REQUEST",
    "table_class": "STANDARD",
    "avg_daily_wru": 45000000,
    "avg_daily_rru": 120000000,
    "monthly_cost_usd": 1125.00
  },
  "recommended_config": {
    "billing_mode": "PROVISIONED",
    "auto_scaling": "Target tracking 70%, Min 50 WCU / 150 RCU",
    "projected_monthly_cost_usd": 380.00
  },
  "financial_impact": {
    "monthly_savings_usd": 745.00,
    "annual_savings_usd": 8940.00,
    "savings_percentage": 66.2
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "Table traffic is steady 24/7; Auto Scaling handles standard peak fluctuations."
  },
  "implementation": {
    "scope": "Engineer/DevOps",
    "effort_estimate": "1 hour",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Capacity Mode (On-Demand -> Prov)** | 8 | $9,400.00 | $6,110.00 | 65.0% | Low |
| **Architecture (Eventually Consistent)**| 22 | $4,800.00 | $2,400.00 | 50.0% | Low |
| **S3 Pointer Pattern (>2KB)** | 5 | $6,200.00 | $4,960.00 | 80.0% | Medium |
| **Waste Elimination (GSIs/TTL)** | 14 | $3,500.00 | $2,100.00 | 60.0% | Low |
| **Total** | **49** | **$23,900.00** | **$15,570.00** | **65.1%** | -- |
