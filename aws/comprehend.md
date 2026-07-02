# AWS Service Cost Research: Amazon Comprehend

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Comprehend is a natural language processing (NLP) service that uses machine learning to find insights and relationships in unstructured text. It can identify key phrases, entities, sentiment, language, and topics. Comprehend offers Standard APIs (pre-trained general models) and Custom APIs (where you train classification or entity recognition models). Because custom real-time endpoints require dedicated compute, they represent a significant fixed cost risk.

---

## 2. Billing Mechanics
Amazon Comprehend bills monthly based on the text volume analyzed and the model type:
1.  **Text Ingestion Units:** Billed per 100-character unit processed, with a **3-unit (300 characters) minimum** per request.
2.  **Custom Model Training:** Billed per hour of active model training compute.
3.  **Custom Real-Time Endpoints (Inference Units):** Billed per second for provisioned endpoint capacity (Inference Units), regardless of whether queries are active.

---

## 3. Key Cost Dimensions

### A. Standard NLP Pricing (us-east-1 Tiers)
*   **The Ingestion Unit:** Billed in units of **100 characters** (UTF-8).
*   **First Tier (First 10 Million units / month):** **$0.0001 per unit** ($1.00 per Million characters).
*   **Second Tier (10 Million to 50 Million units):** **$0.00005 per unit** ($0.50 per Million characters).
*   **Third Tier (Over 50 Million units):** **$0.000025 per unit** ($0.25 per Million characters).
*   *Minimum Billing Surcharge:* Every request is rounded up to at least **3 units (300 characters)**. Sending a 10-character string is billed as 300 characters.

### B. Custom Comprehend (Real-Time Endpoints)
If you train a custom model to identify proprietary entity classes:
*   **Training Compute:** Billed at **$3.00 per hour** of training time.
*   **Endpoint Capacity (Inference Units - IU):** 1 IU represents a processing throughput of up to 100 characters per second.
    *   *Rate:* **$0.0005 per IU per second** (equivalent to **$1.80 per hour**).
    *   *The Fixed Cost Risk:* Custom endpoints do **not scale to 0** when idle. Running a single custom endpoint continuously for a month costs:
        $$\$1.80 \times 730\text{ hours} = \$1,314.00\text{ / month flat per endpoint}$$

---

## 4. Detailed Pricing Rates (us-east-1)

| Comprehend Service | Billing Metric | Rate (us-east-1) | Price per Million characters / Month equivalent |
|--------------------|----------------|------------------|------------------------------------------------|
| **Standard NLP** | Per 100 characters | **$0.00010** | **$1.00** (with a 300-char minimum/req) |
| **Custom Model Train**| Per training hour | **$3.00000** | $3.00 / hour |
| **Custom Endpoint** | Per IU per second | **$0.00050** | **$1,314.00 / month flat per IU** |
| **Custom Model Store**| Flat fee per model | **$0.50000** | $0.50 / model-month |

---

## 5. AWS Free Tier Coverage
*   **Standard APIs:** **50,000 units** (5 Million characters) free per month per API for the first 12 months.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Zombie Custom Inference Endpoints:** Leaving custom real-time endpoints running in development or test environments. Each active endpoint bleeds **$1,314.00/month** even if 0 requests are processed.
*   **High-Frequency Short String Requests:** Calling Comprehend Standard APIs for tiny strings (e.g. 5-character chat messages) in high volumes. Due to the **300-character minimum charge**, you are billed for 30x the character volume.
*   **Analyzing Unfiltered Logs:** Sending raw log directories containing system codes, timestamps, and stack traces to Comprehend, inflating character unit ingestion fees.

---

## 7. Actionable Cost Optimization Strategies
1.  **De-provision Custom Endpoints (Use Batch Instead):**
    *   For staging, development, or non-real-time business reports, do not use Custom Endpoints.
    *   Use **Comprehend Custom Batch Jobs** instead. Batch jobs spin up compute dynamically, process files in S3, and shut down automatically when finished.
    *   **The Savings:** Bypasses the **$1,314.00/month** idle endpoint charge completely, paying only for the exact characters processed.
2.  **Combine Short Text Strings Locally:**
    *   If you have thousands of short text inputs (e.g. user comments), do not send them individually.
    *   Stitch multiple strings together into larger documents (e.g. combining 20 comments separated by line breaks) before calling the Standard APIs.
    *   **The Savings:** Bypasses the 300-character minimum surcharge, saving up to **66%** on high-frequency short-string runs.
3.  **Pre-Filter Text Inputs:** Strip out formatting metadata, HTML elements, repeating footers, and system logs locally using regex before sending payloads to Comprehend.
4.  **Audit and Delete Inactive Custom Models:** Prune custom classification models that are no longer actively referenced by pipelines.
5.  **Shutdown Staging Endpoints Automatically:** If real-time staging endpoints are mandatory, write a scheduled script to deploy the endpoint at 9 AM and delete it at 5 PM.
