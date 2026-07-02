# Cloud Bigtable Cost Optimization & Research

Google Cloud Bigtable is a highly scalable, fully managed NoSQL wide-column database. It is optimized for high-throughput, low-latency analytical and operational workloads. However, due to its provisioned node model, bad schema designs, hotspotting, and missing storage garbage collection can lead to significant cost waste.

---

## 1. Bigtable Billing Components

Bigtable billing is driven by three main factors:
1. **Compute Nodes:** Billed hourly per active node in each cluster. A cluster requires a minimum of 1 node (or more depending on storage and HA requirements).
2. **Storage Type:**
   * **SSD Storage:** Billed at ~$0.17/GB/month. Highly recommended for low-latency operational workloads.
   * **HDD Storage:** Billed at ~$0.026/GB/month. Recommended only for batch processing or archival data (high latency).
3. **Replication & Network Egress:** Adding replicated clusters in other zones or regions doubles or triples both compute node hours and storage GB fees, alongside inter-region network charges.

---

## 2. Core Cost-Reduction Tactics

### A. Enable Bigtable Autoscaling
Running clusters at a fixed, static node count based on peak-traffic expectations leads to high idle costs.
* **The Solution:** Configure **Autoscaling** for all Bigtable clusters.
* **How it works:** Bigtable automatically adjusts the node count between your configured minimum and maximum limits based on CPU utilization and storage capacity.
* **Autoscaling Target Limits:**
  * **Single-Cluster (No replication):** Set target CPU utilization to **70%**.
  * **Replicated Cluster (Multi-cluster routing):** Set target CPU utilization to **35%** (to ensure that if one cluster fails, the remaining clusters can handle the failed cluster's traffic without hitting CPU saturation).

### B. Implement Schema & Row Key Optimization (Prevention of Hotspotting)
An unoptimized row key design causes all read/write traffic to hit a single node (hotspotting), leaving the other nodes idle.
* **The Cost Trap:** Because one node is overloaded, overall database latency spikes, forcing administrators to scale up the node count to compensate. You end up paying for a 10-node cluster when only 1 node is doing actual work.
* **Tactic:** Use **Key Visualizer** in the GCP Console to analyze traffic patterns.
* **Action:** Redesign row keys to distribute read/write operations evenly across the cluster (e.g., avoid sequential timestamps at the start of keys; use reversed domain names, salting, or hashes).

### C. Enforce Strict Garbage Collection (GC) Policies
Bigtable tables store multiple historical versions of data cells. Without active GC policies, old historical data accumulates indefinitely.
* **Action:** Define Garbage Collection rules on every column family:
  * **Max Versions:** Limit the number of cell versions stored (e.g., keep only the 3 most recent versions).
  * **Max Age:** Automatically delete data cells older than a certain duration (e.g., delete rows older than 30 days).
  * *Example GC Rule via `cbt` CLI:*
    ```bash
    cbt setgcpolicy my-table my-column-family maxversions=3 maxage=30d
    ```

### D. Mind the Storage-per-Node Threshold
Autoscaling nodes is bounded by physical storage limits.
* **The Limits:**
  * **SSD Clusters:** Bigtable enforces a maximum limit of **5 TB of data per node** (or 2.5 TB per node if the instance has replicated clusters).
  * **HDD Clusters:** Enforces a maximum of **16 TB of data per node**.
* **The Trap:** If an SSD cluster has 15 TB of data, the autoscaler **cannot scale lower than 3 nodes**, even if CPU utilization is 0%.
* **Mitigation:** Use garbage collection to delete stale records, and export old historical data to Cloud Storage using Dataflow templates to drop below storage-based scaling floors.

### E. Decommission Replication in Non-Production
* Multi-cluster replication is essential for failover and low-latency global routing in production.
* **Action:** Dev and staging environments should only ever use a single-cluster zonal Bigtable setup without replication, reducing node and storage costs by 50% or more.

---

## 3. Cloud Bigtable Audit Checklist

1. [ ] **Autoscaling Verification:** Confirm autoscaling is active on all production clusters.
2. [ ] **CPU Target Calibration:** Ensure target CPU settings match replication topology (35% for replicated, 70% for single cluster).
3. [ ] **Dev Single-Cluster:** Verify no staging/dev Bigtable instances are running replicated clusters.
4. [ ] **Garbage Collection Policies:** Check that every column family in every table has a defined `maxversions` or `maxage` GC policy.
5. [ ] **Key Visualizer Sweep:** Audit Key Visualizer logs to detect and resolve hotspotting keys.
6. [ ] **Storage Footprint Management:** For clusters hitting storage scaling limits, run data archiving/pruning scripts.
