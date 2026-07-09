# Cost-Cutting Playbook: AWS Storage Gateway

> **Companion File:** [storage_gateway.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/storage_gateway/storage_gateway.md)  
> **Last Updated:** July 2026

---

## Executive Summary

AWS Storage Gateway connects on-premises applications to cloud storage (S3, EBS, Glacier). Billing includes **Data Written to AWS (Processing Fee)** ($0.01 per GB uploaded, capped at **$125.00/month per gateway**), **Target Cloud Storage** (Volume Gateway at $0.023/GB-mo; Tape Gateway Glacier at $0.00099–$0.0036/GB-mo), and **Network Data Transfer Out** ($0.09/GB internet egress on cache misses).

Key billing traps include:
1. **Tiny Local Cache Disk Size (Cache Miss Egress Tax):** Provisioning small local SSD cache disks on the gateway VM causes frequent cache evictions. On-premises clients requesting files trigger "cache misses," forcing downloads from AWS over the public internet (**$0.09/GB egress fee**).
2. **Gateway VM Sprawl:** Running multiple separate file gateways for different teams where each gateway hits the $125/month write cap independently ($500/mo for 4 gateways).
3. **EC2 Hosting Fees for Gateway Appliances:** Running Storage Gateway on EC2 instances (e.g. `m5.xlarge`) inside AWS instead of using local VMware/Hyper-V host servers (which are **100% FREE**).

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **35–80% reduction in Storage Gateway spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Right-Size Local SSD Cache Disk (Target > 95% Cache Hit Ratio):** Expanding local cache disk eliminates cache misses and slashes expensive internet egress charges ($0.09/GB).
2. **Consolidate On-Premises Gateways under a Single $125/Mo Write Cap:** Aggregates file shares to share local cache and cap upload fees at $125/mo per physical site.
3. **Transition Virtual Tapes to Glacier Deep Archive ($0.00099/GB-mo):** Slashes Tape Gateway compliance storage costs by **95%**.

---

## Strategy Categories

### 1. Cache Disks & Egress Optimization

#### 1. Expand Local Cache Disks to Achieve > 95% Cache Hit Ratio
- **What:** Monitor CloudWatch metric `CacheHitPercent` on Storage Gateway appliances. Allocate additional local SSD storage to the gateway VM if cache hit ratio drops below 95%.
- **Why It Saves Money:**
  - **Cache Hit:** File is served locally from on-premises SSD cache (**$0.00 data transfer cost**).
  - **Cache Miss:** Gateway must download the file from AWS S3 over the internet, incurring **$0.09 per GB in Internet Egress fees**.
  - A gateway with an under-sized cache processing 10 TB of cache misses wastes **$900.00/month in internet egress**! Expanding local cache disk eliminates cache miss downloads.
- **Detailed Implementation Steps:**
  1. Check `CacheHitPercent` metric in CloudWatch.
  2. Attach additional local SSD disk to gateway VM in VMware vSphere / Hyper-V.
  3. Add disk to Gateway cache in AWS Console / CLI:
     ```bash
     aws storagegateway add-cache \
       --gateway-arn "arn:aws:storagegateway:us-east-1:123456789012:gateway/sgw-12345678" \
       --disk-ids "pci-0000:03:00.0-scsi-0:0:1:0"
     ```
- **Estimated Savings:** **80–95% reduction** in internet egress data transfer fees ($900/mo saved per 10 TB).
- **Risk Level:** Zero risk (improves on-premises application performance).
- **Implementation Scope:** Storage Administrator / VMware Admin
- **Prerequisites:** On-premises SAN/SSD storage capacity available.

---

### 2. Gateway Appliance Consolidation

#### 2. Consolidate Multiple Gateways to Maximize the $125/Mo Write Processing Cap
- **What:** Merge departmental file shares onto a single large Storage Gateway appliance per physical site rather than deploying separate VMs for every team.
- **Why It Saves Money:**
  - AWS charges **$0.01 per GB** of data written to AWS through the gateway, capped at **$125.00 per month per gateway**.
  - Running 5 separate gateways writing 15 TB each costs 5 × $125.00 = **$625.00/month**.
  - Consolidating into 1 central gateway writing 75 TB caps total write processing at **$125.00/month (a $500.00/month savings)**!
- **Detailed Implementation Steps:**
  1. Create consolidated SMB/NFS file shares on central File Gateway.
  2. Decommission individual departmental gateway VMs.
- **Estimated Savings:** **50–80% reduction** in Storage Gateway data write processing fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Systems Engineer / Storage Admin
- **Prerequisites:** On-premises network throughput capacity.

---

### 3. Target Cloud Storage Lifecycle Optimization

#### 3. Transition Tape Gateway Tapes to Glacier Deep Archive ($0.00099/GB-Mo)
- **What:** Configure on-premises backup software (Veeam, Veritas NetBackup, Commvault) to write virtual tapes and immediately eject them to the virtual tape shelf (VTS).
- **Why It Saves Money:**
  - **Active Tape Library Storage:** **$0.0230 per GB-month** ($23.00/TB-mo).
  - **Glacier Flexible Retrieval Archive:** **$0.0036 per GB-month** ($3.60/TB-mo).
  - **Glacier Deep Archive:** **$0.00099 per GB-month** ($0.99/TB-mo).
  - Archiving 50 TB of compliance tapes in Glacier Deep Archive drops storage from $1,150.00/mo to **$49.50/month (a 95.7% savings)**!
- **Detailed Implementation Steps:**
  1. Configure backup software export policy to eject tapes to `GLACIER_DEEP_ARCHIVE` pool.
- **Estimated Savings:** **95.7% reduction** in Tape Gateway storage costs.
- **Risk Level:** Zero risk (retrieval times match 12-hour compliance SLA).
- **Implementation Scope:** Backup Administrator
- **Prerequisites:** Backup software integration with Tape Gateway VTS.

#### 4. Apply S3 Lifecycle Rules on S3 File Gateway Target Buckets
- **What:** Configure S3 Lifecycle Rules on target S3 buckets backed by S3 File Gateway to transition objects to **S3 Standard-IA** ($0.0125/GB-mo) or **S3 Glacier** after 30 days.
- **Why It Saves Money:** S3 File Gateway writes files directly as S3 objects. Transitioning idle files drops storage fees by **45–80%**.
- **Detailed Implementation Steps:**
  1. Add S3 Lifecycle transition rule to target S3 bucket in Terraform.
- **Estimated Savings:** 45–80% target S3 storage fee reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Storage Administrator
- **Prerequisites:** S3 bucket access.

---

### 4. Hosting Infrastructure Sizing

#### 5. Utilize Free On-Premises Hypervisors (VMware / Hyper-V) Instead of EC2
- **What:** Deploy Storage Gateway virtual appliances on local on-premises hypervisors (VMware ESXi, Microsoft Hyper-V, Linux KVM) or hardware appliances instead of hosting the VM on Amazon EC2 inside AWS.
- **Why It Saves Money:**
  - **EC2 Hosted Gateway (`m5.xlarge`):** Costs **$0.192/hr = $140.16/month** in EC2 compute plus EBS volume charges.
  - **On-Premises Hypervisor Software:** **100% FREE ($0.00)**.
- **Detailed Implementation Steps:**
  1. Download Storage Gateway OVA template and deploy to local VMware cluster.
- **Estimated Savings:** **$140.16/month saved** per gateway in EC2 compute fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** VMware Admin / Systems Engineer
- **Prerequisites:** Local hypervisor host capacity.

---

### 5. Network Routing Optimization

#### 6. Route Storage Gateway Egress over Direct Connect (Discounted $0.02/GB Rate)
- **What:** Configure Storage Gateway network routing to download files from AWS over an existing **AWS Direct Connect** private connection.
- **Why It Saves Money:** Direct Connect egress costs **$0.02/GB** compared to public internet egress at **$0.09/GB** (a **78% direct discount**).
- **Detailed Implementation Steps:**
  1. Set up VPC Endpoint for Storage Gateway and route traffic via Private VIF over Direct Connect.
- **Estimated Savings:** **78% reduction** in cache miss retrieval network costs.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Direct Connect deployment operational.

---

### 6. Observability & Governance

#### 7. Audit Volume Gateway EBS Snapshot Schedules
- **What:** Optimize automated snapshot schedules on Volume Gateways to retain daily snapshots for 14 days rather than keeping daily snapshots indefinitely.
- **Why It Saves Money:** Volume Gateway snapshots bill at standard EBS snapshot rates ($0.05/GB-mo).
- **Detailed Implementation Steps:**
  1. Set AWS Backup plan retention for Volume Gateway snapshots.
- **Estimated Savings:** 50–70% reduction in snapshot storage fees.
- **Risk Level:** Low.
- **Implementation Scope:** Storage Administrator
- **Prerequisites:** AWS Backup integration.

#### 8. Maximize 100 GB Free Data Written Allowance
- **What:** Utilize 100 GB free monthly data written allowance for new accounts.
- **Why It Saves Money:** Baseline testing savings.
- **Detailed Implementation Steps:**
  1. Monitor free tier allocation in Billing Console.
- **Estimated Savings:** Free baseline testing.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

#### 9. Enforce CloudWatch Alarms for `CachePercentDirty` Metric
- **What:** Put CloudWatch alarm on `CachePercentDirty` (> 80%).
- **Why It Saves Money:** Instant alert if local cache writes bottleneck, preventing upload failures.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm targeting `AWS/StorageGateway`.
- **Estimated Savings:** Operational risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Systems Engineer
- **Prerequisites:** SNS topic setup.

#### 10. Audit Virtual Tape Retrieval Frequencies
- **What:** Use Bulk retrieval mode for retrieving archived virtual tapes from Glacier.
- **Why It Saves Money:** Reduces tape retrieval processing fees ($0.01 to $0.03/GB).
- **Detailed Implementation Steps:**
  1. Select Bulk retrieval mode during tape restoration.
- **Estimated Savings:** 60–80% tape restore fee optimization.
- **Risk Level:** Zero.
- **Implementation Scope:** Backup Administrator
- **Prerequisites:** None.

#### 11. Right-Size Gateway Appliance RAM Allocation
- **What:** Provision at least 16 GB RAM for File Gateway VMs (32 GB recommended for high-throughput).
- **Why It Saves Money:** Prevents VM memory starvation that causes upload retries and duplicated bandwidth fees.
- **Detailed Implementation Steps:**
  1. Set VM RAM = 16 GB in VMware vSphere.
- **Estimated Savings:** Performance stability optimization.
- **Risk Level:** Zero.
- **Implementation Scope:** VMware Admin
- **Prerequisites:** Local RAM capacity.

#### 12. Decommission Abandoned Virtual Tapes
- **What:** Identify and delete blank or obsolete virtual tapes (`aws storagegateway delete-tape`) that have not been written to in > 1 year.
- **Why It Saves Money:** Reclaims $0.023/GB-mo tape storage fees.
- **Detailed Implementation Steps:**
  1. Delete unused tapes via CLI.
- **Estimated Savings:** Reclaims 100% of abandoned tape storage.
- **Risk Level:** Medium.
- **Implementation Scope:** Backup Administrator
- **Prerequisites:** Tape catalog audit.

#### 13. Enable Bandwidth Rate Limits During Business Hours
- **What:** Configure Storage Gateway Bandwidth Rate Limits to throttle upload bandwidth during peak office hours.
- **Why It Saves Money:** Prevents gateway uploads from saturating corporate WAN links.
- **Detailed Implementation Steps:**
  1. Set upload bandwidth limit in Storage Gateway console settings.
- **Estimated Savings:** WAN bandwidth stability optimization.
- **Risk Level:** Zero.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Gateway network settings access.

#### 14. Standardize Storage Gateway Software Updates
- **What:** Configure automatic software maintenance update windows for Storage Gateway VMs.
- **Why It Saves Money:** Ensures gateway VM uses latest performance fixes and cache optimization algorithms.
- **Detailed Implementation Steps:**
  1. Set maintenance window time in Gateway settings.
- **Estimated Savings:** Software stability.
- **Risk Level:** Zero.
- **Implementation Scope:** Systems Engineer
- **Prerequisites:** Maintenance schedule window.

#### 15. Enforce IAM Least Privilege Gateway Policies
- **What:** Restrict `storagegateway:AddCache` and `storagegateway:DeleteGateway` permissions via IAM.
- **Why It Saves Money:** Security governance best practice.
- **Detailed Implementation Steps:**
  1. Restrict IAM policies.
- **Estimated Savings:** Security risk mitigation.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** IAM policy review.

#### 16. Audit Gateway High Availability (HA) Cluster Setup
- **What:** Deploy Storage Gateway in VMware HA cluster configurations.
- **Why It Saves Money:** Ensures continuous gateway uptime without deploying duplicate passive gateway hardware ($125/mo write cap saved).
- **Detailed Implementation Steps:**
  1. Enable VMware vSphere HA for Storage Gateway VM.
- **Estimated Savings:** Avoids duplicate gateway appliance licensing.
- **Risk Level:** Zero.
- **Implementation Scope:** VMware Admin
- **Prerequisites:** VMware HA cluster configured.

---

## Cross-Service Synergies

```
[ On-Premises Applications ] 
        │
        ├──(Local Cache Disk)─> [ Right-Size SSD Cache (>95% Hit Ratio) ] (Eliminates $0.09/GB internet egress)
        │
        ├──(Gateway Scope)────> [ Consolidate Gateways ($125/mo Cap) ] (Caps write processing fees per site)
        │
        └──(Tape Archiving)───> [ Glacier Deep Archive ($0.00099/GB-mo) ] (Saves 95.7% on Tape Gateway storage)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `GatewayStorage`, `VolumeUsage`, `TapeUsage`, `DataWrite-Bytes`.
- `line_item_resource_id`: Storage Gateway ARN (`arn:aws:storagegateway:us-east-1:123456789012:gateway/sgw-12345678`).

### B. CloudWatch Metrics
- `AWS/StorageGateway` Namespace: `CacheHitPercent`, `CachePercentDirty`, `BytesUploadedToAWS`, `BytesDownloadedFromAWS`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "SGW-CCH-001",
  "service": "Storage Gateway",
  "category": "Cache Disks & Egress Optimization",
  "resource_id": "arn:aws:storagegateway:us-east-1:123456789012:gateway/sgw-12345678",
  "resource_name": "onprem-file-gateway-01",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "local_cache_disk_gb": 150,
    "cache_hit_ratio_percent": 68.4,
    "monthly_cache_miss_egress_tb": 12.0,
    "monthly_egress_cost_usd": 1080.00
  },
  "recommended_config": {
    "local_cache_disk_gb": 600,
    "projected_cache_hit_ratio_percent": 97.5,
    "projected_monthly_egress_cost_usd": 27.00
  },
  "financial_impact": {
    "monthly_savings_usd": 1053.00,
    "annual_savings_usd": 12636.00,
    "savings_percentage": 97.5
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Expanding local SSD cache disk eliminates cache misses, serving files locally while cutting internet egress charges by 97%."
  },
  "implementation": {
    "scope": "Storage Administrator / VMware Admin",
    "effort_estimate": "30 minutes",
    "automation_eligible": false
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Cache Disk Expansion (>95% Hit)**   | 8 | $8,500.00 | $8,287.50 | 97.5% | Zero |
| **Gateway Consolidation ($125 Cap)**  | 5 | $3,125.00 | $2,500.00 | 80.0% | Zero |
| **Tape Gateway Deep Archive**        | 10 | $11,500.00 | $11,005.50 | 95.7% | Zero |
| **On-Premises Hypervisor Migration**  | 6 | $840.96 | $840.96 | 100.0% | Zero |
| **Total** | **29** | **$23,965.96** | **$22,633.96** | **94.4%** | -- |
