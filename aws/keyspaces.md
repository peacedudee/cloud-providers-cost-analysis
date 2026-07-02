# AWS Service Cost Research: Amazon Keyspaces (for Apache Cassandra)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Keyspaces is a serverless, highly available, and scalable Apache Cassandra-compatible database service. It allows you to run Cassandra workloads in the AWS cloud using the same Cassandra Query Language (CQL) code and developer tools. Because it is serverless, you do not manage servers or cluster sizing. Instead, you pay for read/write throughput (Capacity Units), storage, and data transfer.

---

## 2. Billing Mechanics
Keyspaces bills on a monthly cycle based on three main dimensions:
1.  **Throughput Capacity:** Billed using one of two modes:
    *   *On-Demand:* Billed per million Read/Write Request Units.
    *   *Provisioned:* Billed hourly for pre-allocated Read/Write Capacity Units (with auto-scaling support).
2.  **Data Storage:** Billed per GB-month of table data, index storage, and metadata.
3.  **Point-in-Time Recovery (PITR) Storage:** Billed per GB-month for continuous backups.
4.  **Network Data Transfer:** Standard AWS data transfer charges apply for cross-region or internet egress.

---

## 3. Key Cost Dimensions

### A. Throughput Capacity Modes (us-east-1 Standard)
*   **On-Demand Capacity Mode:**
    *   *Write Request Units (WRUs):* **$1.425 per million WRUs**. One WRU represents one write request for a row up to 1 KB. 
    *   *Read Request Units (RRUs):* **$0.285 per million RRUs**. One RRU represents one `LOCAL_QUORUM` read request for a row up to 4 KB.
    *   *Ideal for:* Highly variable, spiky workloads or applications with unpredictable query frequencies.
*   **Provisioned Capacity Mode:**
    *   *Write Capacity Units (WCUs):* **$0.00065 per WCU-hour** (approximately **$0.47 per WCU-month**).
    *   *Read Capacity Units (RCUs):* **$0.00013 per RCU-hour** (approximately **$0.09 per RCU-month**).
    *   *Ideal for:* Stable, predictable traffic workloads where auto-scaling can match capacity.
*   *Rule of Thumb:* Similar to DynamoDB, if your average database utilization is consistently above **15%**, Provisioned Mode is cheaper than On-Demand. Below 15% utilization, On-Demand is more cost-effective.

### B. Read & Write Metering and Consistency
The Cassandra consistency level chosen directly affects the billing:
*   **Writes:** Billed in 1 KB size increments (rounded up). 
*   **Reads:** Billed in 4 KB size increments (rounded up).
    *   *`LOCAL_ONE` and `ONE` consistency reads:* Bill at a **50% discount** (0.5 RRUs per 4 KB).
    *   *`LOCAL_QUORUM` and `QUORUM` consistency reads:* Bill at standard rates (1 RRU per 4 KB).

### C. Data Storage (us-east-1)
*   **Table Storage:** Billed at a flat rate of **$0.25 per GB-month**.
*   **PITR Backup Storage:** If Point-in-Time Recovery is enabled, backups are billed at **$0.228 per GB-month**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | On-Demand Rate | Provisioned Rate | Unit / Details |
|----------------|----------------|------------------|----------------|
| **Write Throughput** | **$1.425 / million WRUs** | **$0.00065 / WCU-hour** | 1 WCU = 1 KB write/sec |
| **Read Throughput** | **$0.285 / million RRUs** | **$0.00013 / RCU-hour** | 1 RCU = 4 KB read/sec |
| **Standard Storage** | $0.2500 / GB-month | $0.2500 / GB-month | Logical data size |
| **PITR Backup Storage**| $0.2280 / GB-month | $0.2280 / GB-month | Incremental backup size |

---

## 5. AWS Free Tier Coverage
*   **Amazon Keyspaces:** **No free tier** is available. Any initialized tables generate standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Using On-Demand for Predictable Batch Ingestion:** Keeping On-Demand enabled during large ETL or migration processes. Writing millions of rows using On-Demand at $1.425/million WRUs is up to **3x more expensive** than temporarily scaling up Provisioned WCU capacity.
*   **Over-Sizing Row Data:** Storing large text blobs or columns of metadata in individual rows. Since writes are billed per 1 KB block, writing a 5 KB row uses 5 WRUs.
*   **Continuous strongly consistent reads (`LOCAL_QUORUM`):** Querying keyspaces using `LOCAL_QUORUM` consistency by default when your application can tolerate eventually consistent `LOCAL_ONE` reads, which are **50% cheaper**.
*   **Over-Provisioning Auto-Scaling Minimums:** Setting high minimum RCU/WCU thresholds on provisioned tables, causing high idle charges during off-peak hours.

---

## 7. Actionable Cost Optimization Strategies
1.  **Toggle Between Modes for Large Migrations:**
    *   Before running large data loads or bulk migrations, switch your Keyspaces tables to **Provisioned Mode** and set high WCU limits.
    *   After the import finishes, switch the tables back to **On-Demand** or scale down Provisioned WCUs.
2.  **Enforce `LOCAL_ONE` Consistency for Reads:** Configure your Cassandra driver or CQL queries to use `LOCAL_ONE` consistency for all non-critical, read-heavy query endpoints. This reduces RRU consumption by **50%**.
3.  **Optimize Row Sizes (Compress/Move Large Payloads):** Never store files or large JSON blobs directly in Keyspaces columns. Upload the raw files to **Amazon S3** (billed at a cheap $0.023/GB) and store only the **S3 object key/metadata** in the Cassandra row.
4.  **Configure Auto-Scaling with 70% Target Utilization:** When using Provisioned Mode, always enable auto-scaling. Set the target utilization to **70%** and set a low minimum capacity limit (e.g. 1 WCU / 1 RCU) to ensure it scales down to near-zero during idle hours.
5.  **Disable PITR on Non-Prod Tables:** Disable Point-in-Time Recovery on development, testing, and staging tables. This eliminates the **$0.228/GB-month backup storage surcharge** entirely.
