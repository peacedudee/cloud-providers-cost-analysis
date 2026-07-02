# AlloyDB Cost Optimization & Research

Google Cloud AlloyDB for PostgreSQL is a fully managed, PostgreSQL-compatible database service designed for demanding enterprise workloads. It decouples compute and storage and integrates a flash-based cache (L2 cache). While it offers 4x the performance of standard PostgreSQL, AlloyDB is a premium database service with high baseline pricing. Managing compute scaling and read pool sizing is essential to control costs.

---

## 1. AlloyDB Billing Components

AlloyDB costs are separated into three primary categories:
1. **Compute Instances:**
   * **Primary Instance:** The main read-write node (billed per vCPU and RAM hour).
   * **Read Pool Instances:** Scalable read-only nodes used for scaling queries and analytics (billed per vCPU and RAM hour).
2. **Storage:**
   * **Database Storage:** SSD-backed storage billed per GB/month. Billed regionally due to automatic replication across three zones.
   * **Ultra-Fast Cache (L2 Cache):** Billed per GB/hour for the local NVMe flash cache attached to compute instances.
   * **Backups:** Billed per GB/month for database backup snapshots.
3. **Data Transfer (Egress):** Charges for data egress to other GCP regions or the internet.

---

## 2. Core Cost-Optimization Levers

### A. Auto-Scale Read Pools
Read Pools allow you to scale query capacity dynamically by spinning up multiple read-only instances.
* **The Waste:** Running a fixed number of large read instances 24/7 when query volume drops by 90% during off-hours.
* **The Solution:** Configure **Read Pool Autoscaling**.
* **Action:** In your AlloyDB instance settings, configure the read pool to automatically scale between a tight minimum and maximum (e.g., scale between 1 and 5 nodes based on CPU utilization).

### B. Disable High Availability (HA) in non-production
* AlloyDB clusters replicate data across three zones. In production, a secondary standby instance is kept active for fast failover.
* **Action:** Ensure development, QA, and staging AlloyDB clusters are configured with a single zonal primary instance (without standby failover), cutting compute costs by **50%**.

### C. Right-Size L2 Cache Allocation
* AlloyDB attaches local NVMe SSDs to instances to serve as a fast read cache (L2 cache).
* **Action:** Monitor cache hit ratios in Cloud Monitoring. If your active working set (frequently accessed rows) fits entirely in the primary RAM cache (L1 cache), you are paying for unused L2 NVMe storage. Downsize the cache size configurations on your instance shapes to match actual data access shapes.

### D. Purchase AlloyDB Committed Use Discounts (CUDs)
* Because databases are persistent, production AlloyDB primary instances must run 24/7.
* **Action:** Purchase 1-year (25% off) or 3-year (52% off) AlloyDB CUDs to cover your primary database vCPU and RAM baselines.

---

## 3. AlloyDB Audit Checklist

1. [ ] **Non-Prod Zonal Check:** Confirm all non-production clusters are configured without HA standby nodes.
2. [ ] **Read Pool Autoscaling:** Ensure all production read pools are configured with autoscaling policies rather than fixed node counts.
3. [ ] **Cache Size Calibration:** Audit cache metrics and right-size L2 NVMe cache allocations.
4. [ ] **AlloyDB CUD Coverage:** Verify that 1-year or 3-year AlloyDB commitments cover at least 80% of your production database compute baseline.
5. [ ] **Query Insights Tuning:** Review Query Insights in the console to index and optimize slow queries, lowering CPU load and allowing compute instance downsizing.
