# AWS Service Cost Research: Amazon EMR (Elastic MapReduce)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon EMR is a managed cluster platform that simplifies running open-source big data frameworks, such as Apache Spark, Apache Hadoop, Apache HBase, Apache Hive, and Presto. EMR allows you to run distributed data processing workloads at a massive scale. EMR pricing is additive: you pay for the underlying AWS compute and storage resources (like EC2 or EKS) plus an hourly EMR management fee.

---

## 2. Billing Mechanics
EMR billing is structured based on the chosen deployment model:
1.  **EMR on Amazon EC2 (Classic):**
    *   *Underlying Compute & Storage:* Standard hourly EC2 instance charges and EBS volume storage fees.
    *   *EMR Management Fee:* An hourly surcharge per running EC2 instance, billed in per-second increments with a 1-minute minimum.
2.  **EMR on Amazon EKS:** Billed an hourly EMR management fee per vCPU and GB of memory used by the Kubernetes pods.
3.  **EMR Serverless:** Billed per-second for the vCPU, memory, and temporary storage resources consumed by active Spark/Hive jobs.

---

## 3. Key Cost Dimensions

### A. EMR on Amazon EC2 (us-east-1)
*   **The Additive Fee:** The EMR management fee is added to the EC2 hourly cost.
    *   *Example (m5.xlarge):* 
        *   Standard `m5.xlarge` EC2 Node: **$0.192 per hour**.
        *   EMR Management Premium: **$0.048 per hour** (a **25% surcharge**).
        *   Total Node Cost: **$0.240 per hour**.
*   **Compute Node Sizing:** EMR divides instances into:
    *   *Master Node:* Manages cluster state (1 instance).
    *   *Core Nodes:* Run tasks and host HDFS storage (minimum 1, runs continuously).
    *   *Task Nodes:* Run compute tasks only (can scale to 0, stateless).
*   **Reserved and Spot Instances:** You can use EC2 Spot Instances for stateless **Task Nodes** to save up to **90%** on underlying compute, but the EMR management fee remains flat.

### B. EMR on Amazon EKS
*   **The Rate:** Billed a flat management fee of **$0.015 per vCPU-hour** and **$0.0035 per GB-hour** for resources allocated to EMR jobs running on EKS.
*   *Note:* This is in addition to standard EKS cluster hourly fees ($0.10/hr) and Fargate or EC2 compute charges.

### C. EMR Serverless (Serverless Spark/Hive)
*   **The Model:** Best for intermittent big data processing. Compute resources scale from 0 and shut down automatically when jobs complete.
*   **Compute Rates (us-east-1):**
    *   *vCPU:* **$0.052624 per vCPU-hour** (billed per-second, 1-minute minimum).
    *   *Memory:* **$0.0057785 per GB-hour**.
    *   *Storage:* **$0.000111 per GB-hour** (for temporary storage beyond the free 20 GB).

---

## 4. Detailed Pricing Rates (us-east-1 EMR on EC2 Examples)

| Instance Type | EC2 Hourly Rate | EMR Hourly Premium | Total Node Hourly Rate | EMR Surcharge Ratio |
|---------------|-----------------|--------------------|------------------------|---------------------|
| **m5.xlarge** | $0.1920 | **$0.0480** | $0.2400 | 25.0% |
| **m5.2xlarge**| $0.3840 | **$0.0960** | $0.4800 | 25.0% |
| **r5.xlarge** | $0.2520 | **$0.0630** | $0.3150 | 25.0% |
| **c5.2xlarge**| $0.3400 | **$0.0850** | $0.4250 | 25.0% |

---

## 5. AWS Free Tier Coverage
*   **Amazon EMR:** **No free tier** is available. Any running cluster generates base EC2 and EMR management billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Keeping EMR Clusters Running 24/7:** Leaving a multi-node EMR cluster active and idle overnight or over weekends when no batch ETL jobs are running. A 10-node `m5.2xlarge` cluster left idle costs **$345.60 per day**.
*   **Over-Sizing Core Nodes for Storage:** Scaling out Core Nodes (which host HDFS) just to increase HDFS storage capacity. This forces you to pay for expensive, idle compute cores.
*   **Using On-Demand EC2 Nodes for Stateless Tasks:** Provisioning all EMR task nodes as On-Demand instances instead of Spot instances.
*   **Serverless App Idle Running Limits:** Leaving EMR Serverless applications in the `STARTED` state with high pre-initialized capacity configured, billing for compute resources that are not running active jobs.

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate Intermittent Jobs to EMR Serverless:** For batch Spark/Hive pipelines that run for short periods (e.g. 1 hour daily):
    *   Migrate from EMR on EC2 to **EMR Serverless**.
    *   **The Savings:** Eliminates the master/core node idle runtimes completely. You only pay for active job execution time, cutting costs by up to **80%**.
2.  **Use Spot Instances for stateless Task Nodes:** Configure your EMR instance groups to use **Spot Instances** for 100% of your **Task Nodes**. Keep only your Master and Core nodes on On-Demand (or Reserved) instances. This drops underlying compute costs by **70–90%**.
3.  **Separate Compute and Storage (Leverage EMRFS / S3):** 
    *   Do not store large datasets in local HDFS (which requires running Core Nodes to maintain).
    *   Store all persistent datasets in **Amazon S3** and access them via the **EMR File System (EMRFS)**.
    *   This allows you to scale down Core Nodes to a minimum of 1 node, and use stateless Spot Task Nodes that scale up/down dynamically.
4.  **Implement Auto-Scaling Policies:** Configure EMR Managed Scaling policies on your clusters. Set scaling thresholds based on YARN memory utilization to automatically add task nodes during heavy processing and terminate them during idle periods.
5.  **Set "Auto-Terminate After Idle" Limits:** Configure your EMR on EC2 clusters with the **Auto-termination policy** (e.g., automatically shut down the cluster if it has been idle for more than 15 or 30 minutes).
