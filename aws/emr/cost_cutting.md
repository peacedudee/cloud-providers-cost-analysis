# Cost-Cutting Playbook: Amazon EMR (Elastic MapReduce)

> **Companion File:** [emr.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/emr/emr.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon EMR is a managed big data platform running Apache Spark, Hadoop, Hive, Presto, and HBase across **EMR on EC2**, **EMR on EKS**, or **EMR Serverless**. EMR pricing is additive: on EC2 clusters, you pay for raw EC2 compute and EBS storage PLUS an **EMR Management Fee surcharge** (a **25% premium** on instance rates, e.g. $0.048/hr per `m5.xlarge`).

Major billing hotspots include 24/7 idle provisioned clusters ($345/day for 10 nodes), over-sizing Core Nodes for local HDFS storage instead of using S3/EMRFS, and running stateless task nodes on On-Demand EC2 instead of Spot.

This playbook provides **18 actionable strategies** across six operational categories, delivering an estimated **35–80% reduction in total EMR big data spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Migrate Intermittent Batch Spark Jobs to EMR Serverless:** Eliminates 24/7 master/core node idle runtimes; pay only for active job seconds (up to **80% savings**).
2. **Deploy EC2 Spot Instances for 100% of Task Nodes:** Cuts underlying compute costs by **70–90%** on stateless processing capacity.
3. **Configure Auto-Termination After Idle (15–30 Mins):** Automatically shuts down provisioned EC2 clusters when YARN queues remain idle.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Configure Auto-Termination After Idle Policy
- **What:** Attach an automated **Auto-Termination Policy** (`AutoTerminationPolicy`) to all EMR on EC2 clusters.
- **Why It Saves Money:** A 10-node `m5.2xlarge` cluster ($0.48/node-hr total) left running idle overnight or over a weekend costs **$115.20 per idle 24-hour period** ($3,456/month). Auto-termination terminates the cluster when YARN has been idle for 15 minutes.
- **Detailed Implementation Steps:**
  1. Attach auto-termination policy via AWS CLI:
     ```bash
     aws emr put-auto-termination-policy \
       --cluster-id j-1234567890 \
       --auto-termination-policy IdleTimeout=900
     ```
  2. Verify policy status in EMR console or cluster description.
- **Estimated Savings:** 100% of idle cluster runtimes ($100s to $1,000s/month saved per cluster).
- **Risk Level:** Zero risk (triggers only after all YARN jobs complete and cluster is idle).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Cluster setup automation via Airflow/Step Functions.

#### 2. Decommission Stale HDFS Cluster Volumes
- **What:** Terminate legacy long-running EMR clusters where HDFS data has been safely migrated to S3.
- **Why It Saves Money:** Reclaims underlying EC2 instance charges and EBS storage fees.
- **Detailed Implementation Steps:**
  1. Sync remaining HDFS files to S3 via `distcp`:
     ```bash
     hadoop distcp hdfs:///user/data/ s3://company-datalake/user-data/
     ```
  2. Terminate cluster:
     ```bash
     aws emr terminate-clusters --cluster-ids j-1234567890
     ```
- **Estimated Savings:** 100% of abandoned cluster costs.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** Verification that HDFS dataset is backed up to S3.

---

### 2. Deployment Model & Serverless Migration

#### 3. Migrate Intermittent Batch Spark Pipelines to EMR Serverless
- **What:** Refactor batch Spark/Hive jobs that run for short periods (e.g. 1 hour daily) from EMR on EC2 to **EMR Serverless**.
- **Why It Saves Money:** EMR Serverless scales compute to **0 vCPUs ($0.00)** when no jobs are executing. A daily 1-hour job on 100 vCPUs costs **$227.21/month** on Serverless, compared to **$10,368.00/month** for a 24/7 provisioned 10-node cluster!
- **Detailed Implementation Steps:**
  1. Create an EMR Serverless Application using AWS CLI:
     ```bash
     aws emr-serverless create-application \
       --type SPARK \
       --name "daily-etl-spark-app" \
       --release-label "emr-7.1.0" \
       --initial-capacity '{}' \
       --maximum-capacity '{"cpu": "200vCPU", "memory": "800GB"}'
     ```
  2. Submit job run:
     ```bash
     aws emr-serverless start-job-run \
       --application-id app-1234567890 \
       --execution-role-arn arn:aws:iam::123456789012:role/EMRServerlessRole \
       --job-driver '{"sparkSubmit": {"entryPoint": "s3://my-bucket/jobs/etl.py"}}'
     ```
- **Estimated Savings:** **60–85% overall cost reduction** for non-continuous batch workloads.
- **Risk Level:** Low to Medium.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Code compatibility with EMR Serverless runtime release.

#### 4. Migrate Shared Kubernetes Workloads to EMR on EKS
- **What:** Run Spark jobs on existing enterprise EKS clusters using **EMR on EKS**.
- **Why It Saves Money:** Eliminates dedicated EC2 master/core node provisioning by sharing worker node pools with other containerized applications, while benefiting from Karpenter bin-packing. EMR management fee is a flat **$0.015/vCPU-hr**.
- **Detailed Implementation Steps:**
  1. Enable EMR on EKS virtual cluster:
     ```bash
     aws emr-containers create-virtual-cluster \
       --name "eks-spark-cluster" \
       --container-provider '{"type": "EKS", "id": "eks-prod-cluster", "info": {"eksInfo": {"namespace": "emr-jobs"}}}'
     ```
- **Estimated Savings:** 30–50% compute infrastructure savings.
- **Risk Level:** Medium.
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** Existing EKS cluster.

---

### 3. Pricing Model & Spot Optimization

#### 5. Deploy Spot Instances for 100% of Stateless Task Nodes
- **What:** Configure EMR Instance Fleets / Groups to use **Spot Instances** for 100% of Task Nodes (stateless compute workers).
- **Why It Saves Money:** Spot instances slash underlying EC2 compute costs by **70–90%** (e.g. `m5.2xlarge` Spot costs ~$0.115/hr vs $0.384/hr On-Demand). Note: EMR management fee ($0.096/hr) remains flat.
- **Detailed Implementation Steps:**
  1. Configure EMR Instance Fleet JSON specifying multiple Spot instance types:
     ```json
     {
       "Name": "TaskFleet",
       "InstanceFleetType": "TASK",
       "TargetSpotCapacity": 40,
       "InstanceTypeConfigs": [
         {"InstanceType": "m5.2xlarge", "BidPriceAsPercentageOfOnDemandPrice": 100},
         {"InstanceType": "m5a.2xlarge", "BidPriceAsPercentageOfOnDemandPrice": 100},
         {"InstanceType": "m6g.2xlarge", "BidPriceAsPercentageOfOnDemandPrice": 100}
       ],
       "LaunchTemplateConfigs": [...]
     }
     ```
  2. Set `AllocationStrategy = CAPACITY_OPTIMIZED` to minimize Spot interruption risk.
- **Estimated Savings:** **50–70% net savings** on total task node billing.
- **Risk Level:** Low (Task nodes are stateless; Spark handles task re-execution automatically).
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** Workload can handle node terminations gracefully.

#### 6. Apply Savings Plans / RIs to Master and Core Nodes Only
- **What:** Purchase Compute Savings Plans or EC2 RIs covering ONLY steady-state Master and Core nodes for continuous 24/7 clusters.
- **Why It Saves Money:** Slashes underlying EC2 compute fees for static nodes by **up to 66%**.
- **Detailed Implementation Steps:**
  1. Calculate static baseline instance hours for Master/Core nodes.
  2. Purchase Compute Savings Plan commitment in Cost Explorer.
- **Estimated Savings:** 20–50% savings on static node compute.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Baseline cluster stability.

---

### 4. Storage Architecture (EMRFS & S3)

#### 7. Decouple Compute and Storage via EMRFS & S3 (Minimize Core Nodes)
- **What:** Store all persistent datasets in **Amazon S3** accessed via the EMR File System (EMRFS) instead of local HDFS on Core Nodes.
- **Why It Saves Money:** Storing data in HDFS forces you to scale out Core Nodes (paying for expensive EC2 compute and EBS volumes) just to get disk space. Decoupling allows reducing Core Nodes to a minimum (1 node) and scaling compute with cheap Spot Task Nodes.
- **Detailed Implementation Steps:**
  1. Change Spark read/write URIs from `hdfs:///` to `s3://company-datalake/`.
  2. Reduce Core Node Group size to 1 instance in cluster configuration.
- **Estimated Savings:** 40–70% reduction in Core Node compute and EBS spend.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** S3 data lake architecture.

#### 8. Convert Raw Spark Data to Apache Parquet / ORC
- **What:** Write Spark job output to S3 in compressed **Apache Parquet** format using Snappy compression.
- **Why It Saves Money:** Parquet reduces storage volume by 80–90% ($0.023/GB-mo S3 storage) and accelerates downstream Spark/Athena read queries by 10x, shortening EMR job runtimes.
- **Detailed Implementation Steps:**
  1. Update Spark DataFrame write statement in PySpark code:
     ```python
     df.write.mode("overwrite").option("compression", "snappy").parquet("s3://bucket/processed-data/")
     ```
- **Estimated Savings:** 80% reduction in storage volume and 50%+ reduction in read job execution time.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Spark code update.

---

### 5. Capacity Rightsizing & Auto-Scaling

#### 9. Enable EMR Managed Scaling
- **What:** Turn on **EMR Managed Scaling** on EMR clusters to dynamically scale Task Nodes based on YARN memory and CPU demand.
- **Why It Saves Money:** Prevents static over-provisioning of task nodes during light processing stages.
- **Detailed Implementation Steps:**
  1. Attach managed scaling policy via AWS CLI:
     ```bash
     aws emr put-managed-scaling-policy \
       --cluster-id j-1234567890 \
       --managed-scaling-policy ComputeLimits='{MinimumCapacityUnits=2,MaximumCapacityUnits=50,UnitType=Instances}'
     ```
- **Estimated Savings:** 25–45% compute optimization.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** EMR version 5.30.0+ or 6.1.0+.

#### 10. Migrate Instance Families to AWS Graviton (`m6g`/`c6g`/`r6g`)
- **What:** Update EMR instance types from x86 (`m5.2xlarge`) to Graviton-based instance families (`m6g.2xlarge`).
- **Why It Saves Money:** Graviton instances provide a **20% lower EC2 hourly rate** and up to **15% lower EMR management fee** while delivering superior Spark execution performance per vCPU.
- **Detailed Implementation Steps:**
  1. Update cluster launch template or `aws emr create-cluster` command:
     ```bash
     --instance-groups InstanceGroupType=MASTER,InstanceType=m6g.xlarge,InstanceCount=1 ...
     ```
- **Estimated Savings:** **20–35% price-performance improvement**.
- **Risk Level:** Zero risk (Spark runtimes natively support ARM64).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

---

### 6. Performance & Job Optimization

#### 11. Tune Spark Executor Memory & Cores Allocation
- **What:** Configure Spark submit parameters (`spark.executor.memory`, `spark.executor.cores`, `spark.driver.memory`) to maximize worker node RAM/CPU utilization.
- **Why It Saves Money:** Poorly tuned Spark submit scripts leave 50% of node RAM unallocated, forcing extra nodes to be launched.
- **Detailed Implementation Steps:**
  1. Add optimal Spark submit flags:
     ```bash
     spark-submit --num-executors 19 --executor-cores 5 --executor-memory 19g ...
     ```
- **Estimated Savings:** 20–40% reduction in cluster node requirements.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Node CPU/RAM sizing calculation.

#### 12. Enable Dynamic Allocation in Apache Spark
- **What:** Set `spark.dynamicAllocation.enabled = true` in `spark-defaults.conf`.
- **Why It Saves Money:** Allows Spark applications to release idle executors back to YARN when job stages complete.
- **Detailed Implementation Steps:**
  1. Add configuration to EMR cluster creation:
     ```json
     [
       {
         "Classification": "spark-defaults",
         "Properties": {
           "spark.dynamicAllocation.enabled": "true",
           "spark.shuffle.service.enabled": "true"
         }
       }
     ]
     ```
- **Estimated Savings:** 15–30% capacity reclamation during job execution.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** EMR Spark default configuration.

#### 13. Optimize S3 Multipart Uploads and Committer
- **What:** Enable the **EMRFS S3 Optimized Committer** (`spark.sql.parquet.fs.optimized.committer.optimization-enabled = true`).
- **Why It Saves Money:** Eliminates expensive S3 rename operations during Spark write stages, accelerating job completion times by 2x–3x.
- **Detailed Implementation Steps:**
  1. Enable EMRFS S3 Committer in Spark properties.
- **Estimated Savings:** 30–50% reduction in job duration (saving hourly compute fees).
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** EMR 5.19.0+ release.

#### 14. Minimize EBS Volume Provisioning per Worker Node
- **What:** Reduce attached EBS volume sizes on worker nodes to the default **32 GB gp3** volume unless local shuffle disk space is strictly required.
- **Why It Saves Money:** EBS volumes attached to EMR nodes bill at standard EBS rates ($0.08/GB-mo). Allocating 500 GB EBS per node across 100 nodes adds **$400/month** in unnecessary storage fees.
- **Detailed Implementation Steps:**
  1. Set `EbsBlockDeviceConfigs` size to 32 GB in launch configuration.
- **Estimated Savings:** 70–90% reduction in worker node EBS storage fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Shuffle disk usage audit.

#### 15. Automate Cluster Orchestration via AWS Step Functions
- **What:** Replace persistent 24/7 clusters with ephemeral clusters spun up on-demand by **AWS Step Functions** for batch processing workflows.
- **Why It Saves Money:** Step Functions creates the cluster, submits the Spark step, waits for completion, and automatically terminates the cluster.
- **Detailed Implementation Steps:**
  1. Define Step Functions State Machine with EMR `CreateCluster`, `AddJobFlowSteps`, and `TerminateClusters` tasks.
- **Estimated Savings:** 60–80% overall cost reduction vs 24/7 running clusters.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** Workflow orchestration setup.

#### 16. Enforce Node Lifecycle Script Timeouts
- **What:** Add strict execution timeouts (e.g. 5 minutes) to EMR Bootstrap Actions.
- **Why It Saves Money:** Prevents hanging bootstrap scripts from keeping nodes in a `STARTING` state indefinitely.
- **Detailed Implementation Steps:**
  1. Add `timeout 300` wrapper in bootstrap shell scripts.
- **Estimated Savings:** Prevents hung cluster initialization fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Custom bootstrap script maintenance.

#### 17. Use CloudWatch Alarms for EMR Idle YARN Memory
- **What:** Set CloudWatch alarm on `YARNMemoryAvailablePercentage = 100%` for > 30 minutes.
- **Why It Saves Money:** Sends immediate alert to DevOps when clusters sit completely idle.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm targeting `AWS/ElasticMapReduce` namespace.
- **Estimated Savings:** Operational cost alert protection.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 18. Leverage EMR Studio Free Tier Allowances
- **What:** Use EMR Studio workspace notebooks for interactive analysis.
- **Why It Saves Money:** EMR Studio user management is free; you pay only for attached cluster compute.
- **Detailed Implementation Steps:**
  1. Configure EMR Studio workspace with IAM Identity Center.
- **Estimated Savings:** Administrative notebook cost savings.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** IAM Identity Center enabled.

---

## Cross-Service Synergies

```
[ Amazon EMR Cluster ] 
        │
        ├──(Stateless Workers)───> [ 100% Spot Task Nodes ] (Saves 70-90% on task compute)
        │
        ├──(Serverless Migration)─> [ EMR Serverless ] (Saves 60-85% for intermittent batch jobs)
        │
        └──(Data Storage)────────> [ S3 + EMRFS + Parquet ] (Decouples compute from storage)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `EMR-Usage`, `BoxUsage:m5.xlarge`, `EMRServerless-vCPU-Hours`, `EMRServerless-GB-Hours`.
- `line_item_resource_id`: EMR Cluster ID (`j-xxxx`) / EMR Serverless Application ID.

### B. CloudWatch & YARN Metrics
- `AWS/ElasticMapReduce` Namespace: `IsIdle`, `YARNMemoryAvailablePercentage`, `ContainerAllocated`, `CoreNodesRunning`, `TaskNodesRunning`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "EMR-SPT-001",
  "service": "EMR",
  "category": "Pricing Model & Spot Optimization",
  "resource_id": "j-1234567890ABCDEF",
  "resource_name": "daily-etl-analytics-cluster",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "master_nodes": "1 x m5.xlarge (On-Demand)",
    "core_nodes": "2 x m5.xlarge (On-Demand)",
    "task_nodes": "20 x m5.2xlarge (On-Demand)",
    "monthly_cost_usd": 7257.60
  },
  "recommended_config": {
    "master_nodes": "1 x m6g.xlarge (On-Demand)",
    "core_nodes": "1 x m6g.xlarge (On-Demand)",
    "task_nodes": "20 x m6g.2xlarge (100% Spot Instances)",
    "projected_monthly_cost_usd": 2480.00
  },
  "financial_impact": {
    "monthly_savings_usd": 4777.60,
    "annual_savings_usd": 57331.20,
    "savings_percentage": 65.8
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "Task nodes are stateless; Spark handles Spot interruptions gracefully via task re-execution."
  },
  "implementation": {
    "scope": "Data Engineer / DevOps",
    "effort_estimate": "1-2 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Spot Task Nodes (Compute)** | 8 | $38,500.00 | $25,333.00 | 65.8% | Low |
| **Serverless Migration (Batch)** | 5 | $18,200.00 | $12,740.00 | 70.0% | Low |
| **Waste Elimination (Auto-Terminate)**| 12 | $14,000.00 | $11,200.00 | 80.0% | Zero |
| **Storage Decoupling (S3/EMRFS)** | 6 | $9,800.00 | $4,900.00 | 50.0% | Low |
| **Total** | **31** | **$80,500.00** | **$54,173.00** | **67.3%** | -- |
