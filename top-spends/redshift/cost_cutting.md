# Cost-Cutting Playbook: Amazon Redshift

> **Companion File:** [redshift.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/redshift/redshift.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon Redshift is a fully managed petabyte-scale data warehouse available as Provisioned Clusters (e.g. `ra3` nodes with decoupled storage) or Redshift Serverless (billed per RPU-second). Because analytics queries scan multi-terabyte datasets, cost hotspots stem from over-provisioned cluster node counts, Serverless Max RPU capacity traps, uncompressed/unpartitioned Redshift Spectrum queries over S3 data lakes, and 24/7 idle cluster execution.

This playbook provides **18 actionable strategies** across six operational categories, delivering an estimated **30–65% reduction in total Redshift expenditure**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Configure Serverless Max RPU Capacity & Daily Usage Limits:** Prevents complex queries from triggering massive auto-scaling spikes ($0.375/RPU-hr).
2. **Compress and Partition S3 Data Lake Files for Spectrum (Parquet/ORC):** Cuts S3 data scanned by up to **90%**, dropping Spectrum charges from $5.00/TB down to $0.50/TB.
3. **Implement Scheduled Pause and Resume on Provisioned Clusters:** Automatically pauses non-24/7 clusters during off-hours (nights/weekends), eliminating compute charges while idle.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Implement Scheduled Pause and Resume on Non-Production Clusters
- **What:** Configure automated **Scheduled Pause and Resume** on provisioned Redshift clusters used for staging, development, or business-hours-only reporting.
- **Why It Saves Money:** An idle 4-node `ra3.xlplus` cluster ($1.086/node-hr) left running 24/7 costs **$3,171.12/month** in compute. Pausing the cluster outside business hours (7 PM to 7 AM and weekends) cuts compute costs by **65%** ($2,060/mo saved). Managed storage ($0.024/GB-mo) remains active.
- **Implementation Steps:**
  1. Go to Redshift Console -> Schedule actions.
  2. Create Pause schedule: `PauseCluster` at 19:00 UTC Monday–Friday.
  3. Create Resume schedule: `ResumeCluster` at 07:00 UTC Monday–Friday.
- **Estimated Savings:** 60–70% compute cost reduction on non-24/7 clusters.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Schedule alignment with BI/ETL team.

#### 2. Decommission Stale Manual Snapshots and Unused Clusters
- **What:** Audit and delete manual Redshift cluster snapshots older than 30 days and decommission abandoned ad-hoc data marts.
- **Why It Saves Money:** Manual snapshots bill indefinitely at Redshift Managed Storage rates ($0.024/GB-month). Retaining 50 TB of legacy manual snapshots costs $1,200/month.
- **Implementation Steps:**
  1. List manual snapshots: `aws redshift describe-cluster-snapshots --snapshot-type manual`.
  2. Delete stale snapshots: `aws redshift delete-cluster-snapshot --snapshot-identifier snap-id`.
- **Estimated Savings:** 100% of abandoned snapshot storage fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Retention policy sign-off.

---

### 2. Rightsizing

#### 3. Right-Size Provisioned Cluster Node Counts (`ra3` Family)
- **What:** Downsize node count (e.g. from 8 `ra3.xlplus` nodes to 4 nodes) where CPU/RAM metrics show persistent low utilization (< 30% average CPU).
- **Why It Saves Money:** `ra3` nodes decouple compute from storage. Halving the node count cuts hourly compute costs by 50% ($3,171/mo -> $1,585/mo for 4 nodes) without losing data storage capacity.
- **Implementation Steps:**
  1. Monitor CloudWatch metrics `CPUUtilization`, `PercentageDiskSpaceUsed`, and `QueryDuration`.
  2. Resize cluster online: `aws redshift modify-cluster-node-db --cluster-identifier id --node-type ra3.xlplus --number-of-nodes 4`.
- **Estimated Savings:** 50% per node count reduction step.
- **Risk Level:** Low (elastic resize executes online).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload query duration audit.

#### 4. Configure Redshift Serverless Max RPU Capacity Caps
- **What:** Set a strict ceiling on **Max RPU Capacity** (e.g. cap at 16 or 32 RPUs) in Redshift Serverless workgroup settings.
- **Why It Saves Money:** Default Serverless settings allow scaling up to 512 RPUs. A complex unoptimized query can trigger maximum scaling, billing at **$192.00/hour** ($0.375/RPU-hr).
- **Implementation Steps:**
  1. Access Redshift Serverless Workgroup configuration.
  2. Set `Max RPU Capacity = 16` (or lowest functional threshold).
  3. Configure Daily/Monthly RPU-hour usage limits with automated alert/stop actions.
- **Estimated Savings:** Prevents 500–1000% unexpected query cost spikes.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Serverless query performance baseline.

---

### 3. Commitment Discounts

#### 5. Purchase Reserved Nodes for Production Clusters
- **What:** Commit to 1-year or 3-year Reserved Nodes for steady-state provisioned production clusters.
- **Why It Saves Money:** Reserved Nodes provide discounts up to **35% (1-year)** or **75% (3-year All Upfront)** off On-Demand hourly compute node rates.
- **Implementation Steps:**
  1. Identify permanent production cluster node counts.
  2. Purchase Reserved Nodes via Redshift console: `aws redshift purchase-reserved-node-offering`.
- **Estimated Savings:** 35–75% reduction in compute node billing.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team / Procurement
- **Prerequisites:** 1 to 3-year data warehouse commitment.

---

### 4. Architecture Changes & Data Lake Optimization

#### 6. Convert S3 Data Lake Files to Apache Parquet / ORC for Redshift Spectrum
- **What:** Convert raw CSV/JSON text files in S3 external data lakes to compressed, columnar formats (**Apache Parquet** or **ORC**).
- **Why It Saves Money:** Redshift Spectrum bills **$5.00 per TB of data scanned**. Text files require scanning 100% of uncompressed data. Columnar Parquet with Snappy compression reduces data scanned by **80–90%**, dropping query costs from $5.00/TB to $0.50/TB.
- **Implementation Steps:**
  1. Add AWS Glue ETL or PySpark job to convert incoming CSVs to Parquet.
  2. Update Spectrum external table schema definitions.
- **Estimated Savings:** **80–90% savings** on Redshift Spectrum query fees.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** ETL pipeline conversion.

#### 7. Enforce Partitioning on S3 Data Lakes (Year/Month/Day)
- **What:** Organize S3 Spectrum files into partitioned folder structures (e.g. `s3://bucket/table/year=2026/month=07/`) and run `MSCK REPAIR TABLE`.
- **Why It Saves Money:** Allows Spectrum queries with `WHERE year = 2026 AND month = 07` to prune 98% of S3 files, scanning only relevant partitions instead of the entire bucket.
- **Implementation Steps:**
  1. Partition data lake files by date/category in S3.
  2. Add `PARTITIONED BY` clause in Glue Data Catalog / Spectrum DDL.
- **Estimated Savings:** 70–95% reduction in data scanned by Spectrum.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Partitioning strategy adoption.

#### 8. Migrate Intermittent Workloads from Provisioned to Redshift Serverless
- **What:** Move low-frequency analytical workloads (e.g. monthly financial reporting or sporadic data science queries) from 24/7 Provisioned Clusters to Redshift Serverless.
- **Why It Saves Money:** Redshift Serverless scales compute to **0 RPUs ($0.00)** when idle, billing only for active query seconds (60s minimum). Eliminates 24/7 provisioned node fees.
- **Implementation Steps:**
  1. Create Redshift Serverless Namespace and Workgroup.
  2. Migrate schemas and queries.
  3. Terminate Provisioned cluster.
- **Estimated Savings:** 40–80% for sporadic/low-utilization data marts.
- **Risk Level:** Medium.
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** Query pattern audit.

---

### 5. Scheduling & Performance Optimization

#### 9. Maximize Free Concurrency Scaling Credits
- **What:** Enable Concurrency Scaling on main cluster WLM (Workload Management) queues, while setting a strict concurrency scaling cluster cap.
- **Why It Saves Money:** AWS provides **45 seconds of free concurrency scaling credits** for every hour your primary cluster is active (up to 30 minutes free credit per day). This handles temporary query queue spikes for free, avoiding permanent cluster upsizing.
- **Implementation Steps:**
  1. Set WLM queue concurrency scaling mode to `Auto`.
  2. Set `max_concurrency_scaling_clusters = 1` or `2` to prevent excessive paid scaling ($5.50/cluster-hour).
- **Estimated Savings:** Absorbs query spikes for free while capping paid overages.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** WLM configuration access.

#### 10. Implement Workload Management (WLM) Query Queues & Timeouts
- **What:** Configure WLM query queues with explicit execution timeouts (e.g. cancel queries running > 30 minutes) and memory limits.
- **Why It Saves Money:** Prevents poorly written Cartesian join queries from consuming cluster resources for hours, blocking critical pipeline runs.
- **Implementation Steps:**
  1. Define WLM queues (e.g. `Short_Queries`, `Long_ETL`, `Adhoc_Analytics`).
  2. Add Query Monitoring Rules (QMR) to abort queries exceeding thresholds.
- **Estimated Savings:** 15–30% capacity efficiency gains.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer / DBA
- **Prerequisites:** Redshift WLM configuration.

#### 11. Automate Database Maintenance (VACUUM & ANALYZE)
- **What:** Ensure Automated Table Maintenance (Auto-Vacuum and Auto-Analyze) is active, or run targeted `VACUUM DELETE` on frequently updated tables.
- **Why It Saves Money:** Reclaims space from deleted rows and updates table statistics, allowing the query planner to generate faster, cheaper execution plans.
- **Implementation Steps:**
  1. Verify Auto-Vacuum status in `SVV_TABLE_INFO`.
  2. Schedule explicit `VACUUM SORT` during low-traffic windows if needed.
- **Estimated Savings:** 10–25% query execution speedup (reducing compute duration).
- **Risk Level:** Low.
- **Implementation Scope:** DBA / Data Engineer
- **Prerequisites:** DBA access.

#### 12. Optimize Sort Keys and Distribution Keys (DISTSTYLE / SORTKEY)
- **What:** Define appropriate Distribution Keys (`DISTKEY` on high-cardinality join columns) and Compound Sort Keys (`SORTKEY` on query filter columns).
- **Why It Saves Money:** Eliminates expensive inter-node data broadcast/redistribution during SQL joins (`DS_BCAST_INNER` steps), speeding up query runtimes by up to 10x.
- **Implementation Steps:**
  1. Analyze `EXPLAIN` query plans for high-cost data redistribution steps.
  2. Re-create tables with optimized `DISTSTYLE KEY` and `SORTKEY`.
- **Estimated Savings:** 50–90% reduction in query execution duration.
- **Risk Level:** Medium (requires DDL update and table reload).
- **Implementation Scope:** Data Engineer / DBA
- **Prerequisites:** Schema design review.

#### 13. Leverage Materialized Views for Repetitive Aggregations
- **What:** Create Materialized Views for complex, multi-table JOIN and `GROUP BY` aggregations used by executive dashboards.
- **Why It Saves Money:** Pre-computes and incrementally updates query results, allowing dashboards to query tiny pre-aggregated views in milliseconds instead of scanning billions of raw table rows repeatedly.
- **Implementation Steps:**
  1. Create Materialized View: `CREATE MATERIALIZED VIEW mv_daily_sales AS SELECT ...`.
  2. Configure `ENABLE AUTO REFRESH YES`.
- **Estimated Savings:** 70–95% query execution compute savings.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** SQL query analysis.

---

### 6. Pricing Model & Storage Optimization

#### 14. Migrate Legacy `dc2` / `ds2` Nodes to Modern `ra3` Node Family
- **What:** Convert legacy `dc2.large` or `ds2.xlarge` clusters to modern `ra3.xlplus` nodes with Redshift Managed Storage (RMS).
- **Why It Saves Money:** Legacy nodes couple compute and storage. `ra3` nodes decouple storage to low-cost S3 ($0.024/GB-mo), allowing independent scaling of compute and storage.
- **Implementation Steps:**
  1. Perform Elastic Resize or Snapshot Restore to `ra3.xlplus` cluster.
- **Estimated Savings:** 20–40% storage and compute optimization.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Cluster migration window.

#### 15. Compress Historical Tables using Columnar Encoding (ZSTD / AZ64)
- **What:** Apply optimal column compression encodings (`AZ64` for numeric, `ZSTD` for text) across all table columns.
- **Why It Saves Money:** Compresses disk footprint by 3x–5x, reducing Managed Storage fees ($0.024/GB-mo) and accelerating I/O read speeds.
- **Implementation Steps:**
  1. Run `ANALYZE COMPRESSION table_name` to get recommended encodings.
  2. Re-create tables with specified column compression.
- **Estimated Savings:** 60–80% reduction in table storage volume.
- **Risk Level:** Medium.
- **Implementation Scope:** DBA / Data Engineer
- **Prerequisites:** Table DDL update.

#### 16. Enforce Redshift Zero-ETL Integrations (Aurora / DynamoDB)
- **What:** Replace custom Glue/Lambda polling ETL pipelines with native AWS Zero-ETL integrations between Aurora/DynamoDB and Redshift.
- **Why It Saves Money:** Eliminates custom Glue ETL worker compute costs ($0.44/DPU-hr) and intermediate S3 staging storage fees.
- **Implementation Steps:**
  1. Create Zero-ETL integration in RDS/DynamoDB console targeting Redshift.
- **Estimated Savings:** 100% of custom Glue/Lambda pipeline processing costs.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Supported database engine versions.

#### 17. Monitor and Restrict Uncompressed Cross-Region Data Sharing
- **What:** Audit Redshift Data Sharing across AWS regions; restrict cross-region datashare queries.
- **Why It Saves Money:** Cross-region data sharing incurs inter-region data transfer egress fees ($0.01–$0.02/GB) for all bytes read by the consumer cluster.
- **Implementation Steps:**
  1. Track cross-region datashare egress in CUR.
- **Estimated Savings:** Eliminates cross-region egress overhead.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region cluster architecture.

#### 18. Use Redshift Data API for Serverless / Transient Query Execution
- **What:** Connect Lambda and web apps to Redshift using the Redshift Data API instead of maintaining persistent JDBC/ODBC connection pools.
- **Why It Saves Money:** Eliminates idle database connection overhead and avoids deploying dedicated proxy EC2 instances.
- **Implementation Steps:**
  1. Refactor Lambda functions to use `aws-sdk` RedshiftDataClient.
- **Estimated Savings:** 100% of connection proxy compute fees.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Data API enabled on cluster.

---

## Cross-Service Synergies

```
[ Redshift Cluster / Serverless ] 
        │
        ├──(Data Lake Querying)───> [ S3 + Parquet/Partitioning ] (Saves 90% Spectrum $5/TB fees)
        │
        ├──(Pipeline Optimization)─> [ Zero-ETL Integration ] (Eliminates Glue DPU $0.44/hr ETL)
        │
        └──(Cluster Scheduling)───> [ Scheduled Pause/Resume ] (Saves 65% compute on non-prod)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Node:ra3.xlplus`, `RedshiftServerless-RPU-Hours`, `ManagedStorage:GB-mo`, `Spectrum-Bytes-Scanned`, `ConcurrencyScaling`.
- `line_item_resource_id`: Redshift Cluster Identifier / Serverless Workgroup ARN.

### B. CloudWatch & System Tables
- CloudWatch `AWS/Redshift` Metrics: `CPUUtilization`, `PercentageDiskSpaceUsed`, `DatabaseConnections`, `QueryDuration`, `ConcurrencyScalingSeconds`.
- System Views: `SVV_TABLE_INFO`, `STL_QUERY`, `SVL_QLOG`, `STL_DIST_EVENT`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "RS-SCH-001",
  "service": "Redshift",
  "category": "Waste Elimination",
  "resource_id": "arn:aws:redshift:us-east-1:123456789012:cluster:bi-staging-cluster",
  "resource_name": "bi-staging-cluster",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "node_type": "ra3.xlplus",
    "number_of_nodes": 4,
    "schedule": "24/7 Continuous",
    "monthly_compute_cost_usd": 3171.12
  },
  "recommended_config": {
    "node_type": "ra3.xlplus",
    "number_of_nodes": 4,
    "schedule": "Pause 19:00-07:00 UTC M-F + Weekend Pause",
    "projected_monthly_compute_cost_usd": 1110.00
  },
  "financial_impact": {
    "monthly_savings_usd": 2061.12,
    "annual_savings_usd": 24733.44,
    "savings_percentage": 65.0
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "Staging cluster has zero query activity outside business hours; storage remains intact."
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
| **Waste Elimination (Pause/Resume)** | 5 | $14,200.00 | $9,230.00 | 65.0% | Low |
| **Spectrum Data Lake Optimization** | 8 | $8,500.00 | $6,800.00 | 80.0% | Low |
| **Rightsizing (Nodes / Serverless RPU)**| 6 | $12,400.00 | $4,960.00 | 40.0% | Low |
| **Commitment Discounts (Reserved Nodes)**| 2 | $18,000.00 | $6,300.00 | 35.0% | Low |
| **Total** | **21** | **$53,100.00** | **$27,290.00** | **51.4%** | -- |
