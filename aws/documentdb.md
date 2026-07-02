# AWS Service Cost Research: Amazon DocumentDB

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon DocumentDB (with MongoDB compatibility) is a fully managed, fast, scalable, and highly available JSON document database service. It is designed to run MongoDB workloads directly in the cloud, utilizing a cloud-native architecture that separates compute instances from a distributed cluster storage volume. DocumentDB's billing components (compute, storage, and I/O) are identical in structure to Amazon Aurora, requiring identical optimization strategies.

---

## 2. Billing Mechanics
DocumentDB is billed based on four primary dimensions:
1.  **Database Instance Hours (Compute):** Hourly charges based on instance family and size (e.g. `db.r6g.large`).
2.  **Storage & IOPS Tiers:**
    *   *DocumentDB Standard:* Lower base storage rates, but bills separately for every read/write I/O request.
    *   *DocumentDB I/O-Optimized:* Higher base storage and compute rates, but charges **$0.00** for all I/O requests.
3.  **Backup Storage:** Billed per GB-month for snapshot storage exceeding 100% of the active database size.
4.  **DocumentDB Extended Support Fee:** Surcharges for running database versions past community end-of-life.

---

## 3. Key Cost Dimensions

### A. Database Instances (Compute - us-east-1)
*   **Node-Based Billing:** You provision instances (e.g., burstable `db.t3.medium` or memory-optimized `db.r6g`).
*   **Redundancy Multiplier:** Running in a Multi-AZ high-availability configuration requires at least 1 Primary Node and 1 Replica Node, which **doubles (2x)** your compute cost.
*   **Reserved Instances:** 1- or 3-year commitments offer up to **50% savings** on compute hours.

### B. Standard vs. I/O-Optimized Configurations
You choose the cluster storage configuration based on query workloads:
*   **DocumentDB Standard (Pay-per-Request I/O):**
    *   *Storage:* **$0.10 per GB-month**.
    *   *I/O Requests:* **$0.20 per million requests**.
    *   *Compute:* Standard instance rates.
*   **DocumentDB I/O-Optimized (Free I/O):**
    *   *Storage:* **$0.225 per GB-month** (125% premium).
    *   *I/O Requests:* **$0.00** (completely free, regardless of request volume).
    *   *Compute:* Instance compute rates are **30% more expensive** (e.g., `db.r6g.large` increases from $0.278/hr to $0.361/hr).
*   **The Decision Rule:** If your I/O request charges under the Standard tier make up **more than 25%** of your total DocumentDB bill, switching to **I/O-Optimized** will reduce your overall costs.

### C. Backup Storage (us-east-1)
*   Includes free backup storage equal to **100% of your active storage size**.
*   Additional backup storage is billed at **$0.095 per GB-month**.

### D. DocumentDB v3.6 Extended Support Fee
*   **The Surcharge:** As of **July 1, 2026**, AWS charges an Extended Support fee for clusters running the older DocumentDB engine version 3.6.
*   **Rate:** Charged per vCPU-hour (typically **$0.100 per vCPU-hour**).
*   *Impact:* Running a 4-vCPU database instance in Extended Support adds **~$292.00/month** in pure support surcharges, making upgrade paths crucial.

---

## 4. Detailed Pricing Rates (us-east-1 DocumentDB Example)

| Billing Component | DocumentDB Standard Rate | DocumentDB I/O-Optimized Rate | Unit |
|-------------------|--------------------------|-------------------------------|------|
| **db.t3.medium Compute** | $0.0780 | $0.1014 | Per instance-hour |
| **db.r6g.large Compute** | $0.2780 | $0.3614 | Per instance-hour |
| **Storage Capacity** | $0.1000 | $0.2250 | Per GB-month |
| **I/O Requests** | $0.2000 | **Free ($0.00)** | Per million requests |
| **Backup Storage** | $0.0950 | $0.0950 | Per GB-month |

---

## 5. AWS Free Tier Coverage
*   **Amazon DocumentDB:** **No free tier** compute nodes are available. Any initialized cluster generates standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **I/O Request Spikes (Standard Tier):** Running continuous index creation, data migrations, or queries without indexes. Queries that perform full-collection scans generate massive I/O request fees ($0.20/million).
*   **Multi-AZ Enabled in Non-Prod:** Running reader instances in dev/test/staging environments. Replicas double the compute bill for databases that do not require failover capabilities.
*   **Extended Support Fees (v3.6):** Forgetting to upgrade legacy 3.6 clusters to version 4.0 or 5.0, resulting in the new $0.10/vCPU-hour surcharge starting July 2026.
*   **Unindexed Queries:** Querying large collections without appropriate indexes. MongoDB queries without indexes must scan every document in the database, generating high compute and I/O request costs.

---

## 7. Actionable Cost Optimization Strategies
1.  **Upgrade DocumentDB 3.6 Clusters Immediately:** Migrate all version 3.6 clusters to version 4.0 or 5.0 to immediately bypass the **Extended Support surcharge**.
2.  **Evaluate I/O-Optimized vs. Standard Monthly:** Review your bills in Cost Explorer.
    *   If I/O request fees under the Standard model exceed 25% of your bill, convert the cluster to **I/O-Optimized**.
    *   If you have a high-volume database but low I/O traffic, keep it on **Standard** to save 30% on compute.
3.  **Implement Dev Auto-Start/Stop Scheduler:** Dev databases are rarely accessed during nights and weekends. Set up a scheduler to stop non-prod DocumentDB clusters during off-peak blocks. Compute fees scale to $0 when stopped.
4.  **Enforce Indexing to Reduce I/O:** Run `db.collection.explain()` on slow queries. Create indexes on frequently queried fields to reduce index scan I/O fees by up to 99%.
5.  **Prune Non-Prod Replicas:** Run all development, test, and QA DocumentDB clusters with **0 reader nodes** (Single-Node configuration) to cut compute costs by 50%.
6.  **Purchase RIs for Production Nodes:** Purchase 1- or 3-year Reserved Nodes for production workloads to save up to **50%** on compute.
