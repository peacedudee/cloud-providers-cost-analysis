# AWS Service Cost Research: AWS IoT SiteWise

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS IoT SiteWise is a managed industrial IoT service that collects, stores, organizes, and visualizes data from industrial equipment (sensors, PLCs, industrial assembly lines, wind farms) at scale. It structures physical equipment into hierarchical asset models, computes real-time performance metrics (Overall Equipment Effectiveness - OEE, performance, availability), and provides no-code web applications (SiteWise Monitor) for plant operators. AWS IoT SiteWise is billed per metric ingested, data stored, and computed metric.

---

## 2. Billing Mechanics
AWS IoT SiteWise billing is calculated across three main dimensions:
1.  **Data Ingestion:** Billed per 1 Million measurement metric values ingested from industrial assets ($0.50 per 1 Million measurement metrics).
2.  **Asset Property Data Storage:** Billed per GB of historical asset property data stored in SiteWise time-series storage ($0.03 per GB-month).
3.  **Asset Model Computations:** Billed per computed transform or metric property value ($0.0001 per computed metric = $100.00 per 1 Million computed values).

---

## 3. Key Cost Dimensions

| SiteWise Feature | Billed Metric | Rate (us-east-1) | Price for 10 Million Metrics |
|------------------|---------------|------------------|------------------------------|
| **Data Ingestion** | Per 1M measurements | **$0.50 / 1M metrics** | **$5.00** |
| **Historical Storage**| Per GB-month | **$0.03 / GB-mo** | $3.00 / month (100 GB) |
| **Metric Computations**| Per computed metric | **$0.0001 / metric**| **$1,000.00** (10M computed) |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Ingestion Rate:** $0.50 per 1 Million metrics ($0.0000005 per measurement).
*   **Storage Rate:** $0.03 per GB-month.
*   **Metric Computation Rate:** $0.0001 per computed value.

---

## 5. AWS Free Tier Coverage
*   **AWS IoT SiteWise Free Tier:** Includes **10,000 free measurement metrics per month**, 10,000 computed metrics, and 1 GB of storage for 2 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **High-Frequency Raw Millisecond Industrial Sensor Ingestion:**
    *   Ingesting raw 10-millisecond PLC sensor readings directly to SiteWise cloud datastores without local aggregation.
    *   *Math:* 1,000 industrial sensors × 100 metrics/sec = 100,000 metrics/sec = 260 Billion metrics/month = **$130,000.00/month in ingestion fees!**

---

## 7. Actionable Cost Optimization Strategies
1.  **Use SiteWise Edge for Industrial Floor Aggregation:**
    *   Deploy **AWS IoT SiteWise Edge** software on local industrial gateway servers on the factory floor.
    *   *Why:* SiteWise Edge computes 1-minute average, min, max, and sum metrics locally on the factory floor and uploads summarized metrics to the cloud.
    *   **The Savings:** Slashes cloud data ingestion and computation billing by **98.3%** ($2,160/mo vs $130,000/mo).
2.  **Optimize Asset Model Computations:** Configure complex asset metric transforms (e.g. OEE calculations) to trigger on 15-minute intervals rather than on every raw sensor reading, avoiding $0.0001 per-computation charges.
