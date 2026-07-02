# AWS Service Cost Research: AWS IoT Analytics

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS IoT Analytics is a fully managed service that automates the collection, filtering, transformation, storage, and analysis of massive volumes of IoT device data for operational insights. It handles messy telemetry data, cleanses and enriches it with metadata, stores it in an optimized time-series datastore, and allows developers to run SQL queries or execute Jupyter Notebook ML models on the processed datasets. AWS IoT Analytics is billed per component usage.

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

---

## 5. AWS Free Tier Coverage
*   **AWS IoT Analytics Free Tier:** Includes **100 GB of channel data ingestion**, 100 GB of pipeline processing, 10 GB of datastore storage, and 10 GB of query scanning per month for 12 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Ingesting Raw Duplicate Telemetry Into Channels ($0.20/GB):** Streaming un-filtered raw sensor data directly into IoT Analytics channels, paying $0.20/GB for ingestion plus $0.20/GB for pipeline processing on junk or duplicate data.

---

## 7. Actionable Cost Optimization Strategies
1.  **Filter Raw Data in IoT Core Rules Before Channel Ingestion:**
    *   Apply SQL filtering in **AWS IoT Core Rules Engine ($0.15/M rules)** to drop heartbeats and unchanged sensor values BEFORE emitting data to IoT Analytics channels ($0.20/GB).
    *   **The Savings:** Slashes data ingestion and pipeline processing billing by **70–90%**.
2.  **Partition Datastores by Timestamp for Targeted Queries:** Configure datastore partitioning keys by year/month/day so SQL queries scan ONLY relevant time windows ($5.00/TB scanned) rather than full historical table scans.
