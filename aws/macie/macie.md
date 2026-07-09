# AWS Service Cost Research: Amazon Macie

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Macie is a fully managed data security and privacy service that uses machine learning and pattern matching to discover and protect sensitive data in Amazon S3. Macie automatically inventories S3 buckets, evaluates bucket-level access policies (detecting unencrypted or publicly accessible buckets), and scans object contents for Protected Health Information (PHI), Personally Identifiable Information (PII), credit cards, and API keys. Macie is serverless, billing based on bucket inventory counts, monitored object counts, and volume of data scanned.

---

## 2. Billing Mechanics
Amazon Macie charges are calculated monthly across three main dimensions:
1. **S3 Bucket Inventory & Assessment:** Billed per bucket per month for evaluating security configurations (public access, encryption, policy exposure).
2. **Automated Sensitive Data Discovery (Sampling):** Billed per 100,000 objects monitored and per GB of sampled data inspected.
3. **Targeted Discovery Jobs (On-Demand Deep Scans):** Billed per GB of uncompressed data scanned during deep inspection jobs (with volume tier discounts above 10 TB).

---

## 3. Key Cost Dimensions

### A. Bucket Assessment & Automated Discovery (us-east-1)
* **S3 Bucket Inventory & Assessment:** **$0.10 per bucket per month** (prorated daily).
* **Automated Sensitive Data Discovery (Continuous Sampling):**
  * *Object Monitoring Fee:* **$0.01 per 100,000 S3 objects** evaluated per month.
  * *Sample Data Inspection Fee:* **$1.00 per GB** of sampled object content inspected.

### B. Targeted Discovery Jobs (On-Demand Deep Scans)
When you configure targeted jobs to scan specific S3 buckets:
* **First 10,000 GB (10 TB) / month:** **$1.00 per GB** evaluated.
* **Next 40,000 GB (10 TB to 50 TB) / month:** **$0.50 per GB** evaluated.
* **Above 50,000 GB (50 TB+) / month:** **$0.25 per GB** evaluated.
* *Math Trap Warning:* Scanning a single 10 TB bucket filled with uncompressed data costs:
  $$10,000\text{ GB} \times \$1.00 = \$10,000.00\text{ / month!}$$

---

## 4. Detailed Pricing Rates (us-east-1)

| Macie Feature | Billing Unit | Rate (us-east-1) | Price for 100 GB / Buckets |
|---------------|--------------|------------------|----------------------------|
| **Bucket Assessment** | Per bucket-month | **$0.10** | $10.00 / 100 buckets/mo |
| **Automated Discovery Monitoring**| Per 100,000 objects | **$0.01** | $0.01 / 100k objects |
| **Sensitive Data Inspection** | Per GB inspected | **$1.00** | **$100.00** |
| **Targeted Job (Tier 1: <10 TB)** | Per GB scanned | **$1.00** | **$100.00** |
| **Targeted Job (Tier 2: 10-50 TB)**| Per GB scanned | **$0.50** | $50.00 |
| **Targeted Job (Tier 3: 50+ TB)** | Per GB scanned | **$0.25** | $25.00 |

---

## 5. AWS Free Tier Coverage
* **Macie 30-Day Free Trial:** Includes a 30-day free trial for all new accounts.
  * *Includes:* 30 days of free bucket inventory evaluation and **150 GB of free data inspection** per account during the trial period.

---

## 6. Common Cost Hotspots & Pitfalls
* **Scanning Uncompressed Media / Binary Datasets (The $10,000 Mistake):**
  * Targeting Macie discovery jobs on S3 buckets containing raw video files (`.mp4`), disk images (`.iso`), or compressed archives (`.zip`).
  * Macie decompresses archives and scans every byte ($1.00/GB), incurring massive fees on files that never contain text PII.
* **Running Full Targeted Jobs When Automated Sampling Suffices:** Scheduling daily targeted deep scans ($1.00/GB) on fast-growing data lakes instead of relying on Automated Sensitive Data Discovery ($1.00/GB on sampled subsets).

---

## 7. Actionable Cost Optimization Strategies
1. **Configure File Extension Filters on All Discovery Jobs:**
   * Go to Macie Discovery Job settings -> **Include/Exclude Rules**.
   * Explicitly include text-based formats: `.csv`, `.json`, `.txt`, `.pdf`, `.parquet`, `.tsv`, `.doc`.
   * Explicitly exclude media and binary formats: `.png`, `.jpg`, `.jpeg`, `.mp4`, `.mp3`, `.zip`, `.tar`, `.gz`, `.iso`.
   * **The Savings:** Slashes data inspection volume by **90–99%** on data lakes.
2. **Use Automated Sensitive Data Discovery Over Full Targeted Scans:**
   * Enable **Automated Sensitive Data Discovery** across your organization.
   * It uses smart statistical sampling to scan small representative portions of objects across all buckets.
   * **The Savings:** Provides continuous PII visibility at **1/10th the cost** of scanning whole buckets.
3. **Use the 30-Day Free Trial to Forecast Data Volumes:** Review the Macie console **Estimated Cost** panel during the trial period to identify top candidate buckets for exclusion before launching paid scans.
4. **Exclude Archival & Public Datasets:** Exclude static public web asset buckets (e.g. static website images) and S3 Glacier Deep Archive buckets from Macie job scopes.
