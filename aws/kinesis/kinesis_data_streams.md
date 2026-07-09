# AWS Service Cost Research: Amazon Kinesis Data Streams

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Kinesis Data Streams is a serverless, real-time data streaming service that can continuously capture gigabytes of data per second from hundreds of thousands of sources. Because Kinesis is deployed for high-throughput data ingestion, choosing between the capacity modes (On-Demand Standard, On-Demand Advantage, or Provisioned), managing stream retention periods, and evaluating consumer pull models (like Enhanced Fan-Out) are the key cost drivers.

---

## 2. Billing Mechanics
Kinesis Data Streams is billed based on the selected capacity mode:
1. **On-Demand Standard Mode:**
   * *Stream Hourly Charge:* Flat fee per stream-hour ($0.045/hr).
   * *Data Ingested/Retrieved:* Billed per GB processed ($0.075/GB ingest, $0.013/GB retrieval).
2. **On-Demand Advantage Mode (Account-Level High Volume):**
   * *No Per-Stream Hourly Charge:* Eliminates the fixed $0.045/hr per-stream fee.
   * *Discounted Data Ingestion/Retrieval:* ~60% lower usage rates ($0.032/GB ingest, $0.016/GB retrieval).
3. **Provisioned Mode:**
   * *Shard Hours:* Billed per hour for each provisioned shard ($0.015/shard-hr).
   * *PUT Payload Units:* Billed per million 25 KB write blocks ($0.014/M units).
4. **Data Retention (All Modes):** Hourly or GB-month charges for retaining data beyond the baseline 24 hours.
5. **Enhanced Fan-Out (EFO):** Hourly fees per consumer-shard ($0.015/hr) and per-GB charges ($0.015/GB) for dedicated read pipes.

---

## 3. Key Cost Dimensions

### A. On-Demand Capacity Modes (us-east-1)
* **On-Demand Standard Mode:**
  * *Stream Hour:* **$0.045 per stream-hour** (~$32.85/month flat per stream).
  * *Data Ingestion:* **$0.075 per GB**.
  * *Data Retrieval:* **$0.013 per GB**.
  * *Ideal for:* Spiky, unpredictable, or low-volume streams.
* **On-Demand Advantage Mode:**
  * *Stream Hour:* **$0.00 (No stream hourly fee)**.
  * *Data Ingestion:* **$0.032 per GB** (~60% lower).
  * *Data Retrieval:* **$0.016 per GB**.
  * *Ideal for:* High-volume accounts with multiple serverless streams.

### B. Provisioned Capacity Mode (us-east-1)
Best for predictable, high-throughput streaming workloads.
* **Shard Hour:** **$0.015 per shard-hour** (~$10.95/month per shard).
  * *Capacity:* 1 shard supports up to 1 MB/sec write (or 1,000 records/sec) and 2 MB/sec read.
* **PUT Payload Units:** Billed at **$0.014 per million** PUT payload units (measured in 25 KB blocks).
* *Rule of Thumb:* If your stream shard utilization is consistently above **15–30%**, Provisioned Mode is cheaper. Below 15% utilization, On-Demand is more cost-effective.

### C. Data Retention Multipliers
* **Base Retention (24 hours):** Included in standard shard/stream rates at no extra charge.
* **Extended Retention (24 hours to 7 days):** Billed an extra hourly surcharge of **$0.020 per shard-hour**.
* **Long-Term Retention (7 days to 365 days):** Billed at a flat rate of **$0.023 per GB-month** of data stored (equivalent to S3 rates).

### D. Enhanced Fan-Out (EFO)
EFO provides consumers with a dedicated 2 MB/sec throughput pipe (instead of sharing the 2 MB/sec shard limit).
* **EFO Consumer Shard Hour:** Billed **$0.015 per consumer-shard hour** (for each active consumer registered).
* **EFO Data Retrieval:** Billed **$0.015 per GB** retrieved.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | On-Demand Standard | On-Demand Advantage | Provisioned Mode | Unit / Details |
|----------------|--------------------|---------------------|------------------|----------------|
| **Compute Base** | **$0.045 / stream-hr** | **$0.00 / stream-hr** | **$0.015 / shard-hr** | Base hourly compute |
| **Data Ingestion** | **$0.075 / GB** | **$0.032 / GB** | **$0.014 / 1M PUTs** | 1 PUT unit = 25 KB block |
| **Data Retrieval** | **$0.013 / GB** | **$0.016 / GB** | **Free ($0.00)** | Standard consumer read |
| **Extend Retention**| N/A | N/A | **+$0.020 / shard-hr** | Up to 7 days |
| **Long Retention** | $0.023 / GB-month | $0.023 / GB-month | $0.023 / GB-month | 7 days to 365 days |
| **EFO Connection** | $0.015 / shard-hr | $0.015 / shard-hr | $0.015 / shard-hr | Per EFO consumer-shard |
| **EFO Retrieval** | $0.015 / GB | $0.015 / GB | $0.015 / GB | Per GB read via EFO |

---

## 5. AWS Free Tier Coverage
* **Amazon Kinesis Data Streams:** No free tier is available. Standard billing applies immediately upon stream provisioning.

---

## 6. Common Cost Hotspots & Pitfalls
* **Leaving High Shard Limits Provisioned:** Provisioning 50 shards to handle a temporary high-volume data load and forgetting to scale down. A 50-shard cluster idle for a month costs **$547.50/month**.
* **Using On-Demand Standard for Steady High-Volume Streams:** Keeping On-Demand Standard active on a stream that processes a constant 10 MB/sec. Billed at $0.075/GB ingestion, this costs **$1,944.00/month**, whereas 10 provisioned shards would cost only **$109.50/month**.
* **Long Extended Retention Windows on High-Volume Streams:** Setting retention to 7 days on a 10-shard stream, adding **$146.00/month** in pure retention hour surcharges.
* **Over-using Enhanced Fan-Out (EFO):** Registering multiple serverless consumers (e.g. Lambdas) to use EFO when they could share standard shard capacity, multiplying consumer shard-hour fees.

---

## 7. Actionable Cost Optimization Strategies
1. **Evaluate On-Demand Advantage Mode for High Stream Count Accounts:**
   * For accounts with high stream counts and large aggregate data volumes, enable **On-Demand Advantage** at the account level.
   * **The Savings:** Eliminates per-stream hourly fees ($32.85/mo per stream) and cuts ingestion rates by ~60% ($0.032/GB vs $0.075/GB).
2. **Toggle Between Modes for Predictable Workloads:**
   * Use **On-Demand Mode** for new, unpredictable, or highly variable developer streams.
   * Switch to **Provisioned Mode** for production streams that have established steady-state throughput baselines (>15–30% capacity utilization).
3. **Enable Auto-Scaling on Provisioned Streams:** Use AWS Application Auto Scaling to dynamically scale shard counts up during peak ingestion hours and down during quiet blocks.
4. **Minimize Extended Retention Periods:** Do not extend retention to 7 days if your consumer applications (like Lambda or Firehose) process and write data to S3 within minutes. Keep stream retention at the baseline **24 hours** (free).
5. **Bypass EFO for Standard Consumers:**
   * Avoid using Enhanced Fan-Out unless you have a strict requirement for sub-100ms real-time delivery and more than 2 independent consumer applications reading from the same stream.
   * Have consumers share the standard 2 MB/sec read limit to save connection and retrieval fees.
6. **Aggregate Small Records Locally:** Do not write small, individual payloads (e.g. 100 bytes) to Kinesis. Use the **Kinesis Producer Library (KPL)** or local buffering to aggregate records into larger blocks (closer to 25 KB payload unit boundaries) before calling `PutRecord`. This cuts PUT payload unit charges by up to 95%.
