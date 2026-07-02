# AWS Service Cost Research: AWS Batch

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Batch enables developers, scientists, and engineers to run hundreds of thousands of batch computing jobs efficiently on AWS. Batch dynamically provisions optimal compute resources (CPU or memory-optimized instances) based on job queue volume and resource requirements. AWS Batch orchestration features are provided at **no additional charge**.

---

## 2. Billing Mechanics
* **Batch Orchestration Fee:** **100% Free ($0.00)**. Zero management charges or queue handling fees.
* **Underlying Compute Costs:** You pay only for underlying AWS compute resources (EC2 instances, Fargate tasks, EKS pods, or ECS tasks) provisioned to execute batch jobs.
* **Billing Models:** Determined by compute environment configuration (e.g., standard EC2 hourly rates, EC2 Spot rates, or Fargate per-second resource pricing).

---

## 3. Key Cost Dimensions

### A. Managed vs. Unmanaged Compute Environments
* **Managed Compute Environments:** AWS Batch dynamically provisions and terminates EC2 instances or Fargate tasks automatically based on job queue load.
  * *EC2 Launch:* Billed at standard EC2 rates.
  * *Fargate Launch:* Billed at standard Fargate rates (per vCPU and memory second).
* **Unmanaged Compute Environments:** You provision and manage your own EC2 instances. AWS Batch dispatches container tasks to existing nodes. Billed for EC2 instances regardless of queue activity.

### B. Spot vs. On-Demand Compute
* **The Primary Lever:** Batch natively supports **Spot Compute Environments**. Because batch jobs are typically stateless and fault-tolerant, using Spot instances yields **70%–90% cost savings** over On-Demand rates.
* **Allocation Strategies:**
  * `SPOT_CAPACITY_OPTIMIZED`: Launches Spot instances from pools with optimal available capacity to minimize job interruptions.
  * `BEST_FIT_PROGRESSIVE`: Selects cheap instance types while scaling progressively.

### C. Ancillary Resource Charges
* **Amazon EBS Disks:** Boot and scratch disk storage for EC2 worker nodes ($0.08/GB-mo).
* **Amazon ECR:** Container image storage and pull bandwidth.
* **Amazon CloudWatch Logs:** Collecting stdout/stderr logs from batch job containers.

---

## 4. Detailed Pricing Rates (us-east-1)

| Resource Type | Billing Basis | Rate (us-east-1) | Cost Management Strategy |
|---------------|---------------|------------------|--------------------------|
| **AWS Batch Fee** | Orchestrator | **$0.00** (Free) | N/A |
| **Fargate Compute** | Per vCPU/Memory second | Standard Fargate Rates | Set tight resource limits in Job Definition |
| **EC2 On-Demand** | Per running second | Standard EC2 Rates | Limit compute environment max vCPUs |
| **EC2 Spot** | Per running second | **Up to 90% discount** | Use `SPOT_CAPACITY_OPTIMIZED` strategy |
| **EBS Storage** | Per GB-month provisioned | **$0.08 / GB-month** | Use smaller root disk sizes |

---

## 5. AWS Free Tier Coverage
* **AWS Batch Orchestration:** Always 100% free.
* **Underlying Compute:** Billed compute resources can leverage standard EC2, EBS, and Fargate free tier allowances where eligible.

---

## 6. Common Cost Hotspots & Pitfalls
* **MinvCPUs > 0 in Managed Compute Environments:** Setting `MinvCPUs` greater than 0 in managed EC2 compute environments. This forces Batch to keep worker instances running 24/7 even when job queues are empty.
* **Hung Jobs Without Timeouts:** Running jobs that hang in infinite loops or wait on unavailable endpoints. Without job timeouts, active instances run indefinitely.
* **Over-Allocated Job Definition Specs:** Requesting 16 vCPUs and 64 GB RAM in job definitions for single-threaded tasks, forcing Batch to launch expensive `c6i.4xlarge` instances and wasting 90%+ compute.

---

## 7. Actionable Cost Optimization Strategies
1. **Set `MinvCPUs = 0` Across All Managed Compute Environments:**
   * Ensure `MinvCPUs = 0` in all managed compute environments (especially Dev/Test).
   * **The Savings:** Guarantees compute scales down to **$0.00** when queues are empty.
2. **Enforce Job Execution Timeouts:** Always configure the `timeout` parameter in Job Definitions (e.g., 30–60 minutes) to terminate hanging tasks automatically.
3. **Use Spot Compute Environments with `SPOT_CAPACITY_OPTIMIZED`:** Deploy `SPOT` compute environments using `SPOT_CAPACITY_OPTIMIZED` allocation to reduce interruption rates while securing 70%–90% savings.
4. **Deploy Small/Short Jobs to AWS Fargate:** For jobs executing in under 15 minutes requiring low compute (<4 vCPUs), run them on Fargate to eliminate instance startup lag and idle node burn.
5. **Implement S3 Checkpointing for Long Jobs:** Write intermediate state checkpoints to S3 for jobs running longer than 2 hours. If a Spot instance is interrupted, the job resumes from the last checkpoint.
