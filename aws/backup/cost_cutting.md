# Cost-Cutting Playbook: AWS Backup

> **Companion File:** [backup.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/backup/backup.md)  
> **Last Updated:** July 2026

---

## Executive Summary

AWS Backup provides centralized data protection across EBS, RDS, DynamoDB, EFS, FSx, S3, and Storage Gateway. Billing includes **Backup Storage Capacity** (Warm Storage at $0.05/GB-mo vs Cold Storage at $0.00099–$0.0125/GB-mo), **Restore Fees** ($0.02–$0.03/GB), and **Cross-Region Replication Transfer** ($0.02–$0.04/GB).

Key billing traps include:
1. **Indefinite Warm Storage Retention:** Retaining daily, weekly, and monthly compliance backups in Warm Storage ($0.05/GB-mo) indefinitely rather than transitioning to Cold Storage ($0.00099–$0.01/GB-mo — **80–98% savings**).
2. **Cold Storage 90-Day Early Deletion Penalties:** Moving backups to Cold Storage but configuring retention periods shorter than the mandatory 90-day minimum (e.g. 30 days), triggering pro-rated early deletion penalty fees.
3. **Daily Full Cross-Region Replication:** Replicating multi-terabyte database snapshots across regions daily instead of using incremental replication.

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **40–85% reduction in total AWS Backup spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Automate Lifecycle Transition to Cold Storage:** Transition backups to Cold Storage after 7–14 days for EFS, EBS, and S3, dropping storage fees by **75–98%**.
2. **Align Cold Storage Retention to 90+ Days:** Eliminates AWS Backup early deletion penalty charges on cold vaults.
3. **Exclude Non-Production Environments via Resource Tagging:** Excludes dev/staging resources from expensive production backup plans.

---

## Strategy Categories

### 1. Storage Lifecycle & Tiering Optimization

#### 1. Automate Backup Transition to Cold Storage Tiers
- **What:** Configure AWS Backup Lifecycle Rules in Backup Plans to move backups from **Warm Storage** ($0.05/GB-mo) to **Cold Storage** ($0.0100/GB-mo for EFS, $0.0125/GB-mo for EBS, $0.00099/GB-mo for S3) after 7 to 14 days.
- **Why It Saves Money:**
  - **Warm Storage:** **$0.0500 per GB-month** ($50.00/TB-mo).
  - **Cold Storage (S3/EFS):** **$0.00099 to $0.0100 per GB-month** ($0.99 to $10.00/TB-mo).
  - Transitioning 10 TB of compliance backups to Cold Storage saves **$400.00 to $490.00/month (an 80–98% cost reduction)**!
- **Detailed Implementation Steps:**
  1. Update Backup Plan lifecycle rule in Terraform:
     ```hcl
     resource "aws_backup_plan" "production_plan" {
       name = "prod-backup-plan"
       rule {
         rule_name         = "daily-warm-monthly-cold"
         target_vault_name = aws_backup_vault.prod_vault.name
         schedule          = "cron(0 12 * * ? *)"
         lifecycle {
           cold_storage_after = 14
           delete_after       = 365
         }
       }
     }
     ```
- **Estimated Savings:** **75–98% reduction** in long-term backup storage spend.
- **Risk Level:** Zero risk (backups remain fully retrievable).
- **Implementation Scope:** Backup Administrator / DevOps
- **Prerequisites:** Supported resource type (EBS, EFS, S3, DynamoDB).

#### 2. Align Cold Storage Retention Durations with the 90-Day Minimum Rule
- **What:** Ensure any Backup Plan rule configured with `cold_storage_after` maintains `delete_after` of **at least 90 days** (or 180 days for S3 Glacier Deep Archive equivalent).
- **Why It Saves Money:** AWS Backup enforces a **minimum 90-day billing duration** on Cold Storage. Deleting a cold backup after 30 days triggers an early deletion penalty fee for the remaining 60 days, destroying the cost savings.
- **Detailed Implementation Steps:**
  1. Audit Backup Plan JSON configurations to ensure `delete_after >= cold_storage_after + 90`.
- **Estimated Savings:** Eliminates early deletion penalty fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Backup Administrator
- **Prerequisites:** Backup plan lifecycle audit.

---

### 2. Resource Scope & Tagging Governance

#### 3. Exclude Non-Production Resources Using Tag-Based Selections
- **What:** Configure AWS Backup Resource Assignments using **Tag-Based Selection** (`Environment = Production`) rather than selecting *All Resources*.
- **Why It Saves Money:** Excludes dev, test, and staging EBS volumes, RDS databases, and EFS file systems from expensive daily backup plans and cross-region replication.
- **Detailed Implementation Steps:**
  1. Define tag selection in Terraform:
     ```hcl
     resource "aws_backup_selection" "prod_resources" {
       iam_role_arn = aws_iam_role.backup_role.arn
       name         = "prod-resource-selection"
       plan_id      = aws_backup_plan.production_plan.id
       selection_tag {
         type  = "STRINGEQUALS"
         key   = "Environment"
         value = "Production"
       }
     }
     ```
- **Estimated Savings:** **50–70% reduction** in total account backup vault storage by excluding non-prod environments.
- **Risk Level:** Zero risk.
- **Implementation Scope:** DevOps / FinOps Team
- **Prerequisites:** Consistent resource tagging (`Environment = Production`).

---

### 3. Cross-Region Replication Optimization

#### 4. Optimize Cross-Region Replication Frequency for Disaster Recovery
- **What:** Restrict Cross-Region Backup Vault Replication to **Weekly or Monthly** snapshots rather than daily continuous backups.
- **Why It Saves Money:** Copying multi-terabyte backups across regions incurs inter-region data transfer egress fees (**$0.02 to $0.04 per GB**) plus duplicate storage fees in the secondary region vault.
- **Detailed Implementation Steps:**
  1. Configure separate cross-region copy rules in Backup Plan with `schedule = "cron(0 12 ? * 1 *)"` (weekly).
- **Estimated Savings:** **75–85% reduction** in cross-region replication bandwidth and storage costs.
- **Risk Level:** Low (verify DR RPO compliance SLA).
- **Implementation Scope:** Security / Backup Administrator
- **Prerequisites:** DR SLA review (RPO/RTO).

---

### 4. Continuous Backup & Point-in-Time Recovery (PITR)

#### 5. Right-Size Continuous Backup Retention Windows (RDS & DynamoDB)
- **What:** Reduce Continuous Backup Point-in-Time Recovery (PITR) retention windows from 35 days down to **7 or 14 days** for non-critical databases.
- **Why It Saves Money:** Continuous backups for RDS and DynamoDB bill **$0.021 to $0.10 per GB-month** for transaction logs and change records.
- **Detailed Implementation Steps:**
  1. Update PITR retention period in Backup Plan rule: `enable_continuous_backup = true`, `completion_window = 14`.
- **Estimated Savings:** 50–60% reduction in continuous transaction log storage.
- **Risk Level:** Low.
- **Implementation Scope:** Database Administrator
- **Prerequisites:** Database RPO alignment.

---

### 5. Vault Security & Lock Governance

#### 6. Verify Backup Vault Lock Policy Retentions Before Enabling
- **What:** Review and test **AWS Backup Vault Lock** policies in `Compliance` mode before applying lock controls.
- **Why It Saves Money:** Vault Lock enforces immutable backup retention policies. Once locked in Compliance mode, **no user or AWS root account can delete backups or reduce retention periods**, creating un-alterable storage billing for the entire duration!
- **Detailed Implementation Steps:**
  1. Test Vault Lock in `Governance` mode first (allows IAM override).
- **Estimated Savings:** Protects against immutable billing mistakes.
- **Risk Level:** High risk mitigation.
- **Implementation Scope:** Enterprise Security Architect
- **Prerequisites:** Legal compliance mandate review.

---

### 6. Observability & Governance

#### 7. Audit and Delete Orphaned Backup Vault Snapshots
- **What:** Identify and delete recovery points in backup vaults (`aws backup list-recovery-points-by-backup-vault`) associated with deleted EC2 instances or terminated RDS databases.
- **Why It Saves Money:** Terminating an EC2 instance does NOT delete its historical AWS Backup vault snapshots. Retaining snapshots for deleted instances incurs ongoing storage fees.
- **Detailed Implementation Steps:**
  1. Delete orphaned recovery points via AWS CLI:
     ```bash
     aws backup delete-recovery-point \
       --backup-vault-name prod-vault \
       --recovery-point-arn "arn:aws:ec2:us-east-1::snapshot/snap-12345678"
     ```
- **Estimated Savings:** Reclaims 100% of storage for deleted resources.
- **Risk Level:** Medium (verify snapshot is not needed for historical audit).
- **Implementation Scope:** Backup Administrator
- **Prerequisites:** 30-day resource termination cross-reference audit.

#### 8. Use AWS Backup Audit Manager for Automated Compliance Checks
- **What:** Deploy AWS Backup Audit Manager framework rules to evaluate backup policy compliance automatically.
- **Why It Saves Money:** Reclaims manual auditing time and identifies over-retention policy drift.
- **Detailed Implementation Steps:**
  1. Enable Audit Manager framework in AWS Backup console.
- **Estimated Savings:** Operational governance efficiency.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps / Compliance Team
- **Prerequisites:** AWS Backup Audit Manager setup.

#### 9. Enforce CloudWatch Alarms for Daily Backup Storage Expansion
- **What:** Put CloudWatch alarm on `BackupVaultBytes` (> 20% growth rate).
- **Why It Saves Money:** Instant alert if a resource starts generating unexpected backup volumes.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm targeting `AWS/Backup`.
- **Estimated Savings:** Proactive billing risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 10. Deduplicate S3 Backups using AWS Backup for S3
- **What:** Enable AWS Backup for S3 with single-instance vault policies.
- **Why It Saves Money:** Avoids duplicating standard S3 bucket lifecycle backups.
- **Detailed Implementation Steps:**
  1. Consolidate S3 backup management in AWS Backup.
- **Estimated Savings:** 30–50% S3 backup cost consolidation.
- **Risk Level:** Zero.
- **Implementation Scope:** Storage Administrator
- **Prerequisites:** S3 bucket backup setup.

#### 11. Optimize Backup Windows to Avoid High-Cost IOPS Spikes
- **What:** Stagger Backup Plan execution schedules (`cron(0 1 * * ? *)`, `cron(0 3 * * ? *)`) across resource groups.
- **Why It Saves Money:** Prevents concurrent snapshot creation from driving up EBS/RDS provisioned IOPS overage fees.
- **Detailed Implementation Steps:**
  1. Stagger backup window schedules in Backup Plans.
- **Estimated Savings:** Avoids database IOPS performance throttling and overage fees.
- **Risk Level:** Zero.
- **Implementation Scope:** DevOps Engineer
- **Prerequisites:** Backup plan schedule configuration access.

#### 12. Consolidate Duplicate Regional Backup Vaults
- **What:** Merge multiple regional vaults into 1 central primary vault per region.
- **Why It Saves Money:** Simplifies vault lock policy administration.
- **Detailed Implementation Steps:**
  1. Consolidate backup selection targets.
- **Estimated Savings:** Administrative cleanliness.
- **Risk Level:** Low.
- **Implementation Scope:** Backup Administrator
- **Prerequisites:** Vault policy review.

#### 13. Audit Item-Level Recovery Usage
- **What:** Use full volume restore for large EBS recoveries instead of individual file-level item extraction.
- **Why It Saves Money:** Avoids additional item-level recovery indexing fees.
- **Detailed Implementation Steps:**
  1. Perform volume restores for large data sets.
- **Estimated Savings:** Restore fee optimization.
- **Risk Level:** Zero.
- **Implementation Scope:** Infrastructure Engineer
- **Prerequisites:** None.

#### 14. Standardize IAM Backup Role Least Privilege
- **What:** Restrict `aws-backup` service roles to authorized resource scopes.
- **Why It Saves Money:** Security governance best practice.
- **Detailed Implementation Steps:**
  1. Update IAM role policies.
- **Estimated Savings:** Security risk mitigation.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** IAM policy review.

#### 15. Utilize Legal Hold Controls Cautiously
- **What:** Remove expired Legal Holds (`aws backup remove-legal-hold`) promptly after legal discovery completes.
- **Why It Saves Money:** Legal Holds prevent automated deletion policies, locking storage billing.
- **Detailed Implementation Steps:**
  1. Release legal hold via CLI.
- **Estimated Savings:** Storage billing termination.
- **Risk Level:** Low (verify legal release approval).
- **Implementation Scope:** Legal / Compliance Team
- **Prerequisites:** Legal clearance.

#### 16. Implement Automated Retries for Failed Backup Jobs
- **What:** Configure `completion_window` and retry settings on Backup Plan rules.
- **Why It Saves Money:** Prevents partial backup failures from missing compliance windows.
- **Detailed Implementation Steps:**
  1. Set `start_window = 60`, `completion_window = 120`.
- **Estimated Savings:** Operational reliability optimization.
- **Risk Level:** Zero.
- **Implementation Scope:** Backup Administrator
- **Prerequisites:** Backup plan configuration.

---

## Cross-Service Synergies

```
[ Active AWS Resources (EBS, EFS, S3, RDS) ] 
        │
        ├──(Tag Selection)─────> [ Exclude Non-Prod (Environment=Production) ] (Cuts backup storage 50-70%)
        │
        ├──(Cold Storage)──────> [ Automate Cold Lifecycle (After 14 Days) ] (Saves 75-98% on storage)
        │
        └──(Cold Retention)────> [ Align Retention to 90+ Days ] (Eliminates early deletion penalty fees)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `WarmStorage-ByteHrs`, `ColdStorage-ByteHrs`, `Restore-Bytes`, `DataTransfer-Out-Bytes`.
- `line_item_resource_id`: AWS Backup Vault ARN / Recovery Point ARN (`arn:aws:backup:us-east-1:123456789012:backup-vault:prod-vault`).

### B. CloudWatch & AWS Backup Metrics
- `AWS/Backup` Namespace: `BackupVaultBytes`, `NumberOfBackupJobsCompleted`, `NumberOfBackupJobsFailed`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "BAK-CLD-001",
  "service": "Backup",
  "category": "Storage Lifecycle & Tiering Optimization",
  "resource_id": "arn:aws:backup:us-east-1:123456789012:backup-vault:compliance-vault",
  "resource_name": "compliance-vault",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "warm_storage_tb": 25.0,
    "cold_storage_enabled": false,
    "retention_days": 365,
    "monthly_cost_usd": 1250.00
  },
  "recommended_config": {
    "cold_storage_after_days": 14,
    "projected_warm_tb": 1.0,
    "projected_cold_tb": 24.0,
    "projected_monthly_cost_usd": 290.00
  },
  "financial_impact": {
    "monthly_savings_usd": 960.00,
    "annual_savings_usd": 11520.00,
    "savings_percentage": 76.8
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Transitioning compliance backups to Cold Storage after 14 days maintains 100% data durability and restore capabilities."
  },
  "implementation": {
    "scope": "Backup Administrator / DevOps",
    "effort_estimate": "30 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Cold Storage Lifecycle Automation**| 12 | $18,500.00 | $14,208.00 | 76.8% | Zero |
| **Non-Prod Tag-Based Exclusion** | 18 | $12,400.00 | $7,440.00 | 60.0% | Zero |
| **Cross-Region Replication Tuning**| 8 | $6,200.00 | $4,960.00 | 80.0% | Low |
| **Continuous PITR Window Sizing** | 10 | $4,500.00 | $2,250.00 | 50.0% | Low |
| **Total** | **48** | **$41,600.00** | **$28,858.00** | **69.4%** | -- |
