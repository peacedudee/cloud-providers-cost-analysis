# Cost-Cutting Playbook: Amazon SageMaker

> **Companion File:** [sagemaker.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/sagemaker/sagemaker.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon SageMaker is an end-to-end machine learning (ML) platform spanning Studio notebooks, model training, and model hosting (inference). Because ML workloads rely heavily on expensive GPU instances (e.g. `ml.p3.8xlarge` at $15.30/hr or `ml.g4dn.xlarge` at $0.736/hr), unmonitored development notebooks and idle 24/7 real-time endpoints generate massive cost leaks.

This playbook outlines **18 actionable strategies** across six operational categories, delivering an estimated **35–75% reduction in total SageMaker spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Install SageMaker Studio Auto-Shutdown Extension:** Automatically stops idle notebook instances after 30 minutes of inactivity, avoiding $100s in weekend GPU charges.
2. **Enable Managed Spot Training with Checkpoints:** Slashes GPU model training costs by up to **90%** using spare AWS capacity.
3. **Migrate Staging/QA Endpoints to Serverless or Asynchronous Inference:** Scales endpoint instance counts to **0** when idle ($0.00 compute fee), eliminating 24/7 real-time hosting charges.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Install SageMaker Notebook Auto-Shutdown Extension
- **What:** Deploy the official SageMaker Auto-Shutdown extension across all SageMaker Studio domains and Jupyter notebook instances.
- **Why It Saves Money:** Data scientists frequently leave GPU notebooks (`ml.g4dn.xlarge` at $0.736/hr) running over nights and weekends. A single GPU notebook left running over a weekend wastes **$35.32 in idle compute**. Across 20 data scientists, this wastes **$2,800/month**.
- **Implementation Steps:**
  1. Attach lifecycle configuration script to SageMaker Studio Domain.
  2. Set idle threshold to 30 minutes (monitors CPU, GPU, and Jupyter kernel activity).
  3. Automatically stop instances when threshold is breached.
- **Estimated Savings:** 50–70% reduction in notebook instance billing.
- **Risk Level:** Low (notebook state and files remain saved on EBS).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SageMaker Lifecycle Configuration access.

#### 2. Terminate Abandoned Dev/Test Real-Time Endpoints
- **What:** Identify and delete SageMaker real-time hosting endpoints in development or QA accounts that have received zero inference invocations (`Invocations = 0`) over 7 days.
- **Why It Saves Money:** Real-time endpoints do NOT scale to 0 when idle. An idle `ml.m5.xlarge` endpoint ($0.23/hr) bills **$167.90/month** continuously per instance. An idle GPU endpoint (`ml.g4dn.xlarge`) bills **$537.28/month**.
- **Implementation Steps:**
  1. Check CloudWatch metric `Invocations` for all SageMaker endpoints.
  2. Delete zero-invocation endpoints: `aws sagemaker delete-endpoint --endpoint-name name`.
- **Estimated Savings:** 100% of abandoned endpoint hosting costs.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Endpoint invocation audit over 7 days.

#### 3. Prune Orphaned Notebook EBS Volumes and Stale Models
- **What:** Clean up attached EBS volumes associated with deleted or long-stopped notebook instances, and remove unreferenced SageMaker Model entities.
- **Why It Saves Money:** Notebook EBS storage bills at **$0.14 per GB-month** continuously even while the notebook is in a "Stopped" state.
- **Implementation Steps:**
  1. List stopped notebooks: `aws sagemaker list-notebook-instances --status-equals Stopped`.
  2. Delete notebooks stopped > 60 days (backing up code to Git).
- **Estimated Savings:** 100% of orphaned notebook storage costs.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Code repository synchronization.

---

### 2. Rightsizing & Model Hosting Optimization

#### 4. Migrate Non-Prod Endpoints to Serverless Inference (Scale to 0)
- **What:** Convert development, QA, and low-frequency production endpoints to **SageMaker Serverless Inference**.
- **Why It Saves Money:** Serverless Inference bills per millisecond of active request duration ($0.000020/vCPU-sec) and **scales to 0 instances ($0.00)** when idle. Eliminates the $167.90/mo baseline for low-traffic endpoints.
- **Implementation Steps:**
  1. Create Serverless Endpoint Configuration: Set `MemorySizeInMB` (1 GB to 6 GB) and `MaxConcurrency`.
  2. Deploy endpoint using Serverless config.
- **Estimated Savings:** 70–95% savings for low-frequency/spiky endpoints.
- **Risk Level:** Low to Medium (serverless endpoints experience cold starts up to several seconds).
- **Implementation Scope:** Data Scientist / DevOps
- **Prerequisites:** Latency tolerance for cold starts.

#### 5. Transition Background Batch Models to Asynchronous Inference
- **What:** Move asynchronous processing models (e.g. daily image analysis or batch document processing) to **SageMaker Asynchronous Inference**.
- **Why It Saves Money:** Queues requests in SQS and supports Auto Scaling down to **0 instances** when the request queue is empty.
- **Implementation Steps:**
  1. Define Asynchronous Inference Config specifying S3 output path.
  2. Configure Application Auto Scaling target `HasBacklogWithoutCapacity` to scale to 0.
- **Estimated Savings:** 50–80% savings on batch inference hosting.
- **Risk Level:** Low.
- **Implementation Scope:** Data Scientist / DevOps
- **Prerequisites:** Asynchronous request workflow compatibility.

#### 6. Consolidate Multiple Models onto Multi-Model Endpoints (MME)
- **What:** Deploy multiple trained models of the same framework (e.g. 50 distinct XGBoost or PyTorch customer models) onto a single **Multi-Model Endpoint (MME)**.
- **Why It Saves Money:** Instead of running 50 separate real-time instances ($8,395/month), MME dynamically loads models into memory on a shared container pool, reducing instance requirements by up to 90%.
- **Implementation Steps:**
  1. Package model artifacts into S3 prefix `s3://bucket/models/`.
  2. Create MME endpoint pointing to S3 model directory.
- **Estimated Savings:** **70–90% reduction** in endpoint compute instance counts.
- **Risk Level:** Medium.
- **Implementation Scope:** Data Scientist / MLOps
- **Prerequisites:** Homogeneous framework model usage.

#### 7. Downsize Over-Provisioned Endpoint Instances
- **What:** Analyze CloudWatch `CPUUtilization` and `GPUUtilization` for real-time endpoints; downsize instance types (e.g. from `ml.g4dn.2xlarge` to `ml.g4dn.xlarge`).
- **Why It Saves Money:** Cuts hourly endpoint hosting costs by 50% ($268/mo saved per instance).
- **Implementation Steps:**
  1. Monitor GPU/CPU utilization metrics during peak traffic.
  2. Update Endpoint Configuration with smaller instance type.
- **Estimated Savings:** 50% per step-down.
- **Risk Level:** Low.
- **Implementation Scope:** MLOps / DevOps
- **Prerequisites:** Inference latency SLA testing.

---

### 3. Commitment Discounts

#### 8. Purchase SageMaker Savings Plans for Baseline Hosting
- **What:** Commit to a 1-year or 3-year SageMaker Savings Plan covering steady-state notebook, training, and real-time endpoint compute.
- **Why It Saves Money:** Provides discounts **up to 64%** off On-Demand rates across all SageMaker compute instance types.
- **Implementation Steps:**
  1. Calculate minimum baseline hourly SageMaker spend in Cost Explorer.
  2. Purchase Savings Plan commitment.
- **Estimated Savings:** 20–64% off On-Demand rates.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team / Procurement
- **Prerequisites:** 1-year ML workload commitment.

---

### 4. Training Optimization

#### 9. Enable Managed Spot Training with S3 Checkpointing
- **What:** Enable **Managed Spot Training** (`EnableManagedSpotTraining = True`) for experimental and non-production ML training jobs.
- **Why It Saves Money:** Uses spare AWS GPU capacity at discounts **up to 90% off On-Demand training rates**. A 10-hour `ml.p3.8xlarge` training job drops from **$153.00 to $15.30**.
- **Implementation Steps:**
  1. Update training job estimator: `train_use_spot_instances = True`.
  2. Configure `checkpoint_s3_uri` in estimator to save model weights every epoch.
  3. Set `max_wait` timeout parameters.
- **Estimated Savings:** **70–90% savings** on model training GPU spend.
- **Risk Level:** Low (checkpointing ensures training resumes automatically if preempted).
- **Implementation Scope:** Data Scientist / MLOps
- **Prerequisites:** Code support for saving/restoring checkpoints.

#### 10. Optimize Training Data Pipeline I/O (FastFile & Pipe Mode)
- **What:** Use **FastFile Mode** or **Pipe Mode** for streaming training datasets directly from S3 to SageMaker training containers instead of downloading datasets in File Mode.
- **Why It Saves Money:** Eliminates 30+ minutes of unbilled startup delay spent downloading 500 GB datasets to local EBS volumes prior to training, reducing total billable GPU training duration.
- **Implementation Steps:**
  1. Update S3 data input channel setting: `input_mode = 'FastFile'`.
- **Estimated Savings:** 20–40% reduction in total billable training job duration.
- **Risk Level:** Low.
- **Implementation Scope:** Data Scientist
- **Prerequisites:** Compatible dataset format.

#### 11. Right-Size Training Job Instance Counts & Families
- **What:** Benchmark training job performance; select optimal GPU instance families (e.g. migrating from older `ml.p3` V100 GPUs to `ml.g5` A10G GPUs or `ml.trn1` Trainium instances).
- **Why It Saves Money:** `ml.g5.2xlarge` ($1.212/hr) provides superior price-performance compared to `ml.p3.2xlarge` ($3.825/hr) — a **68% cost reduction** for equivalent training speeds.
- **Implementation Steps:**
  1. Run single-epoch benchmark tests across `g5`, `p4d`, and `trn1` instance families.
  2. Update training estimator instance type.
- **Estimated Savings:** 40–68% improvement in training cost efficiency.
- **Risk Level:** Low.
- **Implementation Scope:** Data Scientist / MLOps
- **Prerequisites:** Framework support for targeted GPU/Trainium architecture.

---

### 5. Scheduling & Auto-Scaling

#### 12. Configure Target Tracking Auto-Scaling on Real-Time Endpoints
- **What:** Attach Application Auto Scaling to production real-time endpoints based on `SageMakerVariantInvocationsPerInstance` (target 70%).
- **Why It Saves Money:** Dynamically scales endpoint instance counts down during off-peak hours (e.g. dropping from 8 instances to 2 instances overnight), cutting hosting costs by 50%+.
- **Implementation Steps:**
  1. Register endpoint variant as scalable target: `aws application-autoscaling register-scalable-target`.
  2. Define Target Tracking policy.
- **Estimated Savings:** 30–60% off static peak endpoint capacity.
- **Risk Level:** Low.
- **Implementation Scope:** MLOps / DevOps
- **Prerequisites:** Endpoint variant configuration.

#### 13. Schedule Non-Prod Real-Time Endpoints to Scale to Minimum 1 Off-Hours
- **What:** Scale staging endpoints down to minimum instance counts (1 instance) during nights and weekends via Application Auto Scaling scheduled actions.
- **Why It Saves Money:** Saves 65% of multi-instance hosting fees in non-production.
- **Implementation Steps:**
  1. Add scheduled auto-scaling actions for non-prod endpoint variants.
- **Estimated Savings:** 50–65% reduction in non-prod hosting spend.
- **Risk Level:** Low.
- **Implementation Scope:** MLOps / DevOps
- **Prerequisites:** Non-prod SLA alignment.

---

### 6. Data & Storage Optimization

#### 14. Compress Training Artifacts and Model Weights in S3
- **What:** Compress raw training data files and output model artifacts stored in S3 using Gzip/Tar.
- **Why It Saves Money:** S3 storage fees for uncompressed ML datasets (e.g. 50 TB of raw images/text at $0.023/GB-mo = $1,150/mo) drop by 60%+ when compressed.
- **Implementation Steps:**
  1. Compress datasets prior to uploading to S3 training channels.
- **Estimated Savings:** 50–70% reduction in ML dataset S3 storage.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Compression script.

#### 15. Enforce S3 Lifecycle Rules on SageMaker Output Buckets
- **What:** Configure S3 lifecycle policies on SageMaker default output buckets (`s3://sagemaker-us-east-1-account/`) to delete temporary training artifacts after 30 days.
- **Why It Saves Money:** Prevents gigabytes of intermediate model checkpoints and training logs from accumulating permanently in S3 Standard.
- **Implementation Steps:**
  1. Apply lifecycle rule `Expiration: 30 Days` on `/sagemaker-artifacts/` prefixes.
- **Estimated Savings:** 70–90% reduction in historical training artifact storage.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Model registry integration for registered production models.

#### 16. Deploy SageMaker Inference Components for Heterogeneous Container Sharing
- **What:** Deploy SageMaker **Inference Components** on real-time endpoints to run multiple different model containers on shared instance pools.
- **Why It Saves Money:** Maximizes GPU/CPU memory utilization across distinct model types, reducing overall instance provisioned footprint.
- **Implementation Steps:**
  1. Create Endpoint with `InferenceComponents` assignment.
- **Estimated Savings:** 30–60% instance consolidation savings.
- **Risk Level:** Medium.
- **Implementation Scope:** MLOps / Architect
- **Prerequisites:** SageMaker Inference Components API usage.

#### 17. Use AWS Inferentia (`inf1`/`inf2`) for High-Volume Production Inference
- **What:** Migrate real-time deep learning inference models (HuggingFace, PyTorch, TensorFlow) from GPU instances (`g4dn`/`g5`) to AWS Inferentia (`inf1`/`inf2`) instances.
- **Why It Saves Money:** AWS Inferentia instances deliver up to **50% lower cost per inference** compared to GPU instances.
- **Implementation Steps:**
  1. Compile model using AWS Neuron SDK: `torch_neuronx.trace()`.
  2. Deploy on `ml.inf2.xlarge` endpoint.
- **Estimated Savings:** **50% direct reduction** in production inference hosting.
- **Risk Level:** Medium (requires Neuron SDK model compilation).
- **Implementation Scope:** Data Scientist / MLOps
- **Prerequisites:** AWS Neuron SDK compilation testing.

#### 18. Monitor & Delete Unused Feature Store Data Groups
- **What:** Audit SageMaker Feature Store offline stores; apply S3 lifecycle rules to expire stale feature group historical snapshots.
- **Why It Saves Money:** Offline feature store data is stored in S3 and billed at standard S3 rates.
- **Implementation Steps:**
  1. Audit Feature Store groups via CLI: `aws sagemaker list-feature-groups`.
- **Estimated Savings:** Feature store S3 storage optimization.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer / MLOps
- **Prerequisites:** Feature group lifecycle policy.

---

## Cross-Service Synergies

```
[ SageMaker ML Platform ] 
        │
        ├──(Managed Spot Training)─> [ EC2 Spot Capacity ] (Saves 70-90% on GPU training)
        │
        ├──(Serverless / Async)─────> [ SQS + Auto-Scaling ] (Scales hosting to 0 when idle)
        │
        ├──(Inference Hardware)─────> [ AWS Inferentia (inf2) ] (Saves 50% vs GPU inference)
        │
        └──(Commitment Plan)────────> [ SageMaker Savings Plans ] (Up to 64% compute discount)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Studio:ml.t3.medium`, `Hosting:ml.m5.xlarge`, `Training:ml.p3.2xlarge`, `SpotSavings`, `ServerlessInference-GB-Seconds`.
- `line_item_resource_id`: SageMaker Endpoint ARN / Notebook Instance ARN / Training Job ARN.

### B. CloudWatch & Platform Metrics
- CloudWatch `AWS/SageMaker` Metrics: `CPUUtilization`, `GPUUtilization`, `MemoryUtilization`, `GPUMemoryUtilization`, `Invocations`, `ModelLatency`, `OverheadLatency`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "SM-EP-001",
  "service": "SageMaker",
  "category": "Rightsizing & Model Hosting",
  "resource_id": "arn:aws:sagemaker:us-east-1:123456789012:endpoint/qa-fraud-detection-endpoint",
  "resource_name": "qa-fraud-detection-endpoint",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "endpoint_type": "Real-Time",
    "instance_type": "ml.m5.xlarge",
    "instance_count": 2,
    "monthly_invocations": 12000,
    "monthly_cost_usd": 335.80
  },
  "recommended_config": {
    "endpoint_type": "Serverless Inference",
    "memory_size_mb": 2048,
    "max_concurrency": 5,
    "projected_monthly_cost_usd": 12.40
  },
  "financial_impact": {
    "monthly_savings_usd": 323.40,
    "annual_savings_usd": 3880.80,
    "savings_percentage": 96.3
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "QA endpoint receives sporadic requests; serverless cold starts (~2s) are acceptable for QA testing."
  },
  "implementation": {
    "scope": "Data Scientist / MLOps",
    "effort_estimate": "1 hour",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Waste Elimination (Auto-Shutdown)**| 24 | $4,800.00 | $3,360.00 | 70.0% | Low |
| **Managed Spot Training** | 12 | $18,500.00 | $14,800.00 | 80.0% | Low |
| **Serverless / Async Hosting** | 15 | $8,200.00 | $6,150.00 | 75.0% | Medium |
| **Inferentia Migration (inf2)** | 4 | $12,000.00 | $6,000.00 | 50.0% | Medium |
| **Commitment Discounts** | 2 | $15,000.00 | $5,250.00 | 35.0% | Low |
| **Total** | **57** | **$58,500.00** | **$35,560.00** | **60.8%** | -- |
