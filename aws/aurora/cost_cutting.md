# Cost-Cutting Playbook: Amazon Aurora (MySQL & PostgreSQL)

> **Companion File:** [aurora.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/aurora/aurora.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon Aurora provides enterprise-grade MySQL and PostgreSQL-compatible relational database performance with cloud-native distributed storage replicated 6 ways across 3 Availability Zones. While highly durable and scalable, Aurora's multi-dimensional billing model — compute (Provisioned vs Serverless v2), storage configurations (Standard vs I/O-Optimized), per-request I/O fees ($0.20/M), Extended Support fees ($0.10–$0.20/vCPU-hr), and Global Database cross-region replication fees — can create major cost leaks.

This playbook provides **20 detailed strategies** across eight operational categories, delivering an estimated **30–65% reduction in total Aurora database spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Switch Write-Heavy Clusters to Aurora I/O-Optimized:** Eliminates variable I/O request charges ($0.20/M) for clusters where I/O fees exceed 25% of the total monthly bill.
2. **Upgrade Legacy Engine Versions (MySQL 5.7 / PostgreSQL 11):** Bypasses the **$0.10 to $0.20 per vCPU-hour Extended Support surcharge** ($292.00/mo penalty per 4-vCPU node).
3. **Set Strict Max ACU Limits on Serverless v2:** Caps auto-scaling ceiling (e.g. max 8 ACUs in dev/staging) to prevent unoptimized queries from scaling up to 128 ACUs ($15.36/hr).

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Upgrade EOL Engine Versions to Eliminate Extended Support Surcharges
- **What:** Upgrade legacy Aurora MySQL 5.7 or Aurora PostgreSQL 11/12 clusters to active engine versions (MySQL 8.0, PostgreSQL 15/16).
- **Why It Saves Money:** RDS Extended Support levies an extra **$0.10/vCPU-hour in Years 1–2** and **$0.20/vCPU-hour in Year 3**. A 4-vCPU instance (`db.r6g.xlarge`) pays a **$292.00/month penalty** in Year 1 and **$584.00/month** in Year 3 on top of base compute!
- **Detailed Implementation Steps:**
  1. Identify legacy clusters via CLI:
     ```bash
     aws rds describe-db-clusters \
       --query "DBClusters[?EngineVersion=='5.7.mysql_aurora.2.11.2'].{ID:DBClusterIdentifier,Engine:Engine,Version:EngineVersion}" \
       --output table
     ```
  2. Run zero-downtime Blue/Green Deployment:
     ```bash
     aws rds create-blue-green-deployment \
       --blue-green-deployment-name aurora-upgrade-bg \
       --source arn:aws:rds:us-east-1:123456789012:cluster:prod-aurora-cluster \
       --target-engine-version 8.0.mysql_aurora.3.04.0
     ```
  3. Verify Green environment data integrity, then trigger switchover:
     ```bash
     aws rds switchover-blue-green-deployment --blue-green-deployment-identifier bgd-123456789
     ```
- **Estimated Savings:** $292.00 to $584.00 per 4-vCPU node per month (100% of Extended Support fees).
- **Risk Level:** Medium (requires application regression testing against newer database engine releases).
- **Implementation Scope:** Engineer/DevOps & Database Administrator
- **Prerequisites:** Blue/Green deployment prerequisites met (binlog enabled for MySQL).

#### 2. Decommission Stale Manual Database Snapshots
- **What:** Identify and delete legacy manual cluster snapshots exceeding retention mandates (> 30 days).
- **Why It Saves Money:** Automated backups up to 100% of cluster storage size are free; manual snapshots always bill at **$0.095 per GB-month**. Retaining 10 TB of manual snapshots costs **$950.00/month** indefinitely.
- **Detailed Implementation Steps:**
  1. List manual cluster snapshots older than 30 days:
     ```bash
     aws rds describe-db-cluster-snapshots \
       --snapshot-type manual \
       --query "DBClusterSnapshots[?SnapshotCreateTime<='2026-06-01'].{ID:DBClusterSnapshotIdentifier,Size:AllocatedStorage,Time:SnapshotCreateTime}" \
       --output table
     ```
  2. Delete stale manual snapshots:
     ```bash
     aws rds delete-db-cluster-snapshot --db-cluster-snapshot-identifier manual-snap-2025-01-01
     ```
  3. Automate snapshot lifecycle via AWS Backup policies with expiration rules.
- **Estimated Savings:** 100% of manual snapshot storage charges ($0.095/GB-mo).
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compliance team retention confirmation.

#### 3. Stop Idle Non-Production Aurora Clusters
- **What:** Automatically stop non-production Aurora clusters (dev, test, QA) during off-hours (nights and weekends: 7 PM to 7 AM M-F, all weekend).
- **Why It Saves Money:** Stopping clusters eliminates 100% of compute billing (Provisioned or Serverless v2). Running dev databases 50 hours/week instead of 168 hours/week cuts compute costs by **70%** (saving 118 idle hours/week).
- **Detailed Implementation Steps:**
  1. Create AWS Systems Manager Automation document or EventBridge rule calling RDS `StopDBCluster`:
     ```bash
     aws rds stop-db-cluster --db-cluster-identifier dev-aurora-cluster
     ```
  2. Create morning start automation:
     ```bash
     aws rds start-db-cluster --db-cluster-identifier dev-aurora-cluster
     ```
  3. Note: Stopped Aurora clusters automatically restart after 7 days; configure a weekly re-stop Lambda to keep permanent dev clusters stopped.
- **Estimated Savings:** 65–70% reduction in non-production database compute billing.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-production SLA alignment.

---

### 2. Storage Tiering & Configuration Optimization

#### 4. Evaluate & Convert Clusters to Aurora I/O-Optimized Configuration
- **What:** Convert write-heavy Aurora clusters from `Aurora Standard` to `Aurora I/O-Optimized` storage configuration.
- **Why It Saves Money:**
  - **Standard:** Storage is $0.10/GB-mo, but charges **$0.20 per 1 million I/O requests**. Heavy analytics/ETL generating 5 billion I/Os/month pays **$1,000.00/month in I/O fees alone**.
  - **I/O-Optimized:** Storage is $0.225/GB-mo and compute carries a 30% premium, but **I/O requests are 100% FREE ($0.00)**.
  - **Decision Rule:** If I/O request fees exceed **25%** of your total monthly Aurora bill, I/O-Optimized will reduce total spend.
- **Detailed Implementation Steps:**
  1. Check Athena CUR for `line_item_usage_type` like `Aurora:StorageIOUsage`.
  2. If I/O fees > 25% of cluster bill, update cluster configuration online:
     ```bash
     aws rds modify-db-cluster \
       --db-cluster-identifier prod-aurora-cluster \
       --db-cluster-instance-class-config StorageType=aurora-iopt1 \
       --apply-immediately
     ```
  3. Conversion occurs dynamically online without downtime or database reboots.
- **Estimated Savings:** 20–50% net monthly bill reduction on write-heavy clusters.
- **Risk Level:** Zero risk (seamless online modification).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Financial analysis of I/O spend vs compute/storage premium.

#### 5. Leverage Aurora Fast Database Cloning for Testing Environments
- **What:** Use **Aurora Fast Database Cloning** instead of restoring manual physical snapshots or running full data export/import scripts for dev/test environments.
- **Why It Saves Money:** Fast Clones use copy-on-write protocol. A 2 TB cloned database shares the underlying storage blocks with the primary cluster, charging **$0.00 for storage** on all unmodified data pages!
- **Detailed Implementation Steps:**
  1. Create a clone via AWS CLI:
     ```bash
     aws rds restore-db-cluster-to-point-in-time \
       --source-db-cluster-identifier prod-aurora-cluster \
       --db-cluster-identifier dev-aurora-clone \
       --option-group-name default:aurora-mysql-8-0 \
       --restore-type copy-on-write \
       --use-latest-restorable-time
     ```
- **Estimated Savings:** Saves 100% of duplicate storage charges ($0.10–$0.225/GB-mo).
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps & Database Administrator
- **Prerequisites:** Source cluster must be in the same region and account.

---

### 3. Compute Rightsizing & Serverless Tuning

#### 6. Configure Strict Max ACU Boundaries on Aurora Serverless v2
- **What:** Set aggressive `MaxCapacity` ACU thresholds on Aurora Serverless v2 auto-scaling configurations.
- **Why It Saves Money:** Serverless v2 scales up to 128 ACUs ($15.36/hour = $11,212/month). An unindexed query or batch join can trigger maximum scaling unnecessarily. Setting Max ACU to 8 or 16 caps maximum hourly compute exposure.
- **Detailed Implementation Steps:**
  1. Inspect Serverless v2 scaling configuration:
     ```bash
     aws rds describe-db-clusters --db-cluster-identifier dev-aurora-serverless
     ```
  2. Modify scaling limits (e.g. Min 0.5 ACU, Max 8.0 ACUs for non-prod):
     ```bash
     aws rds modify-db-cluster \
       --db-cluster-identifier dev-aurora-serverless \
       --serverless-v2-scaling-configuration MinCapacity=0.5,MaxCapacity=8.0 \
       --apply-immediately
     ```
- **Estimated Savings:** Prevents 500–1000% unexpected query cost spikes.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Baseline peak ACU utilization monitoring.

#### 7. Downsize Over-Provisioned Compute Instances
- **What:** Downsize Provisioned Aurora instance classes where CPU utilization is < 20% and memory headroom is high (e.g. `db.r6g.2xlarge` -> `db.r6g.xlarge`).
- **Why It Saves Money:** Halving instance size cuts compute costs by exactly 50%. Moving `db.r6g.2xlarge` ($0.58/hr = $423.40/mo) down to `db.r6g.xlarge` ($0.29/hr = $211.70/mo) saves **$2,540.40/year per instance**.
- **Detailed Implementation Steps:**
  1. Check 30-day CloudWatch metrics `CPUUtilization` and `FreeableMemory`.
  2. Modify instance type with failover strategy (modify reader first, failover, then modify old writer):
     ```bash
     aws rds modify-db-instance \
       --db-instance-identifier prod-aurora-reader-1 \
       --db-instance-class db.r6g.xlarge \
       --apply-immediately
     ```
- **Estimated Savings:** 50% compute savings per step-down.
- **Risk Level:** Low.
- **Implementation Scope:** Database Administrator / DevOps
- **Prerequisites:** Buffer pool hit ratio > 99%.

#### 8. Migrate Provisioned Workloads with Low Utilization to Serverless v2
- **What:** Migrate low-traffic or variable provisioned clusters (`db.r6g.xlarge` running 24/7 at 5% CPU) to Aurora Serverless v2.
- **Why It Saves Money:** A 24/7 `db.r6g.xlarge` costs $211.70/month regardless of usage. Serverless v2 idling at 0.5 ACU costs **$43.80/month**, scaling up only during active queries.
- **Detailed Implementation Steps:**
  1. Add a Serverless v2 reader instance to the existing cluster:
     ```bash
     aws rds create-db-instance \
       --db-instance-identifier aurora-s2-reader \
       --db-cluster-identifier prod-aurora-cluster \
       --db-instance-class db.serverless \
       --engine aurora-mysql
     ```
  2. Fail over to the Serverless v2 instance, then delete old provisioned instances.
- **Estimated Savings:** 50–75% reduction for low-utilization databases.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Serverless v2 feature compatibility verification.

---

### 4. Commitment Discounts

#### 9. Purchase Database Reserved Instances for Steady-State Production
- **What:** Purchase 1-year or 3-year Database Reserved Instances (RIs) for steady-state provisioned Aurora nodes.
- **Why It Saves Money:** Secures discounts of **up to 38% (1-year No Upfront)** or **up to 55% (3-year All Upfront)** off standard On-Demand hourly compute rates.
- **Detailed Implementation Steps:**
  1. Analyze Cost Explorer RI recommendations based on 30-day baseline instance count.
  2. Purchase RIs via CLI:
     ```bash
     aws rds purchase-reserved-db-instances-offering \
       --reserved-db-instances-offering-id offering-id-12345 \
       --db-instance-count 2
     ```
- **Estimated Savings:** 35–55% compute cost reduction.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team / Procurement
- **Prerequisites:** 1-to-3-year database engine and region commitment.

---

### 5. Architecture & High Availability Optimization

#### 10. Prune Unnecessary Multi-AZ Replica Nodes in Non-Production
- **What:** Remove read replicas and Multi-AZ standby instances from development, testing, and staging Aurora clusters.
- **Why It Saves Money:** A 2-node Aurora cluster (1 Writer + 1 Reader) doubles (2x) compute costs. Dev/test clusters do not require high-availability failover.
- **Detailed Implementation Steps:**
  1. List instances in dev cluster:
     ```bash
     aws rds describe-db-clusters --db-cluster-identifier dev-aurora-cluster --query "DBClusters[0].DBClusterMembers[*].DBInstanceIdentifier"
     ```
  2. Delete non-essential reader instances:
     ```bash
     aws rds delete-db-instance --db-instance-identifier dev-aurora-reader-1 --skip-final-snapshot
     ```
- **Estimated Savings:** **50% direct reduction** in cluster compute billing.
- **Risk Level:** Low (for non-production environments).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-production SLA alignment.

#### 11. Utilize RDS Proxy for Serverless Connection Pooling
- **What:** Deploy **Amazon RDS Proxy** between serverless application backends (AWS Lambda, Fargate) and Aurora clusters.
- **Why It Saves Money:** Prevents thousands of transient Lambda invocations from opening independent database connections, which exhausts RAM, forces high CPU context switching, and drives premature node sizing upgrades.
- **Detailed Implementation Steps:**
  1. Create RDS Proxy via Terraform:
     ```hcl
     resource "aws_db_proxy" "aurora_proxy" {
       name                   = "aurora-proxy"
       engine_family          = "MYSQL"
       auth {
         auth_scheme = "SECRETS_ROUTER"
         secret_arn  = aws_secretsmanager_secret.db_secret.arn
       }
       role_arn               = aws_iam_role.proxy_role.arn
       vpc_subnet_ids         = module.vpc.private_subnets
     }
     ```
  2. Point Lambda environment variables to RDS Proxy endpoint.
- **Estimated Savings:** 25–40% compute efficiency gains (allows smaller instance sizing).
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Secrets Manager integration.

#### 12. Optimize Global Database Replication Regions & Traffic
- **What:** Review Aurora Global Database configurations; remove unused secondary region replicas and optimize cross-region replication.
- **Why It Saves Money:** Global Database charges for secondary region compute + secondary storage + **cross-region data transfer egress ($0.02/GB)**. Replicating 10 TB/month across regions adds $200/mo in network egress plus full secondary compute fees.
- **Detailed Implementation Steps:**
  1. Remove unnecessary secondary region cluster:
     ```bash
     aws rds remove-from-global-cluster \
       --global-cluster-identifier global-aurora-db \
       --db-cluster-identifier-arn arn:aws:rds:eu-west-1:123456789012:cluster:secondary-cluster
     ```
- **Estimated Savings:** 50% savings on global database infrastructure.
- **Risk Level:** Medium (impacts multi-region DR capability).
- **Implementation Scope:** Database Architect / DevOps
- **Prerequisites:** Disaster recovery compliance review.

---

### 6. Query & Index Optimization

#### 13. Identify & Eliminate Unindexed Full-Table Scan Queries
- **What:** Enable Performance Insights and Audit Logging to identify SQL queries performing full-table scans.
- **Why It Saves Money:** On Aurora Standard, full-table scans read millions of 8 KB data pages from disk, generating astronomical I/O request fees ($0.20/M) and spiking CPU. Adding a proper B-tree index drops I/O reads by 99.9%.
- **Detailed Implementation Steps:**
  1. Turn on Performance Insights:
     ```bash
     aws rds modify-db-instance \
       --db-instance-identifier prod-aurora-writer \
       --enable-performance-insights \
       --performance-insights-retention-period 7
     ```
  2. Query `slow_query_log` for queries with `rows_examined >> rows_sent`.
  3. Add missing indexes via `CREATE INDEX idx_user_id ON orders(user_id);`.
- **Estimated Savings:** 50–90% reduction in Standard I/O charges and CPU load.
- **Risk Level:** Low.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** Slow query log inspection.

#### 14. Optimize Read-Replica Auto-Scaling for Burst Read Traffic
- **What:** Attach Application Auto Scaling to Aurora Read Replicas, dynamically adding reader nodes during peak hours and terminating them during quiet periods.
- **Why It Saves Money:** Prevents provisioning static 4-node reader clusters 24/7 to handle 2-hour daily traffic spikes.
- **Detailed Implementation Steps:**
  1. Register Aurora cluster as scalable target:
     ```bash
     aws application-autoscaling register-scalable-target \
       --service-namespace rds \
       --resource-id cluster:prod-aurora-cluster \
       --scalable-dimension rds:cluster:ReadReplicaCount \
       --min-capacity 1 --max-capacity 5
     ```
  2. Apply target tracking scaling policy (target 70% average CPU):
     ```bash
     aws application-autoscaling put-scaling-policy \
       --policy-name rds-cpu-auto-scaling \
       --service-namespace rds \
       --resource-id cluster:prod-aurora-cluster \
       --scalable-dimension rds:cluster:ReadReplicaCount \
       --policy-type TargetTrackingScaling \
       --target-tracking-scaling-policy-configuration '{"TargetValue":70.0,"PredefinedMetricSpecification":{"PredefinedMetricType":"RDSReaderAverageCPUUtilization"}}'
     ```
- **Estimated Savings:** 30–50% reduction in reader node compute fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Read-replica load balancing endpoint usage.

---

### 7. Licensing & Engine Tuning

#### 15. Standardize on Open-Source Compatible Engines (Avoid Commercial DB Lock-In)
- **What:** Migrate legacy Oracle or SQL Server workloads to Aurora PostgreSQL or Aurora MySQL.
- **Why It Saves Money:** Eliminates commercial database licensing surcharges ($0.25 to $3.00+ per vCPU-hour) and vendor audit liabilities.
- **Detailed Implementation Steps:**
  1. Run AWS Schema Conversion Tool (SCT) to evaluate database migration feasibility.
  2. Execute migration using AWS Database Migration Service (DMS).
- **Estimated Savings:** 50–80% overall TCO reduction.
- **Risk Level:** High (major application migration project).
- **Implementation Scope:** Enterprise Architecture / DBA Team
- **Prerequisites:** SCT feasibility report.

#### 16. Tune Aurora Database Parameter Groups for Memory Caching
- **What:** Optimize `innodb_buffer_pool_size` (MySQL) or `shared_buffers` (PostgreSQL) in custom DB Cluster Parameter Groups.
- **Why It Saves Money:** Maximizes RAM cache-hit ratio (> 99%), preventing queries from falling through to disk I/O ($0.20/M I/O tax).
- **Detailed Implementation Steps:**
  1. Modify custom parameter group:
     ```bash
     aws rds modify-db-parameter-group \
       --db-parameter-group-name custom-aurora-mysql8 \
       --parameters "ParameterName=innodb_buffer_pool_size,ParameterValue={DBInstanceClassMemory*3/4},ApplyMethod=pending-reboot"
     ```
- **Estimated Savings:** 20–40% I/O cost reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** Custom parameter group attached to cluster.

---

### 8. Network & Data Transfer Optimization

#### 17. Eliminate Cross-AZ Application-to-Database Traffic
- **What:** Align application compute instances (EC2/EKS/Lambda) to send database read queries to Aurora endpoints located in the **same Availability Zone**.
- **Why It Saves Money:** Cross-AZ traffic between compute in AZ-A and database reader in AZ-B costs **$0.01/GB in each direction ($0.02/GB roundtrip)**. Intra-AZ traffic is **100% FREE ($0.00/GB)**.
- **Detailed Implementation Steps:**
  1. Route read queries through local AZ reader endpoints or use AZ-aware connection pools.
- **Estimated Savings:** 100% of inter-AZ data transfer fees ($20/TB saved).
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Multi-AZ deployment.

#### 18. Use Private IP Endpoints for External Service Access
- **What:** Ensure database connection strings utilize internal VPC private DNS names (`*.cluster-xxxx.us-east-1.rds.amazonaws.com`).
- **Why It Saves Money:** Prevents accidental routing over public internet endpoints ($0.09/GB egress).
- **Detailed Implementation Steps:**
  1. Verify cluster `PubliclyAccessible` parameter is set to `No`.
- **Estimated Savings:** Prevents internet egress charges.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC routing configuration.

#### 19. Optimize Binary Log (Binlog) Retention for Replication
- **What:** Reduce binlog retention hours (`mysql.rds_set_configuration('binlog retention hours', 24);`) to minimum required windows.
- **Why It Saves Money:** Excessive binlog retention fills local instance storage, forcing storage expansion.
- **Detailed Implementation Steps:**
  1. Execute stored procedure:
     ```sql
     CALL mysql.rds_set_configuration('binlog retention hours', 12);
     ```
- **Estimated Savings:** Storage optimization.
- **Risk Level:** Low.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** Replication lag monitoring.

#### 20. Enforce Automated CloudWatch Alarms for Storage & ACU Growth
- **What:** Deploy automated CloudWatch alarms for `VolumeBytesUsed`, `ServerlessDatabaseCapacity`, and `VolumeWriteIOPS`.
- **Why It Saves Money:** Provides instant alerts before runaway queries or memory leaks generate massive end-of-month billing surprises.
- **Detailed Implementation Steps:**
  1. Create CloudWatch Alarm for ACU threshold:
     ```bash
     aws cloudwatch put-metric-alarm \
       --alarm-name "Aurora-Serverless-High-ACU" \
       --metric-name ServerlessDatabaseCapacity \
       --namespace AWS/RDS \
       --statistic Average --period 300 --threshold 16 \
       --comparison-operator GreaterThanThreshold \
       --evaluation-periods 2 \
       --alarm-actions arn:aws:sns:us-east-1:123456789012:billing-alerts
     ```
- **Estimated Savings:** Proactive billing risk mitigation.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic configured for notifications.

---

## Cross-Service Synergies

```
[ Amazon Aurora Cluster ] 
        │
        ├──(Storage Tiering)──> [ I/O-Optimized Configuration ] (Free I/O for write-heavy DBs)
        │
        ├──(Connection Pool)──> [ Amazon RDS Proxy ] (Prevents Lambda connection exhaustion)
        │
        └──(Development Clones)─> [ Aurora Fast Database Cloning ] (100% FREE storage on unmodified data)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Aurora:ServerlessUsage`, `Aurora:StorageIOUsage`, `Aurora:StorageUsage`, `Aurora:iopt1-StorageUsage`, `RDS:ExtendedSupport`.
- `line_item_resource_id`: Aurora DB Cluster ARN (`arn:aws:rds:us-east-1:123456789012:cluster:prod-db`).

### B. CloudWatch & Performance Insights Metrics
- `AWS/RDS` Namespace: `CPUUtilization`, `ServerlessDatabaseCapacity`, `FreeableMemory`, `ReadIOPS`, `WriteIOPS`, `VolumeBytesUsed`, `BufferCacheHitRatio`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "AUR-IO-001",
  "service": "Aurora",
  "category": "Storage Tiering & Configuration",
  "resource_id": "arn:aws:rds:us-east-1:123456789012:cluster:high-io-analytics-db",
  "resource_name": "high-io-analytics-db",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "storage_type": "aurora-standard",
    "allocated_storage_gb": 4500,
    "monthly_io_requests_millions": 18500,
    "monthly_io_cost_usd": 3700.00,
    "total_monthly_cost_usd": 5120.00
  },
  "recommended_config": {
    "storage_type": "aurora-iopt1",
    "projected_monthly_io_cost_usd": 0.00,
    "projected_total_monthly_cost_usd": 2840.00
  },
  "financial_impact": {
    "monthly_savings_usd": 2280.00,
    "annual_savings_usd": 27360.00,
    "savings_percentage": 44.5
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Conversion to I/O-Optimized executes online with zero downtime."
  },
  "implementation": {
    "scope": "Engineer/DevOps",
    "effort_estimate": "30 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Waste Elimination (Extended Support)**| 3 | $2,450.00 | $1,752.00 | 71.5% | Medium |
| **Storage Tiering (I/O-Optimized)** | 5 | $18,200.00 | $7,280.00 | 40.0% | Low |
| **Serverless ACU Tuning / Rightsizing** | 12 | $14,800.00 | $5,920.00 | 40.0% | Low |
| **Commitment Discounts (RIs)** | 4 | $22,000.00 | $7,700.00 | 35.0% | Low |
| **Total** | **24** | **$57,450.00** | **$22,652.00** | **39.4%** | -- |
