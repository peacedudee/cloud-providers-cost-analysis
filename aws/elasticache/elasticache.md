# AWS Service Cost Research: Amazon ElastiCache

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon ElastiCache is a fully managed, in-memory caching and data store service. It supports three primary database engines: **Valkey** (the new open-source, high-performance fork), **Redis OSS**, and **Memcached**. ElastiCache is used to accelerate database performance, manage session stores, and cache real-time application data. Because caching workloads typically maintain high RAM usage, compute node memory size is the primary driver of ElastiCache costs.

---

## 2. Billing Mechanics
ElastiCache offers two deployment and billing models:
1.  **Serverless Mode:** No node provisioning. You are billed based on the amount of data stored (GB-hour) and compute capacity consumed in ElastiCache Processing Units (ECPUs).
2.  **Node-Based Mode (Self-Managed):** You select specific cache node instance types (e.g., `cache.t4g.medium`) and pay a flat hourly rate per running node.
3.  **Data Transfer:** Standard inter-AZ data transfer fees apply when applications access the cache across Availability Zone boundaries.
4.  **Backup Storage:** Charged per GB-month for manual and automated snapshots (beyond 1 free snapshot per active cluster).

---

## 3. Key Cost Dimensions

### A. Serverless Mode vs. Node-Based Mode (us-east-1)
*   **Serverless Mode (Pay-as-you-go):**
    *   *Storage Rate:* Billed per GB-hour of data stored.
    *   *Compute Rate:* Billed per million **ElastiCache Processing Units (ECPUs)** consumed. 1 ECPU represents 1 write or read of up to 1 KB of data.
    *   *Engine Choice Savings:*
        *   **Valkey Engine:** **$0.084 per GB-hour** + **$0.0023 per million ECPUs**.
        *   **Redis OSS / Memcached:** **$0.125 per GB-hour** + **$0.0034 per million ECPUs** (Valkey is **33% cheaper**).
*   **Node-Based Mode (Provisioned Nodes):**
    *   You provision specific nodes (e.g., memory-optimized `cache.r6g` or burstable `cache.t4g`).
    *   *Redundancy Multiplier:* If you configure a cluster with 1 Primary Node and 1 Replica Node for high availability, you are billed for **2 separate nodes** (2x cost multiplier).
    *   *Valkey Node Surcharges:* Valkey node-based rates are **~20% cheaper** than Redis OSS counterparts.
    *   *Reserved Nodes:* Committing to nodes (1 or 3 years) yields discounts of up to **55%** compared to On-Demand rates.

### B. ElastiCache Data Tiering (Redis / Valkey)
For large caches (terabytes of data), keeping everything in RAM is extremely expensive.
*   **The Cost Saver:** Data Tiering (available on `r6gd` node classes) automatically moves less-active cache keys from RAM to local NVMe SSDs.
*   **Savings Impact:** NVMe storage is significantly cheaper than RAM. Enabling data tiering allows you to store up to 5x more data per node, reducing your total node count and cutting your cache bill by up to **60%** with minimal latency impact.

### C. Cross-AZ Cache Access Egress (The Hidden Fee)
*   If your application instances in AZ-A connect to an ElastiCache node in AZ-B, you pay standard inter-AZ data transfer charges (**$0.01 per GB in each direction**). For chatty, high-velocity cache lookups, this network fee can easily exceed the cost of the cache cluster itself.

---

## 4. Detailed Pricing Rates (us-east-1 Serverless vs. Nodes)

### A. Serverless Rates (us-east-1)
| Cache Engine | Storage Rate (/GB-hour) | Monthly Storage (Est. per GB) | Compute Rate (/million ECPUs) |
|--------------|-------------------------|-------------------------------|--------------------------------|
| **Valkey** | $0.000115 ($0.084/mo) | ~$0.084 | $0.0023 |
| **Redis OSS** | $0.000171 ($0.125/mo) | ~$0.125 | $0.0034 |
| **Memcached**| $0.000171 ($0.125/mo) | ~$0.125 | $0.0034 |

### B. Node-Based Rates (us-east-1 On-Demand Valkey example)
*Valkey node-based rates are ~20% cheaper than Redis OSS counterparts:*

| Node Instance | vCPU | Memory (GiB) | Valkey Rate (/hr) | Redis OSS Rate (/hr) | Multi-AZ (1 Replica) (/hr) |
|---------------|------|--------------|-------------------|----------------------|-----------------------------|
| **cache.t4g.medium** | 2 | 3.09 | $0.0220 | $0.0280 | $0.0440 |
| **cache.m6g.large** | 2 | 6.38 | $0.1080 | $0.1360 | $0.2160 |
| **cache.r6g.large** | 2 | 13.07 | $0.1580 | $0.1980 | $0.3160 |
| **cache.r6gd.xlarge** (Tiered)| 4 | 26.24 | $0.3950 | $0.4940 | $0.7900 |

### Example Monthly Cost Calculation
*Workload: An application running in us-east-1 stores 10 GB of cache data in Serverless Valkey mode, generating 200 million ECPUs per month.*

*   **Valkey Serverless Storage Cost:**
    $$\text{Storage Fee} = 10\text{ GB} \times \$0.084/\text{GB-mo} = \$0.84$$
*   **Valkey Serverless Compute Cost:**
    $$\text{Compute Fee} = \frac{200,000,000}{1,000,000} \times \$0.0023 = \$0.46$$
*   **Total Monthly Cost:** **$1.30/month** (A Redis OSS serverless equivalent would cost $1.25 storage + $0.68 compute = $1.93/month, which is 48% more expensive).

---

## 5. AWS Free Tier Coverage
*   **Compute:** 750 hours/month of Single-AZ `cache.t2.micro` or `cache.t3.micro` node usage for the first 12 months (Redis or Memcached).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Choosing Redis OSS Serverless for Small Caches:** Using Redis Serverless for a cache containing only 10 MB of data. Serverless has a **minimum storage metering limit of 1 GB** per cache.
*   **Burstable Cache Throttling (`t4g` Nodes):** Using cheap `t4g` burstable nodes in production. Once CPU credits are exhausted, performance is severely throttled, causing cache-miss database lockups.
*   **Cross-AZ Cache Traffic:** Directing all application cache clients to a single centralized primary node in another AZ, generating massive inter-AZ data transfer fees ($0.01/GB).

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate Redis OSS to Valkey Engine:**
    *   Update your ElastiCache engines to Valkey.
    *   **The Savings:** Instantly yields a **20% direct price cut** on node instances and a **33% direct price cut** on serverless storage/ECPUs.
2.  **Purchase Reserved Nodes for Production:**
    *   For caching clusters that run continuously, commit to 1- or 3-year Reserved Nodes to reduce hourly compute rates by up to **55%**.
3.  **Enable Data Tiering (`r6gd` Nodes) for Large Datasets:**
    *   If your cache size exceeds 500 GB, migrate your cluster to Valkey or Redis using `r6gd` nodes. This lets you move cold cache data to local NVMe SSDs, reducing node counts and storage costs by up to **60%**.
4.  **Use Serverless Mode for Development & Spiky Workloads:**
    *   Switch dev/test environments and spiky production caches to Serverless mode. It automatically scales down when idle.
    *   *Warning:* If cache size is over 5 GB and has a flat, constant read/write rate, switch to **Node-Based Mode**, as continuous ECPU charges will exceed provisioned node costs.
5.  **Enforce AZ-Aware Cache Routing:**
    *   Configure your cache client libraries to read from the local Availability Zone replica node using **Read-Replicas**.
    *   **The Savings:** Routes read queries locally, avoiding inter-AZ egress fees ($0.01/GB).
