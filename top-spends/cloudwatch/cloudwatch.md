# AWS Service Cost Research: Amazon CloudWatch

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon CloudWatch is a comprehensive monitoring, logging, and observability service for AWS resources and applications. It collects operational metrics, log data, events, and traces across infrastructure (EC2, Lambda, ECS, RDS) and custom application components. CloudWatch is a major enterprise cost center billed on a pay-as-you-go consumption model across metrics, log ingestion/storage, alarms, dashboards, and API requests.

---

## 2. Billing Mechanics
CloudWatch billing is calculated monthly across five primary functional areas:
1. **Custom Metrics:** Billed monthly per active metric pushed to CloudWatch (tiered volume rates).
2. **CloudWatch Logs:** Billed per GB of log data ingested (differentiated by log class), per GB of log data stored per month, and per GB of data scanned during CloudWatch Logs Insights queries.
3. **Alarms:** Billed monthly per active metric alarm or composite alarm.
4. **Dashboards:** Billed monthly per custom dashboard provisioned.
5. **Synthetics Canaries & RUM:** Billed per canary run or per 100,000 Real User Monitoring (RUM) events.

---

## 3. Key Cost Dimensions

### A. Metrics & Dashboard Pricing (us-east-1 Rates)
* **Custom Metrics Volume Tiers:**
  * *First 10,000 metrics / month:* **$0.30 per metric-month**.
  * *Next 240,000 metrics / month:* **$0.10 per metric-month**.
  * *Next 750,000 metrics / month:* **$0.05 per metric-month**.
  * *Over 1,000,000 metrics / month:* **$0.02 per metric-month**.
  * *High-Resolution Metrics (1-sec resolution):* Billed as 5 custom metrics ($1.50/mo).
* **Dashboards:** **$3.00 per custom dashboard per month** (3 free).

### B. CloudWatch Logs Pricing (us-east-1 Rates)
* **Log Ingestion (Standard Log Class):** **$0.50 per GB** ingested.
* **Log Ingestion (Infrequent Access Log Class):** **$0.25 per GB** ingested (**50% cheaper**).
* **Log Storage:** **$0.03 per GB per month**.
* **Logs Insights Queries:** **$0.005 per GB** of uncompressed data scanned per query.

### C. Alarms Pricing (us-east-1 Rates)
* **Standard Resolution Alarms (60-second):** **$0.10 per alarm per month**.
* **High-Resolution Alarms (10s/30s):** **$0.30 per alarm per month**.
* **Composite Alarms:** **$0.50 per alarm per month**.

---

## 4. Detailed Pricing Rates (us-east-1)

| CloudWatch Metric / Feature | Billed Unit | Rate (us-east-1) | Price for 100 GB / Units |
|-----------------------------|-------------|------------------|--------------------------|
| **Standard Log Ingestion** | Per GB ingested | **$0.50** | **$50.00** |
| **Infrequent Access Ingestion** | Per GB ingested | **$0.25** | **$25.00** (50% saved) |
| **Log Storage** | Per GB-month | **$0.03** | $3.00 / month |
| **Log Insights Query** | Per GB scanned | **$0.005** | $0.50 per 100 GB query |
| **Custom Metrics (Tier 1)** | Per metric-month | **$0.30** | $30.00 / 100 metrics/mo |
| **Standard Alarms** | Per alarm-month | **$0.10** | $10.00 / 100 alarms/mo |

---

## 5. AWS Free Tier Coverage
* **Amazon CloudWatch Free Tier:** Included per region for all AWS accounts indefinitely.
  * *Metrics:* 10 custom metrics and 10 alarms.
  * *Logs:* **5 GB of log data ingestion** and 5 GB of log data scanned per month.
  * *Dashboards:* 3 custom dashboards.

---

## 6. Common Cost Hotspots & Pitfalls
* **The "Never Expire" Log Storage Leak:** Leaving CloudWatch Log Groups set to the default retention setting of **"Never Expire"**. Over time, stale application logs accumulate gigabytes/terabytes of storage charges ($0.03/GB-mo).
* **High-Cardinality Custom Metric Explosions:** Publishing custom metrics with dynamic dimensions (e.g., embedding `User_ID` or `Session_ID` as a metric dimension). Publishing 100,000 unique dimension values creates 100,000 custom metrics × $0.30 = **$30,000.00/month**!
* **Broad Unfiltered Logs Insights Queries:** Executing queries across multi-terabyte log groups without restricting the time window ($0.005/GB scanned).

---

## 7. Actionable Cost Optimization Strategies
1. **Enforce Explicit Retention Limits on ALL Log Groups:**
   * Never leave log groups set to "Never Expire".
   * Set explicit retention limits (e.g., 7 days for dev/test, 30 days for production) using AWS Systems Manager or CLI scripts.
   * Export long-term compliance logs to **Amazon S3 Standard ($0.023/GB-mo)** or **S3 Glacier ($0.004/GB-mo)**.
   * **The Savings:** Cuts ongoing log storage bills by **70%–90%**.
2. **Use Infrequent Access Log Class for Build & Audit Logs:**
   * Switch non-critical or build log group classes to **Infrequent Access ($0.25/GB)**.
   * **The Savings:** Saves **50%** on log ingestion charges.
3. **Eliminate High-Cardinality Custom Metric Dimensions:**
   * Never use unique customer IDs, IP addresses, or UUIDs as metric dimensions in `PutMetricData`.
   * Embed high-cardinality metadata inside structured JSON log events instead, querying them via Log Insights when needed.
   * **The Savings:** Prevents multi-thousand dollar custom metric billing spikes.
4. **Restrict Logs Insights Queries by Time Window & Log Group:** Always select specific log groups and narrow query timeframes to minimize total gigabytes scanned by CloudWatch Logs Insights.
