# BigQuery Cost Optimization & Research

Google BigQuery is a fully managed, serverless, highly scalable cloud data warehouse. BigQuery separates compute (query analysis) from storage, billing each separately. While this separation makes it easy to store petabytes of data, an unoptimized query or bad table schema can run up thousands of dollars in seconds. This file covers essential cost-saving tactics for BigQuery.

---

## 1. BigQuery Billing Structure

BigQuery billing is divided into three main dimensions:

### A. Analysis (Query Execution)
BigQuery offers two compute pricing models:
1. **On-Demand (Per-TB model):** Billed based on the number of bytes scanned during query execution ($6.25 per TB). The first 1 TB per month is free.
2. **Capacity Pricing (Editions & Slots):** You pay for dedicated query processing capacity, measured in **Slots** (virtual CPUs).
   * Available in **Standard**, **Enterprise**, and **Enterprise Plus** editions.
   * Billed per slot-hour, with support for autoscaling (autoscaler adjusts slots to match workload demands) and 1-year/3-year slot commitments.

### B. Storage Capacity
BigQuery storage pricing is very cost-effective:
* **Active Storage ($0.020/GB/month):** For tables/partitions modified within the last 90 days.
* **Long-Term Storage ($0.010/GB/month):** For tables/partitions **unmodified for 90 consecutive days**. BigQuery automatically drops the storage price by **50%** with no performance degradation.

### C. Data Ingestion
* **Batch Loading (Free):** Loading data from files, Google Cloud Storage, or other databases is free of charge.
* **Streaming Ingestion:** Billed per MB ingested using the Storage Write API or legacy `tabledata.insertAll` API.

---

## 2. Core Cost-Reduction Tactics (Queries)

### A. Stop the `SELECT *` Habit
BigQuery uses a columnar storage architecture. It only reads data from columns explicitly requested in the query.
* **The Cost Trap:** Running `SELECT * FROM table` forces BigQuery to scan every single column in the table, maximizing the bytes processed and billing.
* **Action:** Never use `SELECT *` in production scripts or BI dashboards. Explicitly name only the columns required (e.g., `SELECT user_id, transaction_date FROM table`).

### B. Implement Table Partitioning & Clustering
Every large table (greater than 10 GB) should use partitioning and clustering:
1. **Partitioning:**
   * Divides a table into segments based on a date, timestamp, or integer range column.
   * **Cost Benefit:** When queries filter on the partition column (e.g., `WHERE order_date = '2026-07-01'`), BigQuery only scans that day's partition, reducing bytes processed by up to 99%.
2. **Clustering:**
   * Sorts data within each partition based on the values of up to four columns (e.g., sorting by `customer_id` and `event_type`).
   * **Cost Benefit:** Further narrows the search space by using block metadata to skip blocks that do not match the query filters.

### C. Enforce Query Safety Caps (Max Bytes Billed)
* Developers or analysts can easily execute queries with accidental cartesian products (cross-joins), scanning petabytes of data and generating massive bills.
* **Action:** In the BigQuery Query Settings, set the **Maximum bytes billed** cap. If a query estimates a scan volume higher than this limit, BigQuery will immediately abort the run before executing, costing you $0.
* **Tactic:** Enforce a project-wide daily scan quota (e.g., max 10 TB limit per day per user) using IAM quotas.

### D. Use Dry Runs to Estimate Costs
* Before executing any query in the console, API, or CLI, perform a **Dry Run**.
* A dry run validates the query syntax and returns the exact number of bytes that will be scanned without executing the query or incurring charges.

### E. Leverage BI Engine for Dashboards
* Connecting Looker, Looker Studio, or Tableau directly to BigQuery tables can trigger massive query costs as users click and filter dashboards, spawning duplicate scans.
* **Action:** Allocate a **BI Engine memory reservation** to your project. BI Engine caches tables in-memory, serving dashboard interactions instantly and bypassing BigQuery scan charges.

---

## 3. Storage & Ingestion Optimization

### A. Minimize Streaming Inserts
* Streaming data into BigQuery in real-time is expensive.
* **Action:** Review data ingestion paths. If an application does not require real-time analytics (e.g., updates are fine on a 1-hour delay), buffer the data and write it using **Free Batch Loading** via Cloud Storage.

### B. Table Expiration Policies
* Temporary staging tables, testing tables, and scratch datasets often remain active indefinitely.
* **Action:** Set a default table/dataset expiration policy (e.g., tables expire after 7 days) on all staging or sandbox datasets to automatically purge unwanted data.

### C. Co-locate Data to Avoid Egress Charges
* Loading data from GCS or federated queries across different regions (e.g., reading a bucket in `europe-west1` into a BigQuery dataset in `us-central1`) incurs network egress charges.
* **Action:** Keep GCS source buckets and BigQuery datasets in the exact same region.

---

## 4. BigQuery Audit Checklist

1. [ ] **`SELECT *` Audit:** Search the codebase/dashboard SQL for `SELECT *` and replace with specific columns.
2. [ ] **Partitioning and Clustering Check:** Identify all tables > 10 GB. Ensure they are partitioned and clustered.
3. [ ] **Daily Scan Quotas:** Configure daily query scan quotas at the user and project levels.
4. [ ] **Dry Run Validation:** Ensure all analytical scripts execute a dry run/estimation check prior to execution.
5. [ ] **BI Engine Reservation:** Configure BI Engine for high-concurrency dashboards.
6. [ ] **Staging Expiration:** Set automatic expiration rules on staging and sandbox datasets.
7. [ ] **Streaming vs Batch:** Audit ingestion channels and convert streaming pipelines to batch loads where real-time is not required.
8. [ ] **Unused Table Archive:** Identify tables that have not been queried for > 180 days and export them to GCS Archive storage, then delete them from BigQuery.
