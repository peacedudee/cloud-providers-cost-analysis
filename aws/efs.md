# AWS Service Cost Research: Amazon EFS (Elastic File System)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon EFS is a serverless, fully managed elastic file system designed for shared file storage across multiple EC2 instances, ECS tasks, and Lambda functions. Unlike EBS (where you pay for provisioned space), EFS standard storage charges you only for the **actual data stored**. However, EFS has high base storage rates ($0.30/GB-month for Standard) and bills for data throughput, making throughput configuration the primary cost differentiator.

---

## 2. Billing Mechanics
EFS bills monthly on several distinct dimensions:
1.  **Storage Volume:** GB-month consumed, divided by storage tier (Standard, Infrequent Access, Archive) and replication redundancy (Regional vs. One Zone).
2.  **Throughput Mode:** Billed based on how throughput is provisioned and consumed (Elastic, Provisioned, or Bursting).
3.  **Data Access/Retrieval Fees:** Charged when reading/writing data from colder storage tiers (IA and Archive).
4.  **Cross-AZ Data Transfer:** Charged if clients access EFS from a different Availability Zone than the EFS mount target.

---

## 3. Key Cost Dimensions

### A. Storage Tiers & Redundancy (us-east-1)
*   **Regional EFS (Default):** Replicates data across multiple Availability Zones.
    *   *EFS Standard:* **$0.30 per GB-month**. Designed for active files.
    *   *EFS Infrequent Access (IA):* **$0.016 per GB-month** (95% storage savings). Reading data from IA costs a **$0.01 per GB** data access fee.
    *   *EFS Archive:* **$0.008 per GB-month** (97% storage savings). Designed for files accessed a few times a year. Reading data from Archive costs a **$0.03 per GB** access fee.
*   **One Zone EFS:** Replicates data within a single Availability Zone (approx. 47% cheaper).
    *   *One Zone Standard:* **$0.16 per GB-month**.
    *   *One Zone IA:* **$0.0133 per GB-month** ($0.01/GB retrieval fee).
    *   *One Zone Archive:* **$0.004 per GB-month** ($0.03/GB retrieval fee).
    *   *Risk:* No multi-AZ availability. Data is lost if the AZ suffers a physical disaster.

### B. Throughput Modes
*   **Elastic Throughput (Recommended for Serverless/Spiky):**
    *   *Pricing:* Billed purely based on active data read/written.
    *   *Rates:* **$0.03 per GB read** and **$0.06 per GB written** in `us-east-1`.
    *   No baseline provisioning charges; cost scales down to zero if there are no read/write requests.
*   **Provisioned Throughput (Recommended for high baseline read/write):**
    *   *Pricing:* You specify the throughput you need (in MB/s). 
    *   *Rates:* Billed at **$6.00 per MB/s-month** for any throughput provisioned above what your storage volume automatically includes (which is 1 MB/s per 20 GB of Standard storage).
    *   *Hotspot:* If you provision 100 MB/s for a month, you pay **$600.00** flat, even if the file system is completely idle.
*   **Bursting Throughput:**
    *   *Pricing:* Included in the base storage price. Throughput scales linearly with your storage size (50 KiB/s per GB stored). You accumulate burst credits.
    *   *Hotspot:* If your storage volume is small (e.g., 10 GB), your baseline throughput is tiny (500 KiB/s). If you run out of burst credits during a job, EFS throttles throughput to the baseline, which can stall your applications.

### C. Cross-AZ Data Transfer Egress
*   EFS creates mount targets in each AZ. If an EC2 instance in AZ-A mounts the EFS target in AZ-B (due to dns misconfiguration or subnet routing), all read/write traffic is billed at standard inter-AZ rates (**$0.01 per GB in each direction**).

---

## 4. Detailed Pricing Rates (us-east-1)

| Storage Class | Storage Rate (/GB-mo) | Read Retrieval (/GB) | Write Retrieval (/GB) | Throughput Mode Base |
|---------------|----------------------|----------------------|-----------------------|----------------------|
| **Standard (Regional)** | $0.3000 | Free | Free | Bursting (Included) |
| **Standard (One Zone)** | $0.1600 | Free | Free | Bursting (Included) |
| **Infrequent Access (IA)** | $0.0160 | $0.0100 | $0.0100 | Elastic Read: $0.03/GB |
| **Archive (Regional)** | $0.0080 | $0.0300 | $0.0300 | Elastic Write: $0.06/GB |
| **One Zone IA** | $0.0133 | $0.0100 | $0.0100 | Provisioned: $6.00/MB/s |

---

## 5. AWS Free Tier Coverage
*   **Storage:** 5 GB of Regional EFS Standard storage per month for the first 12 months.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Keeping Large Datasets in EFS Standard:** Storing raw data, archives, or logs on EFS Standard without lifecycle rules. EFS Standard is 13x more expensive than EBS gp3 SSD and 18x more expensive than EFS IA!
*   **Provisioned Throughput Waste:** Configuring Provisioned Throughput (e.g., 200 MB/s) to handle a migration and forgetting to switch back to Elastic or Bursting mode, resulting in a **$1,200.00/month** idle fee.
*   **Elastic Throughput on Constant Read/Write Workloads:** Using Elastic mode for database logs or continuous data streaming workloads that write terabytes of data daily. Paying $0.06/GB written for high-frequency disk I/O will quickly exceed the cost of provisioned throughput.
*   **Cross-AZ Mounts:** Mounting EFS across AZ boundaries instead of using local mount targets, accumulating inter-AZ egress fees on every file read/write.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enable EFS Lifecycle Management Immediately:** Configure lifecycle rules on all EFS file systems. Set it to automatically transition files to **Infrequent Access (IA)** after 7 or 14 days of no access, and to **Archive** after 90 days. This can reduce your storage bill by up to **90%**.
2.  **Audit EFS Throughput Configurations:** Check your file systems' throughput settings. 
    *   Change from **Provisioned** to **Elastic** mode for development, staging, or highly spiky web application workloads.
    *   If you have a large storage volume (e.g., >1 TB), switch to **Bursting** mode, as the large storage volume automatically grants a high baseline throughput for free.
3.  **Configure Local AZ Mount Targets:** Ensure your ECS tasks, Kubernetes pods, or EC2 instances mount EFS using the local AZ endpoint. Use AWS-provided DNS names (which resolve to the local subnet IP of EFS) to prevent cross-AZ data transfer fees.
4.  **Use One Zone EFS for Development Environments:** In non-production environments where high availability is not required, use **One Zone** storage. This instantly cuts your EFS storage costs by **47%**.
5.  **Clean up old EFS Backups:** Audit EFS automatic backups in AWS Backup. Set a strict lifecycle policy to expire dev backups after 7 days to avoid long-term backup storage accumulation.
