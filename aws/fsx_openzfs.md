# AWS Service Cost Research: Amazon FSx for OpenZFS

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon FSx for OpenZFS is a fully managed file system built on the open-source OpenZFS file system. It is designed to deliver ultra-low latencies (sub-100 microseconds) and massive throughput for Linux workloads (such as machine learning training, financial analytics, DevOps pipelines, and web serving). Because ZFS has high-performance features like copy-on-write cloning, snapshots, and LZ4/ZSTD compression, optimizing these settings can yield significant storage savings.

---

## 2. Billing Mechanics
FSx for OpenZFS bills on a monthly cycle based on the following dimensions:
1.  **SSD Storage Capacity:** Charged per GB-month provisioned.
2.  **Provisioned IOPS:** Billed for provisioned SSD IOPS-month beyond the baseline of 3 IOPS per GB.
3.  **Throughput Capacity:** Billed per MBps-month provisioned (independent of storage size).
4.  **Backup Storage:** Charged per GB-month for automatic and manual backups.

---

## 3. Key Cost Dimensions

### A. Storage Tiers & Redundancy (us-east-1)
*   **Single-AZ SSD Storage:** Billed at **$0.11 per GB-month**. Replicates data within a single Availability Zone.
*   **Multi-AZ SSD Storage:** Billed at **$0.203 per GB-month** (84% premium over Single-AZ). Replicates data across two Availability Zones for high availability.
*   *Note: Licensing for OpenZFS is open-source; there are no commercial software licensing fees.*

### B. Throughput Capacity (Provisioned Independently)
Throughput capacity determines the speed at which the file system can read and write data (from 64 MB/s up to 4,096 MB/s). Billed as a flat monthly rate:
*   **Single-AZ Throughput:** **$2.03 per MBps-month**.
*   **Multi-AZ Throughput:** **$4.06 per MBps-month**.
*   *Example:* Provisioning 256 MB/s throughput on a Single-AZ file system costs **$519.68/month** flat, regardless of actual bytes read or written.

### C. Provisioned SSD IOPS
Every GB of SSD storage includes 3 IOPS for free. You can provision extra IOPS (up to 350,000 IOPS per file system):
*   **Single-AZ Additional IOPS:** **$0.05 per IOPS-month**.
*   **Multi-AZ Additional IOPS:** **$0.10 per IOPS-month**.
*   *Example:* Provisioning an extra 10,000 IOPS on a Single-AZ file system adds **$50.00/month**.

### D. ZFS Compression & Cloning Cost Savings
*   **ZFS Compression:** Supports inline **LZ4** and **ZSTD** compression. LZ4 is optimized for low CPU overhead, while ZSTD provides higher compression ratios. Enabling compression typically reduces data storage requirements by **30–50%**, directly lowering the GB-month storage bill.
*   **ZFS Cloning:** Allows you to create instantaneous, block-level clones of datasets. Clones share data blocks with their parent dataset, costing **$0 for storage** until new data is modified or added to the clone.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Component | Single-AZ Rate | Multi-AZ Rate | Unit |
|-------------------|----------------|---------------|------|
| **SSD Storage** | $0.1100 | $0.2030 | Per GB-month |
| **Throughput Capacity** | $2.0300 | $4.0600 | Per MBps-month |
| **Provisioned SSD IOPS** | $0.0500 | $0.1000 | Per IOPS-month |
| **Backup Storage** | $0.0500 | $0.0500 | Per GB-month |

---

## 5. AWS Free Tier Coverage
*   **FSx for OpenZFS:** No free tier available. All provisioned resources generate billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Over-Provisioning Throughput:** Provisioning high throughput capacities (e.g., 512 MB/s or higher) to handle rare, monthly batch processes, resulting in high monthly idle fees ($1,039.36/month for 512 MB/s Single-AZ).
*   **Dormant Snapshots & Clones:** ZFS snapshots track incremental changes. If a dataset has high churn, old snapshots retain blocks that have since been modified, causing storage consumption to balloon silently.
*   **Unnecessary Multi-AZ in Dev/Test:** Deploying Multi-AZ file systems for development, staging, or testing environments where Single-AZ is sufficient.
*   **Deduplication Overhead:** ZFS deduplication requires significant memory (RAM) and can severely degrade performance if not properly sized, often forcing you to upgrade to higher throughput tiers just to get the required RAM, which increases costs.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enable ZSTD or LZ4 Data Compression:** Enable compression on all ZFS datasets. Use **LZ4** for performance-sensitive applications and **ZSTD** for capacity-sensitive archives or logs to instantly reduce storage fees by **30–50%**.
2.  **Right-Size Throughput Capacity Dynamically:** Check CloudWatch metrics for `ThroughputLimit` and `ThroughputUtilization`. Scale throughput down to match average daily peak utilization, and dynamically scale it up only for heavy workload phases.
3.  **Use ZFS Clones for Developer Environments:** Instead of duplicating data physically for testing or CI/CD pipelines, create ZFS clones. This saves 100% of storage costs for the shared base data.
4.  **Set up Automated Snapshot Deletion Policies:** Configure a lifecycle script or use AWS Backup to automatically delete old, non-current ZFS snapshots after a set retention period to prevent block churn storage growth.
5.  **Utilize Single-AZ for Dev/Staging:** Recreate all non-production OpenZFS file systems as Single-AZ. This drops storage and throughput rates by roughly **45–50%**.
