# AWS Service Cost Research: Amazon Macie

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Macie is a fully managed data security and privacy service that uses machine learning and pattern matching to discover and protect sensitive data in Amazon S3. Macie automatically inventorying S3 buckets, detecting unencrypted or publicly accessible buckets, and scanning object contents for Protected Health Information (PHI), Personally Identifiable Information (PII), credit cards, and API keys. Macie is serverless, billing based on bucket inventory counts and volume of data scanned.

---

## 2. Billing Mechanics
Amazon Macie charges are calculated monthly across three main dimensions:
1.  **S3 Bucket Monitoring:** Billed per bucket per month for evaluating bucket-level security configurations (public access, encryption, policy exposure).
2.  **Automated Sensitive Data Discovery (Sampling):** Billed per 100,000 objects discovered and per GB of sampled data scanned.
3.  **Targeted Discovery Jobs (On-Demand Deep Scans):** Billed per GB of uncompressed data scanned during deep inspection jobs.

---

## 3. Key Cost Dimensions

### A. Bucket Assessment & Automated Discovery (us-east-1)
*   **S3 Bucket Inventory & Assessment:** **$0.10 per bucket per month** (prorated daily).
*   **Automated Sensitive Data Discovery (Continuous Machine Learning Sampling):**
    *   *Object Discovery Fee:* **$0.01 per 100,000 S3 objects** evaluated per month.
    *   *Sample Data Inspection Fee:* **$0.10 per GB** of sampled object content scanned.

### B. Targeted Discovery Jobs (On-Demand Deep Scans)
When you manually configure a job to scan specific S3 buckets:
*   **First 10,000 GB (10 TB) / month:** **$1.00 per GB** evaluated.
*   **Next 40,000 GB (10 TB to 50 TB) / month:** **$0.50 per GB** evaluated.
*   **Above 50,000 GB (50 TB+) / month:** **$0.25 per GB** evaluated.
*   *Math Trap Warning:* Scanning a single 10 TB bucket filled with uncompressed data costs:
    $$10,000\text{ GB} \times \$1.00 = \$10,000.00\text{ / month!}$$

---

## 4. Detailed Pricing Rates (us-east-1)

| Macie Feature | Billing Unit | Rate (us-east-1) | Price for 100 GB / Buckets |
|---------------|--------------|------------------|----------------------------|
| **Bucket Assessment** | Per bucket-month | **$0.10** | $10.00 / 100 buckets/mo |
| **Automated Discovery Scan**| Per GB sampled | **$0.10** | **$10.00** |
| **Targeted Job (Tier 1)** | Per GB scanned | **$1.00** | **$100.00** |
| **Targeted Job (Tier 2)** | Per GB scanned | **$0.50** | $50.00 |

---

## 5. AWS Free Tier Coverage
*   **Macie 30-Day Free Trial:** Includes a 30-day free trial for all new accounts.
    *   *Includes:* 30 days of free bucket inventory evaluation and **1 GB of free data inspection** per account per month.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Scanning Uncompressed Media / Binary Datasets (The $10,000 Mistake):**
    *   Targeting Macie discovery jobs on S3 buckets containing raw video files (`.mp4`), disk images (`.iso`), or compressed archives (`.zip`).
    *   Macie decompresses archives and scans every byte ($1.00/GB), incurring massive fees on files that never contain text PII.
*   **Running Full Targeted Jobs When Automated Sampling Suffices:** Scheduling daily targeted deep scans ($1.00/GB) on fast-growing data lakes instead of enabling Automated Sensitive Data Discovery ($0.10/GB).

---

## 7. Actionable Cost Optimization Strategies
1.  **Configure File Extension Filters on All Discovery Jobs:**
    *   Go to Macie Discovery Job settings -> **Include/Exclude Rules**.
    *   Explicitly include text-based formats: `.csv`, `.json`, `.txt`, `.pdf`, `.parquet`, `.tsv`, `.doc`.
    *   Explicitly exclude media and binary formats: `.png`, `.jpg`, `.jpeg`, `.mp4`, `.mp3`, `.zip`, `.tar`, `.gz`, `.iso`.
    *   **The Savings:** Slashes data inspection volume by **90–99%** on data lakes.
2.  **Use Automated Sensitive Data Discovery ($0.10/GB) Over Targeted Jobs ($1.00/GB):**
    *   Enable **Automated Sensitive Data Discovery** across your organization.
    *   It uses smart statistical sampling to scan small representative portions of objects across all buckets.
    *   **The Savings:** Provides continuous PII visibility at **1/10th the cost** of full targeted scans.
3.  **Use the 30-Day Free Trial to Forecast Data Volumes:** Review the Macie console **Estimated Cost** panel during the trial period to identify top candidate buckets for exclusion before launching paid scans.
4.  **Exclude Archival / Public Datasets:** Exclude static public web asset buckets (e.g. static website images) and S3 Glacier Deep Archive buckets from Macie job scopes.
