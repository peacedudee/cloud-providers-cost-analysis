# AWS Service Cost Research: Amazon API Gateway

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. It supports **HTTP APIs**, **REST APIs**, and **WebSocket APIs**. Because API Gateway acts as the entry point to serverless backends (such as AWS Lambda or HTTP endpoints), it is billed primarily on a pay-as-you-go request volume model. However, architectural choices (REST vs. HTTP) or enabling API Gateway caching can create significant cost variations.

---

## 2. Billing Mechanics
API Gateway billing is calculated on a monthly cycle across four main dimensions:
1. **API Requests / Messages:** Billed per million requests processed, determined by API type (REST, HTTP, or WebSocket).
2. **Connection Minutes (WebSockets only):** Billed per million minutes clients remain connected to the WebSocket API.
3. **Data Transfer (Egress):** Standard AWS outbound internet data transfer rates apply to response payloads ($0.09/GB baseline).
4. **API Caching (REST APIs only):** Billed at a fixed hourly rate based on allocated cache memory size.

---

## 3. Key Cost Dimensions

### A. API Types & Request Pricing (us-east-1 Tiers)

* **HTTP APIs (Lightweight, High-Performance, Serverless):**
  * *First 300 Million requests/month:* **$1.00 per million requests**.
  * *Beyond 300 Million requests/month:* **$0.90 per million requests**.
  * *Ideal for:* High-volume, low-latency APIs connecting to Lambda or VPC endpoints without requiring complex enterprise request transformations.

* **REST APIs (Feature-Rich Enterprise APIs):**
  * *First 333 Million requests/month:* **$3.50 per million requests** (3.5x higher cost than HTTP APIs).
  * *Next 667 Million requests/month:* **$2.80 per million requests**.
  * *Next 19 Billion requests/month:* **$2.38 per million requests**.
  * *Beyond 20 Billion requests/month:* **$1.52 per million requests**.
  * *Ideal for:* API integrations requiring AWS WAF protection, inline mapping templates, client SSL certificates, API key management, or request validation natively.

* **WebSocket APIs (Real-time Two-Way Communication):**
  * *Messages (up to 1 Billion/month):* **$1.00 per million messages**.
  * *Messages (over 1 Billion/month):* **$0.80 per million messages**.
  * *Connection Time:* **$0.25 per million connection minutes**.

### B. REST API Caching (Flat Hourly Fee Structure)
Caching can be enabled on REST API stages to store responses and speed up performance. Caching is billed at a flat hourly rate per stage based on cache memory size, regardless of request volume or cache hit ratio:

* *0.5 GB Cache:* **$0.020 per hour** (~$14.60/month).
* *1.6 GB Cache:* **$0.038 per hour** (~$27.74/month).
* *6.1 GB Cache:* **$0.250 per hour** (~$182.50/month).
* *13.5 GB Cache:* **$0.500 per hour** (~$365.00/month).
* *28.4 GB Cache:* **$1.000 per hour** (~$730.00/month).
* *58.2 GB Cache:* **$1.900 per hour** (~$1,387.00/month).
* *118 GB Cache:* **$3.800 per hour** (~$2,774.00/month).
* *237 GB Cache:* **$3.800 per hour** (~$2,774.00/month).

> **Warning:** Enabling a large cache size on dev, test, or QA API stages incurs massive fixed monthly charges even if zero requests are processed.

### C. Data Transfer Egress
* Payload data returned to external clients is billed at standard outbound internet data transfer rates:
  * First 100 GB/month: **Free** (global AWS account allowance).
  * Next 10 TB/month: **$0.09 per GB**.

---

## 4. Detailed Pricing Rates (us-east-1 First Tier)

| API Type | Request Rate (per Million) | Data Egress (per GB) | Connection Minutes (per Million) | 0.5 GB Cache Rate (/hr) |
|----------|----------------------------|----------------------|-----------------------------------|-------------------------|
| **HTTP API** | **$1.0000** | $0.0900 | N/A | N/A (Not supported) |
| **REST API** | **$3.5000** | $0.0900 | N/A | **$0.0200** ($14.60/mo) |
| **WebSocket API** | **$1.0000** | $0.0900 | **$0.2500** | N/A |

---

## 5. AWS Free Tier Coverage
* **Requests:** 1 million free API requests per month (REST or HTTP combined) for the first 12 months.
* **WebSockets:** 1 million messages and 750,000 connection minutes per month for the first 12 months.

---

## 6. Common Cost Hotspots & Pitfalls
* **Defaulting to REST APIs for Simple Proxy Routes:** Selecting REST APIs out of habit when building standard serverless microservices. Using REST instead of HTTP APIs increases request charges by **3.5x**.
* **Unused API Caching in Non-Production Environments:** Enabling caching on staging, QA, or dev API stages, accumulating fixed hourly charges continuously.
* **WebSocket Connection Leaks:** Allowing client WebSocket connections to remain open indefinitely without idle disconnect logic, generating continuous connection-minute fees.
* **Passing Large Payloads / Binary Files Through API Gateway:** Returning multi-megabyte files or large JSON datasets directly through API Gateway responses ($0.09/GB egress) rather than serving them via Amazon S3 / CloudFront.

---

## 7. Actionable Cost Optimization Strategies
1. **Migrate Simple APIs to HTTP APIs:** Review active APIs. For endpoints acting primarily as proxies to Lambda functions or internal VPC targets without requiring complex XML/JSON mapping templates or native WAF, migrate to **HTTP APIs** to achieve an immediate **71% request cost reduction**.
2. **Disable Caching on Non-Production Stages:** Audit all REST API stages. Disable caching on dev, test, and staging environments immediately.
3. **Use Amazon CloudFront for API Caching & Data Egress:**
   * Deploy **Amazon CloudFront** in front of API Gateway.
   * *Benefits:* CloudFront offers lower bandwidth egress rates ($0.085/GB vs $0.09/GB baseline), a 1 TB/month free tier, and cost-effective edge caching without fixed hourly API Gateway cache instance charges.
4. **Enforce Idle Disconnects on WebSocket APIs:** Configure WebSocket integrations to automatically terminate connections that have been idle for a designated period (e.g., 10–15 minutes).
5. **Return Large Payloads via S3 Presigned URLs:** For file downloads or large reports (>1 MB), generate an **S3 Presigned URL** in your API response and let clients download directly from S3.
