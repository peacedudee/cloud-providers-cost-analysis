# Cost-Cutting Playbook: AWS Database Migration Service (DMS)

> **Companion File:** [dms.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/dms/dms.md)  
> **Last Updated:** July 2026

---

## Executive Summary

AWS Database Migration Service (DMS) migrates databases to AWS with minimal application downtime. Billing includes **Provisioned Replication Instances** (`dms.c5.large` Single-AZ at $0.194/hr vs Multi-AZ CDC at $0.388/hr = $283.24/mo), **Replication Storage Volume** ($0.115/GB-mo above 50 GB free), and **DMS Serverless** ($0.10/DCU-hr).

Key billing traps include:
1. **Leaving Multi-AZ CDC Instances Running Post-Cutover:** Forgetting to terminate Multi-AZ `dms.c5.large` instances ($283.24/mo per instance) or `dms.r5.xlarge` instances ($1,127.12/mo) after production cutover is complete.
2. **Missing the 6-Month Free Target Migration Incentive:** Paying for replication compute when migrating target databases to Amazon Aurora, RDS MySQL, or RDS PostgreSQL.
3. **Cross-Region Network Egress Charges:** Provisioning the DMS replication instance in a different region than the target database.

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **40–100% reduction in total AWS DMS spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Immediately Terminate DMS Instances After Production Cutover:** Halts ongoing compute ($283.24/mo to $1,127/mo) instantly upon migration completion.
2. **Leverage 6-Month Free Target Migration Incentive:** AWS waives replication instance compute fees for 6 months when target database is Amazon Aurora or RDS MySQL/PostgreSQL.
3. **Align Replication Instance Region with Target DB:** Eliminates cross-region data transfer egress fees ($0.02 to $0.09/GB).

---

## Strategy Categories

### 1. Post-Cutover Resource Elimination

#### 1. Immediately Terminate DMS Replication Instances Post-Cutover
- **What:** Configure calendar reminders or automated cleanup scripts to terminate DMS replication tasks and delete provisioned replication instances (`aws dms delete-replication-instance`) immediately following production application cutover validation.
- **Why It Saves Money:**
  - **`dms.c5.large` (Multi-AZ CDC):** **$0.388 per hour = $283.24 per month**.
  - **`dms.r5.xlarge` (Multi-AZ CDC):** **$1.544 per hour = $1,127.12 per month**.
  - Forgetting to delete 5 idle post-cutover replication instances wastes **$1,416.20 to $5,635.60 per month**!
- **Detailed Implementation Steps:**
  1. Audit active replication instances via AWS CLI:
     ```bash
     aws dms describe-replication-instances \
       --query "ReplicationInstances[*].{ID:ReplicationInstanceIdentifier,Status:ReplicationInstanceStatus,Type:ReplicationInstanceClass}"
     ```
  2. Terminate replication instance:
     ```bash
     aws dms delete-replication-instance \
       --replication-instance-arn arn:aws:dms:us-east-1:123456789012:rep:4GXXXXXX
     ```
- **Estimated Savings:** **100% savings** on idle migration instance compute ($283.24 to $1,127.12/mo saved per instance).
- **Risk Level:** Zero risk (after production cutover validation completes).
- **Implementation Scope:** Database Administrator / DevOps
- **Prerequisites:** Final application cutover sign-off.

---

### 2. Free Tier & Target Engine Incentive Maximization

#### 2. Leverage the 6-Month Free Target Database Migration Offer
- **What:** Migrate target database workloads to **Amazon Aurora**, **RDS PostgreSQL**, **RDS MySQL**, or **Amazon Redshift**.
- **Why It Saves Money:** AWS waives the provisioned replication instance fee for up to 6 months when migrating to an eligible AWS database engine target.
- **Detailed Implementation Steps:**
  1. Verify target database engine eligibility during migration planning.
  2. Confirm $0.00 compute discount line items on monthly AWS bill.
- **Estimated Savings:** **100% discount** on DMS replication compute for 6 months.
- **Risk Level:** Zero risk.
- **Implementation Scope:** FinOps Team / Database Administrator
- **Prerequisites:** Migration target is Amazon Aurora, RDS MySQL/PostgreSQL, or Redshift.

---

### 3. Instance Sizing & Multi-AZ Mode Selection

#### 3. Use Single-AZ Instances for Initial Bulk Data Loading (Switch to Multi-AZ for CDC Only)
- **What:** Provision a **Single-AZ** instance (e.g. `dms.c5.large` at $0.194/hr) for the initial Full Load phase, and upgrade to **Multi-AZ** ($0.388/hr) only when initiating the continuous Change Data Capture (CDC) phase.
- **Why It Saves Money:** Single-AZ instances are **50% cheaper** than Multi-AZ instances ($0.194/hr vs $0.388/hr). Running initial full load on Single-AZ cuts migration compute spend during data backfill by **50%**.
- **Detailed Implementation Steps:**
  1. Provision Single-AZ instance in Terraform:
     ```hcl
     resource "aws_dms_replication_instance" "migration" {
       replication_instance_id    = "prod-db-migration"
       replication_instance_class = "dms.c5.large"
       multi_az                   = false
       allocated_storage          = 50
     }
     ```
  2. Modify `multi_az = true` before starting CDC phase.
- **Estimated Savings:** **50% reduction** in compute fees during Full Load phase.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** Full load vs CDC phase separation in migration plan.

#### 4. Right-Size Replication Instance Class (`dms.t3` vs `dms.c5` vs `dms.r5`)
- **What:** Use burstable `dms.t3.medium` ($0.036/hr = $26.28/mo) or `dms.c5.large` ($0.194/hr) for low-to-medium throughput migrations rather than defaulting to expensive `dms.r5.xlarge` ($0.772/hr = $563.56/mo).
- **Why It Saves Money:** Slashes compute fees by **65–95%** for smaller database workloads.
- **Detailed Implementation Steps:**
  1. Monitor `CPUUtilization` and `FreeableMemory` metrics during test migration.
  2. Downsize instance class if CPU < 20%.
- **Estimated Savings:** 65–95% compute fee reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** Test migration metrics.

---

### 4. Serverless & Homogeneous Migration Selection

#### 5. Use Serverless Homogeneous Migrations for Like-to-Like Runs
- **What:** Use **DMS Homogeneous Migrations** (serverless) for homogeneous database migrations (e.g. PostgreSQL to RDS PostgreSQL, or MySQL to RDS MySQL).
- **Why It Saves Money:** Eliminates provisioned replication instance storage and idle hourly fees, scaling compute charges strictly to migration execution runtime.
- **Detailed Implementation Steps:**
  1. Select Homogeneous Migration type in AWS DMS Console.
- **Estimated Savings:** 40–80% savings compared to running a dedicated 24/7 provisioned instance.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** Homogeneous engine migration.

---

### 5. Network Egress & Storage Optimization

#### 6. Align DMS Instance Region and AZ with Target Database
- **What:** Launch the DMS replication instance in the **same AWS Region and Availability Zone** as the target Amazon RDS or Aurora database.
- **Why It Saves Money:** Cross-AZ or cross-region transfers incur network egress fees ($0.01 to $0.09 per GB). Aligning AZs makes data transfer to the target database **100% FREE ($0.00/GB)**.
- **Detailed Implementation Steps:**
  1. Specify target DB subnet group and Availability Zone when creating `aws_dms_replication_instance`.
- **Estimated Savings:** Eliminates cross-AZ data transfer fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Network Engineer / DBA
- **Prerequisites:** Network subnet mapping.

#### 7. Keep Provisioned Storage Under 50 GB Free Tier Baseline
- **What:** Keep `allocated_storage = 50` GB on provisioned replication instances unless massive swap log storage is explicitly required.
- **Why It Saves Money:** AWS includes **50 GB of storage for FREE** per replication instance. Provisioning 500 GB incurs $0.115/GB-mo on the additional 450 GB ($51.75/month).
- **Detailed Implementation Steps:**
  1. Set `allocated_storage = 50` in Terraform.
- **Estimated Savings:** Eliminates extra storage billing ($51.75/mo saved per instance).
- **Risk Level:** Low.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** CDC log volume sizing.

---

### 6. Observability & Governance

#### 8. Maximize 6-Month Free Tier Allowance for `dms.t3.micro`
- **What:** Utilize 6 months of free single-AZ `dms.t3.micro` instance + 50 GB storage for small database migrations.
- **Why It Saves Money:** Delivers $0.00 migration compute for small DBs.
- **Detailed Implementation Steps:**
  1. Track free tier usage in Billing Console.
- **Estimated Savings:** Baseline free testing.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** New account or eligible target DB.

#### 9. Enforce CloudWatch Alarms for `CDCLatencySource` and `CDCLatencyTarget`
- **What:** Put CloudWatch alarm on `CDCLatencySource` (> 60 seconds).
- **Why It Saves Money:** Instant alert if CDC replication falls behind, delaying application cutover.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm targeting `AWS/DMS`.
- **Estimated Savings:** Prevents prolonged migration compute runtimes.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch alarm setup.

#### 10. Utilize Free DMS Fleet Advisor & Schema Conversion Tools
- **What:** Use **AWS DMS Fleet Advisor** and **AWS DMS Schema Conversion** for database discovery and code translation.
- **Why It Saves Money:** Fleet Advisor and Schema Conversion tools are **100% FREE ($0.00)**.
- **Detailed Implementation Steps:**
  1. Deploy Fleet Advisor collectors for pre-migration discovery.
- **Estimated Savings:** $0.00 assessment tool cost.
- **Risk Level:** Zero.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** Fleet Advisor collector setup.

#### 11. Disable Logging of Sensitive Payload Data in Task Settings
- **What:** Set `Severity = LOGGER_SEVERITY_DEFAULT` in DMS task logging settings.
- **Why It Saves Money:** Prevents CloudWatch log ingestion overruns ($0.50/GB) on verbose DEBUG logging.
- **Detailed Implementation Steps:**
  1. Update task logging settings JSON.
- **Estimated Savings:** 80–95% CloudWatch log cost reduction for DMS.
- **Risk Level:** Low.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** Task settings configuration.

#### 12. Consolidate Multiple Small Database Migrations onto One Instance
- **What:** Run multiple replication tasks concurrently on a single `dms.c5.xlarge` instance instead of launching separate instances for every table.
- **Why It Saves Money:** Maximizes instance utilization and reduces total hourly compute spend.
- **Detailed Implementation Steps:**
  1. Associate multiple replication tasks with 1 replication instance.
- **Estimated Savings:** 40–60% compute fee consolidation.
- **Risk Level:** Medium.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** Instance memory capacity check.

#### 13. Optimize LOB (Large Binary Object) Parameter Settings
- **What:** Configure `InlineLobMode = true` and `LobChunkSize = 64` KB in task settings for tables with LOB columns.
- **Why It Saves Money:** Accelerates LOB data migration, shortening instance runtime.
- **Detailed Implementation Steps:**
  1. Specify LOB settings in task JSON definition.
- **Estimated Savings:** 30–50% reduction in migration duration.
- **Risk Level:** Low.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** LOB column analysis.

#### 14. Stop Inactive Migration Tasks
- **What:** Stop replication tasks (`aws dms stop-replication-task`) during maintenance windows.
- **Why It Saves Money:** Reduces unnecessary CDC query load on source databases.
- **Detailed Implementation Steps:**
  1. Stop task via CLI.
- **Estimated Savings:** Source database compute protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** Maintenance schedule.

#### 15. Enforce IAM Least Privilege Roles on DMS Tasks
- **What:** Restrict `dms:StartReplicationTask` permissions via IAM.
- **Why It Saves Money:** Security governance best practice.
- **Detailed Implementation Steps:**
  1. Restrict IAM role policies.
- **Estimated Savings:** Security risk mitigation.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** IAM policy review.

#### 16. Implement Automated Cleanup Tags (`AutoDeleteDate`)
- **What:** Apply an `AutoDeleteDate = YYYY-MM-DD` tag to all created DMS replication instances.
- **Why It Saves Money:** Enables automated Reaper scripts to delete forgotten test instances after 14 days.
- **Detailed Implementation Steps:**
  1. Configure Reaper Lambda to terminate instances past `AutoDeleteDate`.
- **Estimated Savings:** Prevents abandoned instance billing overruns.
- **Risk Level:** Zero.
- **Implementation Scope:** DevOps Engineer
- **Prerequisites:** Reaper script deployment.

---

## Cross-Service Synergies

```
[ Migration Project ] 
        │
        ├──(Target Selection)───> [ Amazon Aurora Target ] (6 Months FREE compute discount)
        │
        ├──(Instance Mode)──────> [ Single-AZ for Full Load ] (50% cheaper during backfill)
        │
        └──(Post-Cutover)───────> [ Immediate Instance Termination ] (Stops $283-$1,127/mo billing instantly)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `InstanceUsage:c5.large`, `InstanceUsage:r5.xlarge`, `ReplicationStorage`, `Serverless-DCU-Hours`.
- `line_item_resource_id`: DMS Replication Instance ARN (`arn:aws:dms:us-east-1:123456789012:rep:4GXXXXXX`).

### B. CloudWatch Metrics
- `AWS/DMS` Namespace: `CPUUtilization`, `FreeableMemory`, `CDCLatencySource`, `CDCLatencyTarget`, `FullLoadThroughputBandwidth`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "DMS-TRM-001",
  "service": "DMS",
  "category": "Post-Cutover Resource Elimination",
  "resource_id": "arn:aws:dms:us-east-1:123456789012:rep:POST_CUTOVER_ID",
  "resource_name": "legacy-oracle-migration-instance",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "instance_class": "dms.c5.large",
    "multi_az": true,
    "task_status": "STOPPED",
    "days_idle": 45,
    "monthly_cost_usd": 283.24
  },
  "recommended_config": {
    "action": "Delete Replication Instance Immediately",
    "projected_monthly_cost_usd": 0.00
  },
  "financial_impact": {
    "monthly_savings_usd": 283.24,
    "annual_savings_usd": 3398.88,
    "savings_percentage": 100.0
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Production cutover was validated 45 days ago; replication task is stopped."
  },
  "implementation": {
    "scope": "Database Administrator / DevOps",
    "effort_estimate": "5 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Post-Cutover Instance Elimination**| 8 | $3,400.00 | $3,400.00 | 100.0% | Zero |
| **6-Month Target Free Incentive** | 5 | $2,100.00 | $2,100.00 | 100.0% | Zero |
| **Single-AZ Full Load Selection** | 6 | $1,800.00 | $900.00 | 50.0% | Zero |
| **Instance Class Downsizing** | 10 | $5,600.00 | $4,200.00 | 75.0% | Low |
| **Total** | **29** | **$12,900.00** | **$10,600.00** | **82.2%** | -- |
