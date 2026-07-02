# AWS Service Cost Research: Amazon FSx for Lustre

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon FSx for Lustre is a fully managed, high-performance file system optimized for compute-intensive workloads such as machine learning training, high-performance computing (HPC), video rendering, and financial modeling. It is designed to process massive datasets at sub-millisecond latencies, but its performance tiers carry high baseline costs, making deployment architecture (Scratch vs. Persistent) the primary cost driver.

---

## 2. Billing Mechanics
FSx for Lustre billing is based on provisioned capacity and configuration settings:
1.  **Deployment Type:** Scratch (temporary, single-AZ, non-replicated) vs. Persistent (durable, replicated, with high availability options).
2.  **Storage Type:** SSD (high IOPS, low latency) vs. HDD (capacity-optimized, with SSD caching).
3.  **Throughput Tier:** Billed based on the throughput limit provisioned per TB of storage (e.g., 12, 40, 125, 250, 500, or 1000 MB/s per TB).
4.  **Backup Storage:** Charged per GB-month for data backed up to AWS Backup.
5.  **Metadata Storage:** Some Persistent configurations charge separately for metadata storage.

---

## 3. Key Cost Dimensions

### A. Deployment Types & Storage (us-east-1)
*   **Scratch File Systems (Non-Replicated):** Designed for temporary storage and short-term processing. Data is not replicated, and there is no automatic failover.
    *   *Scratch SSD Storage:* Billed at approximately **$0.043 per GB-month**.
    *   *Cost Advantage:* Extremely cheap performance storage. Ideal for running short ML training cycles where intermediate data is saved back to S3.
*   **Persistent File Systems (Replicated):** Designed for longer-term workloads. Replicates data within a single Availability Zone (Single-AZ) or across zones (Multi-AZ) and supports automatic failover.
    *   *Persistent SSD Storage (Single-AZ):* Starts at **$0.133 per GB-month** (with 12 MB/s/TB throughput).
    *   *Persistent SSD Storage (Multi-AZ):* Starts at **$0.230 per GB-month** (roughly double the cost of Single-AZ).
    *   *Persistent HDD Storage:* Starts at **$0.013 per GB-month** (Single-AZ) or **$0.025 per GB-month** (Multi-AZ). Includes a 20% SSD cache to accelerate hot files.

### B. Throughput Tiers (Capacity Pricing)
Throughput on FSx for Lustre is tied to the amount of storage provisioned. Choosing a higher throughput tier increases the per-GB storage charge.
*   *Example (Single-AZ Persistent SSD Storage pricing in us-east-1):*
    *   **12 MB/s/TB tier:** $0.133 per GB-month
    *   **40 MB/s/TB tier:** $0.170 per GB-month
    *   **125 MB/s/TB tier:** $0.233 per GB-month
    *   **250 MB/s/TB tier:** $0.320 per GB-month
    *   **500 MB/s/TB tier:** $0.490 per GB-month
    *   **1000 MB/s/TB tier:** $0.830 per GB-month
    *   *Warning:* A 10 TB file system on the 1000 MB/s/TB tier costs **$8,300.00/month** compared to **$1,330.00/month** on the 12 MB/s/TB tier!

### C. Data Repository Integration (S3 Sync)
Lustre file systems can link directly to an S3 bucket to import and export data.
*   **API Fees:** When FSx imports metadata from S3, you pay standard S3 `LIST` and `GET` request fees ($0.005/1K Class A, $0.0004/1K Class B).
*   **Data Movement:** Moving data between S3 and FSx in the same region is free. Cross-region data transfer fees apply if the ECR/S3 bucket is in another region.

### D. Backup Storage
*   Automatic and manual backups are stored in S3 and billed at a flat rate of **$0.05 per GB-month** in `us-east-1`.

---

## 4. Detailed Pricing Rates (us-east-1 SSD Tiers)
*Below are the monthly storage pricing rates per GB-month for SSD-based Lustre file systems:*

| Throughput Tier | Scratch SSD (/GB-mo) | Persistent Single-AZ (/GB-mo) | Persistent Multi-AZ (/GB-mo) |
|-----------------|----------------------|--------------------------------|------------------------------|
| **Baseline / 12 MBps**| $0.0430 | $0.1330 | $0.2300 |
| **40 MB/s/TB** | N/A | $0.1700 | $0.2940 |
| **125 MB/s/TB** | N/A | $0.2330 | $0.4030 |
| **250 MB/s/TB** | N/A | $0.3200 | $0.5540 |
| **500 MB/s/TB** | N/A | $0.4900 | $0.8480 |
| **1000 MB/s/TB**| N/A | $0.8300 | $1.4360 |

---

## 5. AWS Free Tier Coverage
*   **FSx for Lustre:** No free tier available. Any provisioned file system generates billing from the moment it is initialized.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Over-Provisioning Storage for Throughput:** Developers often provision larger storage sizes (e.g., 20 TB instead of 2 TB) just to hit a specific throughput target. This is extremely wasteful, as you pay for the unused storage space.
*   **Leaving Persistent Multi-AZ Running for Temp Jobs:** Launching Multi-AZ Persistent file systems for temporary machine learning runs or rendering jobs. This doubles the compute and replication baseline cost unnecessarily.
*   **Forgetting to Delete Scratch Volumes:** Because scratch volumes are designated as "temporary," developers assume they autodelete. However, Scratch file systems remain active and charge **$0.043/GB-month** indefinitely until manually deleted.
*   **High S3 API Charges During Large Metadata Imports:** Linking FSx to an S3 bucket containing hundreds of millions of small files. The initial metadata import will execute millions of S3 API calls, driving up S3 request fees.

---

## 7. Actionable Cost Optimization Strategies
1.  **Use Scratch Tiers for Ephemeral Compute:** For all ML training jobs (e.g., SageMaker or EC2 GPU clusters) and batch rendering, deploy **Scratch SSD** file systems. Hook them up to S3 to auto-load training data at startup, and write results back to S3. Delete the Scratch FSx immediately after the job finishes.
2.  **Enable Data Compression:** Enable **LZ4 compression** on the Lustre file system. This compresses files automatically, reducing the total GB-month storage footprint by 30-50% for typical text or log datasets, yielding direct storage savings.
3.  **Right-Size Throughput Tiers:** Start with the lowest functional throughput tier (e.g., 40 MB/s/TB or 125 MB/s/TB). Monitor the `ThroughputLimit` and `DiskReadBytes/DiskWriteBytes` metrics in CloudWatch. Do not opt for 1000 MB/s/TB unless your GPU compute is actively bottlenecked by storage I/O.
4.  **Use HDD Tiers with SSD Cache for Large Archives:** For large datasets that are read sequentially, use the **Persistent HDD** storage class. The built-in 20% SSD cache automatically caches hot metadata and frequently accessed files for SSD speed, while the base storage is billed at the cheap **$0.013/GB-month** rate.
5.  **Utilize S3 Prefix Filters during Sync:** When linking EFS/FSx to S3, restrict the data repository association to specific prefixes (folders) containing only the active training dataset, rather than the entire bucket. This reduces metadata synchronization time and S3 API request fees.
