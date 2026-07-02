# AWS Service Cost Research: AWS CodePipeline

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS CodePipeline is a fully managed continuous delivery service that automates release pipelines for application and infrastructure updates. CodePipeline orchestrates the build, test, and deploy stages of your release workflow upon every code push. CodePipeline supports two pipeline types: **V1 Pipelines** (flat monthly rate per active pipeline) and **V2 Pipelines** (usage-based action minute execution pricing).

---

## 2. Billing Mechanics
1. **V1 Pipelines (Legacy Fixed Rate):** Billed monthly per active pipeline ($1.00 per active pipeline per month). A pipeline is active if it has existed for >30 days and processed at least one code change during the calendar month.
2. **V2 Pipelines (Execution Minute Pricing):** Billed based on the total minutes that pipeline actions execute ($0.002 per action execution minute).
3. **Free Tier:**
   * *V1 Pipelines:* First 1 active pipeline per month is **100% Free ($0.00)**.
   * *V2 Pipelines:* First 100 action execution minutes per month are **100% Free ($0.00)**.

---

## 3. Key Cost Dimensions

| CodePipeline Version | Billed Unit | Rate (us-east-1) | Price for 10 Pipelines / 1,000 Mins |
|----------------------|-------------|------------------|-------------------------------------|
| **V1 Active Pipeline** | Per pipeline-month | **$1.00** | **$9.00 / month** (1st free) |
| **V2 Action Minute** | Per execution-min | **$0.002** | **$2.00** (per 1,000 mins) |

---

## 4. Detailed Pricing Rates (us-east-1)

* **V1 Active Pipeline Rate:** $1.00 per active pipeline per month.
* **V2 Execution Minute Rate:** $0.002 per action execution minute ($0.12 per hour).

---

## 5. AWS Free Tier Coverage
* **AWS CodePipeline Free Tier:** Includes 1 active V1 pipeline per month or 100 free V2 action execution minutes per month per account indefinitely.

---

## 6. Common Cost Hotspots & Pitfalls
* **Maintaining Abandoned V1 Pipelines:** Keeping dozens of developer feature-branch V1 pipelines active that process 1 minor commit per month ($1.00/mo per pipeline).
* **Unmonitored V2 Action Executions:** Allowing manual approval stages or slow external integration test actions in V2 pipelines to run unmonitored.

---

## 7. Actionable Cost Optimization Strategies
1. **Delete Inactive V1 Pipelines:**
   * Audit pipelines using `aws codepipeline list-pipelines`.
   * Delete unused feature-branch or test pipelines.
   * **The Savings:** Saves $1.00/month per deleted V1 pipeline.
2. **Migrate Low-Frequency Pipelines to V2 Execution Pricing:**
   * Convert pipelines that run infrequently (e.g., weekly infrastructure deployment pipelines) from V1 to **V2 Pipeline Type**.
   * *Why:* In V2, a pipeline executing for 5 minutes per month costs **$0.01/month** instead of the flat $1.00 V1 monthly active fee.
   * **The Savings:** Slashes pipeline management costs by **90%** for low-frequency deployment workflows.
