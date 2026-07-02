# AWS Service Cost Research: AWS Database Migration Service (DMS)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Database Migration Service (DMS) helps you migrate databases to AWS quickly and securely with minimal application downtime. DMS supports homogeneous migrations (e.g. Oracle to Oracle) as well as heterogeneous migrations between different database engines (e.g. Oracle to Amazon Aurora PostgreSQL, or SQL Server to MySQL) using continuous Change Data Capture (CDC). DMS offers provisioned replication instances as well as DMS Serverless options.

---

## 2. Billing Mechanics
1.  **Replication Instance Hourly Fee:** Billed hourly per replication instance based on instance size (`dms.t3.micro`, `dms.c5.large`) and deployment mode (Single-AZ vs. Multi-AZ CDC).
2.  **Replication Storage Volume:** Billed per GB of GP3 storage provisioned for swap/logs ($0.115 per GB-month). First 50 GB included free per instance.
3.  **Free Target Migration Incentive:** **100% FREE for 6 months** when migrating to Amazon Aurora, Amazon RDS (MySQL, PostgreSQL, MariaDB), or Amazon Redshift!

---

## 3. Key Cost Dimensions

### A. Provisioned Instance Pricing (us-east-1 Rates)
*   **`dms.t3.micro` (Single-AZ):** **$0.018 per hour** (~$13.14 per month).
*   **`dms.c5.large` (Single-AZ):** **$0.194 per hour** (~$141.62 per month).
*   **`dms.c5.large` (Multi-AZ CDC):** **$0.388 per hour** (**$283.24 per month**).
*   **`dms.r5.xlarge` (Multi-AZ CDC):** **$1.544 per hour** ($1,127.12 per month).

### B. DMS Serverless Pricing
*   **DMS Capacity Units (DCU):** **$0.10 per DCU-hour** (scales dynamically from 1 to 128 DCUs based on migration throughput).

---

## 4. Detailed Pricing Rates (us-east-1)

| Instance Class | Deployment Mode | Hourly Rate | Monthly Base Cost (24/7) | Target DB Free Incentive |
|----------------|-----------------|-------------|--------------------------|--------------------------|
| **`dms.t3.micro`** | Single-AZ | **$0.018** | **$13.14** | **$0.00 (6 Mos Free)** |
| **`dms.c5.large`** | Single-AZ | **$0.194** | **$141.62** | **$0.00 (6 Mos Free)** |
| **`dms.c5.large`** | Multi-AZ CDC | **$0.388** | **$283.24** | **$0.00 (6 Mos Free)** |

---

## 5. AWS Free Tier Coverage
*   **AWS DMS Free Tier:** **Free for 6 months** (single-AZ `dms.t3.micro` + 50 GB storage) when migrating to Amazon Aurora, RDS MySQL, RDS PostgreSQL, RDS MariaDB, or Redshift.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Leaving Multi-AZ CDC Replication Instances Running Long After Cutover:**
    *   Forgetting to terminate Multi-AZ `dms.c5.large` instances ($283.24/mo) after database migration and production cutover are complete.
    *   Replication instances left running idle for months accumulate hundreds of dollars in unnecessary compute charges.

---

## 7. Actionable Cost Optimization Strategies
1.  **Leverage 6-Month Free Target Migration Offer:**
    *   Migrate target databases to **Amazon Aurora** or **RDS PostgreSQL / MySQL**.
    *   *Why:* AWS waives the replication instance fee for up to 6 months for eligible target engine migrations.
    *   **The Savings:** Provides 100% free database migration infrastructure.
2.  **Immediately Terminate DMS Instances After Production Cutover:**
    *   Set a calendar reminder to terminate the DMS replication task and delete the replication instance immediately after final application cutover validation.
    *   **The Savings:** Stops $283.24/month per instance instantly.
3.  **Use DMS Serverless for Intermittent Batch Migrations:** Use DMS Serverless for non-continuous database sync jobs so compute capacity automatically scales down to zero when sync is complete.
