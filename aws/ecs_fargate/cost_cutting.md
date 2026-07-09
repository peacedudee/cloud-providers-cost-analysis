# Cost-Cutting Playbook: Amazon ECS & AWS Fargate

> **Companion Files:** [ecs.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/ecs_fargate/ecs.md) | [fargate.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/ecs_fargate/fargate.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon ECS orchestration itself is **100% FREE ($0.00)**; billing is driven entirely by the underlying compute launch type: **EC2 Launch Type** (paying for full VM allocations regardless of task utilization) vs **AWS Fargate** (serverless pay-per-second per requested vCPU/RAM). Key cost leaks include over-provisioned task definition limits, ECR image bloat, Fargate tasks running in public subnets with public IP fees, and missing Fargate Spot/Savings Plans commitments.

This playbook provides **20 actionable strategies** across seven categories, yielding an estimated **30–65% reduction in total ECS/Fargate container spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Migrate Fargate Tasks to ARM64 (AWS Graviton):** Instant 20% price reduction per vCPU and GB-hour.
2. **Deploy Fargate Spot Capacity Providers for Non-Prod:** Slashes Fargate compute fees by **70%** for fault-tolerant workloads.
3. **Configure ECR Lifecycle Policies:** Deletes untagged and historical image builds, stopping ECR storage growth.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Implement ECR Lifecycle Policies to Delete Legacy Container Images
- **What:** Configure automated ECR Lifecycle Policies across all private repositories to expire untagged images after 1 day and retain only the last 10 tagged deployment images.
- **Why It Saves Money:** ECR charges **$0.10 per GB-month**. CI/CD pipelines pushing 1 GB images on every commit can accumulate terabytes of historical images, costing hundreds of dollars per month in dead storage.
- **Implementation Steps:**
  1. Add ECR lifecycle policy JSON: Expire untagged images `sinceImagePushed` count = 1.
  2. Set tagged image retention policy `imageCountMoreThan` = 10.
  3. Apply across all ECR repositories via Terraform module.
- **Estimated Savings:** 60–90% reduction in ECR storage fees.
- **Risk Level:** Zero risk (active deployments reference explicit tags).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** ECR repository access.

#### 2. Terminate Stopped or Restart-Looping Tasks (CrashLoopBackOff)
- **What:** Identify and fix ECS tasks stuck in continuous crash-restart loops due to configuration errors or missing environment variables.
- **Why It Saves Money:** Fargate has a **1-minute minimum billable duration** per task launch. A task crashing every 10 seconds and restarting 6 times/minute bills for 6 full task-minutes every minute!
- **Implementation Steps:**
  1. Set CloudWatch Alarm on ECS metric `EssentialContainerExited`.
  2. Configure ECS service deployment circuit breaker with rollback.
  3. Fix root cause or stop service during troubleshooting.
- **Estimated Savings:** Prevents 6x-10x cost inflation during application outages.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch Container Insights enabled.

#### 3. Decommission Idle ECS Clusters & Task Definitions
- **What:** Deregister unused task definition revisions and delete empty ECS clusters in non-prod accounts.
- **Why It Saves Money:** Prevents deployment clutter and stops attached EBS/ENI resources.
- **Implementation Steps:**
  1. Run `aws ecs list-clusters` and filter for `runningTasksCount = 0` and `pendingTasksCount = 0`.
  2. Delete empty clusters.
- **Estimated Savings:** Administrative efficiency and storage cleanup.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Cluster workload verification.

---

### 2. Rightsizing

#### 4. Migrate Fargate Tasks to ARM64 (AWS Graviton) Architecture
- **What:** Update ECS Task Definitions to run on `ARM64` CPU architecture instead of `X86_64`.
- **Why It Saves Money:** Fargate ARM64 pricing is **20% cheaper** than x86 ($0.03238/vCPU-hr vs $0.04048/vCPU-hr for CPU; $0.00356/GB-hr vs $0.004445/GB-hr for RAM).
- **Implementation Steps:**
  1. Update Task Definition JSON: Set `"runtimePlatform": {"cpuArchitecture": "ARM64", "operatingSystemFamily": "LINUX"}`.
  2. Build container images for `linux/arm64`.
  3. Redeploy ECS service.
- **Estimated Savings:** 20% direct reduction in Fargate compute bill.
- **Risk Level:** Low (for Node.js, Python, Java, Go apps).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-architecture container build pipeline.

#### 5. Right-Size Task Definition vCPU and Memory Allocations
- **What:** Analyze Container Insights CPU and Memory utilization metrics for ECS services over 14 days and downsize task allocations (e.g. from 2 vCPU / 4 GB down to 0.5 vCPU / 1 GB).
- **Why It Saves Money:** Fargate bills for requested task size regardless of consumption. Over-allocating by 4x wastes 75% of your Fargate spend ($54/mo per task wasted).
- **Implementation Steps:**
  1. Review Container Insights `CpuUtilized` vs `CpuReserved` and `MemoryUtilized` vs `MemoryReserved`.
  2. Update Task Definition CPU/Memory parameters to match peak usage + 20% safety buffer.
  3. Deploy updated task definition revision.
- **Estimated Savings:** 30–75% reduction in Fargate task billing.
- **Risk Level:** Low (easily revertible via Task Definition revision rollback).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch Container Insights metrics.

#### 6. Optimize Fargate Discrete Configuration Sizing Tiers
- **What:** Align CPU and Memory requests to match valid Fargate configuration tier boundaries (e.g. 0.25 vCPU supports 0.5GB, 1GB, 2GB; 1 vCPU supports 2GB to 8GB in 1GB increments).
- **Why It Saves Money:** Specifying off-tier sizes (e.g. requesting 1.1 vCPU) causes Fargate to automatically round up to 2 vCPU, billing an extra 0.9 vCPU continuously.
- **Implementation Steps:**
  1. Review Task Definitions against official Fargate configuration matrix.
  2. Adjust parameters to exact boundary values.
- **Estimated Savings:** Eliminates 10–40% accidental tier-rounding overhead.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Fargate sizing documentation review.

#### 7. Ephemeral Storage Sizing Optimization
- **What:** Keep Fargate ephemeral storage at the default **20 GB free baseline** unless larger disk storage is required.
- **Why It Saves Money:** Storage configured above 20 GB costs **$0.000111 per GB-hour ($0.08/GB-month)** per task.
- **Implementation Steps:**
  1. Check `ephemeralStorage: { sizeInGiB: XX }` in task definition.
  2. Remove parameter if 20 GB default is sufficient.
- **Estimated Savings:** 100% of extra ephemeral storage fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Task disk space requirement check.

---

### 3. Commitment Discounts

#### 8. Apply Compute Savings Plans to Cover Fargate Baseline Spend
- **What:** Commit to a 1-year or 3-year Compute Savings Plan covering steady-state Fargate spend.
- **Why It Saves Money:** Compute Savings Plans apply automatically to Fargate vCPU and Memory usage, securing discounts up to **20% (1-year)** or **52% (3-year)** off On-Demand rates.
- **Implementation Steps:**
  1. Analyze historical minimum hourly Fargate spend in Cost Explorer.
  2. Purchase Compute Savings Plan matching 75% of baseline.
- **Estimated Savings:** 20–52% off On-Demand Fargate.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Multi-month Fargate spend stability.

---

### 4. Architecture Changes

#### 9. Implement Fargate Spot Capacity Providers for Non-Production & Batch
- **What:** Configure ECS Service Capacity Provider Strategies to route dev, staging, and batch queue tasks to `FARGATE_SPOT`.
- **Why It Saves Money:** Fargate Spot provides a **70% direct discount** off standard Fargate rates.
- **Implementation Steps:**
  1. Add `FARGATE_SPOT` capacity provider to ECS cluster.
  2. Set ECS Service capacity provider strategy: `FARGATE_SPOT` weight = 1, base = 0 (or base = 1 On-Demand, weight = 4 Spot).
- **Estimated Savings:** **70% savings** on compute fees for targeted tasks.
- **Risk Level:** Medium (tasks receive 2-minute interruption notification).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Tasks must handle graceful shutdown upon SIGTERM.

#### 10. Optimize EC2 Launch Type Bin-Packing with `binpack` Placement Strategy
- **What:** For ECS clusters using the EC2 launch type, configure the `binpack` task placement strategy based on memory or CPU.
- **Why It Saves Money:** Packs tasks tightly onto as few EC2 instances as possible, allowing Auto Scaling Groups to terminate empty host instances at the cluster edge.
- **Implementation Steps:**
  1. Update ECS Service placement strategy: `type: "binpack", field: "memory"`.
  2. Configure EC2 ASG capacity provider with target tracking scaling.
- **Estimated Savings:** 20–40% reduction in EC2 host count.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** ECS EC2 Launch Type usage.

#### 11. Evaluate Fargate vs EC2 Launch Type Break-Even Point
- **What:** Compare costs between Fargate and EC2 launch types for large, steady-state production clusters.
- **Why It Saves Money:** For high-density, 24/7 steady-state workloads (> 80% host utilization), EC2 launch type with Graviton instances (`m8g`) and 3-year Savings Plans is ~30–40% cheaper than On-Demand Fargate. For variable/spiky workloads, Fargate is cheaper.
- **Implementation Steps:**
  1. Run break-even analysis comparing total task vCPU/RAM cost on Fargate vs fully packed `m8g.2xlarge` EC2 hosts.
  2. Migrate high-density steady-state services to EC2 launch type or leverage Fargate Spot/Savings Plans.
- **Estimated Savings:** 25–40% structural architecture savings.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Financial model evaluation.

---

### 5. Scheduling & Auto-Scaling

#### 12. Configure Target Tracking Service Auto-Scaling
- **What:** Enable ECS Service Auto Scaling using Target Tracking policies based on `ECSServiceAverageCPUUtilization` or `ECSServiceAverageMemoryUtilization` (target 70%).
- **Why It Saves Money:** Automatically scales task count down during off-peak hours (e.g. dropping from 20 tasks to 2 tasks overnight), matching compute directly to demand.
- **Implementation Steps:**
  1. Register ECS Service as scalable target: `aws application-autoscaling register-scalable-target`.
  2. Put scaling policy: Target value = 70.0%.
- **Estimated Savings:** 30–60% off static peak task capacity.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Stateless task design.

#### 13. Schedule Non-Production ECS Services to Scale to Zero Off-Hours
- **What:** Configure scheduled scaling actions to set `DesiredCount = 0` on dev/test ECS services outside business hours.
- **Why It Saves Money:** Running dev tasks 50 hours/week instead of 168 hours/week cuts non-prod Fargate/EC2 billing by **70%**.
- **Implementation Steps:**
  1. Create Application Auto Scaling scheduled action: `DesiredCount = 0` at 7 PM M-F.
  2. Create morning action: `DesiredCount = 2` at 7 AM M-F.
- **Estimated Savings:** 70% savings on non-prod container spend.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod SLA alignment.

---

### 6. Pricing Model Optimization

#### 14. Mix On-Demand Base with Fargate Spot Scaling
- **What:** Configure ECS Service capacity provider strategy with `FARGATE` base = 1 (1 task On-Demand for high availability) and `FARGATE_SPOT` weight = 4 (80% of scaling tasks on Spot).
- **Why It Saves Money:** Ensures baseline availability while capturing 70% Spot discounts on peak traffic scale-out.
- **Implementation Steps:**
  1. Update ECS Service capacity provider strategy: `[{capacityProvider: "FARGATE", base: 1, weight: 1}, {capacityProvider: "FARGATE_SPOT", base: 0, weight: 4}]`.
- **Estimated Savings:** 55–65% overall cluster cost reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** ECS Capacity Provider setup.

---

### 7. Network & Data Transfer Optimization

#### 15. Deploy ECR & S3 VPC Endpoints to Avoid NAT Processing Fees
- **What:** Create Interface VPC Endpoints for ECR (`ecr.api`, `ecr.dkr`) and Gateway Endpoint for S3 in the ECS VPC.
- **Why It Saves Money:** Fargate tasks running in private subnets pull container images through NAT Gateways ($0.045/GB). Pulling a 1 GB image across 100 task launches costs **$4.50 in NAT processing fees**. VPC Endpoints drop transfer costs to $0.01/GB.
- **Implementation Steps:**
  1. Create ECR Interface Endpoints and S3 Gateway Endpoint in VPC.
  2. Enable Private DNS.
- **Estimated Savings:** **78% savings** on container image pull network costs.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Private subnet Fargate tasks.

#### 16. Eliminate Public IPv4 Address Fees on Fargate Tasks
- **What:** Run Fargate tasks in private subnets with `assignPublicIp = DISABLED`.
- **Why It Saves Money:** Tasks launched in public subnets with `assignPublicIp = ENABLED` incur the **$0.005/hour ($3.65/mo)** public IPv4 address fee per task. 50 tasks with public IPs cost $182.50/mo unnecessarily.
- **Implementation Steps:**
  1. Update ECS Service network configuration: `awsvpcConfiguration: { assignPublicIp: "DISABLED", subnets: [private_subnet_ids] }`.
- **Estimated Savings:** $3.65 per task per month (100% public IP fee elimination).
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Private subnets with VPC endpoints or NAT for outbound traffic.

#### 17. Optimize CloudWatch Logs `awslogs` Ingestion & Retention
- **What:** Set log group retention limits (e.g. 7 days) and filter verbose debug logs from `awslogs` driver output.
- **Why It Saves Money:** CloudWatch Logs charges **$0.50/GB** for ingestion and $0.03/GB-mo for storage. Verbose stdout container logging can easily generate $1,000s/mo in log bills.
- **Implementation Steps:**
  1. Update log group retention to 7 or 14 days.
  2. Adjust application log levels in production from `DEBUG` to `INFO`/`WARN`.
- **Estimated Savings:** 50–80% reduction in container logging fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Logging policy review.

#### 18. Consolidate ALBs across ECS Services using Host/Path-Based Ingress
- **What:** Share a single Application Load Balancer across multiple ECS services using Target Group path-based or host-based routing rules.
- **Why It Saves Money:** Eliminates the ~$20.00/month flat fee per load balancer for isolated microservices.
- **Implementation Steps:**
  1. Add listener rules to primary ALB targeting distinct ECS service target groups.
- **Estimated Savings:** ~$20.00/month saved per consolidated service.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** ALB listener rule management.

#### 19. Co-Locate Tightly Coupled ECS Tasks in the Same Availability Zone
- **What:** Align ECS tasks communicating heavily with specific RDS databases into the same Availability Zone.
- **Why It Saves Money:** Eliminates inter-AZ cross-subnet data transfer fees ($0.02/GB roundtrip).
- **Implementation Steps:**
  1. Use placement constraints or subnet targeting for AZ alignment.
- **Estimated Savings:** $0.02 per GB transferred.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-AZ HA review.

#### 20. Implement AWS Secrets Manager Caching in Task Code
- **What:** Cache Secrets Manager or SSM Parameter Store secrets in task memory rather than fetching secrets on every request or task invocation.
- **Why It Saves Money:** Secrets Manager charges **$0.05 per 10,000 API calls**. Fetching secrets repeatedly across millions of task requests generates unnecessary API charges.
- **Implementation Steps:**
  1. Implement client-side secret caching with 5-minute TTL.
- **Estimated Savings:** 100% of repetitive Secrets Manager API fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Code secret client update.

---

## Cross-Service Synergies

```
[ ECS / Fargate Tasks ] 
        │
        ├──(Architecture)───> [ Graviton ARM64 ] (20% direct compute savings)
        │
        ├──(Pricing Model)──> [ Fargate Spot ] (70% compute savings for non-prod/batch)
        │
        ├──(Image Pulls)────> [ ECR + VPC Endpoints ] (Avoids $0.045/GB NAT fees)
        │
        └──(Image Storage)──> [ ECR Lifecycle Policy ] (Deletes old image tags, stops storage growth)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Fargate-vCPU-Hours`, `Fargate-GB-Hours`, `Fargate-ARM-vCPU-Hours`, `Fargate-ARM-GB-Hours`, `BoxUsage`.
- `line_item_resource_id`: ECS Task Definition ARN / Service Name.

### B. CloudWatch Metrics & Diagnostics
- Container Insights Namespace `ECS/ContainerInsights`: `CpuUtilized`, `CpuReserved`, `MemoryUtilized`, `MemoryReserved`, `StorageUtilized`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "ECS-RS-001",
  "service": "ECS/Fargate",
  "category": "Rightsizing",
  "resource_id": "arn:aws:ecs:us-east-1:123456789012:service/prod-cluster/order-processing-service",
  "resource_name": "order-processing-service",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "launch_type": "FARGATE",
    "architecture": "X86_64",
    "task_vcpu": 2.0,
    "task_memory_gb": 8.0,
    "running_tasks": 4,
    "monthly_cost_usd": 421.20
  },
  "recommended_config": {
    "launch_type": "FARGATE",
    "architecture": "ARM64",
    "task_vcpu": 0.5,
    "task_memory_gb": 2.0,
    "running_tasks": 4,
    "projected_monthly_cost_usd": 84.24
  },
  "financial_impact": {
    "monthly_savings_usd": 336.96,
    "annual_savings_usd": 4043.52,
    "savings_percentage": 80.0
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "Container Insights show peak CPU < 25% and RAM < 1.2GB over 30 days; ARM64 multi-arch build verified."
  },
  "implementation": {
    "scope": "Engineer/DevOps",
    "effort_estimate": "1-2 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Waste Elimination (ECR/Tasks)** | 15 | $2,400.00 | $1,920.00 | 80.0% | Low |
| **Rightsizing (Graviton/Sizing)** | 28 | $11,200.00 | $5,600.00 | 50.0% | Low |
| **Pricing Model (Fargate Spot)** | 10 | $6,500.00 | $4,550.00 | 70.0% | Medium |
| **Network & VPC Endpoints** | 16 | $3,800.00 | $2,850.00 | 75.0% | Low |
| **Total** | **69** | **$23,900.00** | **$14,920.00** | **62.4%** | -- |
