# AWS Service Cost Research: Amazon DynamoDB

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon DynamoDB is a fully managed, serverless NoSQL database service that provides fast, predictable performance with single-digit millisecond latency. Because DynamoDB is serverless, you do not pay for running servers. Instead, you are billed for read/write throughput (measured in Capacity Units), data storage capacity, and optional features (like backups and replication). A major price reduction in late 2024 cut On-Demand throughput costs in half, and the introduction of Database Savings Plans in late 2025 added commitment-based discount options.

---

## 2. Billing Mechanics
DynamoDB billing is driven by the following components:
1.  **Throughput Capacity (Read/Write):** Billed based on your choice of Capacity Mode:
    *   *On-Demand:* Pay-per-request model, billed per million Read/Write Request Units.
    *   *Provisioned:* Pay-per-hour for reserved Read/Write Capacity Units, with optional auto-scaling.
2.  **Data Storage:** Billed per GB-month of table data and secondary indexes.
3.  **Global Tables (Replication):** Extra charges for replicating data across AWS regions.
4.  **Backup and Restore:** Billed per GB-month for Point-in-Time Recovery (PITR) and standard snapshots.
5.  **Database Savings Plans:** Commitment-based plans offering discounts on DynamoDB throughput.

---

## 3. Key Cost Dimensions

### A. Throughput Capacity Modes (us-east-1 Standard Table Class)
*   **On-Demand Mode (Pay-per-Request):**
    *   *Write Request Units (WRUs):* **$0.625 per million WRUs**.
    *   *Read Request Units (RRUs):* **$0.125 per million RRUs**.
    *   *Ideal for:* Spiky, unpredictable workloads or applications with low average utilization.
*   **Provisioned Capacity Mode (Pay-per-Hour):**
    *   *Write Capacity Units (WCUs):* **$0.00065 per WCU-hour** (~$0.47 per WCU-month).
    *   *Read Capacity Units (RCUs):* **$0.00013 per RCU-hour** (~$0.09 per RCU-month).
    *   *Ideal for:* Stable, predictable traffic patterns where auto-scaling can match capacity to usage.
*   *The Break-Even Rule:* If your database average utilization is consistently above **15%**, Provisioned Capacity with auto-scaling is cheaper than On-Demand. Below 15%, On-Demand is more cost-effective.

### B. Read & Write Metering Rules
DynamoDB meters operations in discrete size blocks:
*   **Write Request Unit (WRU / WCU):**
    *   1 write of up to **1 KB** of data = **1 WRU**.
    *   Sizes are rounded up to the nearest 1 KB (e.g., writing a 1.2 KB item costs **2 WRUs**).
    *   Transactional writes require **2 WRUs** per KB.
*   **Read Request Unit (RRU / RCU):**
    *   1 strongly consistent read of up to **4 KB** = **1 RRU**.
    *   2 eventually consistent reads of up to **4 KB** = **1 RRU** (Eventually consistent reads are **50% cheaper**).
    *   Transactional reads require **2 RRUs** per 4 KB.

### C. Database Savings Plans (Commitment Discounts)
Introduced in December 2025, Database Savings Plans allow you to commit to a consistent hourly spend (e.g., $1.00/hour) for a 1- or 3-year term.
*   **On-Demand Throughput Discount:** Up to **18% discount** on WRU/RRU charges.
*   **Provisioned Throughput Discount:** Up to **12% discount** on WCU/RCU charges.

### D. Table Classes (Storage vs. Throughput)
*   **DynamoDB Standard (Default):** Storage is billed at **$0.25 per GB-month**. Designed for tables where throughput dominates the bill.
*   **DynamoDB Standard-IA (Infrequent Access):** Storage is billed at **$0.10 per GB-month** (60% storage savings). However, Read/Write throughput fees are **25% higher**. Designed for archiving or keeping large historical datasets where storage is high but access is rare.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | DynamoDB Standard Class | DynamoDB Standard-IA Class | Unit |
|----------------|-------------------------|----------------------------|------|
| **Data Storage** | $0.2500 | $0.1000 | Per GB-month |
| **On-Demand Write (WRU)**| $0.6250 | $0.7812 | Per million requests |
| **On-Demand Read (RRU)** | $0.1250 | $0.1562 | Per million requests |
| **Provisioned WCU** | $0.00065 | $0.00081 | Per WCU-hour |
| **Provisioned RCU** | $0.00013 | $0.00016 | Per RCU-hour |
| **PITR Backups** | $0.2280 | $0.2280 | Per GB-month |

### Example Monthly Cost Calculation
*Workload: A table stores 100 GB of data in Standard class. It handles 50 million eventually consistent reads (average size 2 KB) and 10 million writes (average size 800 bytes) per month in On-Demand mode.*

*   **Billed Read Units (Eventually consistent reads are 50% cheaper):**
    *   2 KB is under the 4 KB boundary = 1 RRU base. Eventually consistent = 0.5 RRU per read.
    $$\text{RRUs} = 50,000,000\text{ reads} \times 0.5 = 25,000,000\text{ RRUs}$$
    $$\text{Read Cost} = \frac{25,000,000}{1,000,000} \times \$0.125 = \$3.13$$
*   **Billed Write Units:**
    *   800 bytes is under the 1 KB boundary = 1 WRU per write.
    $$\text{WRUs} = 10,000,000\text{ writes} \times 1.0 = 10,000,000\text{ WRUs}$$
    $$\text{Write Cost} = \frac{10,000,000}{1,000,000} \times \$0.625 = \$6.25$$
*   **Storage Cost:**
    $$\text{Storage Cost} = 100\text{ GB} \times \$0.25/\text{GB} = \$25.00$$
*   **Total Cost:** **$34.38/month** (Applying Database Savings Plans can discount the $9.38 throughput portion by up to 18%).

---

## 5. AWS Free Tier Coverage
*   **Storage:** 25 GB of free storage space.
*   **Throughput:** 25 WCUs and 25 RCUs of Provisioned Capacity (enough to handle up to 200 million requests per month for free).

---

## 6. Common Cost Hotspots & Pitfalls
*   **On-Demand Mode on Constant Traffic Tables:** Keeping On-Demand enabled for tables with flat, continuous write workloads. This can be up to **5x more expensive** than Provisioned mode.
*   **Using Strongly Consistent Reads by Default:** Most applications do not require strong consistency (reading the latest written state in milliseconds). Using strongly consistent reads instead of eventual consistency **doubles (2x)** your read throughput costs.
*   **Global Secondary Indexes (GSIs) Cost Multipliers:** Every GSI replicated write consumes write capacity and storage. Creating unnecessary GSIs multiplies the cost of every table write.

---

## 7. Actionable Cost Optimization Strategies
1.  **Right-Size Throughput Capacity Modes:**
    *   For tables with flat or predictable traffic, switch to **Provisioned Capacity with Auto-Scaling** (set minimum/maximum thresholds and a target utilization of 70%).
    *   For development tables or highly sporadic apps, stick to **On-Demand** to scale compute costs to $0 when idle.
2.  **Apply Database Savings Plans to Baseline Throughput:**
    *   If you have continuous DynamoDB workloads, purchase a **Database Savings Plan** to save up to **18%** on on-demand capacity and **12%** on provisioned capacity.
3.  **Enforce Eventually Consistent Reads:**
    *   Ensure your database client queries explicitly configure `ConsistentRead = false` (eventual consistency) unless strong consistency is an absolute requirement. This instantly cuts read costs by **50%**.
4.  **Move Large Payloads to S3 (The "Pointer" Pattern):**
    *   Never store files or large JSON/binary blobs (>2 KB) in DynamoDB. Instead, upload the file to **Amazon S3** (billed at a cheap $0.023/GB) and store the **S3 URL pointer** in the DynamoDB table item.
5.  **Evaluate Standard-IA for Large, Cold Tables:**
    *   Check the ratio of storage cost vs. throughput cost on your tables. If storage makes up more than 50% of the table's total monthly bill, migrate the table to the **Standard-IA** class to save up to 60% on storage.
6.  **Configure Time-to-Live (TTL) for Auto-Deletions:**
    *   Enable TTL on tables containing transient session data or logs. DynamoDB deletes TTL items **for free** (no WCU/WRU consumption) and automatically reclaims storage space.
