# AWS Service Cost Research: AWS Data Pipeline

> **Status:** ⚠️ Deprecated / Legacy Support only
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Data Pipeline is a legacy web service designed to help you reliably process and move data between different AWS compute and storage services, as well as on-premises data sources. It orchestrates scheduled, data-driven workflows. 
*   **Critical Cost Notice:** As of **July 25, 2024**, AWS has deprecated Data Pipeline for new customers. The service remains active strictly for existing workloads. AWS recommends migrating all data integration pipelines to **AWS Glue** or **AWS Step Functions**.

---

## 2. Billing Mechanics
Data Pipeline is billed based on a monthly recurring subscription model:
1.  **Pipeline Frequency Fee:** A flat monthly fee per active pipeline, determined by how often the pipeline runs.
2.  **Pipeline Location Fee:** Billed differently depending on whether the pipeline compute runs on AWS (e.g., EC2) or on-premises.
3.  **Underlying Resources:** Compute (EC2), database (RDS/Redshift), and storage (S3) resources launched by the pipeline are billed separately at standard rates.

---

## 3. Key Cost Dimensions

### A. Pipeline Run Frequency (us-east-1 Monthly Fees)
Pipelines are classified based on execution schedules:
*   **High-Frequency Pipelines:** Runs scheduled more than once per day (e.g., hourly or every 6 hours).
    *   *Running on AWS:* **$5.00 per pipeline per month**.
    *   *Running On-Premises:* **$10.00 per pipeline per month**.
*   **Low-Frequency Pipelines:** Runs scheduled once per day or less (e.g., daily or weekly).
    *   *Running on AWS:* **$1.00 per pipeline per month**.
    *   *Running On-Premises:* **$2.00 per pipeline per month**.
*   *Inactive Pipelines:* A pipeline that has no active runs scheduled is **free ($0.00)**.

### B. Underlying Compute and Storage Charges
*   Data Pipeline does not contain its own compute cluster. It automatically provisions resources (like EC2 instances or EMR clusters) to run your data copy and transformation tasks.
*   These temporary resources are billed at standard, non-discounted hourly rates.

---

## 4. Detailed Pricing Rates (us-east-1 Legacy Workloads)

| Pipeline Run Frequency | Running Location | Monthly Subscription Rate | Notes |
|-------------------|------------------|---------------------------|-------|
| **High-Frequency (>1/day)**| AWS | **$5.00** | Billed monthly |
| **High-Frequency (>1/day)**| On-Premises | **$10.00** | Billed monthly |
| **Low-Frequency (<=1/day)** | AWS | **$1.00** | Billed monthly |
| **Low-Frequency (<=1/day)** | On-Premises | **$2.00** | Billed monthly |
| **Inactive / Suspended** | N/A | **Free ($0.00)** | Billed at $0.00 |

---

## 5. AWS Free Tier Coverage
*   **AWS Data Pipeline:** Legacy free tier included 3 free low-frequency pipelines running on AWS per month.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Paying for Inactive Pipelines left in the Active State:** Leaving pipelines in the `ACTIVATING` or `ACTIVE` state when they have finished their historical migration runs. Even if a pipeline processes no data, you pay the monthly recurring fee unless it is deleted or put into the `FINISHED` state.
*   **EC2 Instances Left Running After Pipeline Failures:** If a copy activity fails, the underlying EC2 instance launched by Data Pipeline may get stuck in an active state due to timeout errors, continuously billing for compute hours.

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate to AWS Step Functions or AWS Glue:** 
    *   Since Data Pipeline is deprecated, proactively migrate all legacy data pipelines.
    *   Use **AWS Glue** for ETL and data transformation tasks.
    *   Use **AWS Step Functions** to orchestrate workflows. Step Functions are billed per execution (serverless), which is significantly cheaper than flat monthly pipeline subscriptions for low-volume runs.
2.  **Delete Completed / Legacy Pipelines:** Review the Data Pipeline console. Locate any historical or completed pipelines and click **Delete** to stop the recurring $1.00 to $10.00 monthly subscription charges.
3.  **Configure Task Timeouts:** Set strict **Timeout** limits on all Data Pipeline activities. This ensures that if a copy task hangs or fails, the underlying EC2 compute instances are terminated automatically within a short window, preventing run-away instance hourly bills.
