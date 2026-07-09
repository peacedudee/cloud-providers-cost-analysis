# AWS Service Cost Research: AWS Database Migration Service (DMS)

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Database Migration Service (DMS) helps you migrate databases to AWS quickly and securely with minimal application downtime. DMS supports homogeneous migrations (e.g. Oracle to Oracle, or PostgreSQL to RDS PostgreSQL) as well as heterogeneous migrations between different database engines (e.g. Oracle to Amazon Aurora PostgreSQL, or SQL Server to MySQL) using continuous Change Data Capture (CDC). DMS offers provisioned replication instances, serverless replication options, and simplified homogeneous migrations.

---

## 2. Billing Mechanics
DMS billing depends on the migration architecture chosen:
1.  **Provisioned Replication Instance Hourly Fee:** Billed hourly based on instance family and size (e.g., `dms.t3.micro`, `dms.c5.large`) and deployment mode (Single-AZ vs. Multi-AZ CDC).
2.  **Replication Storage Volume:** Billed per GB-month for gp3 storage provisioned for swap and logs ($0.115 per GB-month). The first 50 GB of storage is free for each replication instance.
3.  **DMS Serverless (Heterogeneous):** Billed based on DMS Capacity Units (DCUs) consumed ($0.10 per DCU-hour). Scales dynamically between 1 and 128 DCUs.
4.  **DMS Homogeneous Migrations (Serverless):** Uses a serverless execution model that eliminates provisioned replication instances and storage. Instead, you pay a flat hourly service fee for active migrations.
5.  **Free Scoping & Migration Tools:**
    *   **AWS DMS Fleet Advisor:** Automated database discovery and inventory planning is **100% free ($0.00)**.
    *   **AWS DMS Schema Conversion:** Automated code and schema conversion is **100% free ($0.00)**.

---

## 3. Key Cost Dimensions

### A. Provisioned Instance Pricing (us-east-1 Rates)
*   **`dms.t3.micro` (Single-AZ):** **$0.018 per hour** (~$13.14 per month).
*   **`dms.c5.large` (Single-AZ):** **$0.194 per hour** (~$141.62 per month).
*   **`dms.c5.large` (Multi-AZ CDC):** **$0.388 per hour** (**$283.24 per month**).
*   **`dms.r5.xlarge` (Multi-AZ CDC):** **$1.544 per hour** ($1,127.12 per month).

### B. DMS Serverless & Homogeneous Pricing
*   **DMS Capacity Units (DCU):** **$0.10 per DCU-hour** (scales dynamically from 1 to 128 DCUs based on throughput).
*   **DMS Homogeneous Migrations:** Charged a serverless compute hourly fee based on target resources (no instance or storage fees).
*   **Data Transfer:** Inbound data transfer to DMS is **free ($0.00/GB)**. Data transfer within the same Availability Zone is **free**. Cross-AZ or cross-region transfers incur standard egress rates.

---

## 4. Detailed Pricing Rates (us-east-1)

| Deployment Option | Component Type | Hourly Billed Rate | Storage / GB-mo | Free Tier Allowance |
|-------------------|----------------|--------------------|-----------------|---------------------|
| **`dms.t3.micro`** | Single-AZ Instance | **$0.018** | $0.115 | First 50 GB storage free |
| **`dms.c5.large`** | Single-AZ Instance | **$0.194** | $0.115 | First 50 GB storage free |
| **`dms.c5.large`** | Multi-AZ CDC Instance| **$0.388** | $0.115 | First 50 GB storage free |
| **DMS Serverless** | Per DCU-hour | **$0.100** | N/A | N/A |
| **DMS Fleet Advisor**| Discovery Tool | **Free ($0.00)** | N/A | Always Free |
| **Schema Conversion**| Conversion Tool | **Free ($0.00)** | N/A | Always Free |

### Example Monthly Cost Calculation (Continuous CDC)
*Workload: An enterprise replicates an on-premises database to Aurora PostgreSQL using a provisioned `dms.c5.large` Multi-AZ instance for 15 days (360 hours) of continuous Change Data Capture, requiring 150 GB of storage.*

*   **Instance Hourly Cost:**
    $$\text{Compute Cost} = 360\text{ hours} \times \$0.388 = \$139.68$$
*   **Storage Cost (First 50 GB is free):**
    $$\text{Billed Storage} = 150\text{ GB} - 50\text{ GB} = 100\text{ GB}$$
    $$\text{Storage Cost} = 100\text{ GB} \times \$0.115 \times \frac{15}{30}\text{ days} = \$5.75$$
*   **Total Cost:** **$145.43** (Target DB Free incentive may waive the $139.68 compute fee if eligible).

---

## 5. AWS Free Tier Coverage
*   **AWS DMS Free Tier:** Free for 6 months (single-AZ `dms.t3.micro` instance + 50 GB storage) when migrating to Amazon Aurora, RDS MySQL, RDS PostgreSQL, RDS MariaDB, or Amazon Redshift.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Leaving Multi-AZ CDC Replication Instances Running Post-Cutover:**
    *   Forgetting to terminate Multi-AZ `dms.c5.large` instances ($283.24/mo) after database migration and production cutover are complete.
*   **Un-deleted Homogeneous Migration Tasks:**
    *   Leaving homogeneous serverless tasks running in the active migration state when they are no longer in use, generating ongoing hourly serverless compute fees.
*   **Cross-Region Egress on Large Ingests:** Running the DMS replication instance in a different region than the target database, generating high inter-region data transfer fees.

---

## 7. Actionable Cost Optimization Strategies
1.  **Leverage 6-Month Free Target Migration Offer:**
    *   Migrate target databases to **Amazon Aurora** or **RDS PostgreSQL / MySQL**.
    *   *Why:* AWS waives the replication instance fee for up to 6 months for eligible target engine migrations.
    *   **The Savings:** Provides 100% free database migration infrastructure.
2.  **Immediately Terminate DMS Instances After Production Cutover:**
    *   Set a calendar reminder to terminate the DMS replication task and delete the replication instance immediately after final application cutover validation.
    *   **The Savings:** Stops $283.24/month per instance instantly.
3.  **Choose Serverless Homogeneous Migrations for Like-to-Like Runs:**
    *   For simple database migrations (e.g., RDS PostgreSQL to RDS PostgreSQL), use the serverless homogeneous migrations option.
    *   **The Savings:** Bypasses dedicated replication instance storage and licensing fees, scaling costs directly to execution time.
4.  **Align DMS Instance Region with Target DB:**
    *   Always launch the DMS replication instance in the same AWS region and Availability Zone as the target Amazon RDS or Aurora database.
    *   **The Savings:** Eliminates cross-AZ or cross-region network transfer fees ($0.01 to $0.02 per GB).
