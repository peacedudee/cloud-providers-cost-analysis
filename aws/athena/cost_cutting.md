# Cost-Cutting Playbook: Amazon Athena

> **Companion File:** [athena.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/athena/athena.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon Athena is an interactive, serverless analytics query engine that analyzes S3 data lakes using standard SQL. Athena bills on an **On-Demand model ($5.00 per TB scanned)** or via **Capacity Reservations (DPU-hours)**. Athena enforces a **10 MB minimum scan rule** per query.

Major billing traps include executing `SELECT *` queries against unpartitioned, uncompressed CSV or JSON text logs in S3 ($5.00/TB scanned), running frequent polling dashboards that scan hundreds of gigabytes per hour, and leaving Provisioned Capacity DPUs running idle ($24.00/hour = $17,520/month base).

This playbook provides **18 actionable strategies** across six operational categories, delivering an estimated **40–90% reduction in total Athena query spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Convert S3 Log Files from CSV/JSON to Apache Parquet / ORC:** Columnar compression slashes query scanned data by **80–99%**, reducing query costs from $5.00/TB to **$0.05–$1.00/TB**.
2. **Enforce Partition Pruning in SQL Queries (`WHERE year='2026' AND month='07'`):** Skips scanning 95%+ of historical S3 folders.
3. **Set Workgroup Per-Query & Per-Hour Scan Limits:** Caps maximum scanned data per query (e.g. max 100 GB scan limit) to prevent runaway user queries ($5.00/TB).

---

## Strategy Categories

### 1. Storage Format & Compression Optimization (Columnar Storage)

#### 1. Convert Raw CSV/JSON Data to Compressed Apache Parquet or ORC
- **What:** Re-package raw text datasets (CSV, JSON, TSV) in S3 into columnar **Apache Parquet** or **ORC** format using Snappy compression.
- **Why It Saves Money:** Athena bills **$5.00 per TB of data scanned**. Text files force Athena to scan the entire 1 TB dataset even if your query selects only 2 columns. Columnar storage reads *only* the specific columns requested and applies Snappy compression, reducing data scanned by **80% to 99%**. Scanning 1 TB of CSV ($5.00) drops to 10 GB of Parquet (**$0.05**) — a **99% cost reduction**!
- **Detailed Implementation Steps:**
  1. Create a Parquet CTAS (Create Table As Select) statement in Athena:
     ```sql
     CREATE TABLE datalake.orders_parquet
     WITH (
       format = 'PARQUET',
       parquet_compression = 'SNAPPY',
       external_location = 's3://company-datalake/orders_parquet/'
     ) AS
     SELECT * FROM datalake.raw_csv_orders;
     ```
  2. Direct downstream BI tools to query `orders_parquet`.
- **Estimated Savings:** **80–99% reduction** in query scan costs.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** S3 write permissions.

#### 2. Avoid SELECT * (Specify Explicit Column Projections)
- **What:** Enforce team query standards requiring explicit column names in `SELECT` statements (e.g. `SELECT order_id, amount FROM ...`) instead of `SELECT *`.
- **Why It Saves Money:** On Parquet/ORC columnar formats, Athena reads ONLY the byte blocks of the requested columns. Selecting 2 columns out of 50 reads 4% of the file size, dropping query billing by **96%**.
- **Detailed Implementation Steps:**
  1. Add query linting in BI dashboards (QuickSight, Grafana, Tableau).
  2. Replace `SELECT *` with specific field selections.
- **Estimated Savings:** 50–95% cost reduction per columnar query.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Data Analyst / BI Developer
- **Prerequisites:** Parquet/ORC dataset usage.

---

### 2. Partitioning & Data Pruning Strategies

#### 3. Implement S3 Partitioning and Enforce WHERE Clause Pruning
- **What:** Structure S3 data storage paths by logical partition keys (`s3://bucket/table/year=2026/month=07/day=03/`) and require `WHERE` clause partition filters in all queries.
- **Why It Saves Money:** Partition pruning instructs Athena to scan ONLY the specific S3 folder paths matching the query predicate, skipping historical years/months entirely. Querying 1 day out of 3 years of logs drops data scanned by **99.9%** (from $15.00 down to $0.015 per query).
- **Detailed Implementation Steps:**
  1. Define table schema with `PARTITIONED BY`:
     ```sql
     CREATE EXTERNAL TABLE datalake.logs (
       user_id string,
       event string
     )
     PARTITIONED BY (year string, month string, day string)
     STORED AS PARQUET
     LOCATION 's3://company-datalake/logs/';
     ```
  2. Enable **Partition Projection** in table properties to eliminate `MSCK REPAIR TABLE` overhead:
     ```sql
     TBLPROPERTIES (
       'projection.enabled' = 'true',
       'projection.year.type' = 'integer',
       'projection.year.range' = '2024,2026',
       'projection.month.type' = 'integer',
       'projection.month.range' = '1,12'
     )
     ```
- **Estimated Savings:** 90–99.9% query scan cost reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** S3 directory prefix convention alignment.

#### 4. Consolidate Small Files (Avoid 10 MB Minimum Scan Penalty)
- **What:** Run S3 file compaction jobs (using EMR, Glue, or Spark) to consolidate millions of tiny S3 files (< 128 KB) into optimal **128 MB to 512 MB file sizes**.
- **Why It Saves Money:**
  1. Athena enforces a **10 MB minimum billable scan limit per query**. Executing a query over 1,000 tiny 4 KB CSV files bills as 1,000 × 10 MB = **10,000 MB (10 GB = $0.05)**, even though the actual data size is only 4 MB (a **2,500x billing inflation**!).
  2. Small files degrade Athena S3 `LIST` metadata performance by 10x.
- **Detailed Implementation Steps:**
  1. Run compacting PySpark script:
     ```python
     df = spark.read.parquet("s3://bucket/tiny-files/")
     df.coalesce(10).write.mode("overwrite").parquet("s3://bucket/compacted-files/")
     ```
- **Estimated Savings:** 90–99% reduction in small-file billing inflation.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** S3 compaction pipeline setup.

---

### 3. Workgroup & Query Governance

#### 5. Enforce Per-Query & Per-Hour Scan Limits in Athena Workgroups
- **What:** Configure Athena Workgroups with strict data usage limits (`BytesScannedCutoffPerQuery` and `PublishMetrics`).
- **Why It Saves Money:** Prevents a single un-partitioned or un-indexed user query from scanning 20 TB of data lake storage and generating an unexpected **$100.00 bill for a single query**.
- **Detailed Implementation Steps:**
  1. Create or update Athena Workgroup via AWS CLI:
     ```bash
     aws athena update-workgroup \
       --workgroup Analytics-Team \
       --configuration-updates "BytesScannedCutoffPerQuery=107374182400,EnforceWorkGroupConfiguration=true"
     ```
  2. Sets a strict **100 GB per query scan limit ($0.50 max cost)**. Queries exceeding 100 GB are automatically cancelled before incurring fees.
- **Estimated Savings:** Prevents 100% of runaway user query billing spikes.
- **Risk Level:** Zero risk (safeguards production budgets).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workgroup governance setup.

#### 6. Isolate Workloads Across Dedicated Athena Workgroups
- **What:** Assign separate Athena Workgroups to different teams, automated BI dashboards, and ad-hoc developers.
- **Why It Saves Money:** Enables granular cost tracking, per-team monthly budget caps, and independent query result location isolation.
- **Detailed Implementation Steps:**
  1. Create workgroup per department: `Finance-Workgroup`, `Marketing-Workgroup`.
  2. Attach IAM policies restricting developers to their respective workgroup.
- **Estimated Savings:** Cost visibility and accountability governance.
- **Risk Level:** Zero risk.
- **Implementation Scope:** DevOps / FinOps Team
- **Prerequisites:** IAM policy configuration.

---

### 4. BI Dashboard & Caching Optimization

#### 7. Enable Query Result Reuse (Athena Query Result Caching)
- **What:** Enable **Query Result Reuse** (`result_reuse_configuration`) on repetitive dashboard queries or BI tools.
- **Why It Saves Money:** If an identical SQL query is executed within a specified age window (e.g. 60 minutes), Athena returns the cached result from S3 instantly for **100% FREE ($0.00 data scanned)**.
- **Detailed Implementation Steps:**
  1. Pass result reuse parameter in Athena API `StartQueryExecution`:
     ```bash
     aws athena start-query-execution \
       --query-string "SELECT category, SUM(amount) FROM datalake.sales GROUP BY category" \
       --result-reuse-configuration "ResultReuseByAgeConfiguration={Enabled=true,MaxAgeInMinutes=60}"
     ```
- **Estimated Savings:** 50–90% reduction in BI dashboard query fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Software Engineer / BI Developer
- **Prerequisites:** Query pattern repetition.

#### 8. Reduce BI Dashboard Refresh Frequencies
- **What:** Adjust QuickSight, Tableau, or Grafana dashboard auto-refresh rates from 1-minute intervals down to 1-hour or daily schedules.
- **Why It Saves Money:** A Grafana panel scanning 10 GB per refresh every 1 minute executes 43,200 queries/month, scanning 432 TB ($2,160.00/month). Refreshing hourly drops queries to 730/month, scanning 7.3 TB (**$36.50/month**) — saving **$2,123.50/month**!
- **Detailed Implementation Steps:**
  1. Update QuickSight/Tableau schedule settings to hourly/daily.
- **Estimated Savings:** **98% cost reduction** on automated dashboard query spend.
- **Risk Level:** Low.
- **Implementation Scope:** BI Developer / Data Analyst
- **Prerequisites:** User SLA verification for real-time dashboard needs.

---

### 5. Provisioned Capacity vs On-Demand Sizing

#### 9. Audit Capacity Reservations (DPU-Hours) vs On-Demand
- **What:** Evaluate Athena **Capacity Reservations** (DPUs at $0.24 per DPU-hour = $24.00/hr per 100 DPUs) against On-Demand usage ($5.00/TB).
- **Why It Saves Money:** Provisioned Capacity requires a minimum commitment of 100 DPUs (**$17,520.00/month flat**). If your team scans less than 3.5 PB of data per month, On-Demand pricing is significantly cheaper!
- **Detailed Implementation Steps:**
  1. Calculate monthly scan volume in CUR:
     $$\text{Break-Even Scan Volume} = \frac{\$17,520.00}{\$5.00/\text{TB}} = 3,504\text{ TB (3.5 PB)}$$
  2. If monthly scan volume < 3.5 PB, release Capacity Reservation and return to On-Demand.
- **Estimated Savings:** Eliminates up to $17,520.00/month in idle capacity fees.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team / Data Architect
- **Prerequisites:** Monthly scan volume calculation.

---

### 6. Query Code & Engine Tuning

#### 10. Avoid Sorting Large Datasets (`ORDER BY` Without Limits)
- **What:** Remove global `ORDER BY` clauses on multi-terabyte datasets unless coupled with strict `LIMIT` clauses.
- **Why It Saves Money:** Forces Athena engine nodes to spill intermediate data blocks to disk, extending query execution duration and memory overhead.
- **Detailed Implementation Steps:**
  1. Audit slow query logs for `ORDER BY` execution.
- **Estimated Savings:** Query performance speedup.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Data Analyst
- **Prerequisites:** SQL code review.

#### 11. Use `APPROX_DISTINCT` for Cardinality Calculations
- **What:** Replace `COUNT(DISTINCT user_id)` with `APPROX_DISTINCT(user_id)` in analytical summaries.
- **Why It Saves Money:** `APPROX_DISTINCT` uses HyperLogLog algorithm, consuming 90% less memory and processing execution time while maintaining 99%+ accuracy.
- **Detailed Implementation Steps:**
  1. Replace SQL syntax in analytical queries:
     ```sql
     SELECT APPROX_DISTINCT(user_id) FROM datalake.events;
     ```
- **Estimated Savings:** 30–60% query execution duration reduction.
- **Risk Level:** Zero risk (for analytical reporting).
- **Implementation Scope:** Data Analyst
- **Prerequisites:** None.

#### 12. Utilize Athena Engine Version 3
- **What:** Ensure all Athena Workgroups are set to **Athena Engine Version 3**.
- **Why It Saves Money:** Engine v3 includes updated Trino/Presto query optimizer engines, improving query execution speed by up to 20% over legacy v2 engines.
- **Detailed Implementation Steps:**
  1. Update Workgroup engine version:
     ```bash
     aws athena update-workgroup --workgroup primary --configuration-updates "EngineVersion={SelectedEngineVersion='Athena engine version 3'}"
     ```
- **Estimated Savings:** 10–20% performance improvement.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workgroup configuration access.

#### 13. Clean Up Expired Athena Query Results in S3
- **What:** Attach an S3 Lifecycle Policy to the Athena Query Results S3 bucket (`s3://aws-athena-query-results-123456789012-us-east-1/`) to automatically delete CSV/JSON query output files older than 7 days.
- **Why It Saves Money:** Athena stores every query result output in S3 indefinitely by default. Retaining 10 TB of historical query results wastes **$230.00/month** in S3 Standard storage.
- **Detailed Implementation Steps:**
  1. Add S3 Lifecycle Rule to Athena results bucket: Expiration = 7 days.
- **Estimated Savings:** 100% of query result output storage accumulation.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Query result bucket identified.

#### 14. Pre-Aggregate Heavy Summaries via CTAS Scheduled Pipelines
- **What:** Create daily pre-aggregated summary tables via Scheduled CTAS queries (or Glue/EMR) for high-frequency BI reporting.
- **Why It Saves Money:** BI tools query a 1 GB aggregated summary table ($0.005 per scan) instead of running full-table aggregation over a 5 TB raw event dataset ($25.00 per scan).
- **Detailed Implementation Steps:**
  1. Schedule daily CTAS query in EventBridge / Step Functions.
- **Estimated Savings:** **99.9% cost reduction** for repeated BI aggregations.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** ETL pipeline scheduling tool.

#### 15. Enforce CloudWatch Alarms on Monthly Athena Scan Spend
- **What:** Deploy CloudWatch billing alarm on `ProcessedBytes` metric across Athena workgroups.
- **Why It Saves Money:** Provides immediate alerts when team query scan volume approaches monthly budget caps.
- **Detailed Implementation Steps:**
  1. Put CloudWatch metric alarm on `AWS/Athena` `ProcessedBytes`.
- **Estimated Savings:** Operational risk protection.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 16. Optimize Federated Queries via Connector Caching
- **What:** Enable metadata and data caching on Athena Federated Connectors (e.g. Athena DynamoDB or PostgreSQL connectors).
- **Why It Saves Money:** Prevents Athena federated queries from overloading underlying operational databases and multiplying scan execution times.
- **Detailed Implementation Steps:**
  1. Configure `spill_bucket` and `auto_dispose` in connector environment variables.
- **Estimated Savings:** 30–50% duration reduction on federated queries.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Athena Federated Query usage.

#### 17. Use Amazon S3 Express One Zone for High-Frequency Query Scratch Space
- **What:** Store temporary intermediate query tables in S3 Express One Zone for high-throughput sub-millisecond execution.
- **Why It Saves Money:** Reduces query latency for complex multi-stage joins.
- **Detailed Implementation Steps:**
  1. Configure CTAS target to S3 Express One Zone bucket.
- **Estimated Savings:** Latency and efficiency gains.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** S3 Express One Zone bucket setup.

#### 18. Audit Free Tier Allocations
- **What:** Verify non-prod test queries take advantage of initial account allocations.
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
[ Amazon Athena Query Engine ] 
        │
        ├──(Columnar Format)───> [ Apache Parquet + Snappy ] (80-99% scan fee reduction)
        │
        ├──(Query Caching)─────> [ Query Result Reuse (60 Min) ] (100% FREE - $0.00 data scanned)
        │
        └──(Query Governance)──> [ Workgroup Scan Limit (100 GB) ] (Prevents $100+ single query spikes)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Analysis-Bytes-Scanned`, `Capacity-DPU-Hour`.
- `line_item_resource_id`: Athena Workgroup Name (`arn:aws:athena:us-east-1:123456789012:workgroup/primary`).

### B. CloudWatch & Workgroup Metrics
- `AWS/Athena` Namespace: `ProcessedBytes`, `QueryExecutionTime`, `EngineExecutionTime`, `TotalExecutionTime`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "ATH-PRQ-001",
  "service": "Athena",
  "category": "Storage Format & Compression Optimization",
  "resource_id": "arn:aws:athena:us-east-1:123456789012:workgroup/Analytics-Team",
  "resource_name": "Analytics-Team",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "data_format": "RAW_CSV",
    "monthly_scanned_tb": 1250.0,
    "monthly_cost_usd": 6250.00
  },
  "recommended_config": {
    "data_format": "PARQUET_SNAPPY",
    "projected_monthly_scanned_tb": 25.0,
    "projected_monthly_cost_usd": 125.00
  },
  "financial_impact": {
    "monthly_savings_usd": 6125.00,
    "annual_savings_usd": 73500.00,
    "savings_percentage": 98.0
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Converting CSV to Parquet via CTAS retains 100% of data fields with 98% scan compression."
  },
  "implementation": {
    "scope": "Data Engineer",
    "effort_estimate": "2-4 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Columnar Format Conversion (Parquet)**| 12 | $18,500.00 | $18,130.00 | 98.0% | Zero |
| **Partition Pruning Enforcement** | 15 | $14,200.00 | $12,780.00 | 90.0% | Low |
| **BI Query Result Reuse (Caching)** | 8 | $8,400.00 | $6,720.00 | 80.0% | Zero |
| **Workgroup Query Cutoff Safeguards** | 10 | $6,500.00 | $5,200.00 | 80.0% | Zero |
| **Total** | **45** | **$47,600.00** | **$42,830.00** | **90.0%** | -- |
