# AWS Service Cost Research: Amazon FSx for NetApp ONTAP

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon FSx for NetApp ONTAP is a fully managed service that provides highly reliable, scalable, and high-performance file storage built on NetApp’s popular ONTAP file system. It combines ONTAP’s enterprise data management features (like snapshot copies, cloning, and multi-protocol access) with the scalability of AWS. Because ONTAP uses a dual-tier storage architecture (high-speed SSD tier + elastic capacity pool tier), optimizing your data tiering policies is the single most important factor in managing its costs.

---

## 2. Billing Mechanics
FSx for NetApp ONTAP bills on a monthly cycle based on the following dimensions:
1.  **SSD Storage Capacity:** Charged per GB-month provisioned.
2.  **SSD IOPS:** Billed for provisioned IOPS-month above the baseline of 3 IOPS per GB.
3.  **Capacity Pool Storage (Cold Tier):** Billed per GB-month of active data consumed (elastic billing).
4.  **Capacity Pool Request Charges:** Billed per HTTP read/write request when retrieving or sending data to/from the capacity pool.
5.  **Throughput Capacity:** Billed per MBps-month provisioned (independent of storage size).
6.  **Backup Storage:** Billed per GB-month for automatic and manual backups.

---

## 3. Key Cost Dimensions

### A. Dual-Tier Storage Pricing (us-east-1)
FSx for ONTAP splits storage into two distinct tiers:
*   **SSD Storage (Hot Tier):** High-speed storage for active data and metadata.
    *   *Single-AZ:* **$0.148 per GB-month**.
    *   *Multi-AZ:* **$0.263 per GB-month**.
*   **Capacity Pool Storage (Cold Tier):** Serverless, elastic storage backed by S3. Scales automatically based on the volume of cold data moved.
    *   *Single-AZ:* **$0.021 per GB-month** (85% storage cost reduction).
    *   *Multi-AZ:* **$0.0438 per GB-month** (83% storage cost reduction).
    *   *Access Fees:* **$0.01 per GB read** and **$0.02 per GB written** when data is accessed from/written to the capacity pool.

### B. Throughput Capacity (Provisioned Independently)
Throughput capacity determines the speed of the ONTAP controllers (from 125 MB/s to 4,000 MB/s). Billed as a flat monthly rate:
*   **Single-AZ Throughput:** **$2.20 per MBps-month**.
*   **Multi-AZ Throughput:** **$4.40 per MBps-month**.

### C. SSD IOPS
Every GB of SSD storage includes 3 IOPS for free. You can provision extra IOPS:
*   **Single-AZ IOPS:** **$0.08 per IOPS-month**.
*   **Multi-AZ IOPS:** **$0.16 per IOPS-month**.

### D. Data Efficiency Features (Compression & Deduplication)
NetApp ONTAP features native, inline data compression, deduplication, and compaction.
*   **Cost Impact:** These features are active by default and typically achieve **30% to 60% physical storage savings** for databases, container volumes, and general file shares. Since you only pay for the *logical space consumed* after deduplication, this directly reduces both your SSD and Capacity Pool storage bills.

### E. NetApp FlexClones
*   Allows you to create instantaneous logical copies of volumes that share data blocks with the parent volume.
*   **Cost Advantage:** FlexClones consume **$0.00 for storage** for all unmodified blocks. Only changes made to the cloned volume generate new block storage charges.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Component | Single-AZ Rate | Multi-AZ Rate | Unit |
|-------------------|----------------|---------------|------|
| **SSD Storage** | $0.1480 | $0.2630 | Per GB-month |
| **Capacity Pool Storage** | $0.0210 | $0.0438 | Per GB-month |
| **Throughput Capacity** | $2.2000 | $4.4000 | Per MBps-month |
| **Provisioned SSD IOPS** | $0.0800 | $0.1600 | Per IOPS-month |
| **Capacity Pool Reads** | $0.0100 | $0.0100 | Per GB accessed |
| **Capacity Pool Writes** | $0.0200 | $0.0200 | Per GB written |
| **Backup Storage** | $0.0500 | $0.0500 | Per GB-month |

### Example Monthly Cost Calculation (Deduplication & Tiering Impact)
*Workload: A company stores 10 TB of logical user files in a Single-AZ ONTAP environment. Data deduplication yields a 40% logical space saving, bringing active storage down to 6 TB. Of this 6 TB, 1 TB is active SSD data, and 5 TB is cold capacity pool data.*

*   **Logic Capacity Cost (Without Deduplication or Tiering):**
    $$\text{Logical Cost} = 10,000\text{ GB} \times \$0.148/\text{GB-mo} = \$1,480.00\text{ / month}$$
*   **Optimized SSD Tier Cost (1 TB logical active after 40% dedup = 600 GB physical):**
    $$\text{SSD Cost} = 600\text{ GB} \times \$0.148/\text{GB-mo} = \$88.80$$
*   **Optimized Capacity Pool Cost (5 TB logical cold after 40% dedup = 3,000 GB physical):**
    $$\text{Pool Cost} = 3,000\text{ GB} \times \$0.021/\text{GB-mo} = \$63.00$$
*   **Total Optimized Cost:** **$151.80/month** (A **90% savings** compared to flat SSD logical storage).

---

## 5. AWS Free Tier Coverage
*   **FSx for NetApp ONTAP:** No free tier available. All provisioned resources generate billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Poor Tiering Policy Configuration (All SSD):** Setting the volume tiering policy to `None` or keeping large volumes of inactive data in the SSD tier, paying $0.263/GB instead of $0.0438/GB.
*   **Over-Provisioning Throughput:** Provisioning high throughput capacities to handle rare, monthly batch processes, resulting in high monthly idle fees ($4,400/month for 1000 MB/s Multi-AZ).
*   **High Capacity Pool Read Request Fees:** Setting the tiering policy to `Auto` on databases or search indexes that frequently scan the entire dataset. Continuous reads of cold data will generate massive per-GB access fees ($0.01/GB read) that outweigh capacity pool savings.
*   **Unnecessary Multi-AZ in Dev/Test:** Deploying Multi-AZ file systems for development, staging, or testing environments where Single-AZ is sufficient.

---

## 7. Actionable Cost Optimization Strategies
1.  **Configure the `Auto` or `All` Volume Tiering Policy:**
    *   For standard file shares and backup targets, set the volume tiering policy to `Auto` (moves inactive blocks after 31 days) or `All` (moves all user data to capacity pools).
    *   **The Savings:** Cuts storage costs by **70–80%** by leaving only metadata on the SSD tier.
2.  **Enable Deduplication & Compression:**
    *   Verify that ONTAP inline deduplication (`sis on`) and compression are enabled on all volumes. Run background deduplication during off-peak hours.
    *   **The Savings:** Typically saves **30–60%** on general file shares.
3.  **Right-Size Throughput Capacity Dynamically:**
    *   Check CloudWatch metrics for `ThroughputLimit` and `ThroughputUtilization`. Scale throughput down to match average daily peak utilization, and dynamically scale it up only for heavy workload phases (e.g., database restores).
4.  **Use NetApp FlexClones for Dev/Test Databases:**
    *   Instead of creating physical duplicates of volumes for testing, use **NetApp FlexClones**.
    *   **The Savings:** FlexClones create instant logical copies that share data blocks with the parent volume, charging **$0 for storage** for all unmodified blocks.
5.  **Utilize Single-AZ for Dev/Staging:**
    *   Recreate all non-production ONTAP file systems as Single-AZ.
    *   **The Savings:** Drops storage and throughput rates by roughly **40–50%**.
