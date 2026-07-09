# AWS Service Cost Research: Amazon RDS (Relational Database Service)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon RDS is a fully managed service that makes it easy to set up, operate, and scale relational databases (including MySQL, PostgreSQL, MariaDB, Oracle, and Microsoft SQL Server) in the cloud. Managing RDS costs requires understanding that database workloads run continuously (24/7) on dedicated compute nodes. Compute, storage, high-availability replication (Multi-AZ), backup storage, and engine licensing all stack together to form the monthly database bill.

---

## 2. Billing Mechanics
RDS is billed monthly based on five primary dimensions:
1.  **Database Instance Hours (Compute):** Billed hourly based on instance family, size, engine, licensing model, and redundancy.
2.  **Database Storage:** Billed per GB-month provisioned (regardless of disk utilization).
3.  **Provisioned I/O:** Extra charges for provisioned SSD IOPS (above baseline rates).
4.  **Backup Storage:** Charged per GB-month for snapshot storage that exceeds 100% of your active database storage size.
5.  **RDS Extended Support Surcharge:** Hourly per-vCPU fees for running database versions past community end-of-life.

---

## 3. Key Cost Dimensions

### A. Database Instances (Compute - us-east-1)
*   **Instance Families:** Scoped from burstable (`db.t3`, `db.t4g`) to general purpose (`db.m6i`, `db.m7g`) and memory-optimized (`db.r6g`, `db.r7g`).
*   **Licensing Models:**
    *   *License Included:* Default for MySQL, PostgreSQL, MariaDB, and SQL Server Standard/Web. License costs are built into the hourly compute rate.
    *   *Bring Your Own License (BYOL):* Available for Oracle and SQL Server Enterprise, where you pay AWS only for compute and handle software licensing independently.
*   **Database Savings Plans & Reserved Instances:** Committing to database compute usage (1 or 3 years) yields discounts of up to **53–57%** compared to On-Demand rates.

### B. Multi-AZ Cost Multiplier
Enabling High Availability (Multi-AZ) replication affects billing significantly:
*   **Multi-AZ (Standby):** AWS provisions a secondary standby database instance in another Availability Zone. This **doubles (2x)** both your hourly compute instance cost and your provisioned storage/IOPS storage cost.
*   **Multi-AZ (Two Readable Standbys):** Provisions two readable standbys in different AZs. This **triples (3x)** your hourly compute instance cost.

### C. Database Storage (us-east-1)
*   **gp3 (General Purpose SSD):** Billed at **$0.115 per GB-month**. Includes 3,000 IOPS and 125 MB/s throughput for free. Additional IOPS are billed at **$0.020/IOPS-month**; extra throughput is billed at **$0.040 per MB/s-month**.
*   **gp2 (Older SSD):** Same price (**$0.115/GB-month**), but performance scales with volume size (forcing over-provisioning storage to get baseline disk speed).
*   **io1 (Provisioned IOPS SSD):** Billed at **$0.125 per GB-month** plus a high rate of **$0.10 per provisioned IOPS-month**.

### D. Backup Storage (us-east-1)
*   **Free Allocation:** RDS provides free backup storage (automated snapshots) equal to **100% of your total provisioned database storage**.
*   **Overage Rate:** Additional backup storage is billed at **$0.095 per GB-month**.

### E. RDS Extended Support Surcharge
Running database engine versions that are past their community end-of-life (e.g., MySQL 5.7 or PostgreSQL 11) incurs an automatic, high-risk hourly fee:
*   **Years 1 to 3:** **$0.100 per vCPU-hour** surcharge.
*   **Year 3+:** **$0.200 per vCPU-hour** surcharge.
*   *Example:* Running MySQL 5.7 on a `db.m5.2xlarge` instance (8 vCPUs) adds:
    $$8\text{ vCPUs} \times 730\text{ hours} \times \$0.10 = \$584.00\text{ / month extra!}$$
    This surcharge is added directly to standard compute rates, and can double the database bill.

---

## 4. Detailed Pricing Rates (us-east-1 MySQL Example)
*Below are standard On-Demand rates for MySQL (License Included) in N. Virginia:*

| Instance Type | vCPU | Memory (GiB) | Single-AZ Rate (/hr) | Multi-AZ Rate (/hr) | Storage gp3 (/GB-mo) |
|---------------|------|--------------|----------------------|---------------------|----------------------|
| **db.t4g.micro** | 2 | 1 | $0.0160 | $0.0320 | $0.1150 |
| **db.t4g.medium**| 2 | 4 | $0.0520 | $0.1040 | $0.1150 |
| **db.m6i.large** | 2 | 8 | $0.1160 | $0.2320 | $0.1150 |
| **db.r6g.large** | 2 | 16 | $0.1800 | $0.3600 | $0.1150 |

---

## 5. AWS Free Tier Coverage
*   **Compute:** 750 hours/month of Single-AZ `db.t2.micro` or `db.t3.micro` or `db.t4g.micro` (depending on engine) for 12 months.
*   **Storage:** 20 GB of General Purpose (SSD) database storage.
*   **Backups:** 20 GB of backup storage.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Extended Support Neglect:** Running legacy engine versions without upgrading. The vCPU-hour surcharge accumulates silently and represents a massive waste of budget.
*   **Multi-AZ Enabled in Non-Prod:** Launching staging, test, or dev databases with Multi-AZ replication enabled. This instantly doubles the compute and storage bill for databases that do not require 90%+ availability.
*   **Orphaned RDS Snapshots:** Deleting an RDS database and choosing the "Create final snapshot" option. Over time, these legacy manual snapshots accumulate under Backup/RDS storage, costing **$0.095/GB-month**.
*   **Provisioning gp2 instead of gp3:** Standard configurations defaulting to gp2, which requires expanding storage capacity simply to get baseline disk speed.

---

## 7. Actionable Cost Optimization Strategies
1.  **Upgrade Engines to Avoid Extended Support Fees:** Audit your RDS inventory. Upgrade all databases approaching community end-of-life (or already inside it) to modern engine versions (e.g., MySQL 8.0+, PostgreSQL 15+) to immediately eliminate the **$0.10–$0.20/vCPU-hour surcharge**.
2.  **Purchase Database Savings Plans / Reserved Instances:** For any production database running 24/7, commit to a 1- or 3-year RI. This provides up to **57% savings** with zero architectural changes.
3.  **Disable Multi-AZ in Non-Production:** Switch all dev, QA, and staging database instances to Single-AZ. If testing requires replication, enable Multi-AZ only during testing blocks and disable it immediately after.
4.  **Migrate gp2 to gp3 Storage:** Modify RDS storage configurations from gp2 to gp3. Since gp3 provides 3,000 IOPS and 125 MB/s baseline for free, this allows you to right-size database storage capacity downward (since performance is decoupled from size) and reduces standard storage rates.
5.  **Utilize Database Stop/Start for Dev Environments:** Dev databases are rarely accessed during nights and weekends. Set up a scheduler (e.g., via AWS Systems Manager or Lambda) to stop non-prod RDS instances at 7 PM and start them at 7 AM. Since you don't pay compute charges while the instance is stopped, this cuts dev compute costs by **~65%** (storage charges still apply).
6.  **Clean up Manual Snapshots:** Audit and prune old, duplicate manual snapshots. Retain only the most recent snapshots or transition them to cold vaults via AWS Backup.
