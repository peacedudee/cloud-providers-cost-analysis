# Cost-Cutting Playbook: AWS Compute Optimizer
> **Companion File:** [compute_optimizer.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/compute_optimizer/compute_optimizer.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Compute Optimizer is a machine-learning-driven service that analyzes historical utilization metrics to provide right-sizing recommendations across compute, storage, database, and machine learning services. It identifies over-provisioned resources to reduce costs and under-provisioned resources to improve performance. The core 14-day lookback is completely free, making it an essential foundational tool for FinOps. By methodically acting upon Compute Optimizer’s findings—from downsizing EC2 instances and EBS volumes to adopting newer instance generations like Graviton and transitioning to gp3 volumes—organizations can consistently realize 20-40% savings on their AWS infrastructure spend without risking operational stability.

## Strategy Categories

### 1. Waste Elimination

#### 1. Disable Enhanced Metrics for Ephemeral/Non-Prod Workloads
- **What:** Compute Optimizer offers a 3-month Enhanced Infrastructure Metrics lookback for $0.25 per resource-month. This should only be used on production workloads. Disable it for dev/test and short-lived containers.
- **Why It Saves Money:** At scale, paying $0.25/month for thousands of ephemeral test instances or short-lived ECS tasks creates unnecessary overhead. The free 14-day lookback is sufficient for non-prod.
- **Implementation Steps:**
  1. Audit AWS accounts for the Enhanced Infrastructure Metrics setting.
  2. Identify development, sandbox, or test accounts where long-term trends are irrelevant.
  3. Turn off Enhanced Metrics in the Compute Optimizer settings for these environments.
  4. Restrict IAM permissions to prevent developers from turning it on in sandbox accounts.
- **Estimated Savings:** 100% of Compute Optimizer enhanced metric costs in non-prod.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Organizational view of Compute Optimizer enabled.

#### 2. Identify and Remove Completely Idle NAT Gateways
- **What:** Use Compute Optimizer’s analysis of NAT Gateways to find gateways that process zero or near-zero bytes over a 14-day period.
- **Why It Saves Money:** Each idle NAT Gateway costs ~$32/month just to exist, even with no traffic processing.
- **Implementation Steps:**
  1. Open Compute Optimizer and filter for NAT Gateway recommendations.
  2. Identify gateways tagged as idle or with practically zero data processed.
  3. Verify via VPC Flow Logs or CloudWatch that the gateway is truly unused.
  4. Delete the idle NAT Gateway and remove its route from the routing tables.
- **Estimated Savings:** $32/month per idle NAT Gateway.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC Flow logs or deep knowledge of network architecture.

#### 3. Automate Organizational-Wide Recommendation Export
- **What:** Instead of checking account-by-account, export recommendations for the entire AWS Organization to an S3 bucket to create a centralized FinOps backlog.
- **Why It Saves Money:** Un-actioned recommendations represent wasted money. Centralizing the data allows FinOps to mandate and track waste-elimination campaigns globally.
- **Implementation Steps:**
  1. Enable "Organizational View" in Compute Optimizer from the AWS Management Account.
  2. Set up an automated S3 export of all recommendations.
  3. Ingest the CSV/JSON into Amazon Athena or a BI tool (QuickSight) to create FinOps dashboards.
  4. Assign high-value right-sizing tickets to engineering teams.
- **Estimated Savings:** 30–60% compute cost reductions across the organization globally.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** AWS Organizations enabled; delegated administrator for Compute Optimizer configured.

### 2. Rightsizing

#### 4. Rightsize Over-Provisioned EC2 Instances
- **What:** Downgrade EC2 instances that are consistently utilizing a small fraction of their CPU/Memory.
- **Why It Saves Money:** Halving the size of an instance (e.g., from `m5.4xlarge` to `m5.2xlarge`) cuts the hourly compute cost exactly in half.
- **Implementation Steps:**
  1. Review Compute Optimizer EC2 recommendations filtered by `Finding = Overprovisioned`.
  2. Check the "Estimated monthly savings" column to prioritize the biggest monetary wins.
  3. Confirm the target instance type meets memory, network, and storage IOPS requirements.
  4. Update IaC (Terraform/CloudFormation) and restart the instance on the new size during a maintenance window.
- **Estimated Savings:** 50-75% per instance rightsized.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workloads that can tolerate a reboot.

#### 5. Rightsize Amazon RDS Instances
- **What:** Use Compute Optimizer to identify RDS databases (MySQL, PostgreSQL, etc.) that have excessive vCPU and memory.
- **Why It Saves Money:** Database instances are expensive. Halving an RDS instance class saves 50% on hourly DB instance billing, which often dwarfs EC2 costs.
- **Implementation Steps:**
  1. View RDS recommendations in Compute Optimizer.
  2. Cross-reference metrics with Performance Insights to ensure no hidden I/O or memory bottlenecks exist.
  3. Schedule a database modification to scale down the instance class.
- **Estimated Savings:** 50% per database.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-AZ configured (to minimize downtime during the resize).

#### 6. Optimize AWS Lambda Function Memory
- **What:** Compute Optimizer recommends optimal memory sizes for Lambda functions based on execution times and memory usage.
- **Why It Saves Money:** Lambda is billed by GB-seconds. Reducing memory from 1024MB to 512MB halves the cost per millisecond. (However, if execution time doubles, cost remains the same. Compute Optimizer calculates the sweet spot).
- **Implementation Steps:**
  1. Review Lambda recommendations in Compute Optimizer.
  2. Identify functions where memory can be lowered without proportionately increasing execution time.
  3. Update Serverless Framework, SAM, or Terraform templates.
  4. Deploy the updated memory configuration.
- **Estimated Savings:** 10-50% on Lambda compute costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Lambda functions must be invoked at least 50 times over the lookback period.

#### 7. Downsize Over-Provisioned EBS Volumes
- **What:** Compute Optimizer identifies EBS volumes with provisioned IOPS or throughput that vastly exceeds the actual usage.
- **Why It Saves Money:** You pay for provisioned storage, IOPS, and throughput whether you use them or not. Rightsizing IOPS/throughput directly reduces monthly volume costs.
- **Implementation Steps:**
  1. Look for EBS volumes tagged as `Over-provisioned` in IOPS or Throughput.
  2. Review the recommended baseline IOPS.
  3. Modify the volume via the AWS Console or CLI (Note: volumes can be modified without detaching).
- **Estimated Savings:** 20-60% per volume on IOPS/Throughput fees.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 8. Rightsize ECS Services on AWS Fargate
- **What:** Adjust the CPU and memory task definitions for ECS Fargate services based on utilization history.
- **Why It Saves Money:** Fargate bills for requested vCPU and Memory. Dialing these down to match actual consumption directly reduces the hourly run rate.
- **Implementation Steps:**
  1. Go to ECS recommendations in Compute Optimizer.
  2. Identify tasks with low CPU/Memory utilization.
  3. Update the ECS Task Definition to reduce CPU/Memory values.
  4. Force a new deployment of the ECS service to apply the new task sizes.
- **Estimated Savings:** 20-50% on Fargate costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Containerized workloads must be stateless and gracefully handle rolling updates.

#### 9. Rightsize Amazon ElastiCache & MemoryDB Nodes
- **What:** Shrink over-provisioned ElastiCache (Redis/Memcached) and MemoryDB nodes.
- **Why It Saves Money:** In-memory caching instances are highly expensive. Scaling down from an `r6g.2xlarge` to an `r6g.xlarge` halves the node cost.
- **Implementation Steps:**
  1. Review ElastiCache recommendations in Compute Optimizer.
  2. Verify that scaling down won't trigger heavy swapping or evictions.
  3. Modify the cluster configuration to scale down the node type.
- **Estimated Savings:** 50% per node resized.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ensuring adequate buffer memory is maintained for Redis background saves.

#### 10. Rightsize SageMaker ML Endpoints
- **What:** Scale down the instance sizes used for hosting SageMaker ML inference endpoints.
- **Why It Saves Money:** ML instances (like `ml.g4dn` or `ml.p3`) are extremely expensive. If the inference payload and throughput don't require heavy GPU/CPU, a smaller instance saves thousands of dollars.
- **Implementation Steps:**
  1. Access SageMaker endpoint recommendations.
  2. Review current vs. recommended instance types based on latency and invocation metrics.
  3. Update the SageMaker endpoint configuration.
- **Estimated Savings:** 40-70% per endpoint.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Load testing the model on the smaller instance type to ensure latency SLAs are met.

### 3. Commitment Discounts

#### 11. Action Rightsizing Before Purchasing Savings Plans
- **What:** Execute Compute Optimizer rightsizing recommendations *before* committing to a 1- or 3-year Compute or EC2 Instance Savings Plan.
- **Why It Saves Money:** Buying a Savings Plan on an over-provisioned footprint locks you into paying for waste for 1 to 3 years. Rightsizing first lowers the baseline commitment required.
- **Implementation Steps:**
  1. Freeze Savings Plan purchases.
  2. Run a 2-week rightsizing sprint using Compute Optimizer data.
  3. Observe the new, lower baseline compute spend.
  4. Purchase the Savings Plan against the newly optimized baseline.
- **Estimated Savings:** Prevents over-committing by 10-30%.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Close coordination between Engineering and FinOps.

### 4. Architecture Changes

#### 12. Modernize to Graviton Instances (ARM)
- **What:** Act on Compute Optimizer recommendations to migrate x86 workloads (e.g., `m5`) to ARM-based AWS Graviton instances (e.g., `m6g`, `m7g`).
- **Why It Saves Money:** Graviton instances are priced ~20% lower than comparable x86 instances and often deliver up to 40% better performance.
- **Implementation Steps:**
  1. Filter Compute Optimizer for Graviton-compatible instance recommendations.
  2. Verify that the application (AMI, packages, containers) is compiled for ARM64 architecture.
  3. Test the application thoroughly in a staging environment on Graviton.
  4. Switch the production Auto Scaling Groups or standalone instances to the Graviton family.
- **Estimated Savings:** 20% baseline + potential additional performance savings.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application code and dependencies must support ARM64.

#### 13. Upgrade EBS Volumes from gp2 to gp3
- **What:** Use Compute Optimizer to identify `gp2` EBS volumes and transition them to newer `gp3` volumes.
- **Why It Saves Money:** `gp3` volumes are a flat 20% cheaper per GB than `gp2` and offer higher baseline performance (3,000 IOPS and 125 MB/s) without tying IOPS to volume size.
- **Implementation Steps:**
  1. Review EBS volume recommendations.
  2. Identify `gp2` volumes eligible for upgrade.
  3. Modify the volume type via CLI/Console (this happens seamlessly in the background with zero downtime).
  4. Update infrastructure-as-code templates to default to `gp3`.
- **Estimated Savings:** 20% on EBS storage costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 14. Transition to Serverless ElastiCache
- **What:** Use ElastiCache usage data to determine if a workload is highly spiky or variable, making it a candidate for ElastiCache Serverless.
- **Why It Saves Money:** Instead of provisioning for peak cache load 24/7, Serverless scales instantly and you only pay for data stored and compute requests made.
- **Implementation Steps:**
  1. Review Compute Optimizer metrics for Redis nodes.
  2. Identify nodes with extremely low average utilization but high sporadic peaks.
  3. Migrate the cache cluster to ElastiCache Serverless.
- **Estimated Savings:** 30-60% for highly variable workloads.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application must support the ElastiCache Serverless connection handling.

### 5. Scheduling & Auto-Scaling

#### 15. Optimize EC2 Auto Scaling Group (ASG) Configurations
- **What:** Compute Optimizer provides recommendations for Auto Scaling Groups, not just individual instances, to suggest a better instance type for the entire group.
- **Why It Saves Money:** Using a smaller or more modern instance type in an ASG amplifies savings across all horizontally scaled nodes.
- **Implementation Steps:**
  1. Review ASG recommendations in Compute Optimizer.
  2. Create a new Launch Template with the recommended, more cost-effective instance type.
  3. Update the ASG to use the new Launch Template.
  4. Trigger an Instance Refresh to roll out the new instances.
- **Estimated Savings:** 20-50% multiplied by the number of ASG nodes.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** ASG must be using Launch Templates (not legacy Launch Configurations).

#### 16. Scale Down SageMaker Endpoints for Dev/Test
- **What:** Use Compute Optimizer data to prove that Dev/Test SageMaker endpoints are completely idle at night and on weekends.
- **Why It Saves Money:** SageMaker inference endpoints bill by the hour. An idle GPU instance left on over the weekend wastes significant money.
- **Implementation Steps:**
  1. Confirm zero nighttime usage via Compute Optimizer/CloudWatch.
  2. Implement an AWS Lambda function triggered by EventBridge to delete or scale down dev SageMaker endpoints at 7 PM.
  3. Implement a corresponding trigger to restore them at 8 AM.
- **Estimated Savings:** ~70% of dev SageMaker compute costs (running 40 hours instead of 168 hours a week).
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Endpoints must not be required for asynchronous overnight testing.

### 6. Pricing Model Optimization

#### 17. Optimize DynamoDB Capacity Modes
- **What:** Compute Optimizer analyzes DynamoDB table utilization and recommends switching between Provisioned Capacity and On-Demand Capacity.
- **Why It Saves Money:** Provisioned capacity with Auto Scaling is far cheaper for steady, predictable workloads. On-Demand is cheaper for sparse, highly unpredictable workloads (to avoid over-provisioning).
- **Implementation Steps:**
  1. Review DynamoDB recommendations.
  2. For tables tagged for Provisioned, configure read/write capacity units (RCU/WCU) and enable Auto Scaling.
  3. For tables tagged for On-Demand, switch the billing mode directly.
- **Estimated Savings:** 20-70% depending on the workload profile switch.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of the table's baseline and peak traffic patterns.

### 7. Network & Data Transfer Optimization

#### 18. Replace Underutilized NAT Gateways with VPC Endpoints
- **What:** Compute Optimizer flags NAT Gateways with low traffic. For these, routing traffic through VPC Endpoints is often cheaper.
- **Why It Saves Money:** NAT Gateways charge ~$32/month plus $0.045 per GB processed. VPC Gateway Endpoints (for S3 and DynamoDB) are entirely free. VPC Interface Endpoints are cheaper per hour than NAT Gateways.
- **Implementation Steps:**
  1. Identify low-traffic NAT Gateways via Compute Optimizer.
  2. Determine the destination of the traffic (e.g., S3, DynamoDB, Systems Manager).
  3. Deploy VPC Gateway/Interface Endpoints for those specific services.
  4. Update VPC route tables to route traffic to the Endpoints instead of the NAT Gateway.
  5. Delete the NAT Gateway.
- **Estimated Savings:** $32/mo per NAT Gateway + $0.045 per GB transferred.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Deep understanding of VPC routing and application network dependencies.

---
## Cross-Service Synergies
- **Compute Optimizer + AWS Cost Explorer:** Use Compute Optimizer to execute rightsizing *before* using Cost Explorer to calculate and purchase AWS Savings Plans.
- **Compute Optimizer + AWS Systems Manager:** Use Systems Manager Automation Runbooks to programmatically execute the rightsizing recommendations surfaced by Compute Optimizer (e.g., upgrading gp2 to gp3 volumes automatically).
- **Compute Optimizer + AWS Config:** Write custom AWS Config rules to flag newly provisioned resources that violate the optimized baseline established by Compute Optimizer (e.g., banning gp2 volumes after a cleanup).

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Detailed hourly billing data to map Compute Optimizer recommendations back to actual invoiced costs and tags (e.g., to see exactly which team owns the over-provisioned resource).

### B. CloudWatch Metrics
- Granular metrics (CPU, Memory, Network, Disk I/O) that feed Compute Optimizer. Ensure the CloudWatch Agent is installed on EC2 instances to provide OS-level memory metrics, which drastically improves Compute Optimizer's recommendation accuracy.

### C. AWS Config / Trusted Advisor
- Trusted Advisor provides high-level cost optimization checks that overlap slightly with Compute Optimizer but lacks the deep machine learning analysis. Use AWS Config to track configuration changes made during rightsizing.

### D. Company Policies
- Tagging policies are critical so FinOps knows who to contact when Compute Optimizer finds a wildly over-provisioned SageMaker endpoint.

### E. IaC (Optional)
- Terraform state files and modules. Right-sizing manually in the console is an anti-pattern. Engineers must update the IaC templates so the next deployment doesn't overwrite the rightsizing changes.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "FindingID": "CO-004",
  "Strategy": "Rightsize Over-Provisioned EC2 Instances",
  "Category": "Rightsizing",
  "Impact": "High",
  "Effort": "Medium",
  "SavingsPotential": "50-75%"
}
```

### Summary Report Table

| Strategy ID | Strategy Name | Category | Savings Potential | Risk Level |
|-------------|---------------|----------|-------------------|------------|
| CO-001 | Disable Enhanced Metrics for Ephemeral/Non-Prod Workloads | Waste Elimination | 100% (of feature cost) | Low |
| CO-002 | Identify and Remove Completely Idle NAT Gateways | Waste Elimination | $32/mo per NAT | Medium |
| CO-003 | Automate Organizational-Wide Recommendation Export | Waste Elimination | 30-60% | Low |
| CO-004 | Rightsize Over-Provisioned EC2 Instances | Rightsizing | 50-75% | Medium |
| CO-005 | Rightsize Amazon RDS Instances | Rightsizing | 50% | High |
| CO-006 | Optimize AWS Lambda Function Memory | Rightsizing | 10-50% | Low |
| CO-007 | Downsize Over-Provisioned EBS Volumes | Rightsizing | 20-60% | Low |
| CO-008 | Rightsize ECS Services on AWS Fargate | Rightsizing | 20-50% | Medium |
| CO-009 | Rightsize Amazon ElastiCache & MemoryDB Nodes | Rightsizing | 50% | Medium |
| CO-010 | Rightsize SageMaker ML Endpoints | Rightsizing | 40-70% | High |
| CO-011 | Action Rightsizing Before Purchasing Savings Plans | Commitment Discounts | 10-30% | Low |
| CO-012 | Modernize to Graviton Instances (ARM) | Architecture Changes | 20%+ | High |
| CO-013 | Upgrade EBS Volumes from gp2 to gp3 | Architecture Changes | 20% | Low |
| CO-014 | Transition to Serverless ElastiCache | Architecture Changes | 30-60% | Medium |
| CO-015 | Optimize EC2 Auto Scaling Group (ASG) Configurations | Scheduling & Auto-Scaling | 20-50% | Medium |
| CO-016 | Scale Down SageMaker Endpoints for Dev/Test | Scheduling & Auto-Scaling | ~70% | Low |
| CO-017 | Optimize DynamoDB Capacity Modes | Pricing Model Optimization | 20-70% | Low |
| CO-018 | Replace Underutilized NAT Gateways with VPC Endpoints | Network & Data Transfer Optimization | $32/mo + Data costs | Medium |
