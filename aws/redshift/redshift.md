# AWS Service Cost Research: Amazon Redshift

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Redshift is a fully managed, petabyte-scale data warehouse service. It is designed for large-scale business intelligence and analytical querying. Redshift can be deployed as a Provisioned Cluster (where compute and storage are managed via nodes) or in a Serverless model (where AWS scales resources automatically based on active query load). Because analytical queries process massive volumes of data, data scanning and compute sizing are the primary cost drivers.

---

## 2. Billing Mechanics
Redshift billing is structured based on the selected deployment model:
1.  **Provisioned Clusters:**
    *   *Compute Node Hours:* Hourly rate per active node (e.g. `ra3.xlplus`).
    *   *Redshift Managed Storage (RMS):* Billed per GB-month for data stored on the cluster (which scales automatically to S3).
    *   *Concurrency Scaling:* Hourly charges for temporary cluster capacity added to handle query queues.
2.  **Redshift Serverless:**
    *   *Compute (RPU-Hours):* Billed per-second for the **Redshift Processing Units (RPUs)** consumed during active queries.
    *   *Managed Storage:* Billed per GB-month for data stored in the warehouse.
3.  **Redshift Spectrum (S3 Querying):** Billed per TB of data scanned from external S3 buckets.

---

## 3. Key Cost Dimensions

### A. Provisioned Clusters (us-east-1 RA3 Nodes)
Modern Redshift clusters use **RA3 instances**, which decouple compute and storage:
*   **Compute Node Pricing:** Billed hourly per node. For example, a baseline `ra3.xlplus` node costs **$1.086 per hour** in `us-east-1`.
*   **Redshift Managed Storage (RMS):** Data is cached on high-speed local SSDs and pushed to S3. Billed at a flat rate of **$0.024 per GB-month** (roughly standard S3 rates).
*   **Reserved Nodes:** 1- or 3-year commitments offer up to **75% savings** on hourly compute charges.
*   **Concurrency Scaling:** Automatically provisions temporary clusters to handle concurrent queries. Billed at **$5.50 per cluster-hour**.
    *   *Free Credits:* AWS provides **45 seconds of free concurrency scaling** for every hour your primary cluster is active. You can accumulate up to 30 minutes of free scaling credits per day, which handles most temporary query spikes for free.

### B. Redshift Serverless
*   **The Model:** Best for intermittent workloads or applications with unpredictable query loads. Compute scales from 0 when idle.
*   **Compute Rate:** Billed per **RPU-hour**. 1 RPU represents 16 GB of memory and compute capacity.
    *   *Rate:* **$0.375 per RPU-hour** (in `us-east-1`).
    *   *Minimum Charge:* Billed with a **60-second minimum** per query.
*   **Serverless Storage:** Billed at the same rate of **$0.024 per GB-month** for data stored.

### C. Redshift Spectrum (S3 Data Lake Querying)
*   **The Charge:** Billed at a flat rate of **$5.00 per TB** of data scanned from external S3 buckets during queries.
*   **Overage Hazard:** Querying uncompressed, unpartitioned datasets in S3 can result in scanning terabytes of unnecessary data, driving up the bill rapidly.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Component | Provisioned Cluster Rate | Serverless Rate | Unit / Details |
|-------------------|--------------------------|-----------------|----------------|
| **ra3.xlplus Compute**| $1.086 / hour | N/A | 4 vCPU, 32 GiB RAM |
| **Serverless Compute**| N/A | **$0.375 / RPU-hour**| Minimum 8 RPUs scaling |
| **Managed Storage** | $0.0240 | $0.0240 | Per GB-month |
| **Spectrum S3 Query** | $5.0000 | $5.0000 | Per TB of data scanned |
| **Concurrency Scale**| $5.5000 | N/A | Per cluster-hour |

---

## 5. AWS Free Tier Coverage
*   **Provisioned:** 750 free hours/month of a `dc2.large` node (160 GB storage, no decoupling) for 2 months (new accounts).
*   **Serverless:** $300 in free Redshift Serverless usage credits (valid for 90 days).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Over-Provisioning Serverless Max RPU Capacity:** Leaving the serverless maximum capacity at the default (e.g. 128 RPUs). A complex query scan can automatically scale compute, billing at **$48.00/hour**.
*   **Redshift Spectrum Scanning Raw CSVs:** Running Spectrum queries on large, raw CSV/JSON files in S3. Since S3 text files are uncompressed and unpartitioned, a query must scan the entire file, costing $5.00/TB scanned.
*   **Provisioned Clusters Left Active and Idle:** Keeping a multi-node RA3 cluster running 24/7 during weekends and holidays when no analytical reports are being generated. An idle 4-node `ra3.xlplus` cluster costs **$3,171.12/month** in compute fees.
*   **Orphaned Snapshot Storage:** Retaining automatic and manual snapshots of deleted clusters, billed at S3 rates.

---

## 7. Actionable Cost Optimization Strategies
1.  **Configure Serverless RPU Limits:** Set the **Max RPU Capacity** limit in your Serverless configuration to a low, functional ceiling (e.g., 8 or 16 RPUs). Set a **daily or monthly usage limit** (in RPU-hours) to automatically alert or shut down query capabilities if the budget is exceeded.
2.  **Compress and Partition S3 Data for Spectrum:** Convert S3 data lake files to columnar, compressed formats (like **Apache Parquet** or **ORC**) and organize them into folders using **S3 Partitioning** (e.g., by year/month/day). This reduces data scanned by up to **90%**, dropping your Spectrum bill from $5.00/TB to $0.50/TB.
3.  **Implement Pause/Resume on Provisioned Clusters:** For clusters that do not need to run 24/7, configure **Scheduled Pause and Resume** policies. Automatically pause the cluster at 8 PM and resume it at 8 AM. Compute charges scale to $0 during the paused state (you only pay for storage).
4.  **Purchase Reserved Nodes for Production Clusters:** For permanent, 24/7 analytics environments, purchase a 1- or 3-year RI. This cuts compute node costs by up to **75%**.
5.  **Use Concurrency Scaling Limits:** Configure a strict limit on the maximum number of concurrency scaling clusters allowed (e.g., limit it to 1 or 2) to prevent run-away autoscaling query costs.
6.  **Clean up Manual Snapshots:** Set up snapshot lifecycle rules to automatically delete manual Redshift snapshots after 30 days.
