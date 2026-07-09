# Cost-Cutting Playbook: Amazon EC2 (Elastic Compute Cloud)

> **Companion File:** [ec2.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/ec2/ec2.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon EC2 is the single largest contributor to enterprise AWS bills. Because EC2 spend spans compute instance sizing, CPU architecture selection, OS licensing, storage attachments, networking egress, and commitment model coverage, optimizing EC2 requires a multi-layered approach.

This playbook provides **25 actionable strategies** organized across eight operational categories. Implementing these strategies across an enterprise AWS footprint typically yields **30–65% aggregate savings on total compute spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Upgrade `gp2` EBS volumes to `gp3` on all EC2 instances:** Instant 20% storage cost reduction with no downtime.
2. **Release Idle Elastic IPs & Stopped Instance IPs:** Reclaims $0.005/hour ($3.65/month) per unused IPv4 address immediately.
3. **Migrate Stateless/Batch Workloads to Graviton4 (`c8g`/`m8g`/`r8g`):** Instant 20% compute discount and up to 30% performance improvement.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Terminate Unused or Idle EC2 Instances
- **What:** Identify and terminate EC2 instances with average CPU utilization < 5%, network throughput < 5 KB/s, and low disk I/O over a 14-day evaluation window.
- **Why It Saves Money:** An idle `m6i.xlarge` instance ($0.192/hr) left running costs $140.16/month. Terminating 20 idle instances reclaims $2,803/month immediately.
- **Implementation Steps:**
  1. Pull Compute Optimizer recommendations or run AWS CLI: `aws compute-optimizer get-ec2-instance-recommendations`.
  2. Filter for findings labeled `Idle` or `Overprovisioned`.
  3. Verify with workload owners via tags; take an EBS snapshot if backup is needed.
  4. Terminate instances via CLI/Terraform and verify attached EBS volumes are cleaned up.
- **Estimated Savings:** 100% of instance cost (typically 5–15% of total EC2 spend).
- **Risk Level:** Low to Medium (requires application owner sign-off).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch agent installed or basic EC2 metrics enabled; tagging policies active.

#### 2. Release Unattached and Secondary Elastic IPs (Public IPv4)
- **What:** Locate Elastic IP addresses (EIPs) that are either unattached or associated with stopped instances, or secondary IPs attached to single running instances.
- **Why It Saves Money:** AWS charges $0.005/hour ($3.65/month) for every public IPv4 address, including unattached or stopped EIPs. Releasing 100 idle EIPs saves $365/month.
- **Implementation Steps:**
  1. List unattached IPs: `aws ec2 describe-addresses --query "Addresses[?AssociationId==null]"`.
  2. Check associated instance states for attached EIPs to find stopped instances.
  3. Release unneeded EIPs: `aws ec2 release-address --allocation-id eipalloc-xxx`.
- **Estimated Savings:** $3.65 per IP per month (100% of unattached EIP spend).
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Verification that IP is not whitelisted by external partners.

#### 3. Decommission Stale AMIs and Associated EBS Snapshots
- **What:** Clean up custom AMIs created for historical deployments or golden images that are no longer used by any active launch templates or Auto Scaling Groups.
- **Why It Saves Money:** AMIs store underlying EBS snapshots billed at $0.05/GB-month. A 500 GB AMI snapshot left indefinitely costs $25/month. 50 stale AMIs cost $1,250/month.
- **Implementation Steps:**
  1. Deregister AMI: `aws ec2 deregister-image --image-id ami-xxx`.
  2. Delete associated EBS snapshots: `aws ec2 delete-snapshot --snapshot-id snap-xxx`.
  3. Automate image lifecycles using EC2 Image Builder or AWS Systems Manager.
- **Estimated Savings:** 100% of unused AMI storage costs ($0.05/GB-month).
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Audit launch templates to ensure no active ASG references the target AMI.

#### 4. Audit & Remove Unused EC2 Security Groups and ENIs
- **What:** Identify and delete orphaned Elastic Network Interfaces (ENIs) and unattached Security Groups.
- **Why It Saves Money:** While security groups are free, orphaned ENIs with assigned public IPv4 addresses generate $0.005/hr fees ($3.65/mo).
- **Implementation Steps:**
  1. Locate available ENIs: `aws ec2 describe-network-interfaces --filters Name=status,Values=available`.
  2. Delete detached ENIs and release associated public IPs.
- **Estimated Savings:** $3.65/month per public ENI.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

---

### 2. Rightsizing

#### 5. Migrate x86 Instances to AWS Graviton4 (`c8g`, `m8g`, `r8g`)
- **What:** Transition workloads running on x86 (`c6i`, `m6i`, `r6i` or older `c5`, `m5`) to Graviton4 ARM-based instance families.
- **Why It Saves Money:** Graviton4 instances are priced ~20% lower than equivalent x86 instances while delivering up to 30% better price-performance. For example, `c6i.large` ($0.085/hr) vs `c8g.large` ($0.068/hr) saves $124.10/year per instance.
- **Implementation Steps:**
  1. Identify Linux workloads running multi-platform code (Python, Node.js, Java, Go, Docker).
  2. Rebuild container images for `arm64` architecture or test native Linux ARM binaries in staging.
  3. Update Terraform/CloudFormation instance type definitions to `c8g`/`m8g`/`r8g`.
- **Estimated Savings:** 20% direct compute savings + up to 30% performance boost.
- **Risk Level:** Low (for interpreted/managed runtimes) to Medium (for C/C++ compiled binaries).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Code compatibility testing for ARM64 architecture.

#### 6. Downgrade Over-Provisioned Compute Instances
- **What:** Downsize instances where peak CPU utilization remains below 30% and memory utilization remains below 40% during peak hours (e.g. `m6i.2xlarge` -> `m6i.xlarge`).
- **Why It Saves Money:** Cutting instance size by half reduces compute billing by exactly 50%. Moving `m6i.2xlarge` ($0.384/hr = $280.32/mo) down to `m6i.xlarge` ($0.192/hr = $140.16/mo) saves $1,682/year per instance.
- **Implementation Steps:**
  1. Analyze AWS Compute Optimizer or CloudWatch memory/CPU metrics over 30 days.
  2. Schedule a maintenance window for instance resize.
  3. Stop instance, change instance type via `aws ec2 modify-instance-attribute --instance-id i-xxx --instance-type "{\"Value\": \"m6i.xlarge\"}"`, and restart.
- **Estimated Savings:** 50% per instance step-down.
- **Risk Level:** Low (easily revertible if performance bottleneck occurs).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch Agent enabled for RAM monitoring.

#### 7. Transition Fixed-Size Instances to Burstable T4g/T3 for Sporadic Workloads
- **What:** Move non-production, administrative, or low-duty web servers from standard `m6i.large` instances to burstable `t4g.medium` or `t3.medium` instances.
- **Why It Saves Money:** `m6i.large` costs $0.096/hr ($70.08/mo), whereas `t4g.medium` costs $0.0336/hr ($24.53/mo) — a **65% cost reduction**.
- **Implementation Steps:**
  1. Identify instances with baseline CPU usage < 15% and periodic burst patterns.
  2. Set CPU credit mode to `standard` (or `unlimited` with budget alert thresholds).
  3. Change instance type to `t4g.medium` or `t3.medium`.
- **Estimated Savings:** 50–65% reduction on light-workload instances.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Monitoring CPU credit balance via CloudWatch metric `CPUCreditBalance`.

#### 8. Right-Size Red Hat Enterprise Linux (RHEL) vCPU Allocations
- **What:** Optimize instance vCPU count on RHEL instances following the mid-2024 pricing model change.
- **Why It Saves Money:** RHEL pricing changed to **$0.03 per vCPU-hour**. An 8-vCPU RHEL instance pays $0.24/hr ($175.20/mo) in OS licensing alone! Rightsizing from 8 vCPUs to 4 vCPUs slashes RHEL license fees by 50% ($87.60/mo saved per instance).
- **Implementation Steps:**
  1. Audit RHEL instances using `aws ec2 describe-instances`.
  2. Identify low-CPU RHEL servers and downsize instance family size.
  3. Alternatively, consider migrating non-enterprise workloads from RHEL to Amazon Linux 2023 or Ubuntu to eliminate OS licensing fees entirely.
- **Estimated Savings:** $0.03/vCPU-hour licensing fee reduction (50% license savings per step-down).
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps & Procurement
- **Prerequisites:** Verification of RHEL-specific enterprise application support requirements.

---

### 3. Commitment Discounts (RIs / Savings Plans)

#### 9. Commit to Compute Savings Plans for Baseline Spend
- **What:** Purchase 1-year or 3-year Compute Savings Plans (CSP) covering steady-state compute across EC2, Fargate, and Lambda.
- **Why It Saves Money:** Provides discounts up to **66% (3-year No Upfront)** or **72% (3-year All Upfront)** off On-Demand rates without locking into specific regions, instance families, OS, or tenancy.
- **Implementation Steps:**
  1. Analyze Cost Explorer Savings Plans recommendations based on historical 30-day baseline hourly spend.
  2. Set commitment level to 70–80% of minimum hourly spend (leaving headroom for Spot/burst).
  3. Purchase via Cost Explorer console or API: `aws savingsplans create-savings-plan`.
- **Estimated Savings:** 20–66% off On-Demand rates.
- **Risk Level:** Low (Compute SPs automatically apply across all regions and families).
- **Implementation Scope:** FinOps Team / Procurement
- **Prerequisites:** Multi-month baseline spend stability analysis.

#### 10. Leverage EC2 Instance Savings Plans for High-Volume Static Workloads
- **What:** Commit to EC2 Instance Savings Plans for large, static instance pools bound to a specific region and instance family (e.g. `m6i` in `us-east-1`).
- **Why It Saves Money:** Yields higher discounts (up to 72%) than Compute Savings Plans (up to 66%) for the same commitment duration.
- **Implementation Steps:**
  1. Identify database servers, analytics clusters, or legacy apps committed to a single region and family.
  2. Purchase EC2 Instance Savings Plans for that specific family/region.
- **Estimated Savings:** 40–72% discount off On-Demand.
- **Risk Level:** Medium (locks into instance family and region for 1 or 3 years).
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Certainty that instance family will not change during commitment term.

#### 11. Utilize RI Marketplace to Sell or Purchase Short-Term RIs
- **What:** Buy discounted short-term Standard Reserved Instances from the AWS RI Marketplace, or list unneeded Standard RIs for sale.
- **Why It Saves Money:** Allows acquiring commitments with < 12 months remaining at steep discounts, or liquidating unused commitments to recover sunk costs.
- **Implementation Steps:**
  1. Search EC2 RI Marketplace for third-party listed RIs matching target instance types.
  2. List unneeded Standard RIs for sale via EC2 console.
- **Estimated Savings:** 10–40% on remaining commitment value.
- **Risk Level:** Medium.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** US bank account associated with AWS billing for selling RIs.

---

### 4. Architecture Changes

#### 12. Migrate Stateless Web/API Tiers to Amazon ECS / EKS on Fargate
- **What:** Refactor monolithic or microservice EC2 web applications into containerized tasks on ECS or EKS.
- **Why It Saves Money:** Eliminates VM idle overhead and bin-packing inefficiency. Pay only for exact vCPU/RAM requested per task, with Fargate Spot offering 70% discounts.
- **Implementation Steps:**
  1. Containerize applications using Docker.
  2. Deploy on ECS/EKS using Fargate launch type with Graviton (`ARM64`) support.
- **Estimated Savings:** 30–60% reduction in compute overhead.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application containerization readiness.

#### 13. Replace Low-Utilization EC2 Endpoints with AWS Lambda
- **What:** Convert low-traffic API backends or periodic CRON jobs running on dedicated EC2 instances to serverless AWS Lambda functions.
- **Why It Saves Money:** An `m6i.large` instance running 24/7 costs $70.08/mo even if it processes 1,000 requests/day. Lambda processing 1,000 requests/day costs < $0.05/mo.
- **Implementation Steps:**
  1. Refactor web server handlers into serverless functions (e.g. using Serverless Framework, AWS SAM).
  2. Expose via Amazon API Gateway or HTTP Endpoints.
  3. Terminate EC2 instance.
- **Estimated Savings:** 90–99% for low-traffic endpoints.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Code refactoring for stateless execution.

#### 14. Transition from Scale-Up (Vertical) to Scale-Out (Horizontal) Sizing
- **What:** Replace single large `m6i.4xlarge` instances with Auto Scaling Groups of smaller `m8g.large` instances behind an Application Load Balancer.
- **Why It Saves Money:** Prevents paying for high peak capacity 24/7. Smaller instances scale in when demand drops, matching capacity precisely to demand.
- **Implementation Steps:**
  1. Ensure application state is stored in external databases/caches (RDS/ElastiCache).
  2. Create launch templates with smaller Graviton instances.
  3. Configure Auto Scaling Groups with dynamic target tracking.
- **Estimated Savings:** 25–45% off static high-capacity provisioning.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Stateless application tier design.

---

### 5. Storage & Data Tiering

#### 15. Convert All Attached EBS Volumes from `gp2` to `gp3`
- **What:** Modify existing General Purpose SSD volumes from `gp2` to `gp3` across all EC2 instances.
- **Why It Saves Money:** `gp3` costs $0.08/GB-month vs `gp2` at $0.10/GB-month — an immediate **20% direct storage price reduction** with superior baseline performance (3,000 IOPS and 125 MB/s included free).
- **Implementation Steps:**
  1. Run AWS CLI: `aws ec2 modify-volume --volume-id vol-xxx --volume-type gp3`.
  2. Operation executes online with ZERO downtime or performance degradation.
  3. Update IaC templates (Terraform/CloudFormation) default volume types to `gp3`.
- **Estimated Savings:** 20% direct EBS storage savings.
- **Risk Level:** Zero (seamless online modification).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 16. Leverage Instance Store (NVMe) for High-IOPS Temporary Storage
- **What:** Utilize local NVMe instance store volumes (included free with `c6id`/`m6gd` instance types) for temporary files, swap space, and caches instead of paying for high-cost `io2` EBS volumes.
- **Why It Saves Money:** Provisioned IOPS on `io2` costs $0.065/IOPS-month. A 10,000 IOPS `io2` volume costs $650/month. Instance store NVMe provides up to 100,000+ IOPS at $0 additional cost.
- **Implementation Steps:**
  1. Choose instance types with `d` suffix (e.g. `c7gd.large`).
  2. Mount NVMe drive on instance startup script for ephemeral workspace.
- **Estimated Savings:** 100% of provisioned IOPS costs for temporary data.
- **Risk Level:** Medium (data is lost on instance termination/stop).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ephemeral workload suitability.

---

### 6. Scheduling & Auto-Scaling

#### 17. Implement Instance Scheduler for Non-Production Environments
- **What:** Automatically stop non-production EC2 instances (dev, test, QA, staging) during off-hours (nights and weekends: 7 PM to 7 AM Monday-Friday, off all weekend).
- **Why It Saves Money:** Running non-prod instances only 50 hours/week instead of 168 hours/week reduces compute spend by **70%** (118 idle hours saved per week).
- **Implementation Steps:**
  1. Deploy AWS Instance Scheduler via CloudFormation stack.
  2. Tag dev/test instances with `Schedule=office-hours`.
  3. Enforce tag-based auto-shutdown via AWS Systems Manager.
- **Estimated Savings:** 65–70% on non-production compute bills.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Tagging compliance across dev/test accounts.

#### 18. Implement Target Tracking & Predictive Auto-Scaling Policies
- **What:** Configure ASG scaling policies to use Target Tracking (e.g. maintain 60% CPU) combined with Predictive Scaling for forecasted traffic spikes.
- **Why It Saves Money:** Prevents static over-provisioning and eliminates lag associated with reactive step scaling policies.
- **Implementation Steps:**
  1. Set ASG scaling policy to `TargetTrackingScaling`.
  2. Enable `PredictiveScaling` in ASG settings to pre-warm instances prior to known daily traffic spikes.
- **Estimated Savings:** 15–30% capacity optimization.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** ASG configured with launch template.

---

### 7. Pricing Model Optimization

#### 19. Deploy EC2 Spot Instances for Fault-Tolerant Workloads
- **What:** Utilize Spot Instances for stateless web tiers, batch processing, CI/CD runners, and big data workloads (EMR).
- **Why It Saves Money:** Spot instances offer discounts up to **90% off On-Demand rates** (average 70–80% savings).
- **Implementation Steps:**
  1. Configure Auto Scaling Groups with `Attribute-based Instance Type Selection` using flexible instance types (e.g. allow 4+ instance types in allocation strategy).
  2. Use `capacity-optimized` allocation strategy to minimize interruption risk.
  3. Integrate EC2 Instance Rebalance Recommendations and termination notices.
- **Estimated Savings:** 70–90% off On-Demand compute rates.
- **Risk Level:** Medium (susceptible to 2-minute interruption notice).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workloads must handle sudden instance terminations gracefully.

#### 20. Optimize Dedicated Hosts & BYOL Licensing Surcharges
- **What:** Consolidate Bring-Your-Own-License (BYOL) software (Oracle, Windows Server, SQL Server) onto tightly packed EC2 Dedicated Hosts.
- **Why It Saves Money:** Eliminates per-instance OS/database license surcharges by licensing full underlying physical sockets/cores.
- **Implementation Steps:**
  1. Allocate Dedicated Host: `aws ec2 allocate-hosts --instance-type m6i.metal`.
  2. Maximize host socket density to amortize the $2.00/hour dedicated host regional fee.
- **Estimated Savings:** 30–60% on software licensing.
- **Risk Level:** Medium.
- **Implementation Scope:** FinOps & Procurement
- **Prerequisites:** Compliance review of software vendor BYOL licensing rules.

---

### 8. Network & Data Transfer Optimization

#### 21. Route Intra-VPC Traffic via Private IPs (Eliminate Public IP Egress)
- **What:** Ensure inter-instance traffic within the same VPC uses private IP addresses rather than public IPs or Elastic IPs.
- **Why It Saves Money:** Traffic routed via public IPs passes out to the internet border and back, incurring inter-AZ ($0.02/GB) or internet egress charges ($0.09/GB) instead of free intra-AZ private routing ($0.00/GB).
- **Implementation Steps:**
  1. Configure internal DNS names to resolve to private IP addresses.
  2. Update application configuration endpoints to use private DNS hostnames.
- **Estimated Savings:** 100% of accidental egress charges for internal traffic.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Internal Route 53 private hosted zones.

#### 22. Deploy Gateway VPC Endpoints for S3 and DynamoDB
- **What:** Create free Gateway VPC Endpoints for S3 and DynamoDB in all VPC route tables.
- **Why It Saves Money:** Routes S3/DynamoDB traffic directly over the internal AWS network for **free ($0.00/GB)**, completely bypassing NAT Gateways ($0.045/GB processing tax) and internet egress fees ($0.09/GB).
- **Implementation Steps:**
  1. Run AWS CLI: `aws ec2 create-vpc-endpoint --vpc-id vpc-xxx --service-name com.amazonaws.us-east-1.s3 --route-table-ids rtb-xxx`.
  2. Gateway endpoints carry $0.00 hourly fee and $0.00 processing fee.
- **Estimated Savings:** 100% of NAT Gateway processing fees for S3/DynamoDB traffic.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 23. Co-Locate Inter-Communicating EC2 Instances in the Same Availability Zone
- **What:** Group chatty microservices or cluster nodes in the same Availability Zone using Placement Groups or AZ-specific subnet placement.
- **Why It Saves Money:** Intra-AZ private traffic is **100% free ($0.00/GB)**, while inter-AZ traffic costs $0.01/GB in each direction ($0.02/GB roundtrip). Passing 50 TB/month across AZs costs $1,000/month.
- **Implementation Steps:**
  1. Create Cluster Placement Groups for high-throughput node clusters.
  2. Align backend microservices into the same AZ subnet as their primary database master.
- **Estimated Savings:** $0.02 per GB transferred inter-AZ.
- **Risk Level:** Medium (reduces multi-AZ availability for targeted sub-components).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Architectural assessment of HA requirements vs data transfer spend.

#### 24. Utilize Partition Placement Groups for Large Distributed Systems
- **What:** Deploy HDFS, Cassandra, or Kafka nodes into Partition Placement Groups to keep nodes isolated across logical racks while managing network boundaries.
- **Why It Saves Money:** Reduces cross-rack network congestion and optimizes inter-node replication traffic cost.
- **Implementation Steps:**
  1. Launch instances into partition placement groups via launch templates.
- **Estimated Savings:** 10–20% on inter-node data transfer overhead.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Distributed system architecture.

#### 25. Enforce EBS Snapshot Lifecycle Policies via AWS Data Lifecycle Manager (DLM)
- **What:** Configure DLM policies to automatically delete old EC2 volume snapshots after retention windows expire (e.g. delete daily snapshots after 14 days).
- **Why It Saves Money:** Prevents snapshot accumulation. Storing 10 TB of unmanaged legacy snapshots costs $500/month indefinitely.
- **Implementation Steps:**
  1. Create DLM lifecycle policy: `aws dlm create-lifecycle-policy`.
  2. Target instances via tags and set retention schedules.
- **Estimated Savings:** 40–80% snapshot storage cost reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** EC2 instance tagging strategy.

---

## Cross-Service Synergies

EC2 interacts directly with almost every core AWS service. Optimizing EC2 yields compound savings across the broader architecture:

```
[ EC2 Instances ] ──(Attach)──> [ EBS Storage ] (Migrate gp2 -> gp3 saves 20%)
        │
        ├──(Egress Traffic)──> [ NAT Gateway ] (Add Free VPC Endpoint saves $0.045/GB)
        │
        ├──(Replication)─────> [ Inter-AZ Egress ] (Co-locate in same AZ saves $0.02/GB)
        │
        ├──(Refactor)────────> [ ECS / Fargate ] (Containerize + Fargate Spot saves 70%)
        │
        └──(Offload)─────────> [ AWS Lambda ] (Serverless migration saves up to 90%)
```

---

## Required Input Data for Real-World Analysis

To execute automated EC2 cost-cutting discovery, ingest the following data sources:

### A. AWS Cost & Usage Report (CUR 2.0)
Require CUR 2.0 ingested into Amazon Athena with the following columns:
- `line_item_usage_type`: Filter for `BoxUsage`, `HeavyUsage`, `EBS`, `DataTransfer-Out`.
- `line_item_resource_id`: EC2 Instance ID (`i-xxxx`).
- `line_item_unblended_cost`: Net unblended dollar spend per line item.
- `pricing_term`: `OnDemand`, `Reserved`, `Spot`, `SavingsPlan`.
- `product_instance_type`: e.g. `m6i.2xlarge`, `c5.large`.
- `product_operating_system`: `Linux`, `RHEL`, `Windows`.
- `product_tenancy`: `Shared`, `Dedicated`, `Host`.
- `reservation_reservation_a_r_n`: Matching RI coverage.
- `savings_plan_savings_plan_a_r_n`: Matching SP coverage.

### B. CloudWatch Metrics
- `AWS/EC2` Namespace: `CPUUtilization`, `NetworkIn`, `NetworkOut`, `DiskReadOps`, `DiskWriteOps`, `StatusCheckFailed_Instance`.
- Granularity: 1-hour resolution over 30-day lookback period.

### C. AWS Services API & Diagnostic Checks
- **AWS Compute Optimizer:** `get-ec2-instance-recommendations` API outputs.
- **AWS Config Rules:** `ec2-instance-managed-by-systems-manager`, `ec2-volume-inuse-check`.
- **Trusted Advisor:** Idle EC2 Instances check, Unattached Elastic IP check.

### D. Organizational Context & Governance
- Application Tier SLA Classification (Prod vs Non-Prod).
- Maintenance Window Schedule for reboot/resize operations.
- Security & Compliance rules regarding ARM architecture compatibility.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "EC2-RS-001",
  "service": "EC2",
  "category": "Rightsizing",
  "resource_id": "i-0a1b2c3d4e5f67890",
  "resource_name": "legacy-payment-processor",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "instance_type": "c5.2xlarge",
    "vcpu": 8,
    "memory_gib": 16,
    "architecture": "x86_64",
    "operating_system": "Linux",
    "monthly_cost_usd": 248.20
  },
  "recommended_config": {
    "instance_type": "c8g.xlarge",
    "vcpu": 4,
    "memory_gib": 8,
    "architecture": "arm64",
    "operating_system": "Linux",
    "projected_monthly_cost_usd": 99.28
  },
  "financial_impact": {
    "monthly_savings_usd": 148.92,
    "annual_savings_usd": 1787.04,
    "savings_percentage": 60.0
  },
  "risk_assessment": {
    "risk_level": "Medium",
    "reason": "Requires ARM64 binary compilation and vCPU reduction validation."
  },
  "implementation": {
    "scope": "Engineer/DevOps",
    "effort_estimate": "2-4 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Waste Elimination** | 14 | $4,500.00 | $4,120.00 | 91.5% | Low |
| **Rightsizing (Graviton/Sizing)** | 42 | $18,400.00 | $7,360.00 | 40.0% | Medium |
| **Commitment Discounts** | 1 | $32,000.00 | $11,200.00 | 35.0% | Low |
| **Architecture (Containers/Serverless)**| 8 | $6,200.00 | $3,410.00 | 55.0% | Medium |
| **Storage (gp2 -> gp3)** | 65 | $3,800.00 | $760.00 | 20.0% | Low |
| **Scheduling (Dev/Test Off-Hours)** | 28 | $5,600.00 | $3,752.00 | 67.0% | Low |
| **Pricing Model (Spot Fleets)** | 5 | $8,900.00 | $6,675.00 | 75.0% | Medium |
| **Network (VPC Endpoints & IPs)** | 12 | $2,100.00 | $1,680.00 | 80.0% | Low |
| **Total** | **175** | **$81,500.00** | **$38,957.00** | **47.8%** | -- |
