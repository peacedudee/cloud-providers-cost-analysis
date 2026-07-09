# Amazon EC2 - Missing Inputs & Required Telemetry

To ensure maximum accuracy and confidence in the cost-cutting recommendations for Amazon EC2, the following data points (which are NOT natively available in standard FOCUS 1.0 CUR files) are highly beneficial. If these are unavailable, certain optimization levers (like memory rightsizing) will have lower confidence scores or may be skipped entirely.

## 1. Memory (RAM) Utilization
- **Why it's needed:** Standard AWS CloudWatch does not track memory utilization at the OS level by default. We can see CPU, but we don't know if a machine is out of memory. 
- **Impact:** Without memory metrics, downsizing instances (e.g., from `m5.xlarge` to `m5.large`) carries a high risk of triggering Out-Of-Memory (OOM) errors in production.
- **How to provide:** 
  - Install the **CloudWatch Agent** on instances to emit `mem_used_percent`.
  - Provide a **Compute Optimizer Export** (which uses machine learning to infer memory if enabled, or reads it from CloudWatch Agent).
  - Integrate with third-party APM tools (Datadog, Dynatrace, New Relic) via API.

## 2. Process-Level Metrics & Workload Type
- **Why it's needed:** Knowing *what* is running (e.g., an idle JVM taking 10GB of RAM vs. active database queries) helps determine if an instance is safely right-sizable.
- **Impact:** Helps confidently recommend spot instances for stateless worker nodes or Fargate migrations for monolithic APIs.
- **How to provide:** Detailed resource inventory with `WorkloadType` tags, or APM integrations.

## 3. OS License Information (BYOL)
- **Why it's needed:** RHEL and Windows Server carry significant per-vCPU licensing costs.
- **Impact:** Enables the "Dedicated Host Consolidation" and "vCPU Rightsizing" levers.
- **How to provide:** Output of AWS License Manager and exact OS configuration details from the EC2 metadata.

## 4. Auto Scaling Group (ASG) Configurations
- **Why it's needed:** To recommend "Target Tracking & Predictive Scaling."
- **Impact:** We need to know if an instance is part of an ASG, and what the scaling policies are, before we recommend terminating it. (Otherwise, the ASG will just spin it back up).
- **How to provide:** Terraform `.tf` state files or direct AWS CLI output (`aws autoscaling describe-auto-scaling-groups`).
