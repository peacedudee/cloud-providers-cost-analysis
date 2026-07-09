# AWS Service Cost Research: Amazon Timestream for LiveAnalytics

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Timestream for LiveAnalytics is a fast, scalable, serverless time-series database service for IoT, operational telemetry, and application monitoring. It automatically ingests, stores, and processes trillions of time-series events daily. Because Timestream separates data storage into an in-memory tier (for high-speed ingestion and recent queries) and a magnetic tier (for long-term historical analysis), managing the retention windows of these tiers is the key to controlling costs.

---

## 2. Billing Mechanics
Timestream for LiveAnalytics bills on a pay-as-you-go model with four primary dimensions:
1.  **Data Ingestion (Writes):** Billed per GB of data written to the database.
2.  **Memory Store Storage:** Billed per GB-hour of data retained in the high-performance memory tier.
3.  **Magnetic Store Storage:** Billed per GB-month of data retained in the low-cost magnetic tier.
4.  **Queries (Compute Capacity):** Billed based on the compute capacity used to execute queries, measured in **Timestream Compute Unit (TCU) hours**.

---

## 3. Key Cost Dimensions

### A. Data Ingestion (us-east-1)
*   **The Rate:** AWS charges **$0.55 per GB** of data ingested.
*   **Metering:** Ingested bytes are rounded up to the nearest 1 KB per write request. 
*   *Warning:* If your IoT devices send small payload updates (e.g. 100 bytes) individually, the 1 KB rounding rule means you are billed for 10x more data than actually transmitted.

### B. Storage Tiers (The Retention Lever)
Timestream divides data storage into two tiers:
*   **Memory Store (Recent Data):** Used for low-latency writes and immediate queries.
    *   *The Rate:* **$0.036 per GB-hour** (approximately **$26.28 per GB-month** in `us-east-1`).
    *   *High-Risk Hotspot:* Storing gigabytes of data in memory long-term is highly expensive (nearly 900x more expensive than magnetic storage).
*   **Magnetic Store (Historical Data):** Low-cost tier for historical data.
    *   *The Rate:* **$0.03 per GB-month**.
    *   *Data Lifecycle:* Timestream automatically moves data from the Memory Store to the Magnetic Store once the memory retention period expires.

### C. Query Billing (TCUs)
*   **The Compute Unit:** Queries are billed based on the number of **Timestream Compute Units (TCUs)** allocated to execute your query. 1 TCU = 4 vCPUs and 16 GB RAM.
*   **The Rate:** **$0.518 per TCU-hour** in `us-east-1`.
*   **Metering:** Billed per second with a **30-second minimum** charge per query.
    *   *Query Minimum Pitfall:* Even if a query runs in 100 milliseconds, you are billed for a full 30 seconds of compute capacity. High-frequency, low-latency client queries will rapidly accumulate minimum charges.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Billing Basis | Rate (us-east-1) | Monthly Cost (Est.) |
|----------------|---------------|------------------|---------------------|
| **Data Ingestion** | Per GB written | **$0.55 / GB** | Ingestion only |
| **Memory Store Storage** | Per GB-hour | **$0.036 / GB-hour** | **$26.28 / GB-month** |
| **Magnetic Store Storage** | Per GB-month | **$0.030 / GB-month** | $3.00 per 100 GB |
| **Query Compute** | Per TCU-hour | **$0.518 / TCU-hour** | Minimum 30s charge |

---

## 5. AWS Free Tier Coverage
*   **Amazon Timestream:** No free tier available. All writes, storage, and queries generate billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Long Memory Store Retention Settings:** Setting the Memory Store retention to 30 days. Keeping 100 GB in memory costs **$2,628.00/month** compared to only **$3.00/month** in magnetic storage.
*   **IoT Write Payload Rounding (1 KB rule):** Writing single metric updates from thousands of devices individually, causing massive data volume inflation due to the 1 KB minimum write block sizing.
*   **High-Frequency Dashboard Queries (30s Minimum):** Hooking a dashboard (e.g. Grafana) to Timestream and configuring it to refresh queries every 5 seconds. The 30-second minimum billing block will multiply your query costs by 6x.
*   **Forgetting to Set MaxQueryTCU limits:** Complex queries scanning large ranges can scale up compute to 100+ TCUs, creating expensive individual query bills.

---

## 7. Actionable Cost Optimization Strategies
1.  **Minimize Memory Store Retention Windows:** Configure your table’s Memory Store retention period to the absolute minimum required for your real-time processing (e.g., 2 hours or 12 hours). Let the system automatically lifecycle data to the **$0.03/GB-month Magnetic Store** as quickly as possible.
2.  **Batch Write Operations (IoT Payloads):** Do not send single metric updates from edge devices. Batch data locally at the gateway or use an aggregator (like Amazon Kinesis or Firehose) to bundle telemetry metrics into larger write payloads (aiming for 1 KB to 8 KB packets) before writing to Timestream. This bypasses the 1 KB rounding cost.
3.  **Cap `MaxQueryTCU` Limits:** Set the `MaxQueryTCU` parameter in your query client configurations. Limit compute to a low ceiling (e.g. 4 or 8 TCUs) to prevent run-away autoscaling query costs.
4.  **Use Query Caching for Dashboards:** If displaying time-series charts on shared dashboards, implement a caching layer (e.g., in Redis/ElastiCache or local app memory). Avoid querying Timestream directly on every dashboard refresh.
5.  **Evaluate Timestream for InfluxDB for Steady Workloads:** If you have constant, predictable read/write metrics, evaluate **Timestream for InfluxDB**. It is billed per running database instance hour (flat rate), which can be significantly cheaper than pay-as-you-go TCU query billing.
