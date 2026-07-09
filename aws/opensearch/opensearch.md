# AWS Service Cost Research: Amazon OpenSearch Service

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon OpenSearch Service (successor to Amazon Elasticsearch Service) is a managed service that makes it easy to deploy, operate, and scale OpenSearch clusters in the AWS cloud. It is used for real-time application monitoring, log analytics, and web search. OpenSearch can be deployed as a Managed Cluster (where you select instance sizes and count) or in a Serverless model. OpenSearch is compute- and storage-intensive, making storage tiering and deployment model selection the primary cost optimization levers.

---

## 2. Billing Mechanics
OpenSearch billing is structured based on the selected deployment model:
1. **Managed Clusters:**
   * *Instance Hours:* Hourly charges per node (data nodes, master nodes, and warm nodes).
   * *Storage Capacity:* Billed per GB-month for attached EBS volumes (e.g. gp3).
   * *UltraWarm / Cold Storage:* Billed per GB-month for S3-backed historical data tiers.
2. **OpenSearch Serverless:**
   * *OpenSearch Compute Units (OCUs):* Billed per-hour for active indexing and search capacity ($0.24 per OCU-hour; 1 OCU = 6 GB RAM + compute).
   * *Managed Storage:* Billed per GB-month for data stored ($0.024 per GB-month).
3. **Database Savings Plans:** Database Savings Plans can be applied to reduce compute rates on both Managed Clusters and Serverless collections in exchange for a 1- or 3-year usage commitment.

---

## 3. Key Cost Dimensions

### A. Managed Clusters (us-east-1 Nodes)
* **Instance Classes:** Billed hourly per node.
  * *t3.medium.search (Dev Node):* **$0.036 per hour** (~$26.28/month).
  * *r6g.large.search (Production Node):* **$0.187 per hour** (~$136.50/month).
* **Redundancy Multiplier:** A resilient production cluster requires Dedicated Master Nodes (typically 3 nodes) in addition to Data Nodes (minimum 2 nodes for Multi-AZ replication). This 5-node setup multiplies baseline compute costs.
* **Storage Capacity (EBS GP3):** Billed at **$0.122 per GB-month** in `us-east-1`.

### B. Storage Tiering (UltraWarm & Cold Storage)
To manage large log volumes, OpenSearch offers tiered storage:
* **Hot Storage (EBS gp3):** High-performance storage. Billed at **$0.122 per GB-month**.
* **UltraWarm Storage (S3-Backed):** Read-only tier for historical logs. Billed at **$0.024 per GB-month** (a **80% direct discount**).
* **Cold Storage:** Archival tier. Billed at **$0.0135 per GB-month** (a **89% direct discount**). Data must be temporarily restored to the warm tier to be queried.

### C. OpenSearch Serverless
* **Compute Rate:** Billed per **OCU-hour**. 1 OCU represents 6 GB of memory and compute capacity.
  * *Rate:* **$0.24 per OCU-hour** (in `us-east-1`). Indexing and search OCUs are billed separately.
* **Idle Minimum Surcharges (The Serverless Trap):**
  * *Production Collection:* Requires a minimum of 4 OCUs (2 for indexing, 2 for search, spread across AZs) to maintain availability.
    $$\text{Idle Cost: } 4\text{ OCUs} \times 730\text{ hours} \times \$0.24 = \mathbf{\$700.80\text{ / month flat}}$$
  * *Dev/Test Collection:* Can be scaled down to a minimum of 2 OCUs (1 indexing + 1 search).
    $$\text{Idle Cost: } 2\text{ OCUs} \times 730\text{ hours} \times \$0.24 = \mathbf{\$350.40\text{ / month flat}}$$
  * *Warning:* Serverless OpenSearch is highly expensive for low-traffic applications compared to running a small managed cluster.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Managed Cluster Rate | Serverless Rate | Details |
|----------------|----------------------|-----------------|---------|
| **r6g.large Compute** | **$0.1870 / hour** | N/A | Production data node |
| **Serverless Compute**| N/A | **$0.2400 / OCU-hour** | Minimum 2 OCUs (Dev) / 4 OCUs (Prod) |
| **Hot Storage (gp3)** | **$0.1220 / GB-month**| N/A | EBS provisioned storage |
| **UltraWarm Storage** | **$0.0240 / GB-month**| **$0.0240 / GB-month** | S3-backed active query tier |
| **Cold Storage** | **$0.0135 / GB-month**| N/A | S3-backed archival storage |

---

## 5. AWS Free Tier Coverage
* **Managed Clusters:** 750 free hours/month of a `t2.small.search` or `t3.small.search` instance with 10 GB of gp2 storage (new accounts, first 12 months).

---

## 6. Common Cost Hotspots & Pitfalls
* **Using OpenSearch Serverless for Small Dashboards:** Deploying Serverless for internal applications that are rarely accessed. The **$350/month (Dev) or $700/month (Prod) flat idle charge** is a massive waste of budget compared to a small $26/month `t3.medium.search` node.
* **Keeping Historical Logs in Hot Storage:** Leaving 30 or 90 days of logs on high-cost gp3 EBS volumes. Storing 1 TB of logs in hot storage costs **$122.00/month** compared to **$24.00/month** in UltraWarm.
* **Provisioning Dedicated Master Nodes in Dev:** Running 3 dedicated master nodes in development or QA environments when data nodes can act as masters.

---

## 7. Actionable Cost Optimization Strategies
1. **Enforce Index State Management (ISM) Policies:**
   * Define ISM policies to automate data lifecycles.
   * Keep data in **Hot Storage (gp3)** for 3 to 7 days (active log searching).
   * Automatically rollover and migrate indexes to **UltraWarm Storage** for days 8 to 30.
   * Automatically migrate indexes to **Cold Storage** or delete them after 30 days.
   * **The Savings:** Drops storage bills by **70–80%** for older datasets.
2. **Evaluate Serverless vs. Managed Break-Even:**
   * Only use OpenSearch Serverless if your query traffic has massive, unpredictable spikes and you process high volumes of data.
   * For steady-state monitoring, internal search, or small development environments, deploy **Managed Clusters** with small instance types (`t3.medium.search`) to bypass OCU minimum fees.
3. **Apply Database Savings Plans:** Apply Database Savings Plans to both Managed Clusters and Serverless collections to save up to **30-50%** on compute.
4. **Optimize EBS Volume Types (gp3 vs gp2):** Migrate all legacy gp2 storage volumes to **gp3**. Gp3 is **9.6% cheaper** per GB-month in us-east-1 and allows you to scale IOPS and throughput independently without provisioning excess storage space.
5. **Bypass Dedicated Masters in Non-Prod:** Configure development, test, and QA OpenSearch clusters to run as **Single-Node** or Multi-Node without dedicated masters. Let data nodes handle master duties to cut node counts by half.
6. **Enable Data Compression:** Enable `index.codec: best_compression` on all indexes. This utilizes LZ4 compression, reducing search index storage sizes by up to **20–30%**.
