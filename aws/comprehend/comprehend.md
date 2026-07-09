# AWS Service Cost Research: Amazon Comprehend

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Comprehend is a natural language processing (NLP) service that uses machine learning to extract insights and relationships from unstructured text. It offers pre-trained Standard APIs (for sentiment, entities, key phrases, syntax, and language detection) and Custom APIs (allowing you to train proprietary classification and entity recognition models). Because custom real-time endpoints require dedicated compute capacity, they represent a significant fixed-cost risk if left idle.

---

## 2. Billing Mechanics
Amazon Comprehend bills monthly based on the volume of text analyzed, model training runtimes, and provisioned endpoint capacity:
1.  **Text Ingestion Units:** Billed per 100-character unit processed (UTF-8). Every request has a **3-unit (300 characters) minimum** charge.
2.  **Custom Model Training:** Billed per hour of active model training compute.
3.  **Custom Real-Time Endpoints (Inference Units):** Billed per second for provisioned endpoint capacity (Inference Units - IU), regardless of whether queries are active.
4.  **Custom Model Storage:** Charged a low monthly flat fee for storing custom trained model artifacts.

---

## 3. Key Cost Dimensions

### A. Standard NLP Ingestion (us-east-1 Volume Tiers)
*   **Tier 1 (First 10 Million units / month):** **$0.00010 per unit** ($1.00 per Million characters).
*   **Tier 2 (10 Million to 50 Million units / month):** **$0.00005 per unit** ($0.50 per Million characters).
*   **Tier 3 (Over 50 Million units / month):** **$0.000025 per unit** ($0.25 per Million characters).
*   *Note: Standard APIs include Key Phrase Extraction, Sentiment Analysis, Syntax Analysis, Entity Resolution, and Language Detection. Charges accumulate per API invoked.*

### B. Custom Comprehend (Model Training & Storage)
*   **Model Training:** **$3.00 per hour** of active compute training time.
*   **Model Storage:** Flat **$0.50 per model-month** for holding the trained model artifact.

### C. Custom Real-Time Endpoints (Provisioned Inference Units)
*   **Inference Unit (IU) Capacity:** 1 IU handles a processing throughput of up to 100 characters per second.
*   **Pricing:** **$0.0005 per IU per second** (equivalent to **$1.80 per hour**).
*   *Fixed Cost Danger:* Custom endpoints do **not autoscaling to 0**. Running a single custom endpoint continuously for a month costs:
    $$\text{Monthly Cost} = \$1.80 \times 730\text{ hours} = \$1,314.00\text{ / month flat per IU}$$

### D. Asynchronous Batch Processing
*   For bulk data processing, Custom Models can be run asynchronously without provisioning real-time endpoints.
*   Billed purely by the characters processed (using standard ingestion unit pricing) plus the model training/storage fees, eliminating idle compute charges.

---

## 4. Detailed Pricing Rates (us-east-1)

| Comprehend Feature / Metric | Billed Unit | Rate (us-east-1) | Price per Million Characters / Month |
|-----------------------------|-------------|------------------|--------------------------------------|
| **Standard NLP (Tier 1)** | Per 100-character unit | **$0.00010** | **$1.00** (with 300-character request minimum) |
| **Standard NLP (Tier 2)** | Per 100-character unit | **$0.00005** | **$0.50** |
| **Standard NLP (Tier 3)** | Per 100-character unit | **$0.000025** | **$0.25** |
| **Custom Model Training** | Per training hour | **$3.00000** | $3.00 / hour |
| **Custom Model Storage** | Per model-month | **$0.50000** | $0.50 / month |
| **Custom Real-Time IU** | Per IU per second | **$0.00050** | **$1,314.00 / month flat per IU** |

### Example Monthly Cost Calculation
*Workload: A pipeline uses a Custom Model to analyze customer transcripts in real-time. It runs a single custom endpoint (1 IU) continuously. Over the month, it processes 5,000,000 characters of text.*

*   **Custom Endpoint Fee:**
    $$\text{Endpoint Cost} = \$1.80/\text{hour} \times 730\text{ hours} = \$1,314.00$$
*   **Ingestion Fee:**
    $$\text{Ingestion Units} = \frac{5,000,000\text{ characters}}{100\text{ characters/unit}} = 50,000\text{ units}$$
    $$\text{Ingestion Cost (Tier 1)} = 50,000 \times \$0.00010 = \$5.00$$
*   **Total Cost:** **$1,319.00/month** (Note: $1,314.00 of this bill is pure idle provisioning cost).

---

## 5. AWS Free Tier Coverage
*   **Standard APIs:** **50,000 units** (5 Million characters) free per month for **each** individual Standard NLP API (e.g. 5M chars of Sentiment + 5M chars of Entity Extraction) for the first 12 months.
*   **Custom Model APIs:** No free tier available for Custom Model training, storage, or endpoints.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Zombie Custom Inference Endpoints:** Leaving custom real-time endpoints provisioned in non-production environments (development, QA, staging). Each active endpoint bleeds **$1,314.00/month** even if 0 requests are processed.
*   **Short String Surcharges:** Submitting small payloads (e.g. 10-character chat messages) in high frequencies. Due to the **300-character request minimum**, you are billed for 30x the character volume.
*   **Sending Raw Log Metadata:** Submitting system logs, JSON structures, HTML/CSS markup tags, or repeating boilerplate templates to the API, inflating ingestion unit volume.

---

## 7. Actionable Cost Optimization Strategies
1.  **De-provision Real-Time Endpoints (Migrate to Batch Jobs):**
    *   For non-real-time workloads (e.g., overnight report analysis, batch migration of transcripts), delete Custom Endpoints.
    *   Deploy **Comprehend Asynchronous Batch Jobs** (`StartDocumentClassificationJob` or `StartEntitiesDetectionJob`) which run against files in S3 and auto-shutdown upon completion.
    *   **The Savings:** Saves **100% of the $1,314.00/month** idle endpoint charge per model, paying only for the exact characters processed.
2.  **Combine Short Text Inputs Locally:**
    *   Stitch multiple small text strings (e.g., individual customer feedback comments) into larger documents (up to 100 KB) separated by line breaks or delimiter symbols.
    *   Send the combined document in a single API call.
    *   **The Savings:** Avoids the 300-character minimum surcharge, saving up to **66%** on high-frequency short-string runs.
3.  **Pre-Clean Document Text:**
    *   Strip out markup tags, system logs, dates, and repetitive footers locally (using local Python regex tools) before making the API call.
    *   **The Savings:** Lowers ingestion unit counts at the **$1.00/Million** character rate.
4.  **Auto-Teardown Staging Endpoints:**
    *   If real-time staging endpoints are required for testing, automate their deployment and teardown (e.g. using a Lambda cron function to create them at 9 AM and delete them at 5 PM).
    *   **The Savings:** Saves **66%** of staging endpoint costs ($440/mo vs $1,314/mo).
