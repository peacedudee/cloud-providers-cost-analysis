# AWS Service Cost Research: Amazon FSx for Windows File Server

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon FSx for Windows File Server provides fully managed, highly reliable, and scalable shared file storage that is accessible over the industry-standard Server Message Block (SMB) protocol. It is built on Windows Server, making it the default choice for migrating Windows-based applications, SQL Server, and enterprise home directories. Unlike Lustre, throughput and IOPS are provisioned independently of storage capacity, requiring careful tuning to prevent over-provisioning.

---

## 2. Billing Mechanics
FSx for Windows File Server billing is determined by several independent configuration choices:
1.  **Deployment Type:** Single-AZ (local redundancy) vs. Multi-AZ (active/passive replication across zones with automatic failover).
2.  **Storage Type:** SSD (high IOPS, low latency) vs. HDD (capacity-optimized, for cold shares/backups).
3.  **Storage Capacity:** The amount of disk space provisioned (GB-month).
4.  **Throughput Capacity:** Provisioned independently of storage size (measured in MB/s-month).
5.  **Provisioned IOPS:** Custom SSD IOPS provisioned above the baseline (which is 3 IOPS per GB).
6.  **Backup Storage:** Billed per GB-month for data backed up to S3.

---

## 3. Key Cost Dimensions

### A. Deployment Types & Storage Tiers (us-east-1)
*   **Single-AZ SSD:** **$0.130 per GB-month**. Replicates within a single zone.
*   **Single-AZ HDD:** **$0.013 per GB-month** (90% storage savings compared to SSD). Designed for enterprise home directories and departmental shares.
*   **Multi-AZ SSD (High Availability):** **$0.230 per GB-month** (76% premium over Single-AZ). Replicates synchronously across two AZs.
*   **Multi-AZ HDD (High Availability):** **$0.025 per GB-month**.
*   *Note: Licensing fees for Microsoft Windows Server are built directly into these capacity charges; there are no separate licensing fees.*

### B. Throughput Capacity (Provisioned Independently)
Throughput capacity determines the speed at which the file system can deliver data (ranging from 8 MB/s to 2,048 MB/s). You pay a flat monthly fee for this throughput:
*   **Single-AZ Throughput:** **$2.25 per MB/s-month**.
*   **Multi-AZ Throughput:** **$4.50 per MB/s-month** (double the cost).

### C. Provisioned SSD IOPS
By default, SSD file systems include 3 IOPS per GB of storage for free (e.g., a 1,000 GB volume includes 3,000 IOPS). You can provision additional IOPS:
*   **Single-AZ Additional IOPS:** **$0.05 per IOPS-month**.
*   **Multi-AZ Additional IOPS:** **$0.10 per IOPS-month**.

### D. Data Deduplication (Storage Savings)
FSx for Windows supports native Microsoft Data Deduplication.
*   **The Cost Saver:** When enabled, it automatically identifies and removes duplicate data blocks, storing only one copy. For typical enterprise file shares and user home directories, Data Deduplication reduces storage footprints by **50–60%**, directly cutting your GB-month capacity bill in half.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Single-AZ Rate (SSD) | Multi-AZ Rate (SSD) | Single-AZ Rate (HDD) | Multi-AZ Rate (HDD) |
|----------------|----------------------|---------------------|----------------------|---------------------|
| **Storage Capacity** | $0.130 / GB-month | $0.230 / GB-month | $0.013 / GB-month | $0.025 / GB-month |
| **Throughput** | $2.25 / MBps-month | $4.50 / MBps-month | $2.25 / MBps-month | $4.50 / MBps-month |
| **Provisioned IOPS**| $0.050 / IOPS-month | $0.100 / IOPS-month | N/A | N/A |
| **Backup Storage** | $0.050 / GB-month | $0.050 / GB-month | $0.050 / GB-month | $0.050 / GB-month |

### Example Monthly Cost Calculation (Deduplication Savings)
*Workload: A company provisions a 4 TB (4,000 GB) Multi-AZ SSD file system with 128 MB/s throughput. Enabling Microsoft Data Deduplication yields a 50% storage footprint reduction.*

*   **Standard Billing (Without Deduplication):**
    *   Storage Cost:
        $$4,000\text{ GB} \times \$0.23/\text{GB} = \$920.00$$
    *   Throughput Cost:
        $$128\text{ MB/s} \times \$4.50 = \$576.00$$
    *   Total Cost: **$1,496.00/month**
*   **Deduplicated Billing (50% physical storage savings = 2,000 GB billed):**
    *   Storage Cost:
        $$2,000\text{ GB} \times \$0.23/\text{GB} = \$460.00$$
    *   Throughput Cost:
        $$128\text{ MB/s} \times \$4.50 = \$576.00$$
    *   Total Cost: **$1,036.00/month** (Saves **$460.00/month** directly).

---

## 5. AWS Free Tier Coverage
*   **FSx for Windows:** No free tier available. Any provisioned resource incurs active billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Idle Throughput Oversizing:** Provisioning high throughput capacities to handle rare, monthly batch processes, resulting in high monthly idle fees ($1,152/month for 256 MB/s Multi-AZ).
*   **Unnecessary Multi-AZ in Dev/QA:** Deploying Multi-AZ file systems for development, staging, or testing environments. switching to Single-AZ instantly cuts costs by **40–50%**.
*   **Storing Cold Shares on SSD:** Deploying SSD storage for standard user home directories or archive shares. HDD storage is **10x cheaper** than SSD ($0.013/GB vs. $0.130/GB).
*   **Data Deduplication Left Disabled:** Deduplication is disabled by default. Running home directories or software repository shares without enabling deduplication results in paying double for duplicate files.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enable Windows Data Deduplication:**
    *   Set up a scheduled data deduplication job via the FSx console or PowerShell.
    *   **The Savings:** Zero-risk operation that typically yields **50–60% storage savings** on user directories.
2.  **Right-Size Throughput Dynamically:**
    *   Check CloudWatch metrics for `ThroughputLimit` and `ThroughputUtilization`.
    *   Scale down the provisioned throughput capacity during normal operating hours to match your peak utilization. You can dynamically increase throughput (online, with zero downtime) before executing large migrations, and scale it back down immediately after.
3.  **Migrate Dev/Test to Single-AZ:**
    *   Audit all running FSx for Windows file systems. Re-create any non-production file systems as **Single-AZ**.
    *   **The Savings:** Achieves immediate **~50% savings** on storage, throughput, and IOPS.
4.  **Use HDD Storage Tiers for User File Shares:**
    *   For general-purpose user directories, backups, and office document shares, use **HDD storage**.
    *   **The Savings:** Keeps the bulk capacity at the cheap **$0.013/GB-month** tier while the SSD cache automatically handles hot files.
5.  **Clean Up Automatic Backup Retention:** Set backup retention windows in AWS Backup to the minimum required for compliance (e.g., 7 or 14 days for non-prod) to avoid accumulating large backup storage bills.
