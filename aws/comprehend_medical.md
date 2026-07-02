# AWS Service Cost Research: Amazon Comprehend Medical

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Comprehend Medical is a HIPAA-eligible natural language processing (NLP) service that uses machine learning to extract medical information from unstructured clinical text. It can identify entities such as medical conditions, medications, dosages, procedures, and protected health information (PHI). Comprehend Medical also links extracted terms to standardized medical vocabularies (ontologies) like ICD-10-CM, RxNorm, and SNOMED-CT. It is serverless, billing purely based on the volume of text analyzed.

---

## 2. Billing Mechanics
Amazon Comprehend Medical bills monthly based on the text volume processed:
1.  **Text Ingestion Units:** Billed per 100-character unit (UTF-8) analyzed.
2.  **Minimum Request Charge:** Each API request has a **1-unit (100 characters) minimum** charge.
3.  **Tiered Discounts:** Pricing is tiered based on monthly text volume, resetting at the beginning of each calendar month.
4.  **Feature Split:** Pricing varies significantly based on the API operation (e.g. basic PHI detection vs. complex SNOMED-CT ontology linking).

---

## 3. Key Cost Dimensions

### A. Medical Named Entity Recognition (NERe)
Extracts medications, medical conditions, test results, and anatomy from text.
*   **Up to 1 Million units/month:** **$0.0100 per unit** ($100.00 per Million characters).
*   **1 Million to 2 Million units/month:** **$0.0050 per unit** ($50.00 per Million characters).
*   **Over 2 Million units/month:** **$0.0010 per unit** ($10.00 per Million characters).

### B. PHI Detection (Protected Health Information)
Locates and flags patient names, phone numbers, email addresses, and dates to assist with de-identification.
*   **Up to 1 Million units/month:** **$0.0014 per unit** ($14.00 per Million characters).
*   **1 Million to 2 Million units/month:** **$0.0007 per unit** ($7.00 per Million characters).
*   **Over 2 Million units/month:** **$0.00028 per unit** ($2.80 per Million characters).

### C. Ontology Linking APIs
Resolves medical text to standardized industry databases.
*   **SNOMED-CT Linking (Conditions, anatomy, procedures):**
    *   *Up to 1M units:* **$0.00750 per unit** ($75.00 per Million characters).
    *   *1M to 2M units:* **$0.00375 per unit**.
    *   *Over 2M units:* **$0.00075 per unit**.
*   **ICD-10-CM (Diagnosis Codes) & RxNorm (Medications) Linking:**
    *   *Up to 1M units:* **$0.00050 per unit** ($5.00 per Million characters).
    *   *1M to 2M units:* **$0.00050 per unit**.
    *   *Over 2M units:* **$0.00025 per unit**.

---

## 4. Detailed Pricing Rates (us-east-1 Tier 1: 0 to 1M units)

| API Operation | Cost per 100-Char Unit | Cost per 10,000 Characters | Price per Million Characters |
|---------------|-------------------------|----------------------------|------------------------------|
| **ICD-10-CM / RxNorm**| **$0.00050** | $0.05 | **$5.00** |
| **PHI Detection** | **$0.00140** | $0.14 | **$14.00** |
| **SNOMED-CT** | **$0.00750** | $0.75 | **$75.00** |
| **NERe (Standard)** | **$0.01000** | $1.00 | **$100.00** |

---

## 5. AWS Free Tier Coverage
*   **Comprehend Medical Free Tier:** **85,000 units** (8.5 Million characters) free for the first month of usage.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Double Billing by Calling Multiple APIs on the Same Text:** Running a medical transcript through the NERe API ($100/M chars) and then running it again through the SNOMED-CT API ($75/M chars) separately.
*   **Sending Non-Clinical Structural Metadata:** Including PDF formatting codes, headers, footers, system telemetry logs, or long boilerplate hospital agreements in your API payloads.
*   **Processing Empty or Short Keystrokes:** Submitting clinical notes page-by-page or line-by-line as they are typed. Due to the **1-unit (100 characters) minimum**, sending a 10-character word is billed as 100 characters.

---

## 7. Actionable Cost Optimization Strategies
1.  **Stitch Short Transcripts Prior to API Ingestion:**
    *   If you receive short medical logs (e.g. 15-character prescriptions), do not invoke the API for each one.
    *   Concatenate multiple short inputs into a single text block (ideally ~1,000 characters) separated by line breaks.
    *   **The Savings:** Bypasses the 100-character request minimum, cutting charges by up to **85%**.
2.  **Combine Extraction and Linking Tasks (Use Direct Linking APIs):**
    *   Instead of calling the standard NERe API ($0.01/unit) and then calling an ontology linking API ($0.0075/unit) on the outputs, call the **Ontology Linking APIs directly**.
    *   The ICD-10-CM or SNOMED-CT linking APIs perform entity recognition *and* code assignment in a single pass.
    *   **The Savings:** Avoids duplicate character billing, cutting fees in half.
3.  **Pre-Clean Medical Documents Locally:**
    *   Use local Python scripts or regex to redact obvious boilerplate (e.g. "CONFIDENTIAL MEDICAL RECORD DO NOT DISTRIBUTE") and remove HTML markup tags.
    *   This prevents paying the high **$100.00/Million** character rate on redundant non-clinical text.
4.  **Use PHI Detection Only Where Necessary:** If your target database is already hosted in a secure, HIPAA-compliant server and you do not need to share data externally, bypass the PHI Detection API to save $14.00/Million characters.
5.  **Utilize Batch Invocations for Archives:** For asynchronous bulk medical record migrations, use Comprehend Medical Batch APIs (`StartMedicalTranscriptionJob`) to process S3 directories.
