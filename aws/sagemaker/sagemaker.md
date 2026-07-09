# AWS Service Cost Research: Amazon SageMaker

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon SageMaker is a comprehensive, fully managed service that provides developers and data scientists with the tools to build, train, and deploy machine learning (ML) models. SageMaker covers the entire ML lifecycle: data preparation, feature stores, notebooks, model training, and model hosting (inference). Because ML workloads require specialized, high-performance compute resources (like GPUs), managing instance runtimes and idle endpoints is critical to containing SageMaker costs.

---

## 2. Billing Mechanics
SageMaker pricing is highly consumption-based, billed by the second for compute and by the GB for storage across several dimensions:
1.  **Studio Notebooks & Notebook Instances:** Hourly fees based on the notebook instance size (e.g. `ml.t3.medium`).
2.  **Model Training:** Hourly charges for active training jobs, billed per-second.
3.  **Model Hosting (Inference):** 
    *   *Real-Time Endpoints:* Hourly charges per active instance hosting the model.
    *   *Serverless Inference:* Billed per millisecond of active execution time.
    *   *Asynchronous Inference:* Queued model hosting that scales to 0 when idle.
4.  **SageMaker Savings Plans:** Commitment-based discounts of up to 64% on compute.

---

## 3. Key Cost Dimensions

### A. Notebooks & Studio Instances (us-east-1 Compute)
*   **The Charge:** Billed hourly per instance.
    *   *ml.t3.medium (Standard CPU):* **$0.05 per hour** (~$36.50/month).
    *   *ml.m5.large (Baseline Compute):* **$0.115 per hour** (~$83.95/month).
    *   *ml.g4dn.xlarge (GPU Notebook):* **$0.736 per hour** (~$537.28/month).
*   *Warning:* If a data scientist leaves a GPU notebook running over a weekend without active coding, it bills $35.32 in idle compute.

### B. Managed Spot Training (The GPU Discount)
Training deep learning models requires expensive multi-GPU instances (e.g., `ml.p3.8xlarge` at $15.30/hour).
*   **The Discount:** By enabling **Managed Spot Training**, SageMaker runs your training jobs on spare AWS capacity, offering discounts of up to **90%** on compute.
*   *Preemption Rule:* If AWS reclaims the capacity, the job is paused. SageMaker automatically resumes the job from the last checkpoint once capacity is available, making checkpoints mandatory.

### C. Model Hosting (Inference Options)
*   **Real-Time Endpoints (24/7 Active):** Billed hourly per instance.
    *   *ml.m5.xlarge:* **$0.23 per hour** (~$167.90/month per instance).
    *   *The Idle Endpoint Hazard:* Real-time endpoints do not scale to 0 when idle. Running a multi-model production endpoint on a GPU instance (`ml.g4dn.xlarge` at $0.736/hr) continuously costs **$537.28/month**, even if it receives 0 requests.
*   **Serverless Inference (Scale to 0):** Billed per millisecond of active execution time during invocations.
    *   *Compute:* **$0.000020 per vCPU-second** (in `us-east-1`).
    *   *Memory:* **$0.000003 per GB-second** (scaled proportionally to memory).
    *   *Trade-off:* Ideal for spiky, low-frequency workloads. However, serverless endpoints experience **cold start latencies** (up to several seconds) when scaling up from 0.
*   **Asynchronous Inference (Queued Scaling):** Queues requests in SQS. It allows you to configure autoscaling to scale the instance count to **0** when the queue is empty, bypassing idle endpoint charges while supporting large models.

### D. SageMaker Savings Plans
*   Offers up to **64% discount** on compute charges in exchange for a 1- or 3-year hourly spend commitment (e.g. committing to spend $2.00/hour of SageMaker compute). Applies globally across notebooks, training, and inference.

---

## 4. Detailed Pricing Rates (us-east-1 Example Instances)

| SageMaker Instance Role | Instance Type | Hourly Rate (On-Demand) | Billed Granularity |
|-------------------------|---------------|--------------------------|---------------------|
| **Studio Notebook** | `ml.t3.medium` | **$0.0500** | Hourly |
| **Model Training** | `ml.p3.2xlarge` (GPU)| **$3.8250** | Per-second (Spot applies)|
| **Real-Time Endpoint** | `ml.m5.xlarge` | **$0.2300** | Hourly (No scale to 0) |
| **Serverless Compute** | 1 GB RAM / 1 vCPU| **$0.000023 / second** | Per millisecond |
| **EBS Storage** | gp3 | **$0.1400 / GB-month** | Monthly |

---

## 5. AWS Free Tier Coverage
*   **Studio / Notebooks:** 250 free hours/month of `ml.t3.medium` for the first 2 months.
*   **Training:** 50 free hours/month of `ml.m5.xlarge` for the first 2 months.
*   **Real-Time Inference:** 125 free hours/month of `ml.m5.xlarge` for the first 2 months.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Zombie Real-Time Endpoints:** Leaving active endpoints deployed in development or QA environments when they are no longer being queried. An idle GPU endpoint bills continuously.
*   **EBS Volume Leak on Stopped Notebooks:** Stopping a notebook instance halts compute charges, but you continue to pay for the attached EBS volume storage ($0.14/GB-month) until the notebook is deleted.
*   **Training without Checkpointing:** Running multi-hour GPU training jobs without saving checkpoints. If the job fails or is preempted by Spot capacity, you lose all progress and must pay to run the entire training duration again.

---

## 7. Actionable Cost Optimization Strategies
1.  **Shutdown Idle Notebooks Automatically:** Install the **SageMaker Notebook Auto-Shutdown Extension** on your Studio domains. Set it to automatically detect inactivity (e.g., 30 minutes of idle CPU/kernel) and stop the underlying instance.
2.  **Migrate Dev/Test Endpoints to Serverless or Async:**
    *   For staging, development, and QA testing, migrate real-time endpoints to **Serverless Inference** (scales to 0, cost is $0 when idle).
    *   For background processing or reporting models, use **Asynchronous Inference** with autoscaling configured to scale down to **0 instances** when the SQS request queue is empty.
3.  **Enforce Managed Spot Training with Checkpoints:**
    *   Always enable **Managed Spot Training** for experimental or non-production model training.
    *   Configure your training script to save model checkpoints to S3 frequently.
    *   **The Savings:** Cuts training GPU instance costs by up to **90%**.
4.  **Purchase SageMaker Savings Plans:** For permanent production hosting and continuous model development, purchase a 1-year SageMaker Savings Plan to secure up to **64% compute discounts**.
5.  **Prune Stopped Notebook EBS Volumes:** Set up a script to locate and delete notebook volumes that have been in a "Stopped" state for more than 60 days.
