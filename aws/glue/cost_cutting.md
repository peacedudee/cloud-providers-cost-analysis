# Cost-Cutting Playbook: AWS Glue

> **Companion File:** [glue.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/glue/glue.md)  
> **Last Updated:** July 2026

---

## Executive Summary

AWS Glue is a serverless data integration service offering ETL processing (Spark/Python Shell), Data Quality evaluations, and Data Catalog indexing. Because Glue has no provisioned clusters, billing is calculated strictly per Data Processing Unit hour (**$0.44 per DPU-hour** for Standard execution; **$0.29 per DPU-hour** for Flex execution).

Key billing leaks stem from running non-urgent batch jobs on Standard execution instead of Flex (a **34% price penalty**), over-provisioning default 10-DPU Spark jobs for tiny datasets, scheduling frequent Glue Crawlers that hit the **10-minute minimum billing block** per run, and leaving Interactive Notebook Sessions open ($2.20/hr base).

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **30–65% reduction in total AWS Glue spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Enable Flex Execution for Non-Urgent Nightly ETL Jobs:** Instantly cuts compute costs by **34%** ($0.29/DPU-hr vs $0.44/DPU-hr).
2. **Convert Single-File Python ETL Scripts to Fractional Python Shell Jobs:** Replaces 10-DPU Spark jobs ($4.40/hr) with fractional Python Shell jobs (0.0625 DPU at **$0.0275/hr** — a **99% cost reduction**).
3. **Set Auto-Timeout Limits on Interactive Notebook Sessions (10-15 Mins):** Prevents forgotten developer notebooks from billing $2.20/hour continuously while idle.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Configure Auto-Timeout on Interactive Notebook Sessions
- **What:** Enforce strict idle timeout limits (e.g. 10 or 15 minutes) on all Glue Studio interactive notebook sessions.
- **Why It Saves Money:** Interactive notebook sessions default to 5 DPUs (**$2.20 per hour**). Leaving a developer notebook open overnight or over a weekend wastes **$52.80 per 24-hour period**.
- **Detailed Implementation Steps:**
  1. Add `%idle_timeout 15` magic command at the start of Glue notebook scripts.
  2. Set default session timeout via AWS CLI:
     ```bash
     aws glue update-dev-endpoint --dev-endpoint-name dev-ep --extra-arguments '{"--idle-timeout": "15"}'
     ```
- **Estimated Savings:** 100% of idle notebook session billing ($100s/mo per data engineer).
- **Risk Level:** Zero risk.
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** Glue Studio notebook configuration access.

#### 2. Audit & Eliminate High-Frequency Crawler Storms (Bypass 10-Min Billing Minimum)
- **What:** Re-architect Glue Crawlers scheduled to run every 15 minutes down to daily schedules, or replace crawlers with manual partition registration (`ALTER TABLE ADD PARTITION`).
- **Why It Saves Money:** Crawlers bill with a **10-minute minimum charge** ($0.44/DPU-hr for 2 DPUs = $0.147 per run minimum). Running a crawler every 15 minutes that finishes in 30 seconds still bills 10 minutes every time, costing **$425.00/month** for a tiny S3 folder!
- **Detailed Implementation Steps:**
  1. Audit crawler schedules via CLI:
     ```bash
     aws glue list-crawlers --query "CrawlerNames"
     ```
  2. Switch S3 partition addition from Crawlers to Athena DDL in ETL scripts:
     ```sql
     ALTER TABLE catalog_db.my_table ADD PARTITION (year='2026', month='07', day='03') LOCATION 's3://bucket/path/';
     ```
  3. DDL statements in Athena are **100% FREE ($0.00)**.
- **Estimated Savings:** 90–100% reduction in crawler DPU charges.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Partition naming scheme compliance.

---

### 2. Execution Mode & Pricing Model Optimization

#### 3. Enable Flex Execution for Non-Urgent Nightly ETL Jobs
- **What:** Set the execution class of overnight, batch, or non-time-critical ETL jobs to `FLEX`.
- **Why It Saves Money:** Flex execution runs on spare AWS compute capacity, providing a **34% direct price discount** (**$0.29 per DPU-hour** vs Standard at **$0.44 per DPU-hour**).
- **Detailed Implementation Steps:**
  1. Modify job configuration in Terraform or AWS CLI:
     ```bash
     aws glue update-job \
       --job-name "nightly-data-warehouse-etl" \
       --job-update "ExecutionClass=FLEX,Role=GlueExecutionRole"
     ```
  2. In Terraform:
     ```hcl
     resource "aws_glue_job" "nightly_etl" {
       name            = "nightly-data-warehouse-etl"
       role_arn        = aws_iam_role.glue_role.arn
       execution_class = "FLEX"
       command {
         script_location = "s3://my-bucket/scripts/etl.py"
       }
     }
     ```
- **Estimated Savings:** **34% direct discount** on ETL job compute spend.
- **Risk Level:** Low (Flex jobs can experience start delays up to 10 minutes during peak AWS demand; ideal for non-SLA batch jobs).
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** Job must tolerate flexible start/completion windows.

#### 4. Migrate Single-File / Simple Tasks to Python Shell Jobs
- **What:** Convert simple single-file data cleanup, API ingestion, or lightweight CSV transformation tasks from Glue Spark jobs to **Glue Python Shell Jobs**.
- **Why It Saves Money:** Standard Spark jobs require a minimum of 2 to 10 DPUs ($0.88 to $4.40/hr). Python Shell jobs support fractional DPUs down to **0.0625 DPU ($0.0275/hr)** — a **99% cost reduction** for non-distributed tasks!
- **Detailed Implementation Steps:**
  1. Re-package Python script using Pandas/Boto3 instead of PySpark.
  2. Create Python Shell job with 0.0625 DPU:
     ```bash
     aws glue create-job \
       --name "lightweight-csv-cleaner" \
       --role GlueExecutionRole \
       --command '{"Name": "pythonshell", "PythonVersion": "3.9", "ScriptLocation": "s3://bucket/script.py"}' \
       --default-arguments '{"--allocated-capacity": "0.0625"}'
     ```
- **Estimated Savings:** 90–99% cost reduction for small data processing scripts.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Dataset fits in single-node memory (< 1 GB for 0.0625 DPU).

---

### 3. DPU Sizing & Auto-Scaling Optimization

#### 5. Enable AWS Glue Auto Scaling
- **What:** Enable **Glue Auto Scaling** on all Spark ETL jobs.
- **Why It Saves Money:** Standard Glue jobs allocate a fixed number of DPUs for the entire execution runtime. Auto Scaling dynamically adds DPUs during heavy shuffle/transform stages and releases them during single-threaded read/write stages, preventing over-provisioning.
- **Detailed Implementation Steps:**
  1. Enable Auto Scaling in job parameters:
     ```bash
     aws glue update-job \
       --job-name "dynamic-spark-etl" \
       --job-update '{"DefaultArguments": {"--enable-auto-scaling": "true"}, "MaxCapacity": 20.0}'
     ```
- **Estimated Savings:** 20–50% reduction in total DPU-hours billed per job.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** Glue version 3.0 or 4.0+.

#### 6. Right-Size Worker Types (G.1X vs G.2X vs G.025X vs Z.2X)
- **What:** Match Glue worker type parameters (`WorkerType`) to workload memory and CPU requirements:
  - `G.025X` (0.25 DPU / 2 vCPU / 4 GB RAM): Best for streaming/lightweight transformation.
  - `G.1X` (1 DPU / 4 vCPU / 16 GB RAM): Standard baseline for memory-balanced Spark.
  - `G.2X` (2 DPU / 8 vCPU / 32 GB RAM): High-memory workloads / complex joins.
- **Why It Saves Money:** Using `G.2X` workers for simple filter operations doubles DPU usage unnecessarily.
- **Detailed Implementation Steps:**
  1. Inspect job metrics in CloudWatch (`glue.driver.aggregate.cpuUtilization`, `glue.driver.jvm.heap.usage`).
  2. Downsize worker type to `G.1X` or `G.025X` if RAM/CPU utilization is low.
- **Estimated Savings:** 30–50% per worker step-down.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Job metrics analysis.

#### 7. Set Strict `MaxCapacity` Caps on Development & Test Jobs
- **What:** Restrict default `MaxCapacity` on development and testing Glue jobs to 2 DPUs.
- **Why It Saves Money:** Prevents developers from launching 50-DPU test jobs during script debugging, which bill $22.00/hour per run.
- **Detailed Implementation Steps:**
  1. Enforce IAM policy or SCP restricting `--max-capacity` > 2 on non-prod roles:
     ```json
     {
       "Effect": "Deny",
       "Action": ["glue:CreateJob", "glue:UpdateJob"],
       "Resource": "*",
       "Condition": {
         "NumericGreaterThan": {"glue:MaxCapacity": "2.0"}
       }
     }
     ```
- **Estimated Savings:** Prevents multi-hundred dollar developer testing overruns.
- **Risk Level:** Low.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** IAM policy setup.

---

### 4. Performance & Job Code Optimization

#### 8. Set Aggressive Job Timeouts (`--timeout`)
- **What:** Configure the `--timeout` parameter on all Glue jobs to `30` or `60` minutes instead of the default 48-hour (2,880 minute) timeout.
- **Why It Saves Money:** If a Spark job hangs due to a deadlocked S3 connection or infinite loop, a job with default timeout runs for 48 hours, billing **$422.40 per 10-DPU job**.
- **Detailed Implementation Steps:**
  1. Set timeout parameter in job definition:
     ```bash
     aws glue update-job \
       --job-name "sales-etl" \
       --job-update '{"Timeout": 60}'
     ```
- **Estimated Savings:** Prevents 48-hour runaway job cost spikes.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** Baseline job duration tracking.

#### 9. Enable Job Bookmarks to Avoid Re-Processing Historical Data
- **What:** Enable **Glue Job Bookmarks** (`--job-bookmark-option job-bookmark-enable`) on stateful ETL pipelines.
- **Why It Saves Money:** Job Bookmarks maintain state tracking across S3 paths, ensuring subsequent job runs process ONLY newly added files, reducing processing volume and execution duration by 90%+.
- **Detailed Implementation Steps:**
  1. Enable bookmark in job parameters:
     ```bash
     aws glue update-job \
       --job-name "incremental-s3-etl" \
       --job-update '{"DefaultArguments": {"--job-bookmark-option": "job-bookmark-enable"}}'
     ```
- **Estimated Savings:** 50–90% reduction in incremental ETL runtimes.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** S3 data source supported by Glue Bookmarks.

#### 10. Optimize S3 Partition Pruning (Use Pushdown Predicates)
- **What:** Add `push_down_predicate` parameters to Glue DynamicFrame reads: `push_down_predicate = "year == '2026' AND month == '07'"`.
- **Why It Saves Money:** Filters S3 files at the catalog level before reading data into Spark memory, skipping 95%+ of un-needed S3 file reads.
- **Detailed Implementation Steps:**
  1. Update PySpark Glue code:
     ```python
     datasource = glueContext.create_dynamic_frame.from_catalog(
         database = "sales_db",
         table_name = "orders",
         push_down_predicate = "year == '2026' AND month == '07'"
     )
     ```
- **Estimated Savings:** 60–90% duration and data read savings.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Partitioned S3 catalog tables.

#### 11. Avoid Data Shuffling with Custom Grouping & Repartitioning Tuning
- **What:** Minimize `df.repartition()` and expensive wide-dependency transformations (`groupByKey`, `distinct`) in Spark job scripts.
- **Why It Saves Money:** Data shuffling passes data across Glue worker nodes over the network, incurring heavy CPU/memory overhead and extending job runtime.
- **Detailed Implementation Steps:**
  1. Use `coalesce()` instead of `repartition()` when reducing partition counts.
  2. Replace `groupByKey` with `reduceByKey` or `aggregateByKey`.
- **Estimated Savings:** 20–40% execution speedup.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** PySpark code refactoring.

---

### 5. Data Catalog & Quality Optimization

#### 12. Maximize Free Glue Data Catalog Allowance (1M Objects)
- **What:** Clean up unneeded table partitions and expired metadata objects in the Glue Data Catalog.
- **Why It Saves Money:** The Data Catalog provides **1 Million free metadata objects** per month. Beyond 1M, storage costs **$1.00 per 100,000 objects/mo**.
- **Detailed Implementation Steps:**
  1. Run catalog partition pruning scripts:
     ```bash
     aws glue batch-delete-partition \
       --database-name temp_db \
       --table-name staging_table \
       --partitions-to-delete '[{"Values": ["2024","01","01"]}]'
     ```
- **Estimated Savings:** Keeps catalog storage within free tier limits.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Data retention policy audit.

#### 13. Optimize Glue Data Quality Anomaly Detection Surcharges
- **What:** Configure Glue Data Quality evaluation rules selectively on key data pipelines rather than enabling full anomaly detection across all tables.
- **Why It Saves Money:** Anomaly detection monitors incur an extra surcharge of **1 DPU per statistic** monitored for the duration of the detection job.
- **Detailed Implementation Steps:**
  1. Restrict Data Quality rules to critical columns (`completeness`, `uniqueness`).
- **Estimated Savings:** 30–50% reduction in Data Quality DPU overhead.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Data Quality rule review.

---

### 6. Network & Data Transfer Optimization

#### 14. Deploy Gateway VPC Endpoints for S3 in Glue VPC Connections
- **What:** Ensure Glue VPC Connections use subnets attached to route tables with **Gateway VPC Endpoints for S3**.
- **Why It Saves Money:** Prevents Glue workers inside private subnets from transferring S3 datasets through NAT Gateways ($0.045/GB processing tax). S3 Gateway endpoints route data for **100% FREE ($0.00/GB)**.
- **Detailed Implementation Steps:**
  1. Attach Gateway VPC Endpoint to Glue subnet route table.
- **Estimated Savings:** Saves $45.00 per 1 TB of data processed by Glue.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Glue VPC connection setup.

#### 15. Compress Job Outputs (Parquet + Snappy)
- **What:** Configure Glue output writers to use Snappy-compressed Apache Parquet format.
- **Why It Saves Money:** Slashes S3 data storage costs by 80%+ and reduces network write duration.
- **Detailed Implementation Steps:**
  1. Set output format in PySpark:
     ```python
     glueContext.write_dynamic_frame.from_options(
         frame = transformed_frame,
         connection_type = "s3",
         connection_options = {"path": "s3://bucket/output/"},
         format = "parquet",
         format_options = {"compression": "snappy"}
     )
     ```
- **Estimated Savings:** 80% storage cost reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** None.

#### 16. Monitor & Delete Unused Glue Development Endpoints
- **What:** Delete legacy Glue Dev Endpoints (`aws glue delete-dev-endpoint`) that are no longer active.
- **Why It Saves Money:** Dev Endpoints bill continuous hourly DPU charges ($0.44/DPU-hr) while active.
- **Detailed Implementation Steps:**
  1. List dev endpoints: `aws glue get-dev-endpoints`.
  2. Delete unused endpoints.
- **Estimated Savings:** 100% of idle Dev Endpoint fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Developer confirmation.

---

## Cross-Service Synergies

```
[ AWS Glue ETL ] 
        │
        ├──(Execution Class)───> [ Flex Execution ] (34% direct DPU price discount)
        │
        ├──(Small Scripts)─────> [ Python Shell Jobs (0.0625 DPU) ] (99% cost reduction vs Spark)
        │
        └──(Network Routing)───> [ Gateway VPC Endpoint for S3 ] (100% FREE - Avoids NAT $0.045/GB)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Glue-DPU-Hour`, `Glue-Flex-DPU-Hour`, `Glue-PythonShell-DPU-Hour`, `Crawler-DPU-Hour`, `Catalog-Objects`.
- `line_item_resource_id`: Glue Job Name / Crawler Name / Catalog Database.

### B. CloudWatch Metrics
- `AWS/Glue` Namespace: `glue.driver.aggregate.cpuUtilization`, `glue.driver.jvm.heap.usage`, `glue.ALL.s3.filesystem.read_bytes`, `glue.ALL.s3.filesystem.write_bytes`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "GLUE-FLX-001",
  "service": "Glue",
  "category": "Execution Mode & Pricing Model",
  "resource_id": "arn:aws:glue:us-east-1:123456789012:job/nightly-warehouse-refresh",
  "resource_name": "nightly-warehouse-refresh",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "execution_class": "STANDARD",
    "allocated_dpus": 20,
    "avg_runtime_hours": 3.0,
    "monthly_runs": 30,
    "monthly_cost_usd": 792.00
  },
  "recommended_config": {
    "execution_class": "FLEX",
    "allocated_dpus": 20,
    "projected_monthly_cost_usd": 522.00
  },
  "financial_impact": {
    "monthly_savings_usd": 270.00,
    "annual_savings_usd": 3240.00,
    "savings_percentage": 34.1
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "Job runs overnight at 2:00 AM UTC; 10-minute Flex launch buffer does not breach morning SLA."
  },
  "implementation": {
    "scope": "Data Engineer / DevOps",
    "effort_estimate": "15 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Flex Execution (Nightly Jobs)** | 14 | $9,400.00 | $3,196.00 | 34.0% | Low |
| **Python Shell Migration** | 10 | $4,500.00 | $4,050.00 | 90.0% | Low |
| **Crawler Optimization (10-Min Min)** | 8 | $3,200.00 | $2,880.00 | 90.0% | Low |
| **Notebook Auto-Timeout (15-Min)** | 5 | $1,800.00 | $1,800.00 | 100.0% | Zero |
| **Total** | **37** | **$18,900.00** | **$11,926.00** | **63.1%** | -- |
