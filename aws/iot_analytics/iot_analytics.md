# AWS Service Cost Research: AWS IoT Analytics

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS IoT Analytics is a fully managed service that automates the collection, filtering, transformation, storage, and analysis of massive volumes of IoT device data for operational insights. It handles messy telemetry data, cleanses and enriches it with metadata, stores it in an optimized time-series datastore, and allows developers to run SQL queries or execute Jupyter Notebook ML models on the processed datasets. AWS IoT Analytics is billed per component usage.

> [!WARNING]
> **AWS IoT Analytics is a legacy service.** New customer access was **closed effective July 25, 2024**. Existing customers may continue to use the service, but no new features are planned. For new workloads, AWS recommends building architectures using **Amazon Kinesis**, **Amazon S3**, and **Amazon Athena**.

---

## 2. Billing Mechanics
AWS IoT Analytics billing is calculated across four pipeline layers:
1.  **Channel Data Ingestion:** Billed per GB of raw IoT telemetry data ingested into channels ($0.20 per GB ingested).
2.  **Pipeline Data Processing:** Billed per GB of data processed through enrichment and transformation pipelines ($0.20 per GB processed).
3.  **Datastore Data Storage:** Billed per GB of processed data stored in datastores ($0.03 per GB-month).
4.  **SQL Query Execution:** Billed per TB of data scanned during SQL query execution ($5.00 per TB scanned, powered by the Amazon Athena engine).

---

## 3. Key Cost Dimensions

| Pipeline Component | Billed Unit | Rate (us-east-1) | Price for 100 GB Data |
|--------------------|-------------|------------------|-----------------------|
| **Channel Ingestion** | Per GB ingested | **$0.20 / GB** | **$20.00** |
| **Pipeline Processing**| Per GB processed| **$0.20 / GB** | **$20.00** |
| **Datastore Storage** | Per GB-month | **$0.03 / GB-mo** | **$3.00 / month** |
| **SQL Query Execution**| Per TB scanned | **$5.00 / TB** | **$0.50** (100 GB scan) |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Ingestion Rate:** $0.20 per GB.
*   **Pipeline Processing Rate:** $0.20 per GB.
*   **Storage Rate:** $0.03 per GB-month.
*   **Query Rate:** $5.00 per TB scanned.

### Example Monthly Cost Calculation (Ingested Volume vs Query Scanning)
*Workload: A legacy telemetry pipeline ingests 50 GB of raw sensor data per month. The pipeline filters out junk values, outputting 30 GB of processed logs to the datastore. Every week, the team executes SQL queries that scan the entire 30 GB dataset to generate reporting (4 runs/month = 120 GB scanned).*

*   **Channel Ingestion Cost:**
    $$\text{Ingestion Cost} = 50\text{ GB} \times \$0.20/\text{GB} = \$10.00$$
*   **Pipeline Processing Cost:**
    $$\text{Processing Cost} = 50\text{ GB} \times \$0.20/\text{GB} = \$10.00$$
*   **Datastore Storage Cost:**
    $$\text{Storage Cost} = 30\text{ GB} \times \$0.03/\text{GB-mo} = \$0.90$$
*   **SQL Query Cost (120 GB scanned = 0.12 TB):**
    $$\text{Query Cost} = 0.12\text{ TB} \times \$5.00/\text{TB} = \$0.60$$
*   **Total Monthly Cost:** **$21.50/month**

---

## 5. AWS Free Tier Coverage
*   **AWS IoT Analytics Free Tier (Legacy):** Includes **100 GB of channel data ingestion**, 100 GB of pipeline processing, 10 GB of datastore storage, and 10 GB of query scanning per month for 12 months for legacy accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Ingesting Raw Duplicate Telemetry Into Channels:** Streaming un-filtered raw sensor data directly into IoT Analytics channels, paying $0.20/GB for ingestion plus $0.20/GB for pipeline processing on junk or duplicate data.
*   **Full Datastore Scans on SQL Queries:** Executing frequent queries that scan the entire datastore capacity, which bills at $5.00 per TB scanned.

---

## 7. Actionable Cost Optimization Strategies
1.  **Filter Raw Data in IoT Core Rules Before Channel Ingestion:**
    *   Apply SQL filtering in **AWS IoT Core Rules Engine ($0.15/M rules)** to drop heartbeats and unchanged sensor values BEFORE emitting data to IoT Analytics channels ($0.20/GB).
    *   **The Savings:** Slashes data ingestion and pipeline processing billing by **70–90%**.
2.  **Partition Datastores by Timestamp for Targeted Queries:**
    *   Configure datastore partitioning keys by year/month/day so SQL queries scan ONLY relevant time windows ($5.00/TB scanned) rather than full historical table scans.
3.  **Migrate to Modern Serverless IoT Analytics Pipeline:**
    *   Migrate the ingestion flow from IoT Core Rules -> **Amazon Kinesis Data Firehose** -> **Amazon S3** (using Parquet storage) -> **Amazon Athena**.
    *   **The Savings:** Ingestion via Firehose ($0.029/GB) and storage on S3 standard ($0.023/GB) are roughly **90% cheaper** than IoT Analytics channels/datastores.
