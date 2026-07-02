# AWS Service Cost Research: AWS Glue

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Glue is a serverless data integration service that makes it easy to discover, prepare, move, and integrate data from multiple sources for analytics, machine learning, and application development. Glue provides both visual and code-based interfaces for ETL (Extract, Transform, and Load) operations. Because Glue is serverless, you do not manage clusters. Instead, you pay for data catalog storage and the compute capacity (Data Processing Units) consumed during job executions.

---

## 2. Billing Mechanics
AWS Glue bills on a monthly cycle based on three primary dimensions:
1.  **ETL Job & Crawler Compute:** Billed based on **Data Processing Units (DPUs)** consumed per hour of runtime. Billed per-second with a 1-minute minimum.
2.  **Interactive Sessions:** Billed per DPU-hour for notebooks used during development.
3.  **Glue Data Catalog:** Billed for metadata storage (metadata objects) and catalog access requests.

---

## 3. Key Cost Dimensions

### A. ETL Job Compute (us-east-1 DPU Rates)
*   **Standard Spark Jobs:** **$0.44 per DPU-hour**.
    *   *Definition:* 1 DPU provides 4 vCPUs and 16 GB of memory.
    *   *Minimum:* Billed with a **1-minute minimum** per job execution.
*   **Glue Flex execution (Non-Urgent Batches):** **$0.29 per DPU-hour** (a **34% direct discount**).
    *   *Usage:* Best for non-time-critical nightly ETL workloads. Flex jobs run on spare AWS capacity and can be reclaimed (like Spot Instances), with Glue automatically managing job resubmissions.
*   **Python Shell Jobs:** Billed at the standard $0.44/DPU-hour rate, but you can provision fractional DPUs:
    *   *0.0625 DPU:* **$0.0275 per hour** (1 vCPU, 1 GB RAM).
    *   *1 DPU:* **$0.44 per hour**.

### B. Glue Crawlers
Used to automatically scan S3 buckets and database tables to populate the Data Catalog.
*   **The Rate:** Billed at **$0.44 per DPU-hour**.
*   **Minimum Charge:** Billed with a **10-minute minimum** per crawler run. Running a crawler that completes in 30 seconds still bills for 10 minutes of DPU time.

### C. Glue Data Catalog
*   **Metadata Storage:**
    *   First 1 Million objects: **Free**.
    *   Beyond 1 Million: **$1.00 per 100,000 objects per month**.
*   **Catalog Requests:**
    *   First 1 Million requests/month: **Free**.
    *   Beyond 1 Million: **$1.00 per 1 million requests**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Billing Basis | Rate (us-east-1) | Minimum Billed Run |
|----------------|---------------|------------------|---------------------|
| **Standard Job DPU** | Per DPU-hour | **$0.4400** | 1 minute |
| **Flex Job DPU** | Per DPU-hour | **$0.2900** | 1 minute |
| **Glue Crawler DPU** | Per DPU-hour | **$0.4400** | **10 minutes** |
| **Catalog Storage** | Per 100K objects | $1.0000 | Over 1 Million free |
| **Catalog Requests** | Per Million | $1.0000 | Over 1 Million free |

---

## 5. AWS Free Tier Coverage
*   **Glue Data Catalog:** 1 million stored objects and 1 million requests per month are free.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Over-Provisioning DPUs on Simple Jobs:** Running standard Spark ETL jobs with high default DPU settings (e.g. 10 or 20 DPUs) for small datasets. The job completes quickly but bills for massive DPU allocations.
*   **Frequent Short Crawler Runs:** Scheduling Glue Crawlers to run every 15 minutes on small folders. The 10-minute minimum billing block will multiply crawler costs by up to 20x.
*   **Development Interactive Sessions Left Open:** Keeping Glue notebook interactive sessions open and idle. Sessions default to a minimum of 5 DPUs (**$2.20/hour**) and run continuously until terminated.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enable Flex Execution for Non-Urgent Jobs:** Review your batch ETL pipelines. For any overnight data processing or reporting jobs that do not have strict completion deadlines, enable **Flex Execution** to cut DPU costs by **34%**.
2.  **Optimize DPU Sizing (Scale Down Dev/Small Jobs):**
    *   For testing and development jobs, configure the **Maximum DPUs** parameter to the minimum limit (e.g., 2 DPUs).
    *   For simple, single-file processing tasks, write them as **Python Shell Jobs** (using 0.0625 DPU at $0.0275/hr) instead of full Apache Spark jobs (using 10 DPUs at $4.40/hr).
3.  **Optimize Glue Crawler Runs (Avoid Crawler Storms):**
    *   Do not run crawlers on a schedule if S3 schemas do not change. Add partitions manually using the `ALTER TABLE ADD PARTITION` statement in Athena or the Glue catalog API, which is free.
    *   If you must use crawlers, run them in **incremental mode** or schedule them at long intervals (e.g., once daily) to bypass the 10-minute minimum charge.
4.  **Set Auto-Timeout on Interactive Sessions:** When using Glue notebooks or interactive sessions, configure the `idle-timeout` parameter to a low limit (e.g., **10 or 15 minutes**) to automatically close forgotten notebooks.
5.  **Enable Glue Job Autoscaling:** Enable **Glue Auto Scaling** on your Spark jobs. This dynamically scales DPU capacity up or down based on execution stage workloads, ensuring you only pay for compute capacity when needed.
