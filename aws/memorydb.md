# AWS Service Cost Research: Amazon MemoryDB

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon MemoryDB is a Redis and Valkey-compatible, durable, in-memory database service designed for ultra-fast performance. Unlike Amazon ElastiCache (which is designed primarily as a non-durable cache), MemoryDB persists all database updates to a multi-AZ transaction log. This allows it to serve as a primary database. Because it stores all data in memory and writes all transactions to a durable log, it is significantly more expensive than ElastiCache.

---

## 2. Billing Mechanics
MemoryDB billing is calculated on a monthly cycle based on the following dimensions:
1.  **On-Demand Node Hours (Compute):** Hourly rates per active database node in your cluster.
2.  **Data Written (Transaction Log):** Billed per GB of data written (persisted) to the database log.
3.  **Backup/Snapshot Storage:** Billed per GB-month for manual and automated database snapshots.
4.  **Network Data Transfer:** Standard inter-AZ data transfer charges apply when apps access database nodes across Availability Zone boundaries.

---

## 3. Key Cost Dimensions

### A. Compute Node Prices (us-east-1 On-Demand)
Compute instances are billed hourly. You can configure Valkey or Redis OSS engines:
*   **Valkey Engine:** Offers a **20% direct price cut** on compute nodes compared to Redis OSS.
*   **Redis OSS Engine:** Standard pricing.
*   *Redundancy Multiplier:* High availability requires at least 1 Primary Node and 1 Replica Node (2x cost multiplier). Adding a second replica for read capacity scales costs to 3x.
*   *Reserved Nodes:* 1- or 3-year commitments offer up to **55% savings** on node compute fees.

### B. "Data Written" Transaction Log Surcharge
*   **The Charge:** AWS charges **$0.20 per GB** of data written (updates/inserts) to the MemoryDB transaction log.
*   **Impact:** This is the most dangerous cost hotspot in MemoryDB. If your application has a high write rate (e.g., streaming logs, page view counters, or database caches that refresh every second) and writes 10 TB of data a month, you will pay:
    $$10,000\text{ GB} \times \$0.20 = \$2,000.00\text{ / month in log charges}$$
    This is in addition to the hourly node instance rates.

### C. Data Tiering (Valkey / Redis)
*   **The Cost Saver:** Available on `db.r6gd` node classes. It moves cold keys from memory onto local NVMe SSDs.
*   **Savings:** NVMe SSD storage is significantly cheaper than RAM, allowing you to store up to 5x more data on the same instance size, reducing your cluster node count and cutting compute costs by up to **60%**.

### D. Snapshot Storage
*   The first snapshot per active cluster is **free**.
*   Additional snapshots are billed at a warm rate of **$0.085 per GB-month** in `us-east-1`.

---

## 4. Detailed Pricing Rates (us-east-1 Valkey Node Example)

| Billing Component | Valkey Node (/hr) | Redis OSS Node (/hr) | Storage gp3/S3 (/GB-mo) | Data Written Surcharge (/GB) |
|-------------------|-------------------|----------------------|-------------------------|------------------------------|
| **db.t4g.medium** | $0.0290 | $0.0360 | N/A | **$0.2000** |
| **db.m6g.large** | $0.1580 | $0.1980 | N/A | **$0.2000** |
| **db.r6g.large** | $0.2310 | $0.2890 | N/A | **$0.2000** |
| **db.r6gd.xlarge**| $0.5780 | $0.7220 | *NVMe SSD Included* | **$0.2000** |
| **Snapshot Storage**| N/A | N/A | $0.0850 | N/A |

---

## 5. AWS Free Tier Coverage
*   **Amazon MemoryDB:** **No free tier** is available. Any initialized cluster generates standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Using MemoryDB for Transient Caching:** Deploying MemoryDB to cache data that changes constantly (high write volume). Since caching does not require durability, paying the **$0.20/GB log write charge** and node premiums is a massive waste of budget compared to ElastiCache.
*   **Excessive Write Volumes:** Applications writing raw session states or continuous updates without aggregation.
*   **Dev/Test Nodes Left in Multi-AZ:** Running replica nodes in development environments. Dev databases do not require failover capabilities.
*   **Unused Manual Snapshots:** Taking manual snapshots before migrations and leaving them in S3 indefinitely, generating $0.085/GB-month fees.

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate to the Valkey Engine:** Change your cluster engines to Valkey. This is a drop-in replacement that instantly reduces your hourly node compute costs by **20%**.
2.  **Evaluate ElastiCache for Non-Durable Caching:** Audit your database usage. If your application only uses Redis to cache queries or hold ephemeral session states where data loss is not critical, **migrate to ElastiCache**. ElastiCache has **$0 data written log fees** and is significantly cheaper.
3.  **Enable Data Tiering (`db.r6gd` Nodes) for Large Clusters:** If your database exceeds 100 GB in size, migrate to data-tiered node types to store cold keys on cheap NVMe storage, cutting your node count and saving up to **60%** on compute.
4.  **Prune Non-Prod Replicas:** Run all development, test, and QA MemoryDB clusters with **0 replicas** (Single-Node configuration) to instantly cut dev compute bills in half.
5.  **Batch Write Operations:** If possible, write data in batches or aggregate updates in memory before writing to the database. This reduces the number of transactional log writes.
6.  **Set up RI Reservations for Production Nodes:** Purchase 1- or 3-year Reserved Nodes for production workloads to save up to **55%** on compute.
