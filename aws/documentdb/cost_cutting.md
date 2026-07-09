# Cost-Cutting Playbook: Amazon DocumentDB (with MongoDB compatibility)

> **Companion File:** [documentdb.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/documentdb/documentdb.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon DocumentDB is a fully managed, MongoDB-compatible document database service utilizing a cloud-native architecture that separates compute instances from distributed cluster storage. Billing components encompass compute instances (`db.r6g`, `db.t3`), storage tiering (Standard vs I/O-Optimized), per-request I/O fees ($0.20/M), and **Extended Support fees** for legacy versions ($0.10–$0.20/vCPU-hr).

Key billing traps include:
1. **DocumentDB v3.6 Extended Support Penalty:** Running legacy v3.6 clusters past end-of-life (March 2026), which levies an extra **$0.10/vCPU-hour ($292.00/mo penalty per 4-vCPU node)** on top of standard compute!
2. **Unindexed Query Scans (Standard Tier):** Queries performing full collection scans generate billions of 8 KB I/O requests ($0.20/M), resulting in massive monthly I/O billing surges.
3. **Multi-AZ Replicas in Non-Production:** Running 2-node clusters (Primary + Reader) in dev/staging environments, doubling compute spend for non-prod databases.

This playbook provides **18 actionable strategies** across six operational categories, delivering an estimated **30–65% reduction in total DocumentDB spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Upgrade Legacy DocumentDB 3.6 Clusters to 4.0 / 5.0:** Immediately eliminates the **$0.10/vCPU-hour Extended Support fee** ($292.00/mo saved per 4-vCPU node).
2. **Prune Reader Replicas in Dev/Test Clusters (Single-Node Config):** Cuts non-production database compute costs by **50%**.
3. **Convert High-I/O Clusters to DocumentDB I/O-Optimized:** Eliminates variable I/O fees ($0.20/M) for clusters where I/O charges exceed 25% of the total monthly bill.

---

## Strategy Categories

### 1. Waste Elimination & Extended Support Avoidance

#### 1. Upgrade DocumentDB 3.6 Clusters to Version 4.0 or 5.0 (Bypass Extended Support)
- **What:** Upgrade legacy Amazon DocumentDB version 3.6 clusters to version 4.0 or 5.0.
- **Why It Saves Money:** Standard support for DocumentDB 3.6 ended on **March 30, 2026**. Extended Support levies a **$0.10 per vCPU-hour surcharge in Years 1–2** and **$0.20 per vCPU-hour in Year 3**. Running a 4-vCPU instance (`db.r6g.xlarge`) on v3.6 incurs a **$292.00/month penalty** in Year 1 and **$584.00/month** in Year 3!
- **Detailed Implementation Steps:**
  1. Identify v3.6 clusters via AWS CLI:
     ```bash
     aws docdb describe-db-clusters \
       --query "DBClusters[?EngineVersion=='3.6.0'].{ID:DBClusterIdentifier,Engine:Engine,Version:EngineVersion}"
     ```
  2. Perform engine upgrade via AWS CLI:
     ```bash
     aws docdb modify-db-cluster \
       --db-cluster-identifier prod-docdb-cluster \
       --engine-version 5.0.0 \
       --apply-immediately
     ```
- **Estimated Savings:** **$292.00 to $584.00 per 4-vCPU node per month** (100% of Extended Support fees).
- **Risk Level:** Medium (requires application regression testing against MongoDB 4.0/5.0 API compatibility).
- **Implementation Scope:** Database Administrator / DevOps
- **Prerequisites:** Backup verification prior to upgrade.

#### 2. Decommission Manual DocumentDB Cluster Snapshots
- **What:** Delete manual cluster snapshots older than 30 days.
- **Why It Saves Money:** Automated backups up to 100% of cluster storage size are free; manual snapshots bill at **$0.095 per GB-month**. 5 TB of manual snapshots costs **$475.00/month**.
- **Detailed Implementation Steps:**
  1. Delete stale manual snapshot: `aws docdb delete-db-cluster-snapshot --db-cluster-snapshot-identifier manual-snap-2025-01-01`.
- **Estimated Savings:** 100% of manual snapshot storage fees ($0.095/GB-mo).
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Backup policy confirmation.

#### 3. Implement Auto-Start/Stop Scheduler on Dev/Test DocumentDB Clusters
- **What:** Automatically stop development and staging DocumentDB clusters during off-hours (nights and weekends).
- **Why It Saves Money:** Stopping clusters eliminates 100% of compute instance billing. Running dev databases 50 hours/week instead of 168 hours/week cuts compute costs by **70%**.
- **Detailed Implementation Steps:**
  1. Stop cluster: `aws docdb stop-db-cluster --db-cluster-identifier dev-docdb-cluster`.
  2. Start cluster: `aws docdb start-db-cluster --db-cluster-identifier dev-docdb-cluster`.
- **Estimated Savings:** **65–70% reduction** in non-production compute spend.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod SLA alignment.

---

### 2. Storage Tiering & Configuration Optimization

#### 4. Evaluate & Convert Clusters to DocumentDB I/O-Optimized Configuration
- **What:** Convert write-heavy DocumentDB clusters from `Standard` to `I/O-Optimized` storage configuration.
- **Why It Saves Money:**
  - **Standard:** Storage is $0.10/GB-mo, but charges **$0.20 per 1 million I/O requests**. Heavy indexing/ETL generating 10 billion I/Os/month pays **$2,000.00/month in I/O fees**.
  - **I/O-Optimized:** Storage is $0.225/GB-mo and compute is 30% higher, but **I/O requests are 100% FREE ($0.00)**.
  - **Decision Rule:** If I/O request fees exceed **25%** of your total monthly DocumentDB bill, switching to I/O-Optimized will reduce total spend.
- **Detailed Implementation Steps:**
  1. Check CUR for `DocDB:StorageIOUsage`.
  2. If I/O fees > 25% of cluster bill, update cluster storage type online:
     ```bash
     aws docdb modify-db-cluster \
       --db-cluster-identifier prod-docdb-cluster \
       --storage-type iopt1 \
       --apply-immediately
     ```
- **Estimated Savings:** 20–50% net monthly bill reduction on write-heavy clusters.
- **Risk Level:** Zero risk (executes online without downtime).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Financial I/O spend analysis.

---

### 3. Compute Rightsizing & Reserved Instances

#### 5. Prune Reader Replicas in Non-Production Environments (Single-Node Config)
- **What:** Run non-production DocumentDB clusters with **0 reader nodes** (Single-Node Primary configuration).
- **Why It Saves Money:** Running a 2-node cluster (1 Primary + 1 Reader) doubles (2x) compute costs. Dev/test clusters do not require failover redundancy.
- **Detailed Implementation Steps:**
  1. Delete reader instance: `aws docdb delete-db-instance --db-instance-identifier dev-docdb-reader-1`.
- **Estimated Savings:** **50% direct reduction** in cluster compute billing.
- **Risk Level:** Low (for non-prod environments).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod SLA alignment.

#### 6. Migrate Provisioned Nodes to AWS Graviton (`db.r6g`)
- **What:** Update instance classes from x86 (`db.r5.xlarge`) to Graviton-based instances (`db.r6g.xlarge`).
- **Why It Saves Money:** `db.r6g.xlarge` ($0.278/hr) is **5% cheaper** than `db.r5.xlarge` ($0.292/hr) while delivering up to 14% higher throughput per core.
- **Detailed Implementation Steps:**
  1. Modify instance class: `aws docdb modify-db-instance --db-instance-identifier prod-docdb-1 --db-instance-class db.r6g.xlarge --apply-immediately`.
- **Estimated Savings:** 5% price reduction + performance boost.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 7. Purchase DocumentDB Reserved Instances for Steady-State Production
- **What:** Purchase 1-year or 3-year Reserved Instances for production `db.r6g` nodes.
- **Why It Saves Money:** Secures discounts of **up to 38% (1-year)** or **up to 55% (3-year)** off standard On-Demand hourly compute rates.
- **Detailed Implementation Steps:**
  1. Purchase RIs via AWS CLI: `aws docdb purchase-reserved-db-instances-offering --reserved-db-instances-offering-id offering-id-12345 --db-instance-count 2`.
- **Estimated Savings:** 35–55% compute savings.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** 1-to-3 year engine commitment.

---

### 4. Query Indexing & Performance Optimization

#### 8. Enforce MongoDB Indexing to Eliminate Full Collection Scans
- **What:** Identify and create compound indexes on unindexed MongoDB collections using `db.collection.explain("executionStats")`.
- **Why It Saves Money:** Unindexed queries must scan every document in a collection from disk, generating billions of billable 8 KB I/O requests ($0.20/M) and driving CPU utilization to 100%. Proper B-tree indexes drop disk I/O reads by 99.9%.
- **Detailed Implementation Steps:**
  1. Run explain plan in MongoDB shell:
     ```javascript
     db.orders.explain("executionStats").find({ customer_id: "C12345" });
     ```
  2. Create missing index:
     ```javascript
     db.orders.createIndex({ customer_id: 1 }, { background: true });
     ```
- **Estimated Savings:** 50–90% reduction in Standard I/O request charges.
- **Risk Level:** Low.
- **Implementation Scope:** Database Administrator / Software Engineer
- **Prerequisites:** Slow query log analysis.

#### 9. Optimize Projection Documents (Return Only Required Fields)
- **What:** Specify projection fields in MongoDB queries (`db.collection.find({}, { field1: 1, field2: 1 })`).
- **Why It Saves Money:** Prevents returning multi-megabyte JSON documents over the network, reducing cluster RAM usage and network transfer overhead.
- **Detailed Implementation Steps:**
  1. Add projection dictionaries to query calls in application code.
- **Estimated Savings:** 20–40% RAM and network efficiency gains.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Query code update.

---

### 5. High Availability & Backup Tuning

#### 10. Downsize Over-Provisioned Compute Nodes
- **What:** Downsize instance classes where CPU is < 20% and memory cache hit ratio is > 99%.
- **Why It Saves Money:** Halving instance size cuts compute costs by 50% ($200+/mo saved per instance).
- **Detailed Implementation Steps:**
  1. Modify instance type: `db.r6g.2xlarge` -> `db.r6g.xlarge`.
- **Estimated Savings:** 50% compute savings per step-down.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Buffer pool hit ratio monitoring.

#### 11. Audit Backup Retention Windows
- **What:** Reduce automated backup retention days from 35 days down to 7 or 14 days for non-critical environments.
- **Why It Saves Money:** Reduces backup storage charges ($0.095/GB-mo) beyond the 100% free baseline.
- **Detailed Implementation Steps:**
  1. Modify retention period: `aws docdb modify-db-cluster --db-cluster-identifier dev-cluster --backup-retention-period 7`.
- **Estimated Savings:** 50–80% backup storage savings beyond free baseline.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compliance retention alignment.

---

### 6. Network & Security Optimization

#### 12. Eliminate Cross-AZ Application-to-Database Traffic
- **What:** Route application queries to DocumentDB instances in the **same Availability Zone**.
- **Why It Saves Money:** Cross-AZ traffic costs **$0.01/GB in each direction ($0.02/GB roundtrip)**. Intra-AZ traffic is **100% FREE ($0.00/GB)**.
- **Detailed Implementation Steps:**
  1. Use AZ-aware connection routing in MongoDB client driver.
- **Estimated Savings:** Saves $20.00/TB on cross-AZ network egress.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Multi-AZ deployment.

#### 13. Enable Performance Insights for Query Diagnostics
- **What:** Turn on Performance Insights to monitor database load.
- **Why It Saves Money:** Pinpoints expensive queries before they cause billing surges.
- **Detailed Implementation Steps:**
  1. Enable Performance Insights via CLI.
- **Estimated Savings:** Operational risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** None.

#### 14. Restrict Public Accessibility (Use Private VPC Subnets)
- **What:** Ensure `PubliclyAccessible` is set to `false`.
- **Why It Saves Money:** Prevents internet data egress charges ($0.09/GB).
- **Detailed Implementation Steps:**
  1. Modify instance network settings.
- **Estimated Savings:** Prevents internet egress fees.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC routing configuration.

#### 15. Enforce CloudWatch Alarms on Volume Bytes Used
- **What:** Create CloudWatch alarm on `VolumeBytesUsed`.
- **Why It Saves Money:** Instant alert on storage bloat.
- **Detailed Implementation Steps:**
  1. Put CloudWatch metric alarm on `AWS/DocDB`.
- **Estimated Savings:** Proactive risk mitigation.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 16. Monitor CPU and Memory Threshold Alarms
- **What:** Alarm on `CPUUtilization` (> 85%).
- **Why It Saves Money:** Prevents un-optimized query deadlocks.
- **Detailed Implementation Steps:**
  1. Put CloudWatch alarm on `CPUUtilization`.
- **Estimated Savings:** Operational stability.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 17. Utilize Index Build Background Flags
- **What:** Execute index builds with `{ background: true }`.
- **Why It Saves Money:** Prevents write locks that stall application connections.
- **Detailed Implementation Steps:**
  1. Pass background flag in `createIndex()`.
- **Estimated Savings:** Application uptime protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** Index creation workflow.

#### 18. Audit Free Tier & Account Allowances
- **What:** Audit non-prod testing against account allowances.
- **Why It Saves Money:** Operational hygiene.
- **Detailed Implementation Steps:**
  1. Review Billing Console.
- **Estimated Savings:** Administrative hygiene.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

---

## Cross-Service Synergies

```
[ Amazon DocumentDB Cluster ] 
        │
        ├──(Engine Upgrade)─────> [ Version 5.0 (Bypasses Extended Support $292/mo tax) ]
        │
        ├──(Storage Tiering)────> [ I/O-Optimized Storage ] (Free I/O for write-heavy clusters)
        │
        └──(Non-Prod Sizing)────> [ Single-Node Primary Configuration ] (Cuts dev compute by 50%)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `DocDB:ServerlessUsage`, `DocDB:StorageIOUsage`, `DocDB:StorageUsage`, `DocDB:ExtendedSupport`.
- `line_item_resource_id`: DocumentDB Cluster ARN (`arn:aws:rds:us-east-1:123456789012:cluster:prod-docdb`).

### B. CloudWatch & Performance Metrics
- `AWS/DocDB` Namespace: `CPUUtilization`, `FreeableMemory`, `ReadIOPS`, `WriteIOPS`, `VolumeBytesUsed`, `BufferCacheHitRatio`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "DOC-EXT-001",
  "service": "DocumentDB",
  "category": "Waste Elimination & Extended Support Avoidance",
  "resource_id": "arn:aws:rds:us-east-1:123456789012:cluster:legacy-docdb-36",
  "resource_name": "legacy-docdb-36",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "engine_version": "3.6.0",
    "vcpu_count": 8,
    "extended_support_active": true,
    "monthly_extended_support_fee_usd": 584.00,
    "monthly_total_cost_usd": 1820.00
  },
  "recommended_config": {
    "engine_version": "5.0.0",
    "projected_monthly_extended_support_fee_usd": 0.00,
    "projected_monthly_total_cost_usd": 1236.00
  },
  "financial_impact": {
    "monthly_savings_usd": 584.00,
    "annual_savings_usd": 7008.00,
    "savings_percentage": 32.1
  },
  "risk_assessment": {
    "risk_level": "Medium",
    "reason": "Engine upgrade to 5.0 requires application compatibility testing against MongoDB 5.0 API specs."
  },
  "implementation": {
    "scope": "Database Administrator / DevOps",
    "effort_estimate": "2-4 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Extended Support Upgrade (v3.6 -> 5.0)**| 5 | $9,100.00 | $2,920.00 | 32.1% | Medium |
| **Non-Prod Single-Node Pruning** | 8 | $6,400.00 | $3,200.00 | 50.0% | Zero |
| **I/O-Optimized Storage Conversion** | 4 | $12,500.00 | $5,000.00 | 40.0% | Zero |
| **MongoDB Indexing (Full Scan Elimination)**| 10 | $8,200.00 | $5,740.00 | 70.0% | Low |
| **Total** | **27** | **$36,200.00** | **$16,860.00** | **46.6%** | -- |
