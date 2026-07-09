# Cost-Cutting Playbook: Amazon FSx (Lustre, NetApp ONTAP, Windows, OpenZFS)

> **Companion Files:** [fsx_lustre.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/fsx/fsx_lustre.md) | [fsx_netapp.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/fsx/fsx_netapp.md) | [fsx_windows.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/fsx/fsx_windows.md) | [fsx_openzfs.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/fsx/fsx_openzfs.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon FSx provides feature-rich, high-performance managed file systems optimized for specialized workloads: **FSx for Lustre** (HPC / ML training), **FSx for NetApp ONTAP** (enterprise SAN/NAS data management), **FSx for Windows File Server** (native Active Directory SMB integration), and **FSx for OpenZFS** (ultra-low latency Linux shared storage).

Key billing traps include:
1. **Lustre Throughput Tier Multiplier:** Selecting the 1000 MB/s/TB tier ($0.830/GB-mo) instead of the 12 MB/s/TB tier ($0.133/GB-mo) — a **524% price multiplier** on storage capacity!
2. **Leaving Ephemeral Lustre Scratch Filesystems Running:** Deploying Scratch SSD file systems ($0.043/GB-mo) for short ML training runs and leaving them billing continuously 24/7 after the job finishes.
3. **NetApp ONTAP Provisioned SSD Storage Over-Provisioning:** Keeping cold snapshots or inactive files on expensive Primary SSD storage ($0.25/GB-mo) instead of auto-tiering to **Capacity Pool Storage ($0.025/GB-mo — a 90% discount)**.

This playbook provides **20 actionable strategies** across six operational categories, delivering an estimated **35–75% reduction in total Amazon FSx spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Enable NetApp ONTAP FabricPool Auto Data Tiering:** Automatically shifts cold file blocks to Capacity Pool Storage ($0.025/GB-mo vs $0.25/GB-mo — an immediate **90% storage discount**).
2. **Use Scratch SSD File Systems for Ephemeral ML / Rendering Jobs:** Deploys cheap $0.043/GB-mo Scratch storage linked to S3; delete immediately post-job.
3. **Enable Data Compression (LZ4 / ZSTD / ONTAP Inline Compression):** Reduces stored file volume by **30–50%** across Lustre, ONTAP, and OpenZFS.

---

## Strategy Categories

### 1. Deployment Model & Ephemeral Storage Optimization

#### 1. Deploy Ephemeral Scratch SSD File Systems for ML & HPC Workloads (FSx for Lustre)
- **What:** Use **Scratch SSD** file systems for temporary machine learning training cycles, financial modeling runs, or video rendering jobs. Link the file system directly to an S3 bucket for auto-loading training data, and export results back to S3.
- **Why It Saves Money:**
  - **Persistent SSD (Single-AZ):** Starts at **$0.133 per GB-month**.
  - **Scratch SSD:** Billed at **$0.043 per GB-month** (**67.6% cheaper**).
  - Deleting the Scratch file system immediately post-job drops compute costs to $0.00 when idle.
- **Detailed Implementation Steps:**
  1. Deploy Scratch Lustre file system with S3 Data Repository Association in Terraform:
     ```hcl
     resource "aws_fsx_lustre_file_system" "ephemeral_ml" {
       import_path      = "s3://company-ml-datasets/training-v1/"
       export_path      = "s3://company-ml-datasets/results/"
       storage_capacity = 2400
       deployment_type  = "SCRATCH_2"
     }
     ```
  2. Add automated deletion task in Airflow / Step Functions DAG upon job completion.
- **Estimated Savings:** **67.6% lower hourly storage rate** plus 100% savings on idle periods.
- **Risk Level:** Zero risk (data is durably backed up in S3).
- **Implementation Scope:** Data Engineer / Machine Learning Engineer
- **Prerequisites:** S3 data repository integration.

#### 2. Auto-Delete Abandoned Scratch Lustre File Systems
- **What:** Deploy a scheduled Lambda function to query and terminate Scratch Lustre file systems that have been un-mounted or idle for > 24 hours.
- **Why It Saves Money:** A 10 TB Scratch file system left running idle wastes **$430.00/month**.
- **Detailed Implementation Steps:**
  1. Identify idle Lustre file systems via `aws fsx describe-file-systems`.
  2. Terminate idle instances: `aws fsx delete-file-system --file-system-id fs-1234567890`.
- **Estimated Savings:** 100% of abandoned ephemeral file system charges.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Verification that output artifacts were exported to S3.

---

### 2. Storage Tiering & Capacity Optimization

#### 3. Enable NetApp ONTAP Auto Data Tiering (FabricPool)
- **What:** Set the Storage Tiering Policy on FSx for NetApp ONTAP volumes to `AUTO` or `ALL`.
- **Why It Saves Money:**
  - **Primary SSD Storage:** **$0.25 per GB-month**.
  - **Capacity Pool Storage (S3):** **$0.025 per GB-month** (**90.0% storage discount**).
  - Inactive file blocks and snapshots are automatically moved to cheap Capacity Pool Storage, cutting storage spend on a 20 TB volume from $5,000.00/month to **$800.00/month** (saving **$4,200.00/month**)!
- **Detailed Implementation Steps:**
  1. Update volume tiering policy via AWS CLI:
     ```bash
     aws fsx update-volume \
       --volume-id fsvol-1234567890 \
       --ontap-configuration '{"TieringPolicy": {"Name": "AUTO", "CoolingDays": 31}}'
     ```
- **Estimated Savings:** **70–90% reduction** in storage costs for cold/inactive data blocks.
- **Risk Level:** Zero risk (NetApp FabricPool transparently retrieves tiered blocks when accessed).
- **Implementation Scope:** Infrastructure Engineer / DevOps
- **Prerequisites:** ONTAP volume deployed.

#### 4. Right-Size Throughput Tiers (FSx for Lustre & OpenZFS)
- **What:** Downsize provisioned throughput tiers on FSx for Lustre and FSx for OpenZFS to match actual application IOPS demands.
- **Why It Saves Money:**
  - **Lustre 40 MB/s/TB tier:** $0.170 per GB-month.
  - **Lustre 500 MB/s/TB tier:** $0.490 per GB-month (**188% price hike**).
  - **Lustre 1000 MB/s/TB tier:** $0.830 per GB-month (**388% price hike**).
  - Downsizing a 10 TB Lustre volume from 500 MB/s/TB ($4,900.00/mo) to 125 MB/s/TB ($2,330.00/mo) saves **$2,570.00/month ($30,840/year)**!
- **Detailed Implementation Steps:**
  1. Monitor CloudWatch metric `DataReadBytes` and `DataWriteBytes`.
  2. Modify throughput tier in file system settings.
- **Estimated Savings:** **50–70% capacity fee reduction** on over-provisioned throughput tiers.
- **Risk Level:** Low.
- **Implementation Scope:** Infrastructure Engineer
- **Prerequisites:** IOPS baseline monitoring.

---

### 3. Data Compression & Deduplication

#### 5. Enable Native LZ4 / ZSTD Data Compression
- **What:** Turn on native hardware-accelerated **LZ4 compression** (on FSx for Lustre and OpenZFS) or **ZSTD / ONTAP Inline Storage Efficiency** (Deduplication and Compression).
- **Why It Saves Money:** Automatically compresses files as they are written to disk, reducing stored GB volume by **30% to 60%** for text, CSV, log, and source code datasets.
- **Detailed Implementation Steps:**
  1. Enable LZ4 compression on Lustre via AWS CLI:
     ```bash
     aws fsx update-file-system \
       --file-system-id fs-1234567890 \
       --lustre-configuration "DataCompressionType=LZ4"
     ```
  2. Enable ONTAP inline deduplication and compression in ONTAP CLI: `volume efficiency on -vserver svm1 -volume vol1`.
- **Estimated Savings:** **30–60% direct storage footprint reduction**.
- **Risk Level:** Zero risk (CPU overhead is handled transparently by dedicated FSx storage controllers).
- **Implementation Scope:** Systems Administrator / DevOps
- **Prerequisites:** File system supports compression.

#### 6. Enable Deduplication on FSx for Windows File Server
- **What:** Turn on **Data Deduplication** on FSx for Windows File Server volumes.
- **Why It Saves Money:** Identifies and eliminates duplicate file chunks across user home directories or corporate file shares, saving **50–80% storage space**.
- **Detailed Implementation Steps:**
  1. Run PowerShell command on FSx Windows management endpoint:
     ```powershell
     Enable-FsxDedup -Volume "D:"
     Set-FsxDedupSchedule -Name "DailyDedup" -Type Optimization -Start "02:00" -DurationHours 4
     ```
- **Estimated Savings:** 50–80% storage reduction for Windows file shares.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Windows Administrator / DevOps
- **Prerequisites:** Remote PowerShell management access.

---

### 4. Redundancy & Non-Production Sizing

#### 7. Deploy Single-AZ File Systems in Non-Production Environments
- **What:** Create non-production (dev, test, staging) FSx file systems using **Single-AZ** deployment types instead of Multi-AZ.
- **Why It Saves Money:**
  - **Lustre Persistent Multi-AZ:** Starts at $0.230/GB-mo.
  - **Lustre Persistent Single-AZ:** Starts at $0.133/GB-mo (**42.1% discount**).
  - **ONTAP Multi-AZ:** $0.400/GB-mo.
  - **ONTAP Single-AZ:** $0.250/GB-mo (**37.5% discount**).
  - Single-AZ cuts non-prod storage costs by **37–42%**.
- **Detailed Implementation Steps:**
  1. Set `deployment_type = "SINGLE_AZ_1"` or `"SINGLE_AZ"` in Terraform.
- **Estimated Savings:** **37–42% storage savings** in non-prod environments.
- **Risk Level:** Low (non-prod workloads do not require multi-AZ DR failover).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod SLA alignment.

#### 8. Use HDD Storage Classes for Capacity-Optimized Workloads
- **What:** Deploy **Persistent HDD** storage classes (with SSD caching) for large sequential read/write file systems (FSx for Lustre, ONTAP, or Windows).
- **Why It Saves Money:**
  - **Single-AZ SSD Storage (Lustre):** $0.133 per GB-month.
  - **Single-AZ HDD Storage (Lustre):** **$0.013 per GB-month** (**90.2% storage discount**!).
  - Includes a built-in 20% SSD cache to accelerate hot files while billing storage at HDD rates.
- **Detailed Implementation Steps:**
  1. Select HDD storage type during file system creation.
- **Estimated Savings:** **90% reduction** in baseline storage rates.
- **Risk Level:** Low (best for large sequential workloads; avoid for high-random IOPS databases).
- **Implementation Scope:** Infrastructure Architect
- **Prerequisites:** Workload IOPS pattern verification.

---

### 5. Backup & Snapshot Governance

#### 9. Audit AWS Backup Retention for FSx Snapshots
- **What:** Configure AWS Backup lifecycle policies to expire automatic daily FSx backups after 14–30 days.
- **Why It Saves Money:** FSx backup storage costs **$0.05 per GB-month**. Un-capped backup retention accumulates terabytes of historical snapshots ($50.00/TB-mo).
- **Detailed Implementation Steps:**
  1. Set backup retention window = 30 days in AWS Backup Vault.
- **Estimated Savings:** 50–80% backup storage savings.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Retention policy compliance approval.

#### 10. Delete Stale Manual FSx Snapshots
- **What:** Identify and delete legacy manual FSx snapshots (`aws fsx describe-snapshots`) older than 30 days.
- **Why It Saves Money:** Reclaims $0.05/GB-mo backup storage fees.
- **Detailed Implementation Steps:**
  1. Delete snapshot: `aws fsx delete-snapshot --snapshot-id fssnap-1234567890`.
- **Estimated Savings:** 100% of manual snapshot storage fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Backup audit.

---

### 6. Observability & Network Optimization

#### 11. Restrict S3 Data Repository Metadata Import Prefixes
- **What:** Configure FSx for Lustre S3 Data Repository Associations to import metadata ONLY for specific S3 sub-folders (prefixes) containing active datasets.
- **Why It Saves Money:** Linking Lustre to an S3 bucket containing 100 million tiny files forces Lustre to issue 100M S3 `LIST`/`GET` requests, generating **$500.00+ in S3 API request fees** during initialization.
- **Detailed Implementation Steps:**
  1. Specify explicit import prefix: `s3://my-bucket/active-training-data/` instead of `s3://my-bucket/`.
- **Estimated Savings:** 90% reduction in S3 API import request fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** S3 folder organization.

#### 12. Deploy Gateway VPC Endpoints for S3 Synchronization
- **What:** Ensure FSx subnets route S3 data synchronization traffic via Gateway VPC Endpoints.
- **Why It Saves Money:** Prevents S3 data sync traffic from passing through NAT Gateways ($0.045/GB processing tax). Gateway Endpoints are **100% FREE ($0.00/GB)**.
- **Detailed Implementation Steps:**
  1. Attach Gateway VPC Endpoint to FSx subnet route table.
- **Estimated Savings:** Saves $45.00/TB on S3 dataset sync.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** VPC route table access.

#### 13. Enforce CloudWatch Alarms on Storage Capacity Utilization
- **What:** Put CloudWatch alarm on `FreeStorageCapacity` (< 15%).
- **Why It Saves Money:** Prevents out-of-space errors that force emergency storage expansion.
- **Detailed Implementation Steps:**
  1. Put CloudWatch metric alarm on `AWS/FSx`.
- **Estimated Savings:** Operational risk mitigation.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 14. Optimize NetApp ONTAP Storage Capacity Auto-Scaling Limits
- **What:** Configure Storage Auto-Scaling on FSx for ONTAP file systems.
- **Why It Saves Money:** Prevents static over-provisioning of SSD storage.
- **Detailed Implementation Steps:**
  1. Set auto-scaling policy target = 80%.
- **Estimated Savings:** 30–50% storage capacity optimization.
- **Risk Level:** Low.
- **Implementation Scope:** Infrastructure Engineer
- **Prerequisites:** ONTAP volume configuration.

#### 15. Standardize OpenZFS Volume Snapshot Retention
- **What:** Set automatic retention policies on FSx for OpenZFS snapshots.
- **Why It Saves Money:** Reclaims intermediate snapshot storage blocks.
- **Detailed Implementation Steps:**
  1. Set OpenZFS snapshot retention policy via CLI.
- **Estimated Savings:** 20–40% storage space reclamation.
- **Risk Level:** Low.
- **Implementation Scope:** Systems Administrator
- **Prerequisites:** OpenZFS deployment.

#### 16. Consolidate Storage Virtual Machines (SVMs) on ONTAP
- **What:** Share Storage Virtual Machines across tenant workloads.
- **Why It Saves Money:** Reduces fixed SVM management overhead ($0.05/hr per SVM).
- **Detailed Implementation Steps:**
  1. Consolidate SVM configurations in ONTAP management console.
- **Estimated Savings:** Reclaims SVM hourly base fees.
- **Risk Level:** Low.
- **Implementation Scope:** Storage Administrator
- **Prerequisites:** Multi-tenant isolation review.

#### 17. Monitor and Alarm on Network Throughput Saturation
- **What:** Alarm on `NetworkBandwidthUtilization` (> 90%).
- **Why It Saves Money:** Identifies throughput bottlenecks before upgrading throughput tiers.
- **Detailed Implementation Steps:**
  1. Put CloudWatch alarm on network throughput metrics.
- **Estimated Savings:** Avoids premature throughput tier upgrades.
- **Risk Level:** Zero.
- **Implementation Scope:** Infrastructure Engineer
- **Prerequisites:** SNS topic setup.

#### 18. Audit Cross-AZ File Access (Client AZ Alignment)
- **What:** Align Linux/Windows clients to mount FSx endpoints in their local AZ.
- **Why It Saves Money:** Eliminates cross-AZ data transfer fees ($0.01/GB in each direction).
- **Detailed Implementation Steps:**
  1. Mount local AZ IP endpoints.
- **Estimated Savings:** Saves $20.00/TB on cross-AZ mount traffic.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-AZ subnet endpoints.

#### 19. Utilize Free Tier Allocations
- **What:** Audit small non-prod test volumes.
- **Why It Saves Money:** Operational hygiene.
- **Detailed Implementation Steps:**
  1. Review Billing Console.
- **Estimated Savings:** Administrative hygiene.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

#### 20. Enforce Automated Environment Shutdown on Maintenance Windows
- **What:** Schedule non-prod file system maintenance windows during off-hours.
- **Why It Saves Money:** Reduces operational friction.
- **Detailed Implementation Steps:**
  1. Configure maintenance windows via CLI.
- **Estimated Savings:** Administrative efficiency.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Maintenance window schedule.

---

## Cross-Service Synergies

```
[ Amazon FSx Storage ] 
        │
        ├──(ONTAP Auto Tiering)──> [ Capacity Pool Storage (S3) ] (90% storage discount - $0.025/GB)
        │
        ├──(Lustre Ephemeral)────> [ Scratch SSD + S3 Sync ] (67% lower storage rate + $0 idle fee)
        │
        └──(Data Compression)────> [ LZ4 / ONTAP Compression ] (Saves 30-60% stored byte volume)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `FSxLustre-Scratch-SSD-GB-mo`, `FSxLustre-Persistent-SSD-GB-mo`, `FSxONTAP-Primary-SSD-GB-mo`, `FSxONTAP-CapacityPool-GB-mo`, `FSxWin-SSD-GB-mo`.
- `line_item_resource_id`: FSx File System ID (`fs-xxxx`) / Volume ID (`fsvol-xxxx`).

### B. CloudWatch Metrics
- `AWS/FSx` Namespace: `FreeStorageCapacity`, `DataReadBytes`, `DataWriteBytes`, `DataReadOperations`, `DataWriteOperations`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "FSX-ONT-001",
  "service": "FSx",
  "category": "Storage Tiering & Capacity Optimization",
  "resource_id": "arn:aws:fsx:us-east-1:123456789012:volume/fsvol-0123456789abcdef0",
  "resource_name": "prod-enterprise-ontap-vol",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "file_system_type": "FSx for NetApp ONTAP",
    "tiering_policy": "NONE",
    "allocated_ssd_storage_gb": 15000,
    "monthly_cost_usd": 3750.00
  },
  "recommended_config": {
    "tiering_policy": "AUTO",
    "projected_primary_ssd_gb": 3000,
    "projected_capacity_pool_gb": 12000,
    "projected_monthly_cost_usd": 1050.00
  },
  "financial_impact": {
    "monthly_savings_usd": 2700.00,
    "annual_savings_usd": 32400.00,
    "savings_percentage": 72.0
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "ONTAP FabricPool auto-tiering transparently shifts cold file blocks to S3 Capacity Pool storage without downtime."
  },
  "implementation": {
    "scope": "Infrastructure Engineer / DevOps",
    "effort_estimate": "30 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **ONTAP FabricPool Auto Tiering** | 8 | $22,500.00 | $16,200.00 | 72.0% | Zero |
| **Lustre Ephemeral Scratch Migration**| 6 | $8,400.00 | $5,678.40 | 67.6% | Zero |
| **Lustre / OpenZFS Throughput Rightsizing**| 5 | $14,200.00 | $7,100.00 | 50.0% | Low |
| **LZ4 / Windows Data Compression** | 10 | $12,000.00 | $4,800.00 | 40.0% | Zero |
| **Total** | **29** | **$57,100.00** | **$33,778.40** | **59.2%** | -- |
