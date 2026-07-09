# AWS Service Cost Research: AWS Entity Resolution

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Entity Resolution is a machine learning-powered service that helps you match, link, and deduplicate related records across different data sources. You can use rule-based matching or advanced machine learning models to link customer records, product catalogs, or business datasets (e.g., matching "John Smith" and "J. Smith" at the same address). AWS Entity Resolution is billed purely on a pay-as-you-go model based on the volume of data records processed.

---

## 2. Billing Mechanics
Entity Resolution billing is calculated on a monthly cycle based on the number of records processed:
1.  **Rule-Based or Machine Learning-Based Matching:** Billed at a flat rate of **$0.25 per 1,000 records processed** ($250.00 per million records). There are no volume tiers or discounts.
2.  **Data Service Provider Matching (Third-Party):** Billed at a flat rate of **$0.10 per 1,000 records processed** ($100.00 per million records) when using third-party identity providers (such as LiveRamp or TransUnion) linked via AWS Data Exchange.

---

## 3. Key Cost Dimensions

### A. Rule-Based & ML-Based Matching Rates
*   **Definition of a Record:** A record represents a single row of input data, regardless of the number of columns.
*   **Standard Rate:** **$0.25 per 1,000 records** ($250.00 per million).

### B. Data Service Provider Matching
*   **The Rate:** Billed at a flat rate of **$0.10 per 1,000 records** ($100.00 per million).
*   *Provider Surcharges:* This fee is strictly for AWS processing. The data provider (e.g., LiveRamp) will bill their own subscription fees separately through AWS Data Exchange.

### C. Incremental Machine Learning Matching
*   Introduced in May 2026, EKS and S3 integrations now support **Incremental Machine Learning-based matching workflows**.
*   This feature allows the workflow to identify and process only *new or updated records* since the last run, rather than reprocessing the entire historical dataset. This significantly reduces active record metrics and lowers matching costs for recurring pipelines.

---

## 4. Detailed Pricing Rates (us-east-1)

| Matching Workflow Type | Free Tier Allowance | Rate per 1,000 Records | Price per Million Records |
|-------------------------|---------------------|------------------------|---------------------------|
| **Rule-Based Matching** | None | **$0.25** | **$250.00** |
| **ML-Powered Matching** | None | **$0.25** | **$250.00** |
| **Provider Matching** | None | **$0.10** | **$100.00** |

### Example Monthly Cost Calculation (Delta vs Full Reprocessing)
*Workload: A database contains 5 million historical customer records. Each month, 100,000 new customer records are added. We compare running a full monthly re-match vs. using the new Incremental ML-based matching feature.*

*   **Option A: Full Reprocessing (Re-running all records each month):**
    $$\text{Total Records} = 5,000,000\text{ records}$$
    $$\text{Monthly Cost} = \frac{5,000,000}{1,000} \times \$0.25 = \$1,250.00\text{ / month}$$
*   **Option B: Incremental Matching (Only processing new additions):**
    $$\text{Total Records} = 100,000\text{ records}$$
    $$\text{Monthly Cost} = \frac{100,000}{1,000} \times \$0.25 = \$25.00\text{ / month}$$
*   **Net Savings:** **$1,225.00/month** (A **98% cost reduction** using the incremental feature).

---

## 5. AWS Free Tier Coverage
*   **AWS Entity Resolution:** **No free tier** is available. All record processing tasks generate standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Full Database Reprocessing on Every Run:** Running your entire customer dataset through Entity Resolution on a daily or weekly schedule instead of using delta processing.
*   **Processing Uncleaned Input Files:** Running raw, unfiltered target files containing empty rows, duplicate inputs, or irrelevant metadata columns.
*   **Provider Matching without Licensing Budget:** Initiating data provider matching tasks on millions of rows without analyzing the provider's subscription fee structures.

---

## 7. Actionable Cost Optimization Strategies
1.  **Implement Incremental Matching (Delta Processing):**
    *   Do not reprocess historical data. Use the **Incremental Machine Learning** matching feature to process only new or modified customer records added since the previous run.
    *   **The Savings:** Cuts processed record counts by **90–98%**, saving thousands of dollars per run.
2.  **Filter and Deduplicate Locally Before Ingesting:**
    *   Before sending data to AWS Entity Resolution, clean the file locally using SQL or python scripts.
    *   Remove empty rows, null fields, and exact duplicates (e.g., identical email addresses).
3.  **Evaluate Rule-Based Matching First:**
    *   Start by running a cheap Rule-Based matching workflow (exact match on phone number or email) to clear the bulk of duplicates.
    *   Only pass the remaining "fuzzy" unmatched rows to the ML-based model.
4.  **Enforce S3 Lifecycle Rules on Raw Input Files:** Keep raw input files in S3 Glacier or delete them once the matching output has been written to your target database.
