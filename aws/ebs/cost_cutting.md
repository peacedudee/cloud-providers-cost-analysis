# Cost-Cutting Playbook: Amazon EBS

> **Companion File:** [ebs.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/ebs/ebs.md)
> **Last Updated:** July 2026

---

## Executive Summary

Amazon EBS (Elastic Block Store) charges for **provisioned capacity**, not consumed bytes — making it one of the most leak-prone services in any AWS estate. Volumes attached to stopped instances, unattached volumes left behind after instance termination, forgotten Fast Snapshot Restore (FSR) charges at $0.75/hr/AZ, and thousands of legacy `gp2` volumes represent the most common and highest-impact waste categories.

This playbook contains **20 strategies** organised into six categories: Waste Elimination, Rightsizing, Storage & Data Tiering, Architecture Changes, Scheduling & Auto-Scaling, and Pricing Model Optimization. Collectively, applying these strategies across a mid-to-large AWS estate can yield **25–55 % aggregate EBS spend reduction**, with several individual strategies (gp2→gp3 migration, FSR disable, unattached volume cleanup) delivering immediate, risk-free savings.

**Key pricing anchors (us-east-1):**

| Dimension | Rate |
|---|---|
| gp3 storage | $0.08 /GB-month |
| gp2 storage | $0.10 /GB-month (25 % premium over gp3) |
| io2 storage | $0.125 /GB-month |
| io2 IOPS | $0.065 /IOPS-month |
| gp3 extra IOPS (above 3,000) | $0.005 /IOPS-month |
| gp3 extra throughput (above 125 MB/s) | $0.04 /MB/s-month |
| Snapshot (standard) | $0.05 /GB-month |
| Snapshot (archive) | $0.0125 /GB-month (75 % cheaper) |
| Fast Snapshot Restore | $0.75 /hr per snapshot per AZ |

---

## Strategy Categories

### 1. Waste Elimination

#### 1. Delete Unattached EBS Volumes

- **What:** Identify and delete EBS volumes in the `available` state (not attached to any EC2 instance). These accumulate when instances are terminated without the `DeleteOnTermination` flag set to `true`, or when volumes are manually detached and forgotten.
- **Why It Saves Money:** An unattached 500 GB gp3 volume costs `500 × $0.08 = $40.00/month` indefinitely with zero utility. Across hundreds of dev/test accounts, unattached volumes commonly represent 5–15 % of total EBS spend.
- **Implementation Steps:**
  1. Query AWS Config or use the CLI: `aws ec2 describe-volumes --filters Name=status,Values=available`.
  2. Cross-reference each volume's `CreateTime` and last `DetachTime` (from CloudTrail) to confirm the volume is genuinely orphaned.
  3. Take a final protective snapshot before deletion: `aws ec2 create-snapshot --volume-id vol-xxx`.
  4. Delete the volume: `aws ec2 delete-volume --volume-id vol-xxx`.
  5. Deploy an AWS Config rule (`ec2-volume-inuse-check`) or a scheduled Lambda to continuously detect and alert on new unattached volumes.
- **Estimated Savings:** 5–15 % of total EBS storage spend
- **Risk Level:** Low (snapshot taken before deletion provides rollback)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudTrail enabled for EC2 API events; established tagging policy to identify volume owners

#### 2. Remove Orphaned Snapshots

- **What:** Delete EBS snapshots that are no longer associated with any active volume, AMI, or backup retention policy. Snapshots accumulate over time as volumes are deleted but their snapshots are not.
- **Why It Saves Money:** Snapshot storage costs $0.05/GB-month. A 200 GB snapshot abandoned for 12 months costs `200 × $0.05 × 12 = $120`. Across an estate with thousands of orphaned snapshots, this quickly becomes material.
- **Implementation Steps:**
  1. List all snapshots owned by the account: `aws ec2 describe-snapshots --owner-ids self`.
  2. Cross-reference each snapshot against active volumes (`VolumeId` field) and registered AMIs (`aws ec2 describe-images`).
  3. Tag snapshots that are neither volume-backed nor AMI-backed as `orphan-candidate`.
  4. Apply a 14-day grace period (tag-based) before deletion to allow teams to claim ownership.
  5. Delete unclaimed snapshots: `aws ec2 delete-snapshot --snapshot-id snap-xxx`.
  6. Automate with a recurring Lambda or AWS Backup cleanup policy.
- **Estimated Savings:** 3–10 % of EBS snapshot spend
- **Risk Level:** Medium (risk of deleting a snapshot needed for DR; mitigated by grace period and AMI cross-check)
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Snapshot tagging governance; AMI inventory; defined retention policies

#### 3. Disable Unused Fast Snapshot Restore (FSR)

- **What:** Audit and disable Fast Snapshot Restore on snapshots where it is no longer actively required. FSR is typically enabled during migrations or ASG launch events and then forgotten.
- **Why It Saves Money:** FSR is billed at **$0.75 per snapshot per AZ per hour**. A single snapshot with FSR enabled across 3 AZs for a full month costs:
  `1 × 3 AZs × 730 hrs × $0.75 = $1,642.50/month`.
  Five forgotten FSR-enabled snapshots across 3 AZs = **$8,212.50/month** — often exceeding the storage costs of the volumes themselves.
- **Implementation Steps:**
  1. List FSR-enabled snapshots: `aws ec2 describe-fast-snapshot-restores`.
  2. Correlate against CUR 2.0 `line_item_usage_type = EBS:FastSnapshotRestore` to quantify monthly spend.
  3. For each FSR entry, verify with the owning team whether it is still needed for active ASG launches or DR.
  4. Disable unnecessary FSR: `aws ec2 disable-fast-snapshot-restore --source-snapshot-id snap-xxx --availability-zones us-east-1a us-east-1b us-east-1c`.
  5. Set a CloudWatch alarm on `EBS:FastSnapshotRestore` CUR line items exceeding a threshold.
- **Estimated Savings:** $500–$10,000+ per month per account (highly variable; potentially enormous)
- **Risk Level:** Low (FSR affects only initial volume hydration latency, not runtime performance)
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** CUR 2.0 data access; snapshot ownership tagging

#### 4. Reclaim Volumes on Long-Stopped Instances

- **What:** Identify EC2 instances that have been in a `stopped` state for an extended period (e.g., >30 days) and snapshot-then-delete their attached EBS volumes.
- **Why It Saves Money:** Stopping an EC2 instance pauses compute billing, but **EBS storage charges continue at the full provisioned rate**. A stopped instance with 1 TB of gp3 storage burns `1000 × $0.08 = $80.00/month` silently. Replacing the volume with a snapshot costs only `$0.05/GB-month` — a **37.5 % reduction** — and the volume can be restored from the snapshot when needed.
- **Implementation Steps:**
  1. Query stopped instances: `aws ec2 describe-instances --filters Name=instance-state-name,Values=stopped`.
  2. Identify instances stopped for >30 days by comparing `StateTransitionReason` timestamps.
  3. For each attached volume, create a snapshot and tag it with the original instance ID and volume metadata.
  4. Detach and delete the volume.
  5. Document the restoration procedure (create volume from snapshot, attach to instance, start).
  6. Automate with an AWS Config rule + Lambda remediation.
- **Estimated Savings:** 5–10 % of total EBS storage spend (highly dependent on stopped-instance prevalence)
- **Risk Level:** Medium (requires clear restoration runbook; slight data-access latency on first restore from snapshot)
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Stakeholder agreement on the stopped-instance threshold (e.g., 30 days); snapshot restoration SOP

---

### 2. Rightsizing

#### 5. Right-Size Volume Capacity (GB)

- **What:** Reduce the provisioned size of over-allocated EBS volumes. Because EBS bills for provisioned — not consumed — GB, a 500 GB volume with 50 GB of actual data is 10× over-provisioned.
- **Why It Saves Money:** Shrinking a gp3 volume from 500 GB to 100 GB saves `400 × $0.08 = $32.00/month` per volume. Across an estate of hundreds of volumes, this compounds rapidly.
- **Implementation Steps:**
  1. Use CloudWatch `VolumeReadBytes` + `VolumeWriteBytes` and OS-level `df -h` data to measure actual disk utilisation.
  2. Flag volumes at <30 % utilisation as rightsizing candidates.
  3. Since EBS does not support in-place shrink, create a new smaller volume, copy data (`dd`, `rsync`, or filesystem-level tools), swap attachment, and delete the old volume.
  4. Update IaC templates (Terraform/CloudFormation) to reflect the new volume size.
  5. Schedule right-sizing reviews quarterly.
- **Estimated Savings:** 10–30 % of storage costs on targeted volumes
- **Risk Level:** High (requires data migration and downtime or live swap; risk of under-provisioning)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** OS-level disk utilisation monitoring; maintenance window; IaC update process

#### 6. Right-Size Provisioned IOPS on gp3

- **What:** Reduce provisioned IOPS on gp3 volumes that have been over-provisioned beyond their actual workload demands. gp3 includes 3,000 IOPS free; additional IOPS cost $0.005/IOPS-month.
- **Why It Saves Money:** A gp3 volume provisioned at 10,000 IOPS pays for 7,000 extra IOPS = `7,000 × $0.005 = $35.00/month`. If CloudWatch shows actual peak IOPS never exceeds 3,000, the entire $35.00 is waste, recoverable with a single API call.
- **Implementation Steps:**
  1. Query CloudWatch metrics `VolumeReadOps` and `VolumeWriteOps` at 5-minute granularity over the past 30 days.
  2. Calculate peak total IOPS (reads + writes). Add a 20 % headroom buffer.
  3. If (peak IOPS × 1.2) ≤ 3,000, reset provisioned IOPS to the free baseline.
  4. Modify the volume online: `aws ec2 modify-volume --volume-id vol-xxx --iops 3000`.
  5. Monitor for 7 days post-change to confirm no performance degradation.
- **Estimated Savings:** 10–40 % of IOPS charges on targeted gp3 volumes
- **Risk Level:** Low (gp3 IOPS can be increased again online without downtime)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch metrics enabled; 30-day baseline data

#### 7. Right-Size Provisioned Throughput on gp3

- **What:** Reduce provisioned throughput on gp3 volumes where actual throughput usage stays below the free 125 MB/s baseline. Extra throughput costs $0.04/MB/s-month.
- **Why It Saves Money:** A gp3 volume provisioned at 500 MB/s throughput pays for 375 MB/s extra = `375 × $0.04 = $15.00/month`. If `VolumeReadBytes` + `VolumeWriteBytes` converted to MB/s shows peak throughput under 125 MB/s, the full $15.00 is recoverable.
- **Implementation Steps:**
  1. Query CloudWatch `VolumeReadBytes` and `VolumeWriteBytes` at 5-minute granularity.
  2. Convert to MB/s: `(sum_bytes / period_seconds) / 1,048,576`.
  3. Calculate peak throughput over the past 30 days with a 20 % buffer.
  4. If (peak throughput × 1.2) ≤ 125 MB/s, reset provisioned throughput to the free baseline.
  5. Modify online: `aws ec2 modify-volume --volume-id vol-xxx --throughput 125`.
- **Estimated Savings:** 10–30 % of throughput charges on targeted volumes
- **Risk Level:** Low (online modification, instantly reversible)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch metrics enabled; 30-day baseline data

#### 8. Downgrade io2 to gp3 Where Possible

- **What:** Audit io2/io2 Block Express volumes and determine if workloads can be served by gp3 with provisioned IOPS/throughput at a fraction of the cost.
- **Why It Saves Money:** io2 storage costs $0.125/GB-month vs. gp3 at $0.08/GB-month (36 % cheaper). io2 IOPS cost $0.065/IOPS-month vs. gp3 at $0.005/IOPS-month (92 % cheaper). A 500 GB io2 volume with 10,000 IOPS costs:
  `(500 × $0.125) + (10,000 × $0.065) = $62.50 + $650 = $712.50/month`.
  The same workload on gp3 with 10,000 IOPS:
  `(500 × $0.08) + (7,000 × $0.005) = $40.00 + $35.00 = $75.00/month`.
  **Savings: $637.50/month per volume (89 %).**
- **Implementation Steps:**
  1. Identify all io2 volumes: filter CUR by `product_volume_api_name = io2`.
  2. Check if the workload requires >16,000 IOPS (gp3 max) or multi-attach (io2-only feature).
  3. For eligible volumes, plan a migration window and create a gp3 volume from a snapshot.
  4. Test workload performance on gp3 in a staging environment.
  5. Execute the cutover and delete the old io2 volume.
- **Estimated Savings:** 50–90 % per migrated io2 volume
- **Risk Level:** Medium (requires performance validation; gp3 cannot exceed 16,000 IOPS or support multi-attach)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload IOPS/throughput profiling; confirmation that multi-attach is not required; gp3 max IOPS (16,000) and throughput (1,000 MB/s) limits are sufficient

---

### 3. Storage & Data Tiering

#### 9. Migrate All gp2 Volumes to gp3

- **What:** Convert every remaining `gp2` volume in the fleet to `gp3`. This is the single highest-ROI, lowest-risk EBS optimisation available.
- **Why It Saves Money:** gp3 storage is **$0.08/GB-month vs. gp2 at $0.10/GB-month — a straight 20 % price reduction**. Additionally, gp3 provides 3,000 IOPS and 125 MB/s throughput free, while gp2 ties performance to volume size (3 IOPS/GB). To achieve 3,000 IOPS on gp2, you must provision a 1,000 GB volume at $100/month. On gp3, a 10 GB volume with the same 3,000 IOPS costs $0.80/month.
- **Implementation Steps:**
  1. Inventory all gp2 volumes: `aws ec2 describe-volumes --filters Name=volume-type,Values=gp2`.
  2. Validate that no application relies on gp2 burst credits (`BurstBalance` metric) as a performance strategy.
  3. Modify volumes in-place (no downtime, online operation): `aws ec2 modify-volume --volume-id vol-xxx --volume-type gp3`.
  4. Use AWS Systems Manager Run Command to batch-modify across accounts and regions.
  5. Update all IaC templates (Terraform `aws_ebs_volume` resource, CloudFormation, CDK) to default to `gp3`.
  6. Set an SCP or AWS Config rule to prevent future `gp2` volume creation.
- **Estimated Savings:** 20 % direct reduction on all migrated volumes
- **Risk Level:** Low (online, non-disruptive operation; AWS guarantees no data loss or downtime)
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Audit of gp2 burst credit dependency; IaC template updates

#### 10. Archive Cold Snapshots

- **What:** Move infrequently accessed EBS snapshots to the **Snapshot Archive** tier, which costs $0.0125/GB-month vs. $0.05/GB-month for standard snapshots — a 75 % cost reduction.
- **Why It Saves Money:** A 500 GB snapshot stored for 12 months at standard rates costs `500 × $0.05 × 12 = $300`. Archived, it costs `500 × $0.0125 × 12 = $75`. Savings = **$225 per snapshot per year**. Retrieval costs $0.03/GB, so a single retrieval costs `500 × $0.03 = $15` — breakeven after less than 1 month of archive.
- **Implementation Steps:**
  1. Identify snapshots older than 90 days that are not referenced by any active AMI or recent restore.
  2. Archive eligible snapshots: `aws ec2 modify-snapshot-tier --snapshot-id snap-xxx --storage-tier archive`.
  3. Note the minimum archive duration of 90 days (early deletion incurs pro-rated charges).
  4. Tag archived snapshots with `tier=archive` and expected retention date.
  5. Configure AWS Backup lifecycle rules to auto-archive snapshots after the defined cold period.
- **Estimated Savings:** 50–75 % of snapshot storage costs for eligible snapshots
- **Risk Level:** Low (data is preserved; retrieval takes 24–72 hours)
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Defined snapshot retention policy; understanding of 90-day minimum archive period and 24–72 hr retrieval latency

#### 11. Tier Cold Data to sc1 or st1

- **What:** Move infrequently accessed block data (log archives, older database backups, staging data) from gp3 to `sc1` ($0.015/GB-month) or `st1` ($0.045/GB-month) volumes.
- **Why It Saves Money:** sc1 is **81 % cheaper** than gp3 ($0.015 vs. $0.08). For a 2 TB cold data volume, monthly cost drops from `2000 × $0.08 = $160` to `2000 × $0.015 = $30`. Annual savings: **$1,560 per volume**.
- **Implementation Steps:**
  1. Identify volumes with consistently low IOPS (CloudWatch `VolumeReadOps` + `VolumeWriteOps` < 100/day) and sequential access patterns.
  2. Create a new sc1/st1 volume, copy data, attach, and swap.
  3. Validate that workload latency tolerances accept HDD characteristics (higher latency, lower IOPS).
  4. Delete the old gp3 volume.
- **Estimated Savings:** 40–80 % of storage costs per tiered volume
- **Risk Level:** Medium (HDD performance characteristics; not suitable for random I/O workloads)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload I/O profile validation; confirmation that the application is sequential-read dominated

#### 12. Offload Object Data to S3

- **What:** Identify files stored on EBS that are actually objects (logs, media, exports, backups) and migrate them to Amazon S3, where pricing starts at $0.023/GB-month (S3 Standard) or $0.00099/GB-month (S3 Glacier Deep Archive).
- **Why It Saves Money:** EBS gp3 costs $0.08/GB-month. S3 Standard costs $0.023/GB-month (71 % cheaper). S3 Glacier Deep Archive costs $0.00099/GB-month (99 % cheaper). For 1 TB of log files on EBS: `1000 × $0.08 = $80/month`. On S3 Standard: `1000 × $0.023 = $23/month`. Savings: **$57/month per TB**.
- **Implementation Steps:**
  1. Audit EBS volumes for object-like data: `/var/log/*`, media uploads, database dumps, export files.
  2. Implement application-level changes to write directly to S3 (AWS SDK, S3 FUSE mount, or rsync-to-S3 cron).
  3. Migrate existing data using `aws s3 sync`.
  4. Shrink or delete the source EBS volume.
  5. Apply S3 Lifecycle policies (Standard → IA → Glacier) for further tiering.
- **Estimated Savings:** 50–99 % on migrated data depending on S3 tier
- **Risk Level:** Medium (requires application changes; S3 has different access semantics than POSIX block storage)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Identification of object-suitable data; application refactoring capability

---

### 4. Architecture Changes

#### 13. Use Instance Store for Ephemeral/Temp Data

- **What:** Leverage EC2 instance store (locally attached NVMe SSDs) for ephemeral workloads — scratch data, temp files, caches, shuffle space — instead of provisioning EBS volumes.
- **Why It Saves Money:** Instance store volumes are **included in the EC2 instance price at no additional charge**. An `i3.large` instance includes 475 GB NVMe SSD. A comparable 500 GB gp3 EBS volume would cost `500 × $0.08 = $40/month` on top of the instance price.
- **Implementation Steps:**
  1. Identify workloads writing to EBS that produce only temporary, non-durable data (Spark shuffle, ETL scratch, Redis cache, build artifacts).
  2. Select EC2 instance types with instance store (c5d, m5d, i3, r5d families).
  3. Mount instance store volumes at application temp directories.
  4. Remove the corresponding EBS volumes from the launch template.
  5. Ensure application handles instance store volatility (data lost on stop/terminate).
- **Estimated Savings:** 100 % of EBS costs for ephemeral data volumes
- **Risk Level:** Medium (data is ephemeral — lost on stop/terminate; not suitable for persistent data)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload classified as ephemeral/non-durable; instance type supports instance store; application tolerates data loss on stop

#### 14. Replace Shared EBS with Amazon EFS

- **What:** If multiple EC2 instances mount the same dataset via EBS snapshots-and-clones or rsync-based replication, consolidate onto Amazon EFS (Elastic File System), which supports concurrent multi-instance access via NFS.
- **Why It Saves Money:** Instead of maintaining N copies of the same data across N EBS volumes (N × $0.08/GB-month), a single EFS Infrequent Access (IA) volume costs **$0.016/GB-month** (80 % cheaper than gp3). Even EFS Standard at $0.30/GB-month can be cheaper when eliminating 4+ duplicate EBS volumes.
- **Implementation Steps:**
  1. Identify workloads where multiple instances read the same data (shared config, ML model files, CMS assets).
  2. Create an EFS filesystem with lifecycle management (transition to IA after 30 days).
  3. Mount EFS on all instances via NFS.
  4. Migrate data from EBS to EFS: `rsync -avz /ebs-mount/ /efs-mount/`.
  5. Delete redundant EBS volumes.
- **Estimated Savings:** 60–80 % when replacing multiple duplicate EBS volumes with a single EFS IA filesystem
- **Risk Level:** Medium (EFS has higher latency than EBS for random I/O; throughput is provisioned or bursting)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload tolerates NFS latency; security group configuration for NFS (port 2049)

#### 15. Consolidate Low-Usage Volumes

- **What:** Merge multiple small, under-utilised EBS volumes attached to the same instance into a single, appropriately sized volume using LVM (Logical Volume Manager) or filesystem-level consolidation.
- **Why It Saves Money:** Three 100 GB gp3 volumes at 10 % utilisation each cost `300 × $0.08 = $24/month` and provision 9,000 baseline IOPS (3 × 3,000). A single 50 GB gp3 volume with the same data costs `50 × $0.08 = $4/month` — **83 % savings**.
- **Implementation Steps:**
  1. Identify instances with 3+ attached EBS volumes where aggregate utilisation is below 50 %.
  2. Create a single target volume sized to actual utilisation + 30 % buffer.
  3. Use `rsync` or `dd` to migrate data during a maintenance window.
  4. Update `/etc/fstab` and application configurations.
  5. Delete the old volumes after a 7-day validation period.
- **Estimated Savings:** 30–70 % on consolidated volumes
- **Risk Level:** High (requires downtime or careful live migration; risk of data loss during consolidation)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Maintenance window approval; tested backup/restore process; IaC update

---

### 5. Scheduling & Auto-Scaling

#### 16. Automate Snapshot Lifecycle with DLM

- **What:** Use Amazon Data Lifecycle Manager (DLM) to automate EBS snapshot creation and expiration, replacing ad-hoc manual snapshot practices that lead to unbounded snapshot accumulation.
- **Why It Saves Money:** Without automated lifecycle policies, snapshots accumulate indefinitely. DLM enforces retention (e.g., "keep last 7 daily, 4 weekly, 12 monthly") and auto-deletes expired snapshots. For an estate producing 50 GB of incremental snapshots per day at $0.05/GB-month, unbounded accumulation costs `50 × 365 × $0.05 / 2 = $456/year` (average). With 30-day retention: `50 × 30 × $0.05 = $75/year`.
- **Implementation Steps:**
  1. Define retention policies per workload tier (prod: 90 days, staging: 14 days, dev: 7 days).
  2. Create DLM lifecycle policies targeting volumes by tag (e.g., `backup=true`).
  3. Enable cross-region copy within DLM if DR snapshots are needed (avoid manual duplication).
  4. Verify DLM policy execution in the DLM console and set SNS alerts for failures.
  5. Audit and delete legacy pre-DLM snapshots that fall outside the new retention window.
- **Estimated Savings:** 30–60 % of snapshot storage costs
- **Risk Level:** Low (DLM is a managed service with configurable retention; does not affect volume data)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Volume tagging strategy; defined retention policies per environment

#### 17. Scheduled Volume Cleanup Automation

- **What:** Deploy a scheduled Lambda function (or Step Function) that runs daily/weekly to detect and remediate EBS cost waste: unattached volumes, FSR left enabled, gp2 volumes, and over-provisioned IOPS.
- **Why It Saves Money:** Manual audits are infrequent and incomplete. Automated, continuous enforcement catches waste within 24 hours of creation rather than months. Prevents the accumulation patterns that drive strategies 1–4.
- **Implementation Steps:**
  1. Write a Lambda function that:
     - Lists unattached volumes → alerts or auto-snapshots + deletes after N days.
     - Lists FSR-enabled snapshots → alerts the owning team.
     - Lists gp2 volumes → auto-converts to gp3 (or alerts).
     - Lists volumes with provisioned IOPS > CloudWatch peak × 1.3 → alerts.
  2. Schedule via EventBridge (daily at 02:00 UTC).
  3. Output findings to an SNS topic and/or write to a DynamoDB findings table.
  4. Integrate with Slack/PagerDuty for real-time visibility.
- **Estimated Savings:** Indirect — prevents 5–15 % waste accumulation over time
- **Risk Level:** Low (detection mode) / Medium (if auto-remediation is enabled)
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Lambda execution role with EC2 read/modify permissions; SNS topic; tagging governance

#### 18. Dev/Test Environment Volume Scheduling

- **What:** Automatically stop dev/test EC2 instances (and snapshot-then-delete their EBS volumes) outside business hours and on weekends, then restore from snapshots at the start of the next business day.
- **Why It Saves Money:** Dev/test instances typically run only 10 hours/day, 5 days/week (50 hrs out of 168 hrs/week = 30 % utilisation). Deleting EBS volumes during off-hours and restoring from snapshots saves 70 % of the EBS cost. Snapshots cost $0.05/GB-month vs. $0.08/GB-month for gp3 volumes.
- **Implementation Steps:**
  1. Tag dev/test instances with `schedule=business-hours`.
  2. Use AWS Instance Scheduler or a custom Lambda to stop instances and snapshot+delete volumes at 7 PM.
  3. At 7 AM, restore volumes from snapshots, attach to instances, and start instances.
  4. Exempt production and staging environments.
  5. Provide a manual override mechanism (e.g., a Slack command to skip tonight's shutdown).
- **Estimated Savings:** 50–70 % of dev/test EBS spend
- **Risk Level:** Medium (requires reliable restore automation; risk of data loss if snapshot fails silently)
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Robust snapshot/restore automation; environment tagging; stakeholder buy-in for dev/test interruption

---

### 6. Pricing Model Optimization

#### 19. Enforce DeleteOnTermination Defaults

- **What:** Ensure all EBS volumes in launch templates, AMIs, and IaC definitions have `DeleteOnTermination = true` set for non-persistent volumes. This prevents orphaned volume accumulation at the infrastructure level.
- **Why It Saves Money:** Every instance termination that leaves behind a root volume creates a new unattached volume billing at full provisioned rate. For a fleet of 500 instances cycling monthly, with 50 GB root volumes: `500 × 50 × $0.08 = $2,000/month` in orphaned volume costs if `DeleteOnTermination` is `false`.
- **Implementation Steps:**
  1. Audit existing launch templates and AMI block device mappings for `DeleteOnTermination` settings.
  2. Update all launch templates: set `DeleteOnTermination = true` for root and ephemeral volumes.
  3. For running instances, modify the attribute: `aws ec2 modify-instance-attribute --instance-id i-xxx --block-device-mappings "[{\"DeviceName\":\"/dev/xvda\",\"Ebs\":{\"DeleteOnTermination\":true}}]"`.
  4. Set an AWS Config rule to detect and alert on volumes lacking this flag.
  5. Add to IaC linting/policy (e.g., `tflint`, OPA/Rego policies, cfn-guard).
- **Estimated Savings:** Prevents 5–15 % ongoing waste accumulation
- **Risk Level:** Low (only affects behavior at termination time; no impact on running workloads)
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** IaC template review; policy-as-code tooling

#### 20. Leverage AWS Savings Plans & EDP for Predictable EBS Spend

- **What:** For organisations with large, predictable EBS footprints (multi-PB estates), negotiate EBS-inclusive Enterprise Discount Programs (EDPs) or leverage Compute Savings Plans that indirectly reduce the total AWS bill, freeing budget for EBS.
- **Why It Saves Money:** EDPs typically offer 5–15 % discounts on committed annual AWS spend, applied across all services including EBS. For an organisation spending $500K/year on EBS, a 10 % EDP discount saves **$50,000/year**.
- **Implementation Steps:**
  1. Quantify total annual EBS spend from CUR 2.0 (filter by `product_product_name = Amazon Elastic Compute Cloud` and `line_item_usage_type LIKE '%EBS%'`).
  2. Project 12-month EBS growth using historical trends.
  3. Include EBS spend in EDP/PPA negotiations with AWS account team.
  4. Evaluate Compute Savings Plans coverage to ensure EC2 + EBS costs are holistically optimised.
  5. Re-evaluate annually at EDP renewal.
- **Estimated Savings:** 5–15 % of total EBS spend (via EDP)
- **Risk Level:** Low (financial commitment, no technical risk)
- **Implementation Scope:** Procurement/Leadership | FinOps Team
- **Prerequisites:** Minimum annual AWS spend threshold for EDP eligibility (typically $1M+); procurement authority

---

## Cross-Service Synergies

EBS costs do not exist in isolation. Optimising EBS requires coordination with several dependent and adjacent AWS services:

| Service | Synergy with EBS | Actionable Coordination |
|---|---|---|
| **Amazon EC2** | EBS volumes are attached to EC2 instances. Stopped instances continue billing for EBS. Instance type selection (instance store vs. EBS-only) directly impacts EBS cost. | Coordinate instance rightsizing with volume rightsizing. Enforce `DeleteOnTermination`. Use instance store families (c5d, m5d, i3) for ephemeral workloads. |
| **Amazon EKS** | Kubernetes PersistentVolumeClaims (PVCs) dynamically provision EBS volumes via the CSI driver. Orphaned PVCs from deleted pods/namespaces create unattached volumes. | Audit PVC lifecycle. Set `reclaimPolicy: Delete` on StorageClasses. Default StorageClass to `gp3`. Monitor PVCs with `kubectl get pvc --all-namespaces` cross-referenced against EBS volume IDs. |
| **Amazon RDS** | RDS instances use EBS volumes (gp3, io2) for database storage. RDS automated backups create EBS snapshots. | Right-size RDS storage allocation. Audit RDS automated backup retention (default 7 days; reduce if appropriate). Migrate RDS from io2 to gp3 where performance permits. |
| **AWS Backup** | AWS Backup centralises EBS snapshot management across accounts. Misconfigured backup plans can create excessive snapshot retention. | Audit backup plan retention rules. Enable lifecycle transitions to Snapshot Archive. Delete backup vaults with expired recovery points. |
| **Amazon S3** | EBS snapshots are stored in S3 (managed by AWS). Object data on EBS should be migrated to S3 for cost efficiency. | Migrate log files, media, exports from EBS to S3. Use S3 Lifecycle policies for further tiering. Snapshot Archive tier leverages S3 Glacier infrastructure. |

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0

The Cost and Usage Report is the primary data source for EBS spend analysis. Key columns:

| CUR Column | Purpose | Example Values |
|---|---|---|
| `line_item_usage_type` | Identifies the EBS billing dimension | `EBS:VolumeUsage.gp3`, `EBS:VolumeUsage.gp2`, `EBS:VolumeUsage.io2`, `EBS:SnapshotUsage`, `EBS:FastSnapshotRestore` |
| `line_item_resource_id` | Maps cost to a specific volume or snapshot | `vol-0abcdef1234567890`, `snap-0abcdef1234567890` |
| `product_volume_api_name` | Identifies the EBS volume type | `gp3`, `gp2`, `io2`, `st1`, `sc1` |
| `line_item_unblended_cost` | Actual cost for the line item | `42.50` |
| `line_item_usage_amount` | Quantity consumed (GB-months, IOPS-months, DSU-hours) | `500.00` |
| `line_item_usage_start_date` | Billing period start | `2026-07-01T00:00:00Z` |
| `resource_tags_user_*` | User-defined tags for cost allocation | `resource_tags_user_environment = production` |

**Key CUR queries:**
```sql
-- Total EBS spend by volume type
SELECT product_volume_api_name,
       SUM(line_item_unblended_cost) AS total_cost
FROM cur_table
WHERE line_item_usage_type LIKE 'EBS:Volume%'
GROUP BY product_volume_api_name
ORDER BY total_cost DESC;

-- FSR spend (high-priority alert)
SELECT line_item_resource_id,
       SUM(line_item_unblended_cost) AS fsr_cost
FROM cur_table
WHERE line_item_usage_type = 'EBS:FastSnapshotRestore'
GROUP BY line_item_resource_id
ORDER BY fsr_cost DESC;

-- Snapshot spend by snapshot ID
SELECT line_item_resource_id,
       SUM(line_item_unblended_cost) AS snap_cost,
       SUM(line_item_usage_amount) AS snap_gb_months
FROM cur_table
WHERE line_item_usage_type LIKE 'EBS:Snapshot%'
GROUP BY line_item_resource_id
ORDER BY snap_cost DESC;
```

### B. CloudWatch Metrics

CloudWatch metrics are essential for rightsizing and waste detection:

| Metric | Namespace | Purpose | Rightsizing Signal |
|---|---|---|---|
| `VolumeReadOps` | `AWS/EBS` | Count of read operations | Peak IOPS < provisioned IOPS → over-provisioned |
| `VolumeWriteOps` | `AWS/EBS` | Count of write operations | Combined with ReadOps for total IOPS |
| `VolumeReadBytes` | `AWS/EBS` | Bytes read from volume | Convert to MB/s for throughput rightsizing |
| `VolumeWriteBytes` | `AWS/EBS` | Bytes written to volume | Convert to MB/s for throughput rightsizing |
| `VolumeQueueLength` | `AWS/EBS` | Number of pending I/O requests | Consistently 0 → volume is idle; consider deletion |
| `VolumeThroughputPercentage` | `AWS/EBS` | % of provisioned throughput consumed (io2 only) | <20 % → over-provisioned |
| `BurstBalance` | `AWS/EBS` | Remaining burst credits (gp2 only) | Constant 100 % → never bursting; safe to migrate to gp3 |

**Recommended CloudWatch queries:**
```
-- 30-day peak IOPS for a volume
SELECT MAX(VolumeReadOps + VolumeWriteOps) AS peak_iops
FROM SCHEMA("AWS/EBS", VolumeId)
WHERE VolumeId = 'vol-xxx'
GROUP BY BIN(30d)
```

### C. AWS Config / Trusted Advisor

| Check | Source | Purpose |
|---|---|---|
| `ec2-volume-inuse-check` | AWS Config | Detects unattached EBS volumes |
| `ebs-optimized-instance` | AWS Config | Ensures instances support EBS-optimised throughput |
| Underutilised EBS Volumes | Trusted Advisor | Flags volumes with <1 IOPS/day over 14 days |
| Unassociated Elastic IPs | Trusted Advisor | Indirectly signals terminated instances with possible orphaned volumes |
| Amazon EBS Snapshots | Trusted Advisor | Flags public snapshots (security) and old snapshots (cost) |

### D. Company Policies

| Policy Area | Required Input | Impact on Strategy |
|---|---|---|
| Retention requirements | Minimum snapshot retention (days) | Constrains strategies 2, 10, 16 |
| Environment classification | Prod / Staging / Dev / Sandbox definitions | Determines scheduling aggressiveness (strategy 18) |
| Change management | CAB approval requirements | Affects implementation timelines for strategies 5, 8, 13, 14 |
| Tagging standards | Required tags (owner, environment, cost-centre) | Enables all attribution and alerting strategies |
| Compliance frameworks | HIPAA, SOC2, PCI-DSS, etc. | Constrains snapshot deletion and data tiering options |

### E. IaC (Optional)

| IaC Source | What to Audit | Action |
|---|---|---|
| **Terraform** | `aws_ebs_volume` resources: `type`, `size`, `iops`, `throughput` | Update defaults to gp3; enforce `delete_on_termination` |
| **CloudFormation** | `AWS::EC2::Volume` resources | Same as Terraform |
| **CDK** | `ec2.Volume` constructs | Same as Terraform |
| **Helm Charts (EKS)** | StorageClass definitions: `type`, `iopsPerGB`, `reclaimPolicy` | Default to gp3; set `reclaimPolicy: Delete` |
| **Packer** | AMI volume mappings | Ensure `delete_on_termination: true`; default to gp3 |

---

## Output Schema

### Finding Record (JSON)

Each identified cost-cutting opportunity is recorded as a structured JSON finding:

```json
{
  "findingId": "EBS-001",
  "service": "Amazon EBS",
  "strategy": "Delete Unattached EBS Volumes",
  "strategyCategory": "Waste Elimination",
  "resourceId": "vol-0abcdef1234567890",
  "region": "us-east-1",
  "accountId": "123456789012",
  "currentConfig": {
    "volumeType": "gp3",
    "sizeGb": 500,
    "provisionedIops": 3000,
    "provisionedThroughputMbps": 125,
    "attachmentState": "available",
    "daysSinceLastAttach": 45
  },
  "recommendedAction": "Snapshot and delete unattached volume",
  "estimatedMonthlySavings": 40.00,
  "estimatedAnnualSavings": 480.00,
  "currency": "USD",
  "riskLevel": "Low",
  "implementationScope": "Engineer/DevOps",
  "prerequisites": ["Snapshot taken", "Owner notified"],
  "curLineItemUsageType": "EBS:VolumeUsage.gp3",
  "tags": {
    "environment": "development",
    "owner": "team-alpha",
    "cost-centre": "CC-1234"
  },
  "detectedAt": "2026-07-03T09:00:00Z",
  "status": "open"
}
```

**Additional finding examples:**

```json
{
  "findingId": "EBS-002",
  "service": "Amazon EBS",
  "strategy": "Disable Unused Fast Snapshot Restore",
  "strategyCategory": "Waste Elimination",
  "resourceId": "snap-0abcdef1234567890",
  "region": "us-east-1",
  "accountId": "123456789012",
  "currentConfig": {
    "fsrEnabledAzs": ["us-east-1a", "us-east-1b", "us-east-1c"],
    "fsrHourlyRate": 0.75,
    "daysEnabled": 60
  },
  "recommendedAction": "Disable FSR on all AZs",
  "estimatedMonthlySavings": 1642.50,
  "estimatedAnnualSavings": 19710.00,
  "currency": "USD",
  "riskLevel": "Low",
  "implementationScope": "Engineer/DevOps",
  "prerequisites": ["Confirm FSR not needed for active ASG"],
  "curLineItemUsageType": "EBS:FastSnapshotRestore",
  "tags": {},
  "detectedAt": "2026-07-03T09:00:00Z",
  "status": "open"
}
```

```json
{
  "findingId": "EBS-003",
  "service": "Amazon EBS",
  "strategy": "Migrate gp2 to gp3",
  "strategyCategory": "Storage & Data Tiering",
  "resourceId": "vol-09876543210fedcba",
  "region": "us-west-2",
  "accountId": "123456789012",
  "currentConfig": {
    "volumeType": "gp2",
    "sizeGb": 1000,
    "currentMonthlyCost": 100.00
  },
  "recommendedAction": "Modify volume type from gp2 to gp3",
  "estimatedMonthlySavings": 20.00,
  "estimatedAnnualSavings": 240.00,
  "currency": "USD",
  "riskLevel": "Low",
  "implementationScope": "Engineer/DevOps",
  "prerequisites": ["Verify no burst credit dependency"],
  "curLineItemUsageType": "EBS:VolumeUsage.gp2",
  "tags": {
    "environment": "production",
    "owner": "team-beta"
  },
  "detectedAt": "2026-07-03T09:00:00Z",
  "status": "open"
}
```

### Summary Report Table

| Finding ID | Strategy | Resource ID | Current Monthly Cost | Estimated Monthly Savings | Annual Savings | Risk | Scope |
|---|---|---|---|---|---|---|---|
| EBS-001 | Delete Unattached Volumes | vol-0abc... | $40.00 | $40.00 | $480.00 | Low | Engineer/DevOps |
| EBS-002 | Disable Unused FSR | snap-0abc... | $1,642.50 | $1,642.50 | $19,710.00 | Low | Engineer/DevOps |
| EBS-003 | Migrate gp2 → gp3 | vol-0987... | $100.00 | $20.00 | $240.00 | Low | Engineer/DevOps |
| EBS-004 | Right-Size Provisioned IOPS | vol-1111... | $75.00 | $35.00 | $420.00 | Low | Engineer/DevOps |
| EBS-005 | Archive Cold Snapshots | snap-2222... | $25.00 | $18.75 | $225.00 | Low | FinOps Team |
| EBS-006 | Downgrade io2 → gp3 | vol-3333... | $712.50 | $637.50 | $7,650.00 | Medium | Engineer/DevOps |
| EBS-007 | Reclaim Stopped Instance Vols | vol-4444... | $80.00 | $80.00 | $960.00 | Medium | Engineer/DevOps |
| EBS-008 | Offload Objects to S3 | vol-5555... | $80.00 | $57.00 | $684.00 | Medium | Engineer/DevOps |
| ... | ... | ... | ... | ... | ... | ... | ... |
| **TOTAL** | | | | **$2,530.75** | **$30,369.00** | | |
