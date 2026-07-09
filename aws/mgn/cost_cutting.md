# Cost-Cutting Playbook: AWS Application Migration Service (MGN)
> **Companion File:** [mgn.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/mgn/mgn.md)
> **Last Updated:** July 2026

---

## Executive Summary
AWS Application Migration Service (MGN) is a highly efficient lift-and-shift tool, but it can accumulate unexpected costs if migrations stall or staging environments are misconfigured. MGN offers a 90-day free replication period per server, after which a $0.042/hour per-server fee applies. Furthermore, the underlying staging infrastructure (EC2 replication instances, EBS volumes) and data transfer can drive up the monthly bill. This playbook details 20 actionable strategies to optimize MGN migrations, avoid the "replication stagnation" trap, and rightsize both the temporary staging environments and the final target AWS workloads.

## Strategy Categories

### 1. Waste Elimination

#### 1. Disconnect and Terminate Post-Cutover Servers
- **What:** Immediately disconnect source servers in the MGN console once the final cutover is successful and validated.
- **Why It Saves Money:** Disconnecting a server automatically terminates its associated staging EC2 instances and staging EBS volumes. Leaving them running incurs dual-run costs.
- **Implementation Steps:** 
  1. Navigate to the MGN Console.
  2. Select servers that have successfully cut over.
  3. Click "Actions" -> "Disconnect from service".
- **Estimated Savings:** 100% of ongoing staging costs for that server.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Successful production cutover and sign-off.

#### 2. Delete Stagnant/Abandoned Replication Servers
- **What:** Identify and delete servers in MGN that have been replicating for weeks but whose migration projects have been canceled or indefinitely paused.
- **Why It Saves Money:** Avoids the $30.24/month per-server MGN fee (post-90 days) and ongoing staging EBS/EC2 costs for workloads that will never cut over.
- **Implementation Steps:**
  1. Review MGN for servers with no recent test launches.
  2. Confirm migration status with project owners.
  3. Disconnect abandoned servers and delete them from the MGN console.
- **Estimated Savings:** ~$50-$100 per month per abandoned server.
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Clear communication with migration stakeholders.

#### 3. Terminate Test Instances Promptly Post-Validation
- **What:** Destroy test EC2 instances launched by MGN as soon as user acceptance testing (UAT) is complete.
- **Why It Saves Money:** Test instances are fully provisioned target EC2 instances. Leaving them running for days after testing concludes incurs unnecessary on-demand compute and storage charges.
- **Implementation Steps:**
  1. Go to the MGN Console.
  2. Select servers with running test instances.
  3. Choose "Test and Cutover" -> "Revert to ready for testing" (this terminates the test instances).
- **Estimated Savings:** Avoids hundreds of dollars per wave in unnecessary compute.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** UAT completion.

#### 4. Clean Up Orphaned EBS Snapshots from Test Cycles
- **What:** Delete lingering EBS snapshots that were created during repeated MGN test launches but not automatically cleaned up.
- **Why It Saves Money:** EBS snapshots cost $0.05/GB-month. Repeated testing can leave behind a trail of snapshots.
- **Implementation Steps:**
  1. Go to EC2 Console -> Snapshots.
  2. Filter by tags (MGN often tags resources automatically) or descriptions indicating test origins.
  3. Delete snapshots older than the most recent successful test.
- **Estimated Savings:** 5-15% of total EBS snapshot costs.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** None.

#### 5. Uninstall MGN Agents from Decommissioned Source Machines
- **What:** Remove the AWS Replication Agent from source servers that have been decommissioned or ruled out of the migration scope.
- **Why It Saves Money:** Prevents the agent from automatically reconnecting and spinning up staging resources if the on-premise server is accidentally powered on.
- **Implementation Steps:**
  1. Run the uninstallation script (`aws-replication-installer-init`) on the source OS.
  2. Verify the server does not reappear in the MGN console.
- **Estimated Savings:** Variable (prevents accidental staging spin-ups).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Server access.

### 2. Rightsizing

#### 6. Optimize Staging Area EC2 Instance Types
- **What:** Configure the MGN Replication Configuration template to use the most cost-effective EC2 instance types (e.g., `t3.small`) for the staging area, rather than larger default instances.
- **Why It Saves Money:** Staging instances simply write replicated blocks to EBS. They rarely require heavy compute. A `t3.small` ($0.0208/hr) is 60% cheaper than a `t3.large`.
- **Implementation Steps:**
  1. Go to MGN Console -> Settings -> Replication template.
  2. Edit "Staging area instance type".
  3. Select cost-effective shared-tenancy burstable instances (e.g., `t3.small`).
- **Estimated Savings:** 40-60% of staging compute costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ensure network bandwidth of the smaller instance can handle the replication change rate.

#### 7. Utilize Cost-Effective Staging EBS Volumes
- **What:** Change the default staging EBS volume type to `st1` (Throughput Optimized HDD) or `sc1` for large, low-churn disks, rather than defaulting all staging disks to `gp3`.
- **Why It Saves Money:** `st1` costs ~$0.045/GB-month, roughly half the cost of `gp3` ($0.08/GB-month). This adds up massively for large file servers.
- **Implementation Steps:**
  1. Modify the Replication Configuration template in MGN.
  2. Adjust the EBS volume type rules for staging volumes based on size and expected IOPS.
- **Estimated Savings:** Up to 40% on staging storage costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Source server write churn must not exceed `st1` limits.

#### 8. Rightsize Target EC2 Instances via Launch Templates
- **What:** Avoid migrating a 16-core on-premise VM to a 16-core EC2 instance if the on-premise VM only utilized 10% CPU. Define right-sized instance types in the MGN EC2 Launch Template prior to cutover.
- **Why It Saves Money:** Prevents bringing on-premise over-provisioning waste into the cloud environment.
- **Implementation Steps:**
  1. Analyze historical performance data (e.g., via AWS Migration Evaluator or third-party tools).
  2. In MGN, update the EC2 Launch Template for the server.
  3. Select the appropriate right-sized instance type (e.g., `c6a.xlarge` instead of `c6a.4xlarge`).
- **Estimated Savings:** 30-70% on final run costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Performance monitoring data from the source environment.

#### 9. Rightsize Target EBS Volumes
- **What:** While MGN replicates blocks 1:1, you can sometimes shrink empty volumes post-migration, or use `gp3` instead of `io2` for the final target disks if high IOPS aren't strictly required.
- **Why It Saves Money:** Overprovisioned block storage is a top cloud waste vector. `gp3` is up to 20% cheaper than `gp2`.
- **Implementation Steps:**
  1. Review target EBS volume types in the EC2 Launch Template.
  2. Ensure `gp3` is selected as the default.
  3. Manually shrink partitions and volumes post-cutover if the source disks were highly underutilized (requires OS-level work).
- **Estimated Savings:** 20% on target storage costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Knowledge of source disk utilization.

#### 10. Exclude Non-Essential Disks from Block Replication
- **What:** Configure the MGN agent to exclude temporary disks, pagefile/swap drives, or backup repositories attached to the source server.
- **Why It Saves Money:** Reduces the size of staging EBS volumes, reduces replication data transfer, and reduces the size of the final target EBS volumes.
- **Implementation Steps:**
  1. During MGN agent installation, or via the MGN console, select the specific source disks to replicate.
  2. Exclude disks containing ephemeral or recreatable data.
- **Estimated Savings:** 10-30% on storage and data transfer costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Clear understanding of source application disk layouts.

### 3. Commitment Discounts

#### 11. Cover Staging Area Compute with Compute Savings Plans (Long Migrations)
- **What:** If a large-scale migration will take 1-2 years, purchase a partial Compute Savings Plan (CSP) to cover the continuous fleet of staging EC2 instances.
- **Why It Saves Money:** Staging instances run 24/7 during replication. A 1-year No Upfront CSP can discount these instances by 20-30%.
- **Implementation Steps:**
  1. Analyze average steady-state spend on MGN staging EC2 instances.
  2. Purchase a conservative Compute Savings Plan to cover a portion of this baseline.
- **Estimated Savings:** 20-30% on staging compute.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Commitment to a long-term migration timeline.

#### 12. Purchase EC2 Savings Plans for Target Workloads Post-Cutover
- **What:** Shortly after the final cutover and a brief stabilization/rightsizing period, cover the new target EC2 instances with Savings Plans.
- **Why It Saves Money:** On-Demand pricing is highly expensive. Securing 1- or 3-year commitments yields up to 72% savings.
- **Implementation Steps:**
  1. Wait 2-4 weeks post-migration to ensure stability and final rightsizing.
  2. Use AWS Cost Explorer Savings Plan Recommendations.
  3. Procure EC2 Instance Savings Plans or Compute Savings Plans.
- **Estimated Savings:** 30-72% on post-migration run costs.
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Finalized target architectures.

### 4. Architecture Changes

#### 13. Replatform Databases to Amazon RDS (Avoid EC2 Lift-and-Shift)
- **What:** Instead of using MGN to lift-and-shift SQL Server or Oracle VMs onto EC2, use AWS Database Migration Service (DMS) to migrate them directly to Amazon RDS or Aurora.
- **Why It Saves Money:** RDS reduces DBA management overhead, optimizes licensing (e.g., License Included), and scales more efficiently than self-managed DBs on EC2.
- **Implementation Steps:**
  1. Identify database servers in the migration wave.
  2. Exclude them from MGN block replication.
  3. Set up AWS DMS and target RDS instances.
- **Estimated Savings:** 15-40% TCO reduction (factoring operational overhead).
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | Database Administrators
- **Prerequisites:** Application compatibility with RDS.

#### 14. Migrate File Data via AWS DataSync (Instead of Block Replication)
- **What:** Do not use MGN to migrate massive (multi-TB) file servers. Instead, migrate the compute node, and use AWS DataSync to copy the raw files directly into Amazon FSx or Amazon EFS.
- **Why It Saves Money:** MGN requires staging EBS volumes equal to the size of the source data. A 50TB file server requires 50TB of staging EBS ($4,000/mo) *plus* the target EBS. DataSync directly to FSx avoids staging storage entirely.
- **Implementation Steps:**
  1. Exclude the massive data drives from the MGN agent.
  2. Provision Amazon FSx/EFS.
  3. Run AWS DataSync tasks to transfer the data.
- **Estimated Savings:** 50% on storage costs during the migration window.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Appropriate network connectivity for DataSync.

### 5. Scheduling & Auto-Scaling

#### 15. Stop/Pause Replication for Paused Migration Waves
- **What:** If a specific migration wave is delayed by months due to business freeze, stop the MGN replication agent or pause replication.
- **Why It Saves Money:** Halting replication stops the continuous data churn, allowing you to downsize or terminate staging resources temporarily, and pauses the march toward the 90-day free window expiration.
- **Implementation Steps:**
  1. Identify severely delayed migration waves.
  2. Stop the AWS Replication Agent service on the source machines.
  3. Terminate staging resources (note: full initial sync will be required upon resumption).
- **Estimated Savings:** 100% of staging run costs during the pause.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | Project Managers
- **Prerequisites:** Acceptance that a full re-sync will be needed later.

#### 16. Implement Auto-Scaling for Migrated Target EC2 Instances
- **What:** Post-migration, convert standalone EC2 instances into Auto Scaling Groups (ASGs) if the application supports it.
- **Why It Saves Money:** Lift-and-shift often results in 24/7 static fleets. ASGs allow you to scale down compute during off-peak hours (nights/weekends).
- **Implementation Steps:**
  1. Create an AMI of the successfully migrated target instance.
  2. Create a Launch Template and an Auto Scaling Group.
  3. Implement schedule-based or metric-based scaling policies.
- **Estimated Savings:** 30-50% on target compute costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Stateless application architecture.

### 6. Pricing Model Optimization

#### 17. Complete Migrations Within the 90-Day MGN Free Window
- **What:** Strictly enforce a policy to complete all testing and final cutovers within 90 days of installing the MGN agent on a source server.
- **Why It Saves Money:** MGN is completely free for 90 days per server. Exceeding this triggers a $30.24 per month, per server fee. For 500 servers, this is $15,120/month in easily avoidable fees.
- **Implementation Steps:**
  1. Do not deploy MGN agents until the business is ready to execute the wave within 2 months.
  2. Track agent installation dates in a migration dashboard.
  3. Alert project managers 14 days before the 90-day window expires.
- **Estimated Savings:** $30.24 per server per month.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Project Managers
- **Prerequisites:** Tight project management.

#### 18. Leverage AWS Migration Acceleration Program (MAP) Credits
- **What:** Enroll the migration project into the AWS MAP program to receive substantial AWS credits that offset migration bubble costs.
- **Why It Saves Money:** MAP provides credits (often 15-25% of the projected post-migration Annual Recurring Revenue) to offset the cost of staging infrastructure (MGN EC2/EBS) and dual-running on-premise environments.
- **Implementation Steps:**
  1. Engage your AWS Account Team or an AWS MAP Partner.
  2. Perform an OLA (Optimization and Licensing Assessment).
  3. Tag all MGN and target resources with the provided MAP `map-migrated` tag to claim credits.
- **Estimated Savings:** Covers most or all of the MGN staging costs.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Large-scale migration ($500k+ ARR).

### 7. Network & Data Transfer Optimization

#### 19. Optimize Network Path for Source Data Egress
- **What:** If migrating *from* another cloud provider (e.g., Azure to AWS) or a colocation facility with high bandwidth costs, route MGN replication traffic over private interconnects (AWS Direct Connect) rather than the public internet.
- **Why It Saves Money:** Cloud providers charge heavily for public internet egress (e.g., $0.08/GB). Routing over Direct Connect can lower these fees significantly.
- **Implementation Steps:**
  1. Provision AWS Direct Connect or a Site-to-Site VPN.
  2. Ensure the MGN agent is configured to route traffic through the private VPC endpoint rather than public MGN endpoints.
- **Estimated Savings:** 50-70% on source data egress costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps | Network Team
- **Prerequisites:** Existing or planned Direct Connect infrastructure.

#### 20. Align Staging and Target Availability Zones
- **What:** Ensure that the MGN staging subnet is located in the same Availability Zone (AZ) where the final target EC2 instance will be launched.
- **Why It Saves Money:** When target instances are launched in a different AZ than the staging area, AWS charges cross-AZ data transfer fees ($0.01/GB in each direction) during the instantiation of the final EBS volumes.
- **Implementation Steps:**
  1. Determine the target AZ for the workload.
  2. Configure the MGN Replication Template to place the staging subnet in that exact AZ.
- **Estimated Savings:** $0.02 per GB of migrated data.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC subnet planning.

---

## Cross-Service Synergies
- **AWS Compute Optimizer:** Use Compute Optimizer *after* the migration settles to validate the right-sizing assumptions made during the MGN Launch Template configuration phase.
- **AWS Cost Explorer / CUR:** Monitor the specific costs tagged with `AWSServiceName: Application Migration Service` to catch stalled migrations exceeding the 90-day free window.
- **AWS Systems Manager:** Use Systems Manager Run Command to orchestrate the mass-uninstallation of MGN agents from source servers if a rollback or cancellation occurs.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
- Query `line_item_product_code` = `AWSApplicationMigrationService` to identify post-90-day charges.
- Query `line_item_operation` for EC2 and EBS usage grouped by resource tags indicating the MGN staging area.

### B. CloudWatch Metrics
- Monitor CPUUtilization and DiskReadOps/DiskWriteOps on staging EC2 instances to validate if `t3.small` instances are keeping up with the replication churn.

### C. AWS Config / Trusted Advisor
- Use AWS Config to track changes to EC2 Launch Templates created by MGN to ensure rightsizing rules are being followed prior to cutover.

### D. Company Policies
- Enforce a strict "90-Day Migration Wave" policy to ensure no server exceeds the MGN free replication tier.

### E. IaC (Optional)
- Use Terraform to standardize the MGN Replication Configuration Templates across all AWS accounts to enforce cheap staging resources (`t3.small` + `st1`).

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "MGN-017",
  "service": "AWS Application Migration Service",
  "strategy_category": "Pricing Model Optimization",
  "finding_name": "Stalled Migrations Exceeding 90-Day Free Tier",
  "description": "50 source servers have been replicating for over 90 days, incurring per-hour MGN service charges.",
  "estimated_monthly_savings": 1512.00,
  "risk_level": "Low",
  "action_required": "Execute cutovers immediately or pause replication."
}
```

### Summary Report Table

| Finding ID | Strategy Category | Finding Name | Est. Monthly Savings | Risk Level | Implementation Scope |
|------------|-------------------|--------------|----------------------|------------|----------------------|
| MGN-017 | Pricing Model Optimization | Stalled Migrations Exceeding 90-Day Free Tier | $1,512.00 | Low | FinOps Team, Project Managers |
| MGN-006 | Rightsizing | Oversized Staging EC2 Instances | $850.00 | Low | Engineer/DevOps |
| MGN-001 | Waste Elimination | Orphaned Staging Resources for Cutover Servers | $1,200.00 | Low | Engineer/DevOps |
