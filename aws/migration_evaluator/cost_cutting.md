# Cost-Cutting Playbook: AWS Migration Evaluator
> **Companion File:** [migration_evaluator.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/migration_evaluator/migration_evaluator.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Migration Evaluator is a complimentary assessment service that generates data-driven business cases for cloud migration. While the service itself is 100% free, it is arguably the most critical FinOps tool available *before* resources are provisioned on AWS. By leveraging agentless collectors or importing existing inventory (VMware, RVTools), Migration Evaluator analyzes real-world CPU, RAM, disk, and licensing usage. This playbook details how to use Migration Evaluator's insights to preemptively eliminate waste, right-size compute and storage, optimize expensive software licenses, and architect for maximum cost efficiency on day one of migration.

## Strategy Categories

### 1. Waste Elimination

#### 1. Identify and Decommission Zombie Servers
- **What:** Use Migration Evaluator inventory reports to identify on-premises servers with consistently sub-5% CPU and memory utilization over a 30-day period.
- **Why It Saves Money:** Eliminates the cost of migrating, hosting, and licensing servers that perform no active business function. (Pricing math: $0 migrated = 100% savings on those workloads).
- **Implementation Steps:** 
  1. Run Migration Evaluator for a 30-day assessment period.
  2. Filter the resulting TCO report for instances with peak CPU utilization under 5%.
  3. Validate business context with application owners.
  4. Exclude these servers from the AWS migration wave plan and decommission them on-premises.
- **Estimated Savings:** 5-15% of total projected compute costs
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** 30+ days of Migration Evaluator collector data

#### 2. Deprecate Unused Software Licenses
- **What:** Analyze the software inventory discovered by Migration Evaluator to identify expensive enterprise software (e.g., Oracle, SQL Server) installed on non-production or underutilized servers.
- **Why It Saves Money:** Enterprise licenses can cost $10,000+ per core. Removing unused installations prevents paying for cloud-equivalent BYOL support or License Included AWS EC2/RDS pricing.
- **Implementation Steps:**
  1. Extract software inventory lists from the Migration Evaluator export.
  2. Cross-reference expensive software footprints with actual connection/usage metrics.
  3. Uninstall software from idle servers prior to migration, or exclude them from the cloud license model.
- **Estimated Savings:** 10-30% on licensing costs
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Active software inventory collection enabled in Migration Evaluator

#### 3. Retire Legacy Out-of-Support OS Environments
- **What:** Identify servers running end-of-life (EOL) operating systems (e.g., Windows Server 2008/2012) and plan for modernization rather than lift-and-shift.
- **Why It Saves Money:** Running EOL OS on AWS requires expensive Extended Security Updates (ESUs) or forces workloads onto sub-optimal older generation instances.
- **Implementation Steps:**
  1. Filter Migration Evaluator OS summary for EOL versions.
  2. Plan a refactor or platform upgrade (e.g., to Amazon Linux 2023 or Windows Server 2022) as part of the migration.
  3. Avoid paying ESU premiums.
- **Estimated Savings:** 5-10% of OS costs
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Complete OS inventory data

### 2. Rightsizing

#### 4. CPU & RAM Right-Sizing Based on Actual Utilization
- **What:** Transition from 1-to-1 physical core mapping to EC2 instances sized based on actual measured peak CPU and RAM utilization.
- **Why It Saves Money:** On-premises environments are typically provisioned for worst-case peaks (e.g., 64 vCPUs running at 5%). Sizing an EC2 instance to match actual need (e.g., 4 vCPUs) drastically cuts the hourly EC2 rate.
- **Implementation Steps:**
  1. In the Migration Evaluator output, select the "Right-Sized" business case view rather than "Direct Match".
  2. Review the recommended EC2 instance families (e.g., choosing t3/t4g over m5/m6i for burstable workloads).
  3. Map the migration wave directly to the recommended right-sized instance IDs.
- **Estimated Savings:** 30-50%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Minimum 2-4 weeks of time-series utilization data

#### 5. Storage IOPS and Throughput Right-Sizing
- **What:** Use disk IOPS and throughput metrics captured by the collector to select the most cost-effective EBS volume types and sizes.
- **Why It Saves Money:** Defaulting to provisioned IOPS (io2) for every on-prem SAN volume is cost-prohibitive. Most workloads run perfectly on gp3 with baseline IOPS included for free.
- **Implementation Steps:**
  1. Analyze peak disk IOPS in the Migration Evaluator report.
  2. Map volumes with <3,000 IOPS and <125 MB/s throughput to baseline `gp3` volumes.
  3. Only select `io2` for databases exceeding `gp3` maximums.
- **Estimated Savings:** 20-40% on storage costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Disk performance monitoring enabled on the collector

#### 6. Transition to Modern Instance Generations
- **What:** Ensure the migration plan maps on-premises hardware to the absolute newest AWS instance generations (e.g., M7i/M7g instead of M5).
- **Why It Saves Money:** Newer instance generations typically offer 10-20% better price/performance ratios than older generations.
- **Implementation Steps:**
  1. Ensure the Migration Evaluator pricing configuration is updated to the latest available instance types.
  2. Filter recommendations for older generation instances and manually override them to the newest equivalents if applicable.
- **Estimated Savings:** 10-20% on compute
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of current AWS instance families

### 3. Commitment Discounts

#### 7. Compute Savings Plans Modeling
- **What:** Utilize the Migration Evaluator output to model the financial impact of 1-year or 3-year Compute Savings Plans based on the post-right-sizing compute footprint.
- **Why It Saves Money:** Savings Plans reduce on-demand EC2/Fargate/Lambda pricing by up to 72% in exchange for a committed hourly spend.
- **Implementation Steps:**
  1. Finalize the right-sized EC2 footprint in the evaluator.
  2. Extract the baseline steady-state compute requirement.
  3. Work with AWS financial analysts to simulate a 3-year No Upfront Compute Savings Plan for 70-80% of the baseline.
- **Estimated Savings:** 25-50% on compute
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Finalized right-sized instance inventory

#### 8. Dedicated Hosts vs. Shared Tenancy for BYOL
- **What:** Use Migration Evaluator's licensing tools to determine if consolidating BYOL licenses (like Windows Server Datacenter or SQL Server Enterprise) on EC2 Dedicated Hosts is cheaper than shared tenancy.
- **Why It Saves Money:** Dedicated Hosts allow you to license the physical cores of the host rather than virtual cores (vCPUs), which is vastly cheaper for dense, highly virtualized workloads.
- **Implementation Steps:**
  1. Enter current enterprise license entitlements into the Migration Evaluator licensing module.
  2. Generate a comparison report for BYOL on Shared Tenancy vs. BYOL on Dedicated Hosts.
  3. Select the deployment model with the lowest TCO.
- **Estimated Savings:** 30-60% on licensing costs
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Accurate inventory of existing software entitlements (Enterprise Agreements)

#### 9. Database Reserved Instances Planning
- **What:** Plan for RDS Reserved Instances for databases that will be migrated to Amazon RDS based on steady-state requirements mapped out by Migration Evaluator.
- **Why It Saves Money:** Similar to Savings Plans, RDS RIs offer significant discounts for 1- or 3-year commitments on database instance types.
- **Implementation Steps:**
  1. Identify permanent databases in the migration plan.
  2. Ensure they are right-sized based on Migration Evaluator data.
  3. Plan to purchase RDS RIs post-migration once performance is validated.
- **Estimated Savings:** 30-50% on RDS compute
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Right-sized DB instance types determined

### 4. Architecture Changes

#### 10. Identify Workloads for Graviton (ARM) Transition
- **What:** Flag open-source workloads (Linux, Java, Node.js, containerized apps) in the inventory as candidates for AWS Graviton processors.
- **Why It Saves Money:** Graviton instances offer up to 40% better price performance over comparable x86-based instances.
- **Implementation Steps:**
  1. Filter Migration Evaluator inventory for Linux-based open-source application servers.
  2. In the target architecture mapping, switch these from x86 instances (e.g., m6i) to Graviton instances (e.g., m7g).
  3. Validate application compatibility pre-migration.
- **Estimated Savings:** 15-20% on compute
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compatibility testing for ARM64 architecture

#### 11. Migrate Commercial DBs to Open Source (Aurora/RDS)
- **What:** Use the database inventory to target Oracle or SQL Server databases for migration to Amazon Aurora (PostgreSQL/MySQL) via the AWS Schema Conversion Tool.
- **Why It Saves Money:** Eliminates expensive commercial database licensing and support contracts completely.
- **Implementation Steps:**
  1. Extract the list of Oracle/SQL Server databases from the Migration Evaluator report.
  2. Assess complexity and target the easiest/lowest-risk databases for refactoring to Aurora.
  3. Model the target state TCO using Aurora pricing instead of EC2 BYOL pricing.
- **Estimated Savings:** 70-90% on database licensing
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Database schema and application code refactoring capability

#### 12. Consolidate SQL Server Instances
- **What:** Identify multiple underutilized SQL Server instances and plan to consolidate them onto fewer, larger EC2 instances or RDS endpoints.
- **Why It Saves Money:** Licensing multiple small instances requires paying for minimum core counts (usually 4 cores minimum per VM for SQL Server). Consolidating reduces the total licensed core footprint.
- **Implementation Steps:**
  1. Review SQL Server inventory and peak IOPS/CPU metrics.
  2. Group compatible databases that can share a single underlying host.
  3. Map the target architecture to a single larger right-sized instance.
- **Estimated Savings:** 40-60% on SQL licensing
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compatibility analysis for database colocation

### 5. Scheduling & Auto-Scaling

#### 13. Identify Dev/Test Environments for Scheduled Uptime
- **What:** Analyze the time-series utilization data from Migration Evaluator to definitively identify non-production environments that sit idle on nights and weekends.
- **Why It Saves Money:** Stopping EC2 instances when not in use stops compute billing. Running 40 hours a week instead of 168 saves 76% on compute.
- **Implementation Steps:**
  1. Review the daily/weekly utilization heatmaps in the Migration Evaluator output.
  2. Tag servers exhibiting 0% utilization outside of business hours as Dev/Test.
  3. Apply an AWS Instance Scheduler policy to these instances immediately upon migration.
- **Estimated Savings:** 60-70% on non-prod compute
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Utilization heatmaps from the collector

#### 14. Plan Auto-Scaling Groups for Fluctuating Workloads
- **What:** Identify web or application servers that show highly variable, cyclical CPU utilization patterns in the assessment data.
- **Why It Saves Money:** Instead of provisioning a static instance sized for the absolute peak, you can deploy a smaller baseline instance in an Auto Scaling Group (ASG) that only adds compute (and cost) when traffic spikes.
- **Implementation Steps:**
  1. Look for servers with low average CPU but extreme, short-lived spikes in the Migration Evaluator metrics.
  2. Architect the migration target to use a baseline instance size with ASG rules for scale-out.
  3. Remove the oversized static server from the direct migration plan.
- **Estimated Savings:** 30-60% compared to peak-provisioning
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Stateless application architecture

### 6. Pricing Model Optimization

#### 15. Compare License Included (LI) vs. Bring Your Own License (BYOL)
- **What:** Use the assessment to do a strict ROI comparison on migrating current licenses vs. dropping them and paying the AWS hourly rate for Windows/SQL.
- **Why It Saves Money:** Depending on the age of your Enterprise Agreement and right-sized core counts, it may be significantly cheaper to stop paying on-prem maintenance and switch to AWS License Included pricing (or vice versa).
- **Implementation Steps:**
  1. Generate both a BYOL and an LI business case in Migration Evaluator.
  2. Add on-premises software maintenance costs to the BYOL TCO side.
  3. Compare the 3-year TCO and select the optimal licensing path.
- **Estimated Savings:** 15-30% on licensing TCO
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Enterprise Agreement renewal dates and costs

#### 16. Spot Instances for Fault-Tolerant Batch Jobs
- **What:** Identify batch processing servers in the inventory that can tolerate interruption, and map them to target Spot Instances.
- **Why It Saves Money:** Spot instances offer up to 90% off On-Demand prices for interruptible workloads.
- **Implementation Steps:**
  1. Identify non-critical, asynchronous processing servers in the inventory.
  2. Model their cost in the business case using Spot pricing.
  3. Architect the migration using Spot Fleet or Auto Scaling with Spot.
- **Estimated Savings:** 70-90% on targeted compute
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload must be stateless and interrupt-tolerant

#### 17. Optimize Windows Server Licensing with Datacenter Edition
- **What:** If choosing BYOL, calculate if you have enough density to warrant buying Windows Server Datacenter licenses for Dedicated Hosts instead of Standard licenses.
- **Why It Saves Money:** Datacenter edition allows unlimited Windows Server VMs on the licensed physical host, dramatically lowering the cost per VM in high-density environments.
- **Implementation Steps:**
  1. Group Windows servers in the Migration Evaluator plan by target placement.
  2. Calculate the vCPU to physical core ratio.
  3. License the AWS Dedicated Hosts with Windows Server Datacenter Edition.
- **Estimated Savings:** 30-50% on Windows licensing at scale
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** High density of Windows Server VMs

### 7. Network & Data Transfer Optimization

#### 18. Assess Inter-Node Communication for Placement Groups
- **What:** Analyze application dependencies mapping (if utilizing advanced discovery tools alongside Migration Evaluator) to group high-traffic instances together.
- **Why It Saves Money:** Keeping chatty instances in the same Availability Zone (AZ) eliminates cross-AZ data transfer charges ($0.01/GB).
- **Implementation Steps:**
  1. Identify high-bandwidth dependencies (e.g., Web tier to DB tier).
  2. Plan to migrate them into the same subnet/AZ.
  3. Use Cluster Placement Groups for extreme low-latency/high-bandwidth needs.
- **Estimated Savings:** 100% of inter-AZ data transfer fees for grouped instances
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Network dependency mapping data

#### 19. Evaluate Data Egress Patterns for Direct Connect Viability
- **What:** Evaluate the outgoing bandwidth metrics captured for the on-premises environment to model AWS Data Transfer Out costs.
- **Why It Saves Money:** If sustained data egress to on-premises is extremely high (e.g., hybrid cloud architecture), AWS Direct Connect data transfer rates are cheaper than standard internet egress rates.
- **Implementation Steps:**
  1. Sum the outbound network traffic metrics in the assessment data.
  2. Calculate the monthly AWS Egress cost.
  3. Compare against the port hour + reduced egress cost of AWS Direct Connect.
- **Estimated Savings:** 20-50% on data egress costs
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Network throughput metrics from the collector

---

## Cross-Service Synergies

- **AWS Cost Explorer / Compute Optimizer:** Migration Evaluator is for *pre-migration*. Once workloads are running on AWS, Compute Optimizer takes over to provide continuous right-sizing recommendations based on live CloudWatch data.
- **AWS Application Discovery Service:** Works alongside Migration Evaluator to provide deeper application dependency mapping, crucial for planning network optimization and move groups.
- **AWS License Manager:** Once the optimal licensing strategy (BYOL vs LI) is determined via Migration Evaluator, AWS License Manager is used to enforce and track those licenses in the live environment.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
Not applicable for Migration Evaluator (pre-migration). Once migrated, CUR provides exact billing data to validate the business case projections.

### B. CloudWatch Metrics
Not applicable pre-migration. On-premises equivalents (vCenter stats, WMI, perfmon) are ingested by the Migration Evaluator Collector.

### C. AWS Config / Trusted Advisor
Not applicable pre-migration. Used post-migration to ensure the deployed architecture matches the optimized plan.

### D. Company Policies
- Enterprise License Agreements (Microsoft, Oracle, IBM)
- Hardware depreciation schedules (to time the migration for maximum ROI)
- Risk tolerance for Spot instance usage
- Corporate policies on open-source software adoption

### E. IaC (Optional)
Terraform or CloudFormation templates should be configured to deploy the *right-sized* instances determined by Migration Evaluator, rather than lifting and shifting legacy specifications.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "MIGEVAL-RIGHTSIZE-001",
  "category": "Rightsizing",
  "service": "Migration Evaluator",
  "resource_id": "onprem-server-db01",
  "issue_description": "On-premises server db01 has 32 cores but max CPU utilization is 8%. Migration target is over-provisioned.",
  "recommendation": "Deploy as r7i.xlarge (4 vCPUs) instead of r7i.8xlarge.",
  "potential_savings_monthly": 1850.00,
  "effort_to_fix": "Low",
  "status": "Open"
}
```

### Summary Report Table

| Finding ID | Strategy | Resource | Est. Savings (Mo) | Effort | Status |
|---|---|---|---|---|---|
| MIGEVAL-ZOMBIE-001 | Decommission Zombie Servers | 14 Idle Dev VMs | $2,100 | Low | Pending |
| MIGEVAL-RIGHTSIZE-002 | CPU Right-Sizing | Web Farm Cluster | $4,300 | Low | Planned |
| MIGEVAL-LICENSE-003 | Optimize SQL Licensing | DB Server 05 | $5,800 | Medium | Under Review |
| MIGEVAL-SCHEDULE-004 | Scheduled Uptime | UAT Environment | $1,200 | Low | Approved |
