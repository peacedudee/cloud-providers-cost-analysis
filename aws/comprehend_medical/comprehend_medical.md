# AWS Service Cost Research: Amazon Comprehend Medical

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Comprehend Medical is a HIPAA-eligible natural language processing (NLP) service that uses machine learning to extract medical information from unstructured clinical text. It can identify entities such as medical conditions, medications, dosages, procedures, and protected health information (PHI). Comprehend Medical also links extracted terms to standardized medical vocabularies (ontologies) like ICD-10-CM, RxNorm, and SNOMED-CT. It is serverless, billing purely based on the volume of text analyzed.

---

## 2. Billing Mechanics
Amazon Comprehend Medical bills monthly based on the text volume processed:
1.  **Text Ingestion Units:** Billed per 100-character unit (UTF-8) analyzed.
2.  **Minimum Request Charge:** Each API request has a **1-unit (100 characters) minimum** charge.
3.  **Tiered Discounts:** Standard pricing is tiered based on monthly text volume per API (NERe, PHI, ICD-10-CM, SNOMED-CT), resetting at the beginning of each calendar month. RxNorm is billed at a flat rate regardless of volume.
4.  **Feature Split:** Pricing varies significantly based on the API operation (e.g. basic RxNorm ontology linking vs. complex NERe named entity recognition).

---

## 3. Key Cost Dimensions

### A. Medical Named Entity Recognition (NERe)
Extracts medications, medical conditions, test results, and anatomy from text.
*   **Up to 1 Million units/month:** **$0.0100 per unit** ($100.00 per Million characters).
*   **1 Million to 2 Million units/month:** **$0.0050 per unit** ($50.00 per Million characters).
*   **Over 2 Million units/month:** **$0.0010 per unit** ($10.00 per Million characters).

### B. PHI Detection (Protected Health Information)
Locates and flags patient names, phone numbers, email addresses, and dates to assist with de-identification.
*   **Up to 1 Million units/month:** **$0.00140 per unit** ($14.00 per Million characters).
*   **1 Million to 2 Million units/month:** **$0.00050 per unit** ($5.00 per Million characters).
*   **Over 2 Million units/month:** **$0.00025 per unit** ($2.50 per Million characters).

### C. Ontology Linking APIs
Resolves medical text to standardized industry databases.
*   **SNOMED-CT Linking (Conditions, anatomy, procedures):**
    *   *Up to 1M units:* **$0.00750 per unit** ($75.00 per Million characters).
    *   *1M to 2M units:* **$0.00375 per unit**.
    *   *Over 2M units:* **$0.00075 per unit**.
*   **ICD-10-CM Linking (Diagnosis Codes):**
    *   *Up to 1M units:* **$0.00050 per unit** ($5.00 per Million characters).
    *   *1M to 2M units:* **$0.00050 per unit**.
    *   *Over 2M units:* **$0.00025 per unit**.
*   **RxNorm Linking (Medication Codes):**
    *   **Flat $0.00025 per unit** ($2.50 per Million characters) across all volume tiers.

---

## 4. Detailed Pricing Rates (us-east-1 Tier 1: 0 to 1M units)

| API Operation | Cost per 100-Char Unit | Cost per 10,000 Characters | Price per Million Characters |
|---------------|-------------------------|----------------------------|------------------------------|
| **RxNorm** | **$0.00025** | $0.025 | **$2.50** |
| **ICD-10-CM** | **$0.00050** | $0.05 | **$5.00** |
| **PHI Detection** | **$0.00140** | $0.14 | **$14.00** |
| **SNOMED-CT** | **$0.00750** | $0.75 | **$75.00** |
| **NERe (Standard)** | **$0.01000** | $1.00 | **$100.00** |

### Example Monthly Cost Calculation
*Workload: A clinic processes 1.5 Million units (150 Million characters) of clinical notes. The notes are processed through both PHI Detection (to de-identify) and RxNorm Linking (to extract and code medications).*

1.  **PHI Detection Cost:**
    *   First 1,000,000 units:
        $$1,000,000 \times \$0.00140 = \$1,400.00$$
    *   Next 500,000 units (Tier 2):
        $$500,000 \times \$0.00050 = \$250.00$$
    *   *Total PHI Cost:* **$1,650.00**

2.  **RxNorm Linking Cost:**
    *   Flat rate of $0.00025 per unit across all 1,500,000 units:
        $$1,500,000 \times \$0.00025 = \$375.00$$
    *   *Total RxNorm Cost:* **$375.00**

*   **Total Monthly Cost:** **$2,025.00/month**

---

## 5. AWS Free Tier Coverage
*   **Comprehend Medical Free Tier:** **85,000 units** (8.5 Million characters) free for the first month of usage, applying to both new and existing AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Double Billing via Redundant API Runs:** Running clinical notes through standard NERe ($100/M chars) and then running the same text through SNOMED-CT ($75/M chars) or RxNorm ($2.50/M chars) separately.
*   **Sending Non-Clinical Structural Metadata:** Including PDF formatting codes, HTML elements, repeating headers, footers, system telemetry logs, or long boilerplate hospital agreements in your API payloads.
*   **Processing Empty or Short Keystrokes:** Submitting clinical notes page-by-page or line-by-line. Due to the **1-unit (100 characters) minimum**, sending a 10-character word is billed as 100 characters.

---

## 7. Actionable Cost Optimization Strategies
1.  **Call Specific Ontology Linking APIs Directly:**
    *   Instead of calling standard NERe ($0.01/unit) and then calling an ontology linking API on the output, call **RxNorm Linking ($0.00025/unit)** or **ICD-10-CM Linking ($0.0005/unit)** directly.
    *   The linking APIs perform entity recognition *and* code mapping in a single pass.
    *   **The Savings:** By calling RxNorm Linking directly instead of NERe + RxNorm, you save **97.5%** on medication extraction.
2.  **Stitch Short Transcripts Prior to Ingestion:**
    *   Concatenate multiple short text inputs (e.g. doctor voice transcripts, short prescriptions) into a single larger block (up to 1,000 characters) separated by delimiters before calling the API.
    *   **The Savings:** Bypasses the 100-character request minimum, cutting charges by up to **85%**.
3.  **Pre-Clean Medical Documents Locally:**
    *   Use local scripts or regex to strip out non-clinical text (e.g. confidential notices, hospital footers, HTML tags) locally before ingestion.
    *   **The Savings:** Lowers unit count billed at the high standard rate.
4.  **Use PHI Detection Only Where Necessary:**
    *   If data remains strictly inside a HIPAA-compliant DB and is not shared with third parties, bypass PHI detection to save $14.00/Million characters.
5.  **Utilize Batch Invocations for Archives:**
    *   For bulk historical records, use Asynchronous Batch APIs (`StartMedicalTranscriptionJob`) instead of real-time streaming, allowing for larger block packing and predictable execution.
