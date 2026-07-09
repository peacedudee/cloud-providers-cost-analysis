# Cost-Cutting Playbook: Amazon EFS (Elastic File System)

> **Companion File:** [efs.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/efs/efs.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon EFS provides serverless POSIX shared file storage for EC2, ECS, and Lambda. Unlike EBS (where you pay for provisioned space), EFS Standard storage charges per actual GB stored (**$0.30 per GB-month** for Regional Standard).

Key cost traps include:
1. **Keeping Active Data on EFS Standard:** Storing cold files or logs on EFS Standard ($0.30/GB-mo) instead of lifecycle transitioning to **Infrequent Access (IA)** ($0.0165/GB-mo — 95% savings) or **Archive** ($0.008/GB-mo — 97% savings).
2. **The 128 KiB Small-File IA Billing Trap:** Moving millions of files smaller than 128 KiB to IA or Archive, where every small file is billed as if it were **128 KiB minimum**, causing a **32x storage charge inflation**!
3. **Provisioned Throughput Overhead:** Paying $6.00 per MB/s-month flat for provisioned throughput on idle file systems.

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **40–90% reduction in total EFS spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Enable EFS Lifecycle Management (Transition to IA / Archive):** Automatically transition files to Infrequent Access (IA) after 7 or 14 days of no access for an immediate **95% storage discount** ($0.0165/GB-mo vs $0.30/GB-mo).
2. **Audit & Switch Provisioned Throughput to Elastic Mode:** Eliminates $6.00/MB/s-month baseline charges for spiky web or developer workloads.
3. **Configure Local AZ Mount Targets:** Eliminates cross-AZ data transfer egress ($0.01/GB in each direction) by mounting local subnet EFS endpoints.

---

## Strategy Categories

### 1. Storage Tiering & Lifecycle Management

#### 1. Enable Automated EFS Lifecycle Management (Standard -> IA -> Archive)
- **What:** Configure Lifecycle Policies on all Regional EFS file systems to automatically transition files un-accessed for 7–14 days to **EFS Infrequent Access (IA)**, and files un-accessed for 90 days to **EFS Archive**.
- **Why It Saves Money:**
  - **Standard Storage:** **$0.30 per GB-month**.
  - **Infrequent Access (IA):** **$0.0165 per GB-month** (**94.5% storage discount**).
  - **Archive Storage:** **$0.008 per GB-month** (**97.3% storage discount**).
  - Storing 10 TB on Standard costs **$3,000.00/month**. Lifecycle transitioning 80% cold data to IA drops storage cost down to **$732.00/month** (saving **$2,268.00/month**)!
- **Detailed Implementation Steps:**
  1. Update lifecycle configuration via AWS CLI:
     ```bash
     aws efs put-lifecycle-configuration \
       --file-system-id fs-1234567890 \
       --lifecycle-policies TransitionToIA=AFTER_14_DAYS,TransitionToArchive=AFTER_90_DAYS
     ```
- **Estimated Savings:** **70–90% reduction** in total file system storage billing.
- **Risk Level:** Zero risk (retrievals from IA are fully automatic with sub-millisecond access latency).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** File system access patterns align with 14-day inactivity.

#### 2. Avoid Transitioning Small Files (< 128 KiB) to IA / Archive (The 128 KiB Trap)
- **What:** Identify file systems containing millions of small files (< 128 KiB, such as 4 KB web icons, source code repos, or log files) and compress them into `.tar.gz` archives BEFORE transitioning to IA/Archive tiers.
- **Why It Saves Money:** EFS IA and Archive enforce a **minimum billable file size of 128 KiB**. Moving 1 million 4 KB files (4 GB actual data) to IA bills as 1,000,000 × 128 KiB = **128 GB billed storage** — a **32x billing inflation**!
- **Detailed Implementation Steps:**
  1. Scan file system for small files: `find /mnt/efs -type f -size -128k | wc -l`.
  2. If small file count > 100,000, pack folders into tarballs before lifecycle transition:
     ```bash
     tar -czf /mnt/efs/archives/logs_2025.tar.gz /mnt/efs/raw_logs_2025/ && rm -rf /mnt/efs/raw_logs_2025/
     ```
- **Estimated Savings:** Prevents 500–3,000% storage inflation charges on small file repositories.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** File archiving workflow script.

---

### 2. Throughput Mode & Capacity Optimization

#### 3. Audit & Convert Provisioned Throughput to Elastic Mode
- **What:** Change EFS throughput mode from `Provisioned` to `Elastic` for non-constant or spiky workloads.
- **Why It Saves Money:** Provisioned Throughput charges a flat **$6.00 per MB/s-month** regardless of usage. Provisioning 100 MB/s costs **$600.00/month flat**, even if the file system is completely idle. Elastic Throughput charges purely on data read/written ($0.03/GB read, $0.06/GB write) and scales down to **$0.00 when idle**.
- **Detailed Implementation Steps:**
  1. Modify throughput mode via AWS CLI:
     ```bash
     aws efs update-file-system \
       --file-system-id fs-1234567890 \
       --throughput-mode elastic
     ```
- **Estimated Savings:** **$600.00 per month saved** per 100 MB/s provisioned on low/variable utilization file systems.
- **Risk Level:** Zero risk (Elastic mode scales throughput dynamically up to 3 GiB/s read / 1 GiB/s write).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Throughput utilization audit.

#### 4. Switch Large Storage File Systems (> 1 TB) to Bursting Mode
- **What:** For large storage volumes (> 1 TB), evaluate switching throughput mode to `Bursting`.
- **Why It Saves Money:** EFS Bursting mode is **100% FREE (included in storage price)**. Storage automatically grants 50 MB/s baseline throughput per 1 TB stored, plus burst credits up to 100 MB/s. If your file system stores 2 TB, you get 100 MB/s baseline for free without paying $600/mo for Provisioned mode.
- **Detailed Implementation Steps:**
  1. Modify throughput mode: `aws efs update-file-system --file-system-id fs-1234567890 --throughput-mode bursting`.
- **Estimated Savings:** 100% of provisioned throughput fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Storage volume > 1 TB.

---

### 3. Redundancy & Non-Production Deployment

#### 5. Deploy One Zone EFS Storage for Non-Production Environments
- **What:** Create non-production (dev, test, QA) file systems using the **One Zone EFS** storage class.
- **Why It Saves Money:** One Zone EFS stores data in a single Availability Zone, cutting storage rates from $0.30/GB-mo down to **$0.16 per GB-month** — an instant **46.7% storage discount**!
- **Detailed Implementation Steps:**
  1. Create One Zone EFS in Terraform:
     ```hcl
     resource "aws_efs_file_system" "dev_efs" {
       creation_token   = "dev-efs"
       availability_zone_name = "us-east-1a"
       lifecycle_policy {
         transition_to_ia = "AFTER_14_DAYS"
       }
     }
     ```
- **Estimated Savings:** **46.7% storage savings** in non-prod environments.
- **Risk Level:** Low (for non-prod workloads that do not require multi-AZ DR resilience).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod SLA alignment.

---

### 4. Network & Mount Target Optimization

#### 6. Configure Local AZ Mount Targets (Eliminate Cross-AZ Egress)
- **What:** Ensure EC2, ECS, or Lambda clients connect to the EFS mount target located in their **same Availability Zone**.
- **Why It Saves Money:** If an EC2 instance in AZ-A mounts an EFS target in AZ-B (due to hardcoded IPs or bad DNS resolution), all file read/write traffic incurs standard inter-AZ egress fees (**$0.01/GB in each direction = $0.02/GB roundtrip**). Reading 10 TB across AZs adds **$200.00/month** in unnecessary network tax.
- **Detailed Implementation Steps:**
  1. Mount EFS using the standard AWS DNS format (`file-system-id.efs.region.amazonaws.com`), which automatically resolves to the local subnet IP of EFS in that AZ.
  2. Verify mount configuration in `/etc/fstab`:
     ```text
     fs-1234567890.efs.us-east-1.amazonaws.com:/ /mnt/efs efs _netdev,noresvport,tls 0 0
     ```
- **Estimated Savings:** 100% of inter-AZ data transfer fees ($20.00/TB saved).
- **Risk Level:** Zero risk.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** EFS mount targets deployed in all active VPC subnets.

---

### 5. Alternative Storage Architecture

#### 7. Offload Unstructured Static Media to Amazon S3
- **What:** Refactor web application code to store static media uploads (images, PDFs, user avatars) in **Amazon S3** instead of mounting EFS shared drives.
- **Why It Saves Money:** EFS Standard ($0.30/GB-mo) is **13x more expensive** than S3 Standard ($0.023/GB-mo). Storing 5 TB of web media on EFS costs **$1,500.00/month**, vs **$115.00/month** on S3.
- **Detailed Implementation Steps:**
  1. Use AWS SDK (Boto3/S3Client) to upload files directly to S3.
- **Estimated Savings:** **92.3% storage cost reduction**.
- **Risk Level:** Medium (requires application code update).
- **Implementation Scope:** Software Engineer
- **Prerequisites:** S3 integration feasibility.

#### 8. Utilize Amazon FSx for OpenZFS for High-IOPS Linux Shared Storage
- **What:** Evaluate migrating high-throughput shared Linux file systems to **FSx for OpenZFS**.
- **Why It Saves Money:** FSx for OpenZFS provides baseline storage at **$0.06/GB-mo** (vs EFS Standard at $0.30/GB-mo) with high IOPS and block-level data compression.
- **Detailed Implementation Steps:**
  1. Compare IOPS requirements and cost via AWS Pricing Calculator.
- **Estimated Savings:** 40–60% TCO reduction for large Linux shared volumes.
- **Risk Level:** Medium.
- **Implementation Scope:** Infrastructure Architect / DevOps
- **Prerequisites:** File system migration planning.

---

### 6. Cleanup & Monitoring Governance

#### 9. Decommission Orphaned EFS File Systems
- **What:** Identify and delete EFS file systems with 0 active mount targets or zero client connections over 30 days.
- **Why It Saves Money:** Reclaims storage billing on abandoned test volumes.
- **Detailed Implementation Steps:**
  1. List EFS file systems and count active mount targets:
     ```bash
     aws efs describe-file-systems --query "FileSystems[*].{ID:FileSystemId,Size:SizeInBytes.Value,Targets:NumberOfMountTargets}"
     ```
  2. Delete orphaned file systems: `aws efs delete-file-system --file-system-id fs-xxx`.
- **Estimated Savings:** 100% of abandoned file system spend.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Backup verification.

#### 10. Enable AWS Backup Expiration Rules for EFS Automatic Backups
- **What:** Configure AWS Backup lifecycle rules to expire automatic EFS daily backups after 14–30 days.
- **Why It Saves Money:** Prevents infinite accumulation of backup snapshots ($0.05/GB-mo).
- **Detailed Implementation Steps:**
  1. Update AWS Backup Vault lifecycle policy with retention limit = 30 days.
- **Estimated Savings:** 50–80% backup storage savings.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compliance retention approval.

#### 11. Delete Stale EFS Manual Snapshots
- **What:** Delete legacy manual snapshots older than 30 days.
- **Why It Saves Money:** Reclaims backup storage charges ($0.05/GB-mo).
- **Detailed Implementation Steps:**
  1. List and delete snapshots via AWS Backup CLI.
- **Estimated Savings:** Reclaims manual backup fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Backup audit.

#### 12. Monitor CloudWatch `PercentIOLimit` Metrics
- **What:** Set CloudWatch alarm on `PercentIOLimit` (> 90%).
- **Why It Saves Money:** Alerts before throughput throttling stalls application performance.
- **Detailed Implementation Steps:**
  1. Put CloudWatch metric alarm on `AWS/EFS` `PercentIOLimit`.
- **Estimated Savings:** Operational risk protection.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch alarm setup.

#### 13. Enable Compression on Local Application Mounts
- **What:** Enable gzip/zstd compression on client applications writing text logs to EFS.
- **Why It Saves Money:** Slashes stored byte volume by 60–80%.
- **Detailed Implementation Steps:**
  1. Configure logrotate to compress logs locally before saving to EFS.
- **Estimated Savings:** 60–80% storage footprint reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Logrotate setup.

#### 14. Standardize IAM Authorization Policies to Prevent Unapproved Creation
- **What:** Attach SCP restricting `elasticfilesystem:CreateFileSystem` without `Environment` tags.
- **Why It Saves Money:** Prevents un-tracked developer file system proliferation.
- **Detailed Implementation Steps:**
  1. Create SCP requiring tagging on creation.
- **Estimated Savings:** Governance enforcement.
- **Risk Level:** Zero.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** SCP configuration access.

#### 15. Leverage Free Tier Storage (5 GB/Mo)
- **What:** Utilize 5 GB free monthly storage for new accounts.
- **Why It Saves Money:** Baseline testing savings.
- **Detailed Implementation Steps:**
  1. Track free tier allocation in Billing Console.
- **Estimated Savings:** Baseline testing.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

#### 16. Monitor and Alarm on EFS Storage Growth
- **What:** Create CloudWatch alarm on `StorageBytes` (> 80% expected threshold).
- **Why It Saves Money:** Proactive alerts against unexpected file system bloat.
- **Detailed Implementation Steps:**
  1. Put CloudWatch alarm on `AWS/EFS` `StorageBytes`.
- **Estimated Savings:** Operational risk mitigation.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

---

## Cross-Service Synergies

```
[ Amazon EFS File System ] 
        │
        ├──(Lifecycle Rules)───> [ Infrequent Access (IA) ] (95% storage discount - $0.0165/GB)
        │
        ├──(Throughput Mode)───> [ Elastic Throughput ] (Eliminates $6/MB/s-mo flat base fee)
        │
        └──(Network Routing)───> [ Local AZ Mount Target ] (100% FREE - Avoids cross-AZ egress)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `EFS-StorageUsage:TimedStorage-ByteHrs`, `EFS-IAStorageUsage`, `EFS-ProvisionedThroughput-MBpsHrs`, `EFS-ElasticThroughput-IOBytes`.
- `line_item_resource_id`: EFS File System ID (`fs-xxxx`).

### B. CloudWatch Metrics
- `AWS/EFS` Namespace: `StorageBytes`, `PercentIOLimit`, `PermittedThroughput`, `DataReadIOBytes`, `DataWriteIOBytes`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "EFS-LFC-001",
  "service": "EFS",
  "category": "Storage Tiering & Lifecycle Management",
  "resource_id": "arn:aws:elasticfilesystem:us-east-1:123456789012:file-system/fs-1234567890",
  "resource_name": "prod-shared-app-data",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "storage_class": "Regional Standard (No Lifecycle)",
    "allocated_storage_gb": 8500,
    "monthly_cost_usd": 2550.00
  },
  "recommended_config": {
    "action": "Enable Lifecycle Policy (14 Days -> IA)",
    "projected_standard_gb": 1500,
    "projected_ia_gb": 7000,
    "projected_monthly_cost_usd": 565.50
  },
  "financial_impact": {
    "monthly_savings_usd": 1984.50,
    "annual_savings_usd": 23814.00,
    "savings_percentage": 77.8
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Cold files transition automatically to IA; reads execute transparently with sub-millisecond latency."
  },
  "implementation": {
    "scope": "Engineer/DevOps",
    "effort_estimate": "15 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **EFS Lifecycle Management (IA)** | 10 | $18,500.00 | $14,393.00 | 77.8% | Zero |
| **Elastic Throughput Conversion** | 6 | $4,800.00 | $4,800.00 | 100.0% | Zero |
| **Local AZ Mount Egress Avoidance**| 8 | $3,200.00 | $3,200.00 | 100.0% | Zero |
| **One Zone Dev Deployment** | 4 | $2,400.00 | $1,120.80 | 46.7% | Low |
| **Total** | **28** | **$28,900.00** | **$23,513.80** | **81.4%** | -- |
