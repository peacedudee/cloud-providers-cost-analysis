# AWS Service Cost Research: AWS Backup

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Backup is a fully managed, centralized data protection service that automates backup orchestration across AWS services (including EBS, RDS, DynamoDB, EFS, FSx, S3, and EC2) and hybrid workloads. AWS Backup provides centralized compliance policies, vault lock security, and lifecycle management across AWS accounts and regions. Billing is separate from underlying resource storage and introduces specific rate cards for warm storage, cold storage, restores, and cross-region copies.

---

## 2. Billing Mechanics
AWS Backup charges across four primary cost dimensions:
1. **Backup Storage Capacity:** Billed per GB-month for data stored in backup vaults, split into **Warm Storage** (fast access) and **Cold Storage** (archival, with retention minimums).
2. **Restore Fees:** Billed per GB of data restored from backup vaults back into active AWS resources.
3. **Cross-Region Transfer:** Billed per GB for copying backups to secondary AWS regions for disaster recovery.
4. **Item-Level Recovery Fees:** Additional charges for extracting individual files (e.g., from EBS backups or VMware VMs) rather than restoring entire volumes.

---

## 3. Key Cost Dimensions

### A. Backup Storage Tiers (us-east-1)

* **Warm Storage Tier (Instant Restore Access):**
  * *EBS, EFS, FSx, Storage Gateway:* **$0.05 per GB-month**.
  * *Amazon S3 Backups:* **$0.05 per GB-month** (Standard Warm) or **$0.035 per GB-month** (Warm Low Access).
  * *Amazon DynamoDB:* **$0.10 per GB-month**.
  * *Amazon RDS & Aurora:* **$0.021 per GB-month** (applies to backup storage exceeding the 100% free backup allowance included with active databases).

* **Cold Storage Tier (Archival Storage):**
  * *EFS:* **$0.0100 per GB-month** (80% lower cost than Warm).
  * *EBS:* **$0.0125 per GB-month** (75% lower cost than Warm).
  * *DynamoDB:* **$0.0300 per GB-month**.
  * *Amazon S3 Cold:* **$0.00099 per GB-month** (Glacier Deep Archive tier equivalent).

### B. Restore Fees (us-east-1)
* **Warm Restores:** **$0.02 per GB** restored for EFS, S3, and Storage Gateway. (Restores for EBS and RDS from warm storage carry no additional AWS Backup restore fee).
* **Cold Restores:** **$0.03 per GB** restored across all supported resources due to data retrieval overhead from cold vaults.

### C. Cold Storage 90-Day Minimum Retention Rule
* **The Constraint:** Any backup moved to the Cold storage tier carries a **minimum billing duration of 90 days**.
* **Early Deletion Penalty:** If a cold backup is deleted after 15 days, AWS Backup bills for the remaining 75 days at standard cold rates.

### D. Cross-Region Copy Fees
* Copying backups to secondary regions (e.g., `us-east-1` to `us-west-2`) incurs standard inter-region data transfer egress fees (**$0.02 to $0.04 per GB**) plus storage fees in the target region vault.

---

## 4. Detailed Pricing Rates (us-east-1)

| Source Resource Type | Warm Storage Rate (/GB-mo) | Cold Storage Rate (/GB-mo) | Warm Restore Rate (/GB) | Cold Restore Rate (/GB) |
|----------------------|----------------------------|----------------------------|-------------------------|-------------------------|
| **Amazon EBS** | $0.0500 | $0.0125 | Free | $0.0300 |
| **Amazon EFS** | $0.0500 | $0.0100 | $0.0200 | $0.0300 |
| **Amazon DynamoDB** | $0.1000 | $0.0300 | Free | $0.0300 |
| **Amazon RDS / Aurora** | $0.0210 | N/A (Not supported) | Free | N/A |
| **Amazon S3** | $0.0500 | $0.00099 | $0.0200 | $0.0300 |
| **Storage Gateway** | $0.0500 | N/A | $0.0200 | N/A |

---

## 5. AWS Free Tier Coverage
* **AWS Backup:** No free tier available. All backup storage, restores, and data transfer carry immediate usage charges.

---

## 6. Common Cost Hotspots & Pitfalls
* **Indefinite Warm Storage Retention:** Storing daily, weekly, and monthly backups in warm storage ($0.05/GB-mo) indefinitely rather than transitioning compliance backups to cold storage ($0.01/GB-mo).
* **Cold Storage Early Deletion Penalties:** Moving backups to cold storage but setting deletion schedules shorter than 90 days (e.g., 30-day retention), triggering early deletion penalty charges.
* **Daily Full Cross-Region Snapshot Syncs:** Replicating multi-terabyte database snapshots across regions daily instead of replicating incremental backups or logs.

---

## 7. Actionable Cost Optimization Strategies
1. **Automate Transition to Cold Storage:** Configure backup plans so that daily backups expire after 14–30 days, and long-term compliance backups transition to **cold storage** after 7–14 days.
   * **The Savings:** Drops backup storage costs by **up to 80%**.
2. **Align Retention Durations with 90-Day Cold Minimum:** Set retention periods for cold storage backups to **at least 90 days** to eliminate early deletion fees.
3. **Exclude Non-Production Resources:** Create separate backup policies for non-production environments with short retention periods (e.g., 7 days warm, no cold, no cross-region replication) or exclude non-prod resources entirely using resource tags.
4. **Use Backup Vault Lock Cautiously:** Ensure retention periods are exact before applying AWS Backup Vault Lock, as locked policies enforce immutable storage charges for the entire specified duration.
