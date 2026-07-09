# AWS Service Cost Research: AWS Data Pipeline

> **Status:** ⚠️ Deprecated / Legacy Support Only  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Data Pipeline is a legacy web service designed to help you reliably process and move data between different AWS compute and storage services, as well as on-premises data sources. It orchestrates scheduled, data-driven workflows.
*   **Deprecation Timelines:**
    *   **July 25, 2024:** AWS closed access to new customers. The service remains active strictly for existing workloads.
    *   **April 30, 2023:** AWS completely removed console access for the service. It can now only be managed and accessed via the AWS CLI and API.
*   **Migration Path:** AWS strongly recommends migrating all legacy workloads to **AWS Glue** (for ETL and data integration), **AWS Step Functions** (for serverless workflow orchestration), or **Amazon MWAA** (Managed Workflows for Apache Airflow).

---

## 2. Billing Mechanics
Data Pipeline is billed based on a monthly recurring subscription model for existing accounts:
1.  **Pipeline Frequency Fee:** A flat monthly fee per active pipeline, determined by how often the pipeline runs.
2.  **Pipeline Location Fee:** Billed differently depending on whether the pipeline compute runs on AWS (e.g., EC2) or on-premises.
3.  **Underlying Resources:** Compute (EC2), database (RDS/Redshift), and storage (S3) resources launched by the pipeline are billed separately at standard, non-discounted hourly rates.

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
*   **Inactive Pipelines:** A pipeline that has no active runs scheduled is **free ($0.00)**.

### B. Underlying Compute and Storage Charges
*   AWS Data Pipeline does not contain its own compute cluster. It automatically provisions resources (like EC2 instances or EMR clusters) to run your data copy and transformation tasks.
*   These temporary resources are billed at standard, non-discounted hourly rates.

---

## 4. Detailed Pricing Rates (us-east-1 Legacy Workloads)

| Pipeline Run Frequency | Running Location | Monthly Subscription Rate | Notes |
|------------------------|------------------|---------------------------|-------|
| **High-Frequency (>1/day)**| AWS | **$5.00** | Billed monthly per pipeline |
| **High-Frequency (>1/day)**| On-Premises | **$10.00** | Billed monthly per pipeline |
| **Low-Frequency (<=1/day)**| AWS | **$1.00** | Billed monthly per pipeline |
| **Low-Frequency (<=1/day)**| On-Premises | **$2.00** | Billed monthly per pipeline |
| **Inactive / Suspended**| N/A | **Free ($0.00)** | Billed at $0.00 |

### Example Monthly Cost Calculation
*Workload: An existing client runs 5 legacy low-frequency daily data copy pipelines on AWS and 2 high-frequency hourly pipelines on-premises. During executions, the pipelines launch a total of 1,000 EC2 t3.medium instance-hours ($0.0416/hr) over the month.*

*   **Pipeline Subscription Fee:**
    $$\text{Sub Fee} = 5\text{ AWS Low-Freq} \times \$1.00 + 2\text{ On-Prem High-Freq} \times \$10.00$$
    $$\text{Sub Fee} = \$5.00 + \$20.00 = \$25.00$$
*   **Underlying EC2 Compute Cost:**
    $$\text{EC2 Cost} = 1,000\text{ hours} \times \$0.0416 = \$41.60$$
*   **Total Monthly Cost:** **$66.60/month**

---

## 5. AWS Free Tier Coverage
*   **AWS Data Pipeline:** The legacy free tier included 3 free low-frequency pipelines running on AWS per month.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Orphaned Active Pipelines:** Leaving pipelines in the `ACTIVATING` or `ACTIVE` state when they have finished their historical migration runs. Even if a pipeline processes no data, you pay the monthly recurring fee unless it is deleted or put into the `FINISHED` state.
*   **EC2 Instances Stuck on Failures:** If a copy activity fails, the underlying EC2 instance launched by Data Pipeline may get stuck in an active state due to timeout errors, continuously billing for compute hours.
*   **Inefficient Airflow/ETL Orchestration:** Retaining monthly pipeline subscriptions for tasks that could easily be run serverlessly.

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate to Modern Serverless Alternatives:**
    *   Proactively migrate all legacy pipelines to **AWS Glue**, **AWS Step Functions**, or **Amazon MWAA**.
    *   **The Savings:** By moving to Step Functions or Glue, you eliminate flat monthly pipeline subscriptions ($1.00 to $10.00/mo) and pay only for active, serverless execution runs.
2.  **Delete Completed / Legacy Pipelines:**
    *   Review your pipelines via the AWS CLI or API. Locate any historical or completed pipelines and call `DeletePipeline`.
    *   **The Savings:** Stops the recurring $1.00 to $10.00 monthly subscription charges.
3.  **Configure Strict Activity Timeouts:**
    *   Set strict **Timeout** limits on all Data Pipeline activities. This ensures that if a copy task hangs or fails, the underlying EC2 compute instances are terminated automatically within a short window, preventing run-away instance hourly bills.
