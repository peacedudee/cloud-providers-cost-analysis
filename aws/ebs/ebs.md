# AWS Service Cost Research: Amazon EBS (Elastic Block Store)

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon EBS provides high-performance block storage volumes for use with Amazon EC2 instances. Unlike S3 (which charges only for actual stored data), EBS volumes charge for **provisioned capacity** (meaning you pay for the full size of the volume regardless of whether your operating system actually writes data to it). EBS is a frequent cost leak because volumes often remain provisioned after their host instances are stopped or terminated.

---

## 2. Billing Mechanics
EBS billing is calculated on a monthly cycle based on three primary dimensions:
1.  **Provisioned Capacity:** Billed per GB-month depending on the selected volume type.
2.  **Provisioned Performance (IOPS & Throughput):** Billed per provisioned IOPS-month and MB/s-month (above baseline values).
3.  **Snapshot Storage:** Billed per GB-month for data backed up to S3.
4.  **Fast Snapshot Restore (FSR):** Billed per hour for instant volume hydration from snapshots.
5.  **Snapshot Archive:** Billed at a cheaper storage rate for cold snapshots, but incurs retrieval fees.

---

## 3. Key Cost Dimensions

### A. Volume Types & Performance (us-east-1)
*   **gp3 (General Purpose SSD - Recommended):**
    *   *Storage:* **$0.08 per GB-month**.
    *   *Performance:* Includes **3,000 IOPS** and **125 MB/s throughput** for free.
    *   *Overage Performance:* Provisioning more IOPS costs **$0.005/IOPS-month** (up to 16,000). Provisioning more throughput costs **$0.04/MB/s-month** (up to 1,000).
*   **gp2 (Older General Purpose SSD):**
    *   *Storage:* **$0.10 per GB-month** (25% more expensive than gp3).
    *   *Performance:* Tied directly to volume size (3 IOPS per GB, minimum 100). To get 3,000 IOPS on gp2, you are forced to provision a **1,000 GB volume**, costing $100/month. On gp3, a 10 GB volume with 3,000 IOPS costs just **$0.80/month**.
*   **io2 / io2 Block Express (Provisioned IOPS SSD):**
    *   *Storage:* **$0.125 per GB-month**.
    *   *Performance:* Billed for provisioned IOPS. **$0.065 per IOPS-month** (up to 32,000 IOPS).
    *   *Block Express:* Billed at tiered rates for IOPS > 32,000. Designed for enterprise databases (e.g., SAP Hana, Oracle).
*   **st1 (Throughput Optimized HDD):** Billed at **$0.045 per GB-month**. Designed for large, sequential workloads (like Hadoop or log processing).
*   **sc1 (Cold HDD):** Billed at **$0.015 per GB-month**. The cheapest block storage, designed for cold data with low access frequencies.

### B. EBS Snapshots
*   **Standard Storage Cost:** Snapshots are incremental and compressed, stored in S3, and billed at **$0.05 per GB-month**.
*   **Archive Storage Cost:** Cold snapshots can be moved to the Snapshot Archive tier, billed at **$0.0125 per GB-month** (a 75% savings). Moving data back to standard storage (retrieval) costs **$0.03 per GB**.
*   **Snapshot Copy:** Copying snapshots across regions incurs standard inter-region data transfer fees ($0.01 to $0.02 per GB) plus destination storage.

### C. Fast Snapshot Restore (FSR)
*   **The Charge:** Billed at **$0.75 per hour** (Data Services Unit-Hour) for *every* snapshot in *every* Availability Zone where FSR is enabled.
*   **Monthly Impact:** Enabling FSR on a single snapshot across 3 AZs for a full month costs:
    $$\text{Monthly Cost} = 1\text{ snapshot} \times 3\text{ AZs} \times 730\text{ hours} \times \$0.75 = \$1,642.50\text{ / month}$$
*   This is a severe, high-risk billing hotspot frequently left active in developer or migration accounts.

---

## 4. Detailed Pricing Rates (us-east-1)

| Volume Type | Storage (/GB-mo) | Baseline IOPS | Baseline Throughput | Extra IOPS (/IOPS-mo) | Extra Throughput (/MBps-mo) |
|-------------|------------------|---------------|----------------------|-----------------------|-----------------------------|
| **gp3** | $0.0800 | 3,000 (Free) | 125 MB/s (Free) | $0.0050 | $0.0400 |
| **gp2** | $0.1000 | 3 per GB | Scale-with-size | N/A (Locked) | N/A (Locked) |
| **io2** | $0.1250 | Provisioned | Provisioned | $0.0650 | N/A |
| **st1** | $0.0450 | N/A | N/A | N/A | N/A |
| **sc1** | $0.0150 | N/A | N/A | N/A | N/A |

---

## 5. AWS Free Tier Coverage
*   **Storage:** 30 GB of total EBS storage (any combination of gp2, gp3, or magnetic), plus 2 million I/Os and 1 GB of snapshot storage.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Orphaned EBS Volumes (Stopped EC2 Instances):** Stopping an EC2 instance pauses compute charges, but **EBS storage charges continue**. Leaving a 500 GB gp3 volume attached to a stopped instance bills at:
    $$500\text{ GB} \times \$0.08 = \$40.00\text{ / month}$$
*   **Unattached Volumes:** Deleting an EC2 instance without deleting its attached volumes. These volumes sit unattached, completely invisible to the host, while generating standard storage charges.
*   **Fast Snapshot Restore Left Enabled:** Setting up FSR to quickly hydrate a volume during a migration, and forgetting to disable FSR after the migration completes, generating $0.75/hour per AZ.
*   **Legacy gp2 Volumes:** Thousands of legacy gp2 volumes created by default templates. Migrating to gp3 immediately yields a **20% direct cost reduction** and higher baseline performance.

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate gp2 to gp3 Immediately:**
    *   Convert all existing `gp2` volumes to `gp3`. This is a non-disruptive, online operation (no downtime, can be executed while the instance is running).
    *   **The Savings:** Provides a **20% direct price reduction** per GB and decouples performance from storage size.
2.  **Delete Unattached EBS Volumes:**
    *   Set up an automation script or AWS Config rule to find and delete unattached EBS volumes. Always take a final snapshot before deletion (which costs $0.05/GB-month vs. $0.08/GB-month for gp3 storage).
3.  **Disable Fast Snapshot Restore (FSR):**
    *   Audit your snapshots. Locate any snapshots with FSR enabled and disable them if they are not actively being used for high-velocity auto-scaling groups or critical DR hydration.
4.  **Implement AWS Backup Lifecycle Rules:**
    *   Set up automated lifecycle policies to transition EBS snapshots to **Snapshot Archive ($0.0125/GB-month)** or delete them after a retention period.
5.  **Right-Size Provisioned IOPS on gp3:**
    *   Check CloudWatch metrics for `VolumeReadOps` and `VolumeWriteOps`. If you are paying for provisioned IOPS on gp3 (above 3,000) but actual usage never exceeds 3,000 IOPS, reduce the provisioned IOPS back to the free 3,000 baseline.
