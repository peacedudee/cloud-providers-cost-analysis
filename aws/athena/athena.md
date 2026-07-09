# AWS Service Cost Research: Amazon Athena

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Athena is an interactive, serverless query service that allows you to analyze data directly in Amazon S3 using standard SQL. Athena requires no infrastructure provisioning or management. You pay only for the queries you execute. Because Athena pricing is determined directly by the volume of data scanned from S3 during query execution, data formatting, compression, and partitioning in S3 are the primary levers for controlling Athena costs.

---

## 2. Billing Mechanics
Athena is billed on an on-demand, usage-based model with three primary components:
1. **Data Scanned (On-Demand SQL Queries):** Billed per TB of data read by Athena from your S3 buckets during query execution ($5.00 per TB scanned).
2. **Athena for Apache Spark & Provisioned Capacity (Optional):** Billed per Data Processing Unit (DPU) hour for dedicated capacity or Spark execution ($0.35/DPU-hr for Spark).
3. **Underlying S3 Storage & Request Fees:** Standard S3 storage fees ($0.023/GB-mo) and GET request fees apply for data scanned and query results stored.

---

## 3. Key Cost Dimensions

### A. SQL Query Data Scanned (us-east-1 On-Demand)
* **Standard Query Rate:** **$5.00 per TB** ($0.005 per GB) of data scanned from S3.
* **Minimum Scan Charge:** Each query is billed with a **10 MB minimum** scan threshold. If a query scans 50 KB, it is billed for 10 MB.
* **Free DDL Operations:** DDL statements (`CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`) and partition management commands (`MSCK REPAIR TABLE`) are **100% free ($0.00)**.
* **Canceled / Failed Queries:** Billed for the data scanned up to the point of cancellation or failure.

### B. Athena for Apache Spark
* **The Rate:** **$0.35 per DPU-hour** (billed per second with a 1-minute minimum).
* *Resource Specification:* 1 DPU provides 4 vCPUs and 16 GB of RAM.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Billing Basis | Rate (us-east-1) | Example Calculation |
|----------------|---------------|------------------|---------------------|
| **SQL Queries (On-Demand)** | Per TB scanned | **$5.00** | 100 GB scanned = **$0.50** |
| **Spark Execution** | Per DPU-hour | **$0.35** | 20 DPUs for 1 hour = **$7.00** |
| **Query Metadata / DDL** | Per query | **Free ($0.00)** | Running `CREATE TABLE` |
| **Query Results Storage** | Per GB-month | Standard S3 rates | Target query results S3 bucket |

---

## 5. AWS Free Tier Coverage
* **Amazon Athena:** No free tier available. All queries generate immediate usage billing.

---

## 6. Common Cost Hotspots & Pitfalls
* **Querying Uncompressed Text Files (CSV / JSON):** Running queries on raw text files. A query scanning for a single record must scan the entire uncompressed file dataset.
* **Lack of S3 Partitioning:** Storing millions of S3 log files in a single unpartitioned folder, forcing Athena to read every object in the prefix.
* **`SELECT *` Queries on Wide Tables:** Executing `SELECT *` on wide tables, reading all columns across all rows unnecessarily.
* **`LIMIT` Clause Misconception:** Adding `LIMIT 10` to an unpartitioned SQL query does **not** cut query cost; Athena still scans the full dataset before sorting/truncating results.
* **Orphaned Query Results S3 Bucket:** Leaving accumulated query result CSVs and metadata in the default `aws-athena-query-results-*` S3 bucket without lifecycle expiration rules.

---

## 7. Actionable Cost Optimization Strategies
1. **Convert S3 Data to Columnar Formats (Apache Parquet / ORC):**
   * Convert raw CSV/JSON files into **Parquet** or **ORC** using AWS Glue, EMR, or Athena CTAS queries.
   * *Benefit:* Columnar formats allow Athena to scan ONLY the specific columns requested in the SQL query.
   * **The Savings:** Cuts data scanned and query fees by **80% to 99%** (e.g., dropping a $5.00 query to **$0.05**).
2. **Compress All Data Files:** Store data using high-efficiency compression algorithms (such as **Snappy** or **GZIP**). Since Athena charges based on compressed bytes read from S3, compression directly reduces query costs by **3x to 5x**.
3. **Structure S3 Paths into Partitions:** Organize S3 keys into partitions (`s3://bucket/year=2026/month=07/day=02/`) and specify partition filters in `WHERE` clauses to prevent full table scans.
4. **Configure Workgroups with Per-Query & Data Usage Caps:** Set up Athena **Workgroups** to establish hard per-query data limits (e.g., max 50 GB per query) to automatically cancel runaway queries.
5. **Set S3 Lifecycle Rules on Query Results Bucket:** Apply an S3 Lifecycle rule to expire and delete query result files in your Athena results bucket after **7 to 30 days**.
