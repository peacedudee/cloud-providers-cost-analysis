# AWS Service Cost Research: AWS Entity Resolution

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Entity Resolution is a machine learning-powered service that helps you match, link, and deduplicate related records across different data sources. You can use rule-based matching or advanced machine learning models to link customer records, product catalogs, or business datasets (e.g., matching "John Smith" and "J. Smith" at the same address). AWS Entity Resolution is billed purely on a pay-as-you-go model based on the volume of data records processed.

---

## 2. Billing Mechanics
Entity Resolution billing is calculated on a monthly cycle based on the number of records processed:
1.  **Rule-Based or Machine Learning-Based Matching:** Billed per 1,000 records processed, with tiered discounts for large monthly volumes.
2.  **Data Service Provider Matching (Third-Party):** Billed per 1,000 records processed when using third-party identity providers (such as LiveRamp or TransUnion) linked via AWS Data Exchange.

---

## 3. Key Cost Dimensions

### A. Rule-Based & ML-Based Matching (us-east-1 Tiers)
*   **Definition of a Record:** A record represents a single row of input data, regardless of the number of columns.
*   **Tiered Ingestion Rates:**
    *   *First 1 Million records/month:* **$0.25 per 1,000 records** ($250.00 per million).
    *   *Next 9 Million records/month:* **$0.15 per 1,000 records** ($150.00 per million).
    *   *Beyond 10 Million records/month:* **$0.05 per 1,000 records** ($50.00 per million).

### B. Data Service Provider Matching
*   **The Rate:** Billed at a flat rate of **$0.10 per 1,000 records** ($100.00 per million).
*   *Provider Surcharges:* This fee is strictly for AWS processing. The data provider (e.g. LiveRamp) will bill their own subscription fees separately through AWS Data Exchange.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cumulative Records Processed / Month | Rule/ML Matching Rate (per 1K) | Provider Matching Rate (per 1K) | Price per Million (Rule/ML) |
|--------------------------------------|--------------------------------|---------------------------------|-----------------------------|
| **0 to 1 Million** | **$0.2500** | $0.1000 | **$250.00** |
| **1 to 10 Million** | **$0.1500** | $0.1000 | **$150.00** |
| **10 Million+** | **$0.0500** | $0.1000 | **$50.00** |

---

## 5. AWS Free Tier Coverage
*   **AWS Entity Resolution:** **No free tier** is available. All record processing tasks generate standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Full Database Reprocessing on Every Run:** Running your entire customer dataset (e.g., 5 million rows) through Entity Resolution daily to check for new entries. Reprocessing 5 million rows daily costs **$25,500.00/month**.
*   **Processing Uncleaned Input Files:** Running raw, unfiltered target files containing empty rows, duplicate inputs, or irrelevant metadata columns.
*   **Provider Matching without licensing budget:** Initiating data provider matching tasks on millions of rows without analyzing the provider's subscription fee structures.

---

## 7. Actionable Cost Optimization Strategies
1.  **Implement Incremental Matching (Delta Processing):**
    *   Do not reprocess historical data.
    *   Set up your data pipelines to separate new or modified customer records (deltas) from the primary database.
    *   Run only these new records through AWS Entity Resolution using the **Incremental Matching** feature.
    *   **The Savings:** Reduces the record count processed by 90–98%, dropping monthly bills from thousands to dollars.
2.  **Filter and Deduplicate Locally Before Ingesting:**
    *   Before sending data to AWS Entity Resolution, clean the file locally using SQL or simple Python scripts.
    *   Remove empty rows, null fields, and exact duplicates (e.g., identical email addresses).
    *   Only upload unique, unresolved candidate rows for advanced matching.
3.  **Evaluate Rule-Based Matching first:**
    *   ML-powered matching requires heavier compute than simple deterministic rules.
    *   Start by running a cheap **Rule-Based** matching workflow (e.g. exact match on phone number or email) to clear the bulk of duplicates.
    *   Only pass the remaining "fuzzy" unmatched rows to the ML-based model.
4.  **Enforce S3 Lifecycle Rules on Raw Input Files:** Keep raw input files in S3 Glacier or delete them once the matching output has been written to your target database.
