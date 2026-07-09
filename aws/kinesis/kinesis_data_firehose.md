# AWS Service Cost Research: Amazon Data Firehose (formerly Kinesis Data Firehose)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Data Firehose (formerly known as Kinesis Data Firehose) is a fully managed, serverless streaming delivery service. It captures, transforms, and loads streaming data into destinations like Amazon S3, Amazon Redshift, Amazon OpenSearch, Apache Iceberg, Snowflake, HTTP endpoints, and third-party service providers (like Datadog, Splunk, or New Relic). Firehose is serverless, requiring no throughput provisioning. Instead, billing is usage-based per GB of data ingested and processed, making data compression, payload formatting, and destination selection critical cost metrics.

---

## 2. Billing Mechanics
Amazon Data Firehose bills on a monthly cycle based on the volume of data processed, with additional charges for optional format conversion, decompression, and destination features:
1. **Data Ingestion Fee (Tiered):** Billed per GB of data ingested ($0.075/GB for the first 250 TB/month).
2. **Format Conversion Fee (Optional):** Billed per GB processed when converting input data (JSON) into columnar formats (Parquet/ORC).
3. **Dynamic Partitioning Fee (Optional):** Billed per GB processed when partitioning S3 delivery paths dynamically using JSON keys.
4. **Decompression Fee (Optional):** Charged per GB when decompressing incoming data streams (such as CloudWatch Logs).
5. **VPC & Specialized Destination Delivery Fees:** Billed per GB delivered when routing traffic privately to targets inside your VPC or specialized sinks like Snowflake or Apache Iceberg.

---

## 3. Key Cost Dimensions

### A. Data Ingestion & Destination Billing Rules (us-east-1 Tiers)
* **Ingestion Rates (Standard Destinations / Direct PUT):**
  * *First 250 TB/month:* **$0.075 per GB**.
  * *Next 750 TB/month:* **$0.060 per GB**.
* **The 5 KB Write Rounding Rule (High Cost Risk):** 
  * For standard S3, OpenSearch, and Redshift destinations, Firehose rounds the size of every individual record written up to the nearest **5 KB**.
  * *Rounding Pitfall:* Writing 100-byte JSON logs directly to Firehose bills 5 KB for every write, multiplying active ingestion fees by **50x**!
* **Snowflake Destination Rule:**
  * Billed on the **higher of ingested bytes vs. delivered bytes** ($0.075/GB).
  * **No 5 KB Record Rounding:** Snowflake delivery streams do NOT round individual records up to 5 KB.
* **Apache Iceberg Destination Rule:**
  * Billed per GB of data processed into Iceberg tables ($0.075/GB base tier).

### B. Optional Processing Features (us-east-1)
* **Format Conversion (JSON to Parquet/ORC):** Billed an extra **$0.018 per GB** processed.
* **Dynamic Partitioning (S3 folder customization):** Billed an extra **$0.020 per GB** processed.
* **CloudWatch Logs Decompression:** Billed per GB decompressed before delivery/processing.
* **VPC Delivery (Private delivery to VPC targets):** Billed an extra **$0.010 per GB** processed.

---

## 4. Detailed Pricing Rates (us-east-1 Baseline)

| Cost Component | Pricing Tier | Rate (us-east-1) | Billed Unit |
|----------------|--------------|------------------|-------------|
| **Data Ingestion (Std/Iceberg/Snowflake)** | First 250 TB / month | **$0.0750** | Per GB processed |
| **Data Ingestion (Std/Iceberg/Snowflake)** | Next 750 TB / month | **$0.0600** | Per GB processed |
| **Format Conversion**| All volumes | **$0.0180** | Per GB converted |
| **Dynamic Partitioning**| All volumes | **$0.0200** | Per GB partitioned |
| **VPC Delivery** | All volumes | **$0.0100** | Per GB delivered |

---

## 5. AWS Free Tier Coverage
* **Amazon Data Firehose:** No free tier is available. All ingested data streams generate standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
* **Writing Small Payload Records Directly (5 KB Rounding Trap):** Streaming raw application events individually to standard S3 Firehose streams, inflating ingestion fees from a few dollars to thousands.
* **Using Firehose Inline Lambda Transforms for Heavy Workloads:** Deploying a Lambda function inside Firehose to perform complex data formatting. Since Firehose calls Lambda for every buffer batch, you incur heavy Lambda execution and duration charges.
* **Unoptimized Dynamic Partitioning:** Partitioning S3 data by high-cardinality keys (e.g. `customer_id` or `session_id`). This forces Firehose to write to thousands of separate S3 folders simultaneously, creating millions of tiny S3 files and driving up S3 PUT request fees ($0.005/1,000 requests).

---

## 7. Actionable Cost Optimization Strategies
1. **Buffer and Aggregate Records Locally (Pre-Firehose):**
   * Do not write individual 100-byte logs directly to standard Firehose streams.
   * Aggregate logs locally in your application using agents (like Fluentbit, Logstash, or the CloudWatch agent) or the **Kinesis Producer Library (KPL)**.
   * Batch events into payloads between **5 KB and 10 KB** before writing to Firehose. This bypasses the 5 KB minimum rounding tax, instantly cutting ingestion fees by up to **90%**.
2. **Evaluate Format Conversion Break-Even:**
   * Using Firehose to convert JSON to Parquet adds **$0.018/GB**.
   * *S3 / Athena Math:* If this data is queried heavily by Athena, the Parquet format will reduce S3 data scanned by up to 99%, easily saving more in Athena query costs ($5.00/TB scanned) than you pay in Firehose conversion fees. Keep it enabled for Athena targets, but disable it if data is simply stored for archive storage.
3. **Optimize Dynamic Partitioning Cardinality:**
   * Only partition S3 directories by low-cardinality keys (like `year=2026/month=07/`).
   * Avoid partitioning by high-cardinality values like user IDs or device IDs. This prevents "small file syndrome" in S3, which drives up request fees and slows down Athena query performance.
4. **Increase Firehose Buffer Intervals:**
   * Configure the Firehose delivery stream **Buffer Hints** (size and time limits) to their maximum values (e.g., **128 MB** and **900 seconds** / 15 minutes).
   * This allows Firehose to bundle more records before writing to S3, reducing the total number of S3 PUT request fees.
