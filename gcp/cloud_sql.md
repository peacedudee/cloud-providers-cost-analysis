# Cloud SQL Cost Optimization & Research

Google Cloud SQL is a fully managed database service for MySQL, PostgreSQL, and SQL Server. Databases are typically the most critical and expensive components of an application stack. Because databases are persistent, teams often over-provision them to handle worst-case load spikes, resulting in high baseline costs. This file documents optimization strategies for Cloud SQL.

---

## 1. Cloud SQL Billing Components

Cloud SQL billing is comprised of four primary items:
1. **Compute (vCPU & RAM):** Billed hourly based on instance type. Custom shapes can be constructed to decouple vCPUs from RAM.
2. **High Availability (HA) Replication:** Doubles the vCPU and RAM billing by running a synchronous standby instance in a different zone.
3. **Storage (SSD or HDD):** Billed per GB provisioned. SSD is the default and recommended for performance; HDD is available for basic workloads.
4. **Data Transfer (Network Egress):** Billed for data moving out of the database (e.g., to the internet or across GCP regions).

---

## 2. Core Cost-Reduction Tactics

### A. Disable High Availability (HA) in Non-Production
High Availability is essential for production databases to ensure automatic failover. However, running HA in development, sandbox, or staging environments is a massive waste of resources.
* **Action:** Audit all databases. For any non-production database, change the availability type to **Zonal** (single zone). This immediately cuts the compute cost of that database by **50%**.

### B. Automated Start/Stop Scheduling (Dev/Test)
Unlike Compute Engine VMs, which teams routinely stop, Cloud SQL instances are often left running 24/7.
* **Tactic:** Google Cloud provides APIs to start and stop Cloud SQL instances.
* **Action:** Schedule Cloud SQL instances in development and staging environments to stop at 7:00 PM and start at 7:00 AM on weekdays, and remain stopped on weekends. This saves **~70%** of compute costs for these instances.

### C. The Storage Auto-Increase Trap
Cloud SQL features an "automatic storage increase" setting to prevent database crashes when disk space runs out.
* **The Trap:** While storage automatically expands, **Cloud SQL storage can never be shrunk**. If a temporary query or batch ingestion write causes the database disk to balloon to 1 TB, you are locked into paying for 1 TB of SSD storage forever, even if you delete the data immediately after.
* **Mitigation Strategy:**
  1. Size your database disk conservatively and cap the auto-increase limit (e.g., maximum 200 GB) instead of leaving it unlimited.
  2. If a database storage volume has already bloated far beyond the actual data size:
     * Create a new Cloud SQL instance with a smaller, correct storage size.
     * Export the data from the old instance.
     * Import the data into the new instance.
     * Update application connection strings and delete the bloated database.

### D. Leverage Committed Use Discounts (CUDs)
For production databases that must run 24/7, never pay on-demand rates.
* **Action:** Buy 1-year (25% off) or 3-year (52% off) Cloud SQL CUDs. CUDs are **not** engine-specific; they apply to aggregate Cloud SQL compute usage within a region across MySQL, PostgreSQL, and SQL Server. Ensure your production engine baseline is covered.

### E. Optimize Instance Shapes with Custom Machine Types
Google offers pre-defined database classes (e.g., `db-custom-16-61440` which has 16 vCPUs and 60 GB RAM). If your database is memory-intensive but CPU utilization is under 15%, you can provision a Custom Machine Type.
* **Action:** Downscale the vCPUs while keeping the RAM high (e.g., `db-custom-4-32768` with 4 vCPUs and 32 GB RAM) to save on vCPU licensing and compute charges.

### F. Query Optimization & Indexing (Indirect Savings)
High database CPU utilization forces you to buy larger instance classes.
* **Tactic:** Use Cloud SQL **Query Insights** to find the top query patterns driving CPU usage.
* **Action:** Add proper indexes, optimize slow joins, and cache frequently read data in Redis/Memcached. Lowering database CPU load allows you to resize the Cloud SQL instance to a smaller class without degrading application performance.

---

## 3. Cloud SQL Audit Checklist

1. [ ] **Non-Prod Zonal Check:** Confirm all dev, test, and staging databases are zonal, not HA.
2. [ ] **Dev/Test Schedules:** Implement automated start/stop scripts for all non-prod database instances.
3. [ ] **Storage Utilization Audit:** Run queries to check the ratio of `active data size` vs `provisioned disk size`. Flag databases where provisioned disk is > 3x the active data.
4. [ ] **Query Insights Review:** Audit Query Insights weekly. Target queries causing CPU spikes for optimization.
5. [ ] **CUD Coverage Match:** Ensure at least 80% of your production database compute baseline is covered by 1-year or 3-year database CUDs.
6. [ ] **Read Replicas Audit:** Identify and delete unused or oversized read replicas.
