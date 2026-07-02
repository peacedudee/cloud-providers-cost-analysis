# AWS Service Cost Research: Amazon Neptune

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Neptune is a fast, reliable, fully managed graph database service optimized for storing and querying highly connected datasets. It supports popular graph models (Property Graph and W3C's RDF) and their respective query languages (Apache TinkerPop Gremlin, openCypher, and SPARQL). Like Aurora and DocumentDB, Neptune uses a cloud-native architecture that separates compute instances from a distributed storage volume, meaning its cost structure is identical in dimensions and rates.

---

## 2. Billing Mechanics
Neptune is billed based on five primary dimensions:
1.  **Database Compute:** Scoped into two models:
    *   *Provisioned Nodes:* Billed hourly per instance (e.g. `db.r6g.large`).
    *   *Serverless Mode:* Billed per-second based on **Neptune Capacity Units (NCUs)**.
2.  **Storage & IOPS Tiers:**
    *   *Neptune Standard:* Lower base storage rates, but bills separately for every read/write I/O request.
    *   *Neptune I/O-Optimized:* Higher base storage and compute rates, but charges **$0.00** for all I/O requests.
3.  **Backup Storage:** Billed per GB-month for snapshot storage exceeding 100% of the active database size.
4.  **Neptune Analytics Compute:** Hourly charges based on Memory-NCUs (m-NCUs) for in-memory graph analytics.

---

## 3. Key Cost Dimensions

### A. Compute: Provisioned vs. Serverless (us-east-1)
*   **Provisioned Instances:** Hourly rates per instance. Memory-optimized `db.r6g` classes are standard. Supported by Reserved Instances (up to 50% discount).
*   **Neptune Serverless:** Automatically scales database capacity (CPU and memory) in response to active query loads.
    *   *Neptune Capacity Unit (NCU) rate:* **$0.12 per NCU-hour** (in `us-east-1`).
    *   *Scaling:* Scales in 0.5 NCU increments (minimum 2.5 NCUs, maximum 128 NCUs).
    *   *Idle Cost Pitfall:* Neptune Serverless must run at a minimum of 2.5 NCUs, generating a flat idle compute charge of:
        $$2.5\text{ NCUs} \times 730\text{ hours} \times \$0.12 = \$219.00\text{ / month per instance}$$

### B. Standard vs. I/O-Optimized Storage Configurations
*   **Neptune Standard (Pay-per-Request I/O):**
    *   *Storage:* **$0.10 per GB-month**.
    *   *I/O Requests:* **$0.20 per million requests**.
    *   *Compute:* Standard instance/serverless rates.
*   **Neptune I/O-Optimized (Free I/O):**
    *   *Storage:* **$0.225 per GB-month** (125% premium).
    *   *I/O Requests:* **$0.00** (completely free, regardless of request volume).
    *   *Compute:* Compute rates are **30% more expensive** (Serverless rate is **$0.156 per NCU-hour**).
*   **The Decision Rule:** Graph queries traversing deep relationships (many hops) generate massive numbers of random physical I/Os. If your I/O request charges under the Standard tier make up **more than 25%** of your total Neptune bill, switching to **I/O-Optimized** will reduce your overall costs.

### C. Neptune Analytics (In-Memory Graph Engine)
*   Neptune Analytics is a separate capability designed for high-speed, in-memory graph analytics (e.g. running PageRank or graph neural networks).
*   Billed per **m-NCU hour** based on the memory capacity provisioned.
*   **Cost Saver (Paused State):** You can pause your Neptune Analytics graph when not running active analysis. While paused, you are billed a discounted rate of **10% of the active compute rate** (a **90% discount**), plus standard storage charges.

---

## 4. Detailed Pricing Rates (us-east-1 Neptune Example)

| Billing Component | Neptune Standard Rate | Neptune I/O-Optimized Rate | Unit |
|-------------------|-----------------------|----------------------------|------|
| **Serverless Compute** | $0.1200 | $0.1560 | Per NCU-hour |
| **db.r6g.large Compute** | $0.3480 | $0.4524 | Per instance-hour |
| **Storage Capacity** | $0.1000 | $0.2250 | Per GB-month |
| **I/O Requests** | $0.2000 | **Free ($0.00)** | Per million requests |
| **Backup Storage** | $0.0950 | $0.0950 | Per GB-month |

---

## 5. AWS Free Tier Coverage
*   **Amazon Neptune:** **No free tier** compute nodes are available. Any initialized cluster generates standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Unoptimized Graph Traversal I/O Storms:** Graph queries that scan a large percentage of the graph (e.g. deep nested Gremlin loops or openCypher scans). These generate huge numbers of read I/Os, accumulating high fees under the Standard storage tier.
*   **Idle Neptune Serverless Clusters:** Leaving Serverless enabled on dev/test environments. Since the minimum scale is 2.5 NCUs, an idle primary-replica cluster costs **$438.00/month** in pure idle compute fees.
*   **Neptune Analytics Left Active:** Leaving in-memory graph analytics instances running at full compute capacity after finishing analysis, rather than pausing them to leverage the 90% paused discount.

---

## 7. Actionable Cost Optimization Strategies
1.  **Analyze I/O-Optimized vs. Standard Monthly:** Graph queries are notoriously I/O intensive. Review your Neptune bills. If I/O request fees under the Standard model exceed 25% of your bill, convert the cluster to **I/O-Optimized**.
2.  **Pause Neptune Analytics Graphs Immediately:** Set up automation or scripts to pause Neptune Analytics graphs immediately after running batch graph algorithms. This reduces compute charges by **90%** during idle hours.
3.  **Implement Dev Auto-Start/Stop Scheduler:** Dev graph databases are rarely accessed during nights and weekends. Set up a scheduler to stop non-prod Neptune clusters during off-peak hours.
4.  **Prune Non-Prod Replicas:** Run all development, test, and QA Neptune clusters with **0 reader nodes** (Single-Node configuration) to cut compute costs by 50%.
5.  **Set Serverless Max NCU Limits:** Cap the Serverless Max NCU configuration in your cluster settings to a sensible limit (e.g. 16 or 32 NCUs) to prevent run-away autoscaling query costs.
6.  **Purchase RIs for Production Nodes:** Purchase 1- or 3-year Reserved Nodes for production workloads to save up to **50%** on compute.
