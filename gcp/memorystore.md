# Memorystore Cost Optimization & Research

Google Cloud Memorystore is a fully managed in-memory data store service for Redis, Valkey, and Memcached. It provides sub-millisecond data access for caching and session management. Because Memorystore charges are based on **provisioned memory capacity** (billed per GB-hour) and replication tier, over-provisioning storage capacity or running high-availability configurations in dev/test can lead to high idle charges.

---

## 1. Memorystore Billing Components

Memorystore pricing is determined by three main dimensions:
1. **Service Tier:**
   * **Basic Tier (Single Node):** A single in-memory database node. No high-availability, no replication.
   * **Standard Tier (High Availability):** Automatically replicates data across two zones and handles failover. **Standard Tier costs exactly 2x the Basic Tier price.**
2. **Provisioned Capacity (GB):** Billed per GB-hour based on the capacity you provision (e.g., a 10 GB instance is billed for 10 GB, even if it contains only 50 MB of data).
3. **Engine Type:** Redis, Valkey, or Memcached (unit rates vary slightly but follow similar capacity-scaling rules).

---

## 2. Core Cost-Optimization Levers

### A. Downgrade Non-Production to Basic Tier
* **The Waste:** Running Standard Tier instances (HA) in development, QA, or staging environments.
* **Action:** Audit all Memorystore instances. For any environment outside of production, modify the instance configuration to **Basic Tier**.
* **The Benefit:** Cuts the compute cost of these cache databases by **50%** immediately.

### B. Right-Size Provisioned Memory Capacity
* **The Waste:** Provisioning a large capacity (e.g. 30 GB) to handle "potential growth" or "worst-case memory spikes," while actual memory consumption never exceeds 2 GB.
* **Action:**
  1. Check the `redis.googleapis.com/stats/memory/usage_ratio` metric in Cloud Monitoring.
  2. If the memory usage ratio is consistently below 50%, scale down the capacity of the instance.
  3. Memorystore supports online scaling, allowing you to increase or decrease instance sizes without downtime.

### C. Configure Active Key Eviction Policies
To prevent out-of-memory (OOM) errors without over-provisioning storage capacity:
* **Action:** Configure an active eviction policy in your Redis configurations (e.g., `maxmemory-policy: allkeys-lru` or `volatile-lru`).
* **The Benefit:** When the cache reaches its memory limit, Redis automatically evicts the least recently used keys to make room for new writes, allowing you to run a smaller database footprint without risking application crashes.

### D. Keep Network Latency & Egress Internal
* Cache queries happen at high frequency (thousands of ops/sec).
* **Action:** Deploy Memorystore instances in the **same region and zone** as the Compute Engine VMs, GKE clusters, or Cloud Run services that read from them. This prevents latency and avoids inter-zone network charges ($0.01/GB).

---

## 3. Memorystore Audit Checklist

1. [ ] **Basic Tier Verification:** Confirm all non-production Memorystore instances are running on the `Basic Tier`.
2. [ ] **Memory Usage Audit:** Audit actual memory utilization ratios. Downscale instances with < 50% average memory usage.
3. [ ] **Eviction Policy Check:** Verify `maxmemory-policy` is set to an active eviction model (e.g. `allkeys-lru`) on all cache instances.
4. [ ] **Zone Alignment:** Check that calling compute clients and Memorystore instances reside in the same zone.
5. [ ] **Idleness Clean Up:** Delete Memorystore instances in projects where cache-hits are zero (abandoned projects).
