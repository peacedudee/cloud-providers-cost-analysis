# Cloud Spanner Cost Optimization & Research

Google Cloud Spanner is a fully managed, enterprise-grade, globally distributed relational database. While Spanner provides strong consistency and high availability, it is one of the most expensive database options in the cloud. Because Spanner compute capacity is billed 24/7 and does not naturally pause, minimizing compute and storage waste is critical.

---

## 1. Billing Components & Pricing Editions

Spanner pricing is calculated along three axes:
1. **Compute Capacity:** Measured in **Nodes** or **Processing Units (PUs)**. 1,000 PUs equals 1 Node. Billed hourly.
2. **Database Storage:** Billed per GB/month of active tables and indexes.
3. **Backup Storage:** Billed per GB/month of backups.
4. **Replication Topology:** Single-region vs Multi-region. Multi-region instances replicate data and compute across multiple zones/regions, multiplying costs.

### Pricing Editions (2026 Update)
Spanner now offers three editions to match your budget and capability requirements:
* **Standard:** Best for single-region workloads without advanced cross-region multi-model features. Lowest baseline cost.
* **Enterprise:** Multi-region capabilities, enterprise security.
* **Enterprise Plus:** Advanced features (Graph, Vector Search, maximum throughput).
* **Cost Strategy:** Use the **Standard Edition** for all databases that do not strictly require cross-region replication or specialized Enterprise Plus models.

---

## 2. Core Cost-Reduction Tactics

### A. Enable Spanner Managed Autoscaling
Running Spanner at fixed node counts leads to over-provisioning for peak loads.
* **The Solution:** Enable the native **Managed Autoscaler**.
* **How it works:** The autoscaler dynamically increases or decreases PUs/Nodes based on CPU utilization and storage constraints.
* **Autoscaling Target Limits:**
  * **Single-Region:** Target high-priority CPU utilization is typically set to **65-80%**.
  * **Multi-Region:** Target high-priority CPU utilization is set to **45%** to ensure there is enough headroom to handle regional failovers.

### B. Consolidate Development Databases (Shared Instances)
Unlike VMs, Spanner cannot be paused (started/stopped) to save costs during off-hours. A 100 PU instance bills 24/7.
* **The Waste:** Creating a separate Spanner instance for each developer or microservice in dev/test projects.
* **Action:** Consolidate multiple schemas into **databases inside a single shared Spanner instance** in development. A single Spanner instance can support up to 100 databases.

### C. Right-Size Dev instances using Processing Units
* Do not provision full Nodes (1,000 PUs) for non-production environments.
* **Action:** Scale development instances down to the minimum **100 PUs** (which costs exactly 1/10th of a full node).

### D. Optimize Schema to Reduce Storage Footprint
Spanner storage is SSD-backed and expensive ($0.30/GB/month Regional, higher for multi-region).
* **Use BYTES instead of STRING:** Storing binary values like UUIDs, session hashes, or hex tokens as `STRING` forces Spanner to encode them as UTF-8, which can double the storage footprint. Store them as `BYTES` instead.
* **Audit Secondary Indexes:** Every secondary index in Spanner is stored as a hidden table containing the index keys and primary keys. If you have 5 indexes on a table, writing a row writes to 6 places, and storage scales accordingly. Delete unused indexes.
* **Clean up Interleaved Tables:** Verify that tables using Parent-Child Interleaving are not accumulating orphaned child records.

### E. Monitor the Compute-to-Storage Ratio Limit
* Spanner enforces a maximum storage limit per Node/PU.
  * In standard setups, this is **10 TB of storage per 1,000 PUs** (or 1 TB per 100 PUs).
* **The Trap:** If your database contains 15 TB of data, Spanner **forces a minimum compute capacity of 2 Nodes (2,000 PUs)**, even if your CPU utilization is 2%.
* **Mitigation:**
  * Archive old transactional data to Cloud Storage (using GCS Coldline or Archive) to keep Spanner database size small.
  * Understand that 10 TB per node is a hard limit, so storage-heavy architectures must plan for compute scaling accordingly.

---

## 3. Cloud Spanner Audit Checklist

1. [ ] **Managed Autoscaler:** Ensure all production Spanner instances have the Managed Autoscaler enabled.
2. [ ] **Standard Edition Check:** Verify that databases not requiring multi-region replication are running on Spanner `Standard Edition`.
3. [ ] **Consolidation:** Check dev/test projects for multiple Spanner instances. Consolidate databases onto a single shared instance where possible.
4. [ ] **PUs for Dev:** Ensure all non-production Spanner instances are downscaled to 100 PUs.
5. [ ] **Unused Index Audit:** Scan and drop secondary indexes that have not been read in the last 30 days.
6. [ ] **Storage-to-Node Ratio:** Audit instances where compute is scaled high solely due to storage limits. Identify archiving candidates.
7. [ ] **Bytes vs String:** Confirm UUID/hash columns are defined as `BYTES` rather than `STRING`.
