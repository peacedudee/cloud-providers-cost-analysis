# AWS Service Cost Research: Amazon Security Lake

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Security Lake automatically centralizes security data from AWS environments, on-premises sources, and third-party security vendors into a purpose-built security data lake. It normalizes incoming log data into the Open Cybersecurity Schema Framework (OCSF) standard format, storing data in Amazon S3 for query access via Amazon Athena, Amazon OpenSearch, or third-party SIEM tools. Security Lake is serverless, billing based on the volume of log data ingested and normalized.

---

## 2. Billing Mechanics
Security Lake billing is calculated monthly based on three core components:
1.  **Log Ingestion & OCSF Normalization:** Billed per GB of security data ingested into the data lake.
2.  **S3 Data Storage:** Standard S3 storage rates apply for storing normalized parquet files.
3.  **Analytics & Querying:** Standard query costs apply for services used to investigate security events (e.g. Amazon Athena query fees at $5.00/TB scanned).

---

## 3. Key Cost Dimensions

### A. Data Ingestion Pricing (us-east-1 Tiers)
*   **CloudTrail Management Events:**
    *   *Rate:* **$0.75 per GB** ingested.
*   **Other AWS Security Logs (VPC Flow Logs, DNS Query Logs, GuardDuty, Security Hub, EKS):**
    *   *First 10 TB / month:* **$0.25 per GB** ($250.00 per TB).
    *   *Next 20 TB (10 TB to 30 TB) / month:* **$0.15 per GB** ($150.00 per TB).
    *   *Next 20 TB (30 TB to 50 TB) / month:* **$0.075 per GB**.
    *   *Over 50 TB / month:* **$0.05 per GB**.

### B. Secondary Storage and Query Fees
*   **Amazon S3 Standard Storage:** **$0.023 per GB-month**.
*   **Amazon Athena Queries:** **$5.00 per TB** of data scanned during security investigations.

---

## 4. Detailed Pricing Rates (us-east-1)

| Security Data Source | Usage Tier | Rate per GB | Price per 1 TB Ingested |
|----------------------|------------|-------------|-------------------------|
| **CloudTrail Management**| All volume | **$0.75** | **$750.00** |
| **VPC / DNS / GuardDuty**| 0 - 10 TB | **$0.25** | **$250.00** |
| **VPC / DNS / GuardDuty**| 10 TB - 30 TB | **$0.15** | $150.00 |
| **VPC / DNS / GuardDuty**| 30 TB - 50 TB | **$0.075**| $75.00 |

---

## 5. AWS Free Tier Coverage
*   **Amazon Security Lake 15-Day Free Trial:** 15-day free trial for new accounts, including 10 GB of free log data ingestion per month.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Ingesting Unfiltered VPC Flow Logs from Dev/Test Clusters:** Ingesting high-volume network logs from non-production VPCs into Security Lake ($0.25/GB).
*   **Unpartitioned Athena Security Investigation Queries:** Running wildcard SQL queries (`SELECT * FROM security_lake WHERE ...`) across years of unpartitioned logs in Athena, scanning terabytes of data ($5.00/TB).

---

## 7. Actionable Cost Optimization Strategies
1.  **Filter Ingested Log Sources by Account & Region:**
    *   Configure Security Lake log source settings.
    *   Enable VPC Flow Logs and DNS logs **ONLY for production accounts**.
    *   Keep CloudTrail Management Events enabled across all accounts (cheap, critical audit coverage).
    *   **The Savings:** Cuts Security Lake ingestion fees by **70–80%** by skipping dev/test network traffic.
2.  **Automate S3 Storage Tiering (Lifecycle Rules):**
    *   Set S3 Lifecycle rules on Security Lake S3 buckets.
    *   Move OCSF security logs to **S3 Glacier Instant Retrieval ($0.004/GB-mo)** after 30 days, and **Glacier Deep Archive ($0.00099/GB-mo)** after 90 days.
    *   **The Savings:** Slashes ongoing data lake storage bills by **80–95%**.
3.  **Enforce Partition Pruning in Athena Queries:** Train security analysts to include date and time range filters (`WHERE eventday >= '2026-07-01'`) in Athena queries. Athena reads only the specific daily OCSF partitions, reducing scanned data volume by up to 99%.
