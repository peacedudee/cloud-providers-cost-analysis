# AWS Service Cost Research: Amazon MemoryDB

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon MemoryDB is a Valkey and Redis OSS-compatible, durable, in-memory database service designed for ultra-fast performance. Unlike Amazon ElastiCache (which is designed primarily as a non-durable cache), MemoryDB persists all database updates to a multi-AZ transaction log. This allows it to serve as a primary database. Because it stores all data in memory and writes all transactions to a durable log, evaluating engine choices (Valkey vs. Redis OSS) and write volumes is critical to managing costs.

---

## 2. Billing Mechanics
MemoryDB billing is calculated on a monthly cycle based on four dimensions:
1. **On-Demand Node Hours (Compute):** Hourly rates per active database node in your cluster.
2. **Data Written (Transaction Log):** Billed per GB of data written to the database log (with generous free write allowances on Valkey).
3. **Backup/Snapshot Storage:** Billed per GB-month for manual and automated database snapshots.
4. **Network Data Transfer:** Standard inter-AZ data transfer charges apply when apps access database nodes across Availability Zone boundaries.

---

## 3. Key Cost Dimensions

### A. Compute Node Prices & Engine Selection (us-east-1 On-Demand)
* **Valkey Engine (Recommended):** Offers a **20% direct price cut** on compute nodes compared to Redis OSS.
* **Redis OSS Engine:** Standard pricing.
* *Redundancy Multiplier:* High availability requires at least 1 Primary Node and 1 Replica Node (2x cost multiplier).
* *Reserved Nodes:* 1- or 3-year commitments offer up to **55% savings** on node compute fees.

### B. "Data Written" Transaction Log Rates (Valkey vs. Redis OSS)
* **Valkey Engine Advantage:** **FREE data written for the first 10 TB per month**. Data written above 10 TB/month is billed at **$0.04 per GB** (an 80% discount).
* **Redis OSS Engine:** Billed at **$0.20 per GB** written across all volumes.
* *Math Impact:* Writing 10 TB of data per month:
  * *Redis OSS:* $$10,000\text{ GB} \times \$0.20 = \mathbf{\$2,000.00\text{ / month in log fees}}$$
  * *Valkey:* $$0\text{ to }10\text{ TB} = \mathbf{\$0.00\text{ / month in log fees (Save \$2,000/mo!)}}$$

### C. Data Tiering (Valkey / Redis)
* Available on `db.r6gd` node classes. It moves cold keys from memory onto local NVMe SSDs.
* **Savings:** NVMe SSD storage is significantly cheaper than RAM, allowing you to store up to 5x more data on the same instance size, reducing your cluster node count and cutting compute costs by up to **60%**.

### D. Snapshot Storage
* The first snapshot per active cluster is **free**.
* Additional snapshots are billed at a warm rate of **$0.085 per GB-month** in `us-east-1`.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Component | Valkey Node (/hr) | Redis OSS Node (/hr) | Storage (NVMe/S3) | Data Written Surcharge (/GB) |
|-------------------|-------------------|----------------------|-------------------|------------------------------|
| **db.t4g.medium** | **$0.0290** | $0.0360 | N/A | **Valkey: Free up to 10 TB** ($0.04/GB after) |
| **db.m6g.large** | **$0.1580** | $0.1980 | N/A | **Redis OSS: $0.2000 / GB** |
| **db.r6g.large** | **$0.2310** | $0.2890 | N/A | **Redis OSS: $0.2000 / GB** |
| **db.r6gd.xlarge**| **$0.5780** | $0.7220 | *NVMe SSD Included* | **Valkey: $0.04 / GB (>10 TB)** |
| **Snapshot Storage**| N/A | N/A | $0.0850 / GB-mo | N/A |

---

## 5. AWS Free Tier Coverage
* **Amazon MemoryDB:** **No free tier** is available. Any initialized cluster generates standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
* **Using MemoryDB for Transient Caching:** Deploying MemoryDB for non-durable query caching where data loss is acceptable. Using ElastiCache instead avoids transactional log write fees entirely.
* **Staying on Redis OSS Engine Instead of Valkey:** Running Redis OSS nodes instead of Valkey, missing out on 20% compute savings and paying $0.20/GB for log writes.
* **Dev/Test Nodes Left in Multi-AZ:** Running replica nodes in non-production environments.

---

## 7. Actionable Cost Optimization Strategies
1. **Migrate to the Valkey Engine:**
   * Change your database engine setting to **Valkey**.
   * *Why:* Drops node compute fees by **20%** and makes your first **10 TB/month of data written completely FREE** (saving $2,000/mo in log fees).
   * **The Savings:** Up to **$2,000+ per month** in data written and compute savings.
2. **Evaluate ElastiCache for Non-Durable Caching:** If your application only uses in-memory data for transient caching or ephemeral session states, migrate to **Amazon ElastiCache** ($0 data written log fees).
3. **Enable Data Tiering (`db.r6gd` Nodes) for Large Datasets:** Migrate clusters >100 GB to data-tiered node types to store cold keys on cheap local NVMe storage, cutting node counts and saving up to **60%** on compute.
4. **Prune Non-Prod Replicas:** Run all development and test MemoryDB clusters with **0 replicas** (Single-Node configuration) to cut non-prod compute bills in half.
5. **Set up Reserved Nodes for Production Clusters:** Purchase 1- or 3-year Reserved Nodes for production workloads to save up to **55%** on compute.
