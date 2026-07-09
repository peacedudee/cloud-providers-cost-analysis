# AWS Service Cost Research: AWS IoT SiteWise

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS IoT SiteWise is a managed industrial IoT service that collects, stores, organizes, and visualizes data from industrial equipment (sensors, PLCs, industrial assembly lines, wind farms) at scale. It structures physical equipment into hierarchical asset models, computes real-time performance metrics (Overall Equipment Effectiveness - OEE, performance, availability), and provides no-code web applications (SiteWise Monitor) for plant operators. AWS IoT SiteWise is billed per metric ingested, data stored, computed metric, active user, and edge gateway pack.

---

## 2. Billing Mechanics
AWS IoT SiteWise billing is calculated across the following main dimensions:
1. **Data Ingestion & Retrieval (Messaging):** Billed at a flat rate of **$1.00 per 1 Million messages** ingested or retrieved.
   * *Near Real-Time Ingestion:* Metered in **1 KB increments** or **10 data points** per data stream.
   * *Buffered Ingestion:* Metered in **5 KB increments** or **60 data points** across multiple data streams (subject to a minimum charge of 10 MB or 2,000 messages per ingestion period).
2. **Asset Property Data Storage:** Multi-tiered storage options:
   * *Hot Tier (Real-Time):* **$0.30 per GB-month** (optimized for low-latency dashboards).
   * *Warm Tier (Historical):* **$0.029 per GB-month** (optimized for BI and analytics).
   * *Cold Tier (Archival):* Stores data in the customer's S3 bucket, billed at standard Amazon S3 rates.
3. **Asset Model Computations:** Billed at **$0.50 per 1 Million computed values** (aggregates, metrics, and transforms).
4. **SiteWise Monitor:** Billed at **$10.00 per unique active user per month** for web portal access.
5. **SiteWise Edge:** Gateway packs deployed on-premises:
   * *Data Collection Pack:* **Free ($0.00)**.
   * *Data Processing Pack:* **$200.00 per active gateway per month** (enables local transforms, metrics, and local monitor portals).
6. **Alarms:** Powered by AWS IoT Events, billed at **$0.10 per active alarm per month** plus message evaluation charges.

---

## 3. Key Cost Dimensions

| SiteWise Feature / Component | Billed Metric | Rate (us-east-1) | Price for 10M Units / Month |
|------------------------------|---------------|------------------|-----------------------------|
| **Data Ingestion / Retrieval**| Per 1M messages (metered) | **$1.00 / 1M msgs** | **$10.00** |
| **Hot Storage** | Per GB-month | **$0.30 / GB-mo** | **$3.00** (for 10 GB) |
| **Warm Storage** | Per GB-month | **$0.029 / GB-mo** | **$0.29** (for 10 GB) |
| **Asset Computations** | Per computed aggregate | **$0.50 / 1M calculations**| **$5.00** |
| **SiteWise Monitor Portal** | Per unique user-month | **$10.00 / user-month** | $10.00 per active user |
| **Edge Data Processing Pack** | Per active gateway-month| **$200.00 / gateway-mo**| $200.00 per gateway |
| **Active Alarms** | Per active alarm-month | **$0.10 / alarm-month** | $1.00 (for 10 alarms) |

---

## 4. Detailed Pricing Rates (us-east-1)

* **Ingestion Rate:** $1.00 per 1 Million messages.
* **Storage Rates:** $0.30 per GB-month (Hot Tier), $0.029 per GB-month (Warm Tier).
* **Aggregate Computation Rate:** $0.50 per 1 Million aggregates computed.
* **Portal User Rate:** $10.00 per active user per month.
* **Edge Gateway Processing Pack:** $200.00 per active gateway per month.

---

## 5. AWS Free Tier Coverage
* **AWS IoT SiteWise Free Tier:** Includes **10,000 free measurement metrics**, 10,000 computed metrics, and 1 GB of storage per month for 2 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
* **High-Frequency Raw Millisecond Industrial Sensor Ingestion:**
  * Ingesting raw 10-millisecond PLC sensor readings directly to the SiteWise cloud hot tier without local aggregation.
  * *Math:* 1,000 industrial sensors × 100 metrics/sec = 100,000 metrics/sec = 260 Billion metrics/month.
    If sent unaggregated using near real-time ingestion, this translates to 260 Billion messages/month:
    $$260,000 \text{ Million messages} \times \$1.00 / \text{Million} = \mathbf{\$260,000.00/month\text{ in ingestion fees!}}$$
    Plus, storing 260 Billion metrics in the Hot Tier (~3.9 TB/month) costs:
    $$3,900 \text{ GB} \times \$0.30 / \text{GB-mo} = \mathbf{\$1,170.00/month\text{ in hot storage fees.}}$$

---

## 7. Actionable Cost Optimization Strategies
1. **Use SiteWise Edge for Local Ingestion & Industrial Aggregation:**
   * Deploy the **AWS IoT SiteWise Edge Data Processing Pack** ($200/month) on local industrial gateways on the factory floor.
   * *Why:* SiteWise Edge computes 1-minute average, min, max, and sum metrics locally on the factory floor and uploads only the summarized metrics (60 data points/hour per stream) to the cloud.
   * **The Savings:** Slashes data ingestion from 260 Billion messages down to 3.6 Million messages/month. Ingestion cost drops from **$260,000.00 down to $3.60/month**, saving more than **99.9%** (recovering the $200/month gateway cost instantly).
2. **Leverage Buffered Ingestion:**
   * For non-critical telemetry streams, use buffered ingestion (5 KB or 60 data point increments) to optimize payloads and reduce total messaging counts.
3. **Configure Storage Lifecycle Retention Policies:**
   * Transition data from the expensive Hot Tier ($0.30/GB-month) to the Warm Tier ($0.029/GB-month) after 30 days. This drops storage costs by **over 90%** for older data.
