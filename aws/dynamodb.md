# AWS Service Cost Research: Amazon DynamoDB

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon DynamoDB is a fully managed, serverless, server-side-scaling NoSQL database service that provides fast, predictable performance with single-digit millisecond latency. Because DynamoDB is serverless, you do not pay for running servers. Instead, you are billed for read/write throughput (measured in Capacity Units), data storage capacity, and optional features (like backups and replication). A 2024 price reduction cut On-Demand throughput costs in half, making it highly competitive for spiky workloads.

---

## 2. Billing Mechanics
DynamoDB billing is driven by the following components:
1.  **Throughput Capacity (Read/Write):** Billed based on your choice of Capacity Mode:
    *   *On-Demand:* Pay-per-request model, billed per million Read/Write Request Units.
    *   *Provisioned:* Pay-per-hour for reserved Read/Write Capacity Units, with optional auto-scaling.
2.  **Data Storage:** Billed per GB-month of table data and indexes.
3.  **Global Tables (Replication):** Extra charges for replicating data across AWS regions.
4.  **Backup and Restore:** Billed per GB-month for Point-in-Time Recovery (PITR) and standard snapshots.
5.  **DynamoDB Streams:** Billed per read request when streaming database updates.

---

## 3. Key Cost Dimensions

### A. Throughput Capacity Modes (us-east-1 Standard Table Class)
*   **On-Demand Mode (Pay-per-Request):**
    *   *Write Request Units (WRUs):* **$0.625 per million WRUs** (Note: this is a 50% price reduction from the legacy $1.25/million rate).
    *   *Read Request Units (RRUs):* **$0.125 per million RRUs** (reduced from $0.25/million).
    *   *Ideal for:* Spiky, unpredictable workloads or applications with low average utilization.
*   **Provisioned Capacity Mode (Pay-per-Hour):**
    *   *Write Capacity Units (WCUs):* **$0.00065 per WCU-hour** (approximately **$0.47 per WCU-month**).
    *   *Read Capacity Units (RCUs):* **$0.00013 per RCU-hour** (approximately **$0.09 per RCU-month**).
    *   *Ideal for:* Stable, predictable traffic patterns where auto-scaling can match capacity to usage.
*   *Rule of Thumb:* If your database utilization is consistently above **15%**, Provisioned Capacity with auto-scaling is cheaper than On-Demand. Below 15%, On-Demand is more cost-effective.

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

### C. Table Classes (Storage vs. Throughput)
*   **DynamoDB Standard (Default):** Storage is billed at **$0.25 per GB-month**. Throughput pricing is standard. Designed for tables where throughput dominates the bill.
*   **DynamoDB Standard-IA (Infrequent Access):** Storage is billed at **$0.10 per GB-month** (60% storage savings). However, Read/Write throughput fees are **25% higher**. Designed for archiving or keeping large historical datasets where storage is high but access is rare.

### D. Global Tables (Cross-Region Replication)
*   You pay for the storage of the replicated table in all target regions.
*   Writes are billed as Replicated Write Request Units (rWRUs) or Replicated Write Capacity Units (rWCUs) to sync databases, and you incur standard inter-region data transfer fees ($0.01–$0.02/GB) for bytes sent.

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

---

## 5. AWS Free Tier Coverage
*   **Storage:** 25 GB of storage space.
*   **Throughput:** 25 WCUs and 25 RCUs of Provisioned Capacity (enough to handle up to 200 million requests per month for free).

---

## 6. Common Cost Hotspots & Pitfalls
*   **On-Demand Mode on Constant Traffic Tables:** Keeping On-Demand enabled for tables with flat, continuous write workloads. This can be up to **5x more expensive** than Provisioned mode.
*   **Over-Sizing Write Items:** Storing large binary payloads or text blocks in table items. Since writes are billed per 1 KB block, storing a 10 KB object multiplies the cost of every write by 10x.
*   **Using Strongly Consistent Reads by Default:** Most applications do not require strong consistency (reading the latest written state in milliseconds). Using strongly consistent reads instead of eventual consistency **doubles (2x)** your read throughput costs.
*   **Neglecting Local Secondary Indexes (LSIs):** LSIs share write capacity with the main table. Every time you write/update an item, all LSIs must also be written, multiplying your WCU consumption. LSIs cannot be deleted without recreating the table.

---

## 7. Actionable Cost Optimization Strategies
1.  **Right-Size Throughput Capacity Modes:**
    *   Audit your table utilization. For tables with flat or predictable traffic, switch to **Provisioned Capacity with Auto-Scaling** (set minimum/maximum thresholds and a target utilization of 70%).
    *   For development tables or highly sporadic apps, stick to **On-Demand** to scale compute costs to $0 when idle.
2.  **Enforce Eventually Consistent Reads:** Ensure your database client queries explicitly configure `ConsistentRead = false` (eventual consistency) unless strong consistency is an absolute application requirement. This instantly cuts read costs by **50%**.
3.  **Move Large Payloads to S3 (The "Pointer" Pattern):** Never store files, PDFs, or large JSON/binary blobs (>2 KB) in DynamoDB. Instead, upload the file to **Amazon S3** (billed at a cheap $0.023/GB) and store the **S3 URL pointer** in the DynamoDB table item.
4.  **Evaluate Standard-IA for Large, Cold Tables:** Check the ratio of storage cost vs. throughput cost on your tables. If storage makes up more than 50% of the table's total monthly bill, migrate the table to the **Standard-IA** class to save up to 60% on storage.
5.  **Configure Time-to-Live (TTL):** Enable DynamoDB TTL on tables containing transient session data or logs. Define a timestamp attribute, and DynamoDB will automatically delete expired items.
    *   **The Benefit:** DynamoDB deletes TTL items **for free** (no WCU/WRU consumption) and automatically reclaims storage space.
6.  **Avoid LSIs, Use GSIs Instead:** Use Global Secondary Indexes (GSIs) instead of Local Secondary Indexes (LSIs) wherever possible. GSIs have their own provisioned throughput, allowing you to scale them down or delete them entirely if they are no longer needed.
