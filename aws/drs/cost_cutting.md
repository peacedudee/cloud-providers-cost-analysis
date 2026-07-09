# Cost-Cutting Playbook: AWS Elastic Disaster Recovery
> **Companion File:** [drs.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/drs/drs.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Elastic Disaster Recovery (DRS) provides scalable and cost-effective disaster recovery by maintaining a low-cost staging area instead of a fully active replica environment. However, costs can silently accumulate through oversized staging resources, excessive Point-in-Time (PIT) snapshot retention, unchecked data transfer, and abandoned drill instances. This playbook provides targeted strategies to minimize the total cost of ownership for DRS while maintaining rigorous Recovery Point Objectives (RPO) and Recovery Time Objectives (RTO).

## Strategy Categories
### 1. Waste Elimination
### 2. Rightsizing
### 3. Commitment Discounts
### 4. Architecture Changes
### 5. Scheduling & Auto-Scaling
### 6. Pricing Model Optimization
### 7. Network & Data Transfer Optimization
---
## Cross-Service Synergies
- **AWS Compute Optimizer:** Use to rightsize the target EC2 instances defined in your DRS Launch Templates.
- **AWS Backup:** Integrate for long-term retention rather than keeping extensive PIT snapshots within DRS itself.
- **AWS Cost Explorer / CUR:** Monitor the "Elastic Disaster Recovery" service and EC2/EBS usage in your designated staging subnet.
---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Filter by `lineItem/ProductCode` = `AWSElasticDisasterRecovery`, `AmazonEC2` (for staging and drill instances).
- Analyze EBS snapshot and volume costs linked to DRS tags.
### B. CloudWatch Metrics
- Monitor staging server CPU and network utilization.
### C. AWS Config / Trusted Advisor
- Track unattached EBS volumes or orphaned snapshots in the staging region.
### D. Company Policies
- Review documented RPO/RTO requirements to justify snapshot retention policies.
### E. IaC (Optional)
- Terraform/CloudFormation templates defining DRS replication settings and launch templates.
---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "DRS-001",
  "strategy_category": "Waste Elimination",
  "resource_id": "drs-recovery-instance-xxxx",
  "recommendation": "Terminate abandoned DR drill instance",
  "estimated_monthly_savings": 250.00,
  "effort_level": "Low"
}
```

### Summary Report Table
| ID | Strategy | Category | Est. Savings | Risk | Scope |
|---|---|---|---|---|---|
| DRS-001 | Terminate Abandoned Drill Instances | Waste Elimination | High | Low | FinOps/Eng |
| DRS-002 | Exclude Temp/Scratch Volumes | Waste Elimination | Medium | Low | Eng |
| DRS-003 | Optimize PIT Retention | Rightsizing | High | Medium | Eng |
| DRS-004 | Use sc1 for Staging Storage | Rightsizing | High | Low | Eng |
| DRS-005 | Consolidate Replication Servers | Rightsizing | Medium | Low | Eng |
| DRS-006 | Stop Replicating Decommissioned Servers | Waste Elimination | Medium | Low | Eng |
| DRS-007 | Rightsize Launch Template Instances | Rightsizing | High | Medium | Eng |
| DRS-008 | Clean Up Orphaned Snapshots | Waste Elimination | Medium | Low | FinOps/Eng |
| DRS-009 | Leverage Savings Plans for Staging | Commitment Discounts | Medium | Low | FinOps |
| DRS-010 | Tiered DR Architecture | Architecture Changes | High | High | Leadership |
| DRS-011 | Auto-terminate Drill Instances | Scheduling | Medium | Low | DevOps |
| DRS-012 | Leverage 30-Day Free Trial | Pricing Model | Low | Low | FinOps |
| DRS-013 | Optimize Intra-Region Data Transfer | Network | Medium | Medium | Eng |
| DRS-014 | Rightsize Replication Servers | Rightsizing | Low | Low | Eng |
| DRS-015 | Tag-Based Exclusions | Architecture Changes | High | Medium | Eng/Leadership |

---

#### 1. Terminate Abandoned Drill Instances
- **What:** Identify and terminate EC2 instances launched during DR drills that were left running after the test concluded.
- **Why It Saves Money:** Drill instances run on full-priced, typically large EC2 instance types and hydrated high-speed gp3 EBS volumes. 
- **Implementation Steps:** 
  1. Go to the AWS DRS console.
  2. Navigate to "Recovery instances".
  3. Identify instances used for testing that are no longer needed.
  4. Select the instances and click "Delete recovery instances".
- **Estimated Savings:** 50-90% of compute costs (depending on drill frequency and duration).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** DR drill testing must be complete.

#### 2. Exclude Temp/Scratch Volumes from Replication
- **What:** Configure the AWS Replication Agent to ignore swap space, temp directories, or database staging disks on source machines.
- **Why It Saves Money:** Avoids unnecessary block writes, reducing staging EBS volume size, snapshot size, and replication data transfer fees.
- **Implementation Steps:**
  1. Access the source server where the replication agent is installed.
  2. Reconfigure the agent settings to exclude specific disks/paths (e.g., pagefiles, swap partitions).
  3. Restart the replication agent.
- **Estimated Savings:** 10-20% of staging storage and data transfer costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Identification of volumes containing strictly ephemeral data.

#### 3. Optimize PIT Snapshot Retention
- **What:** Reduce the Point-in-Time (PIT) snapshot retention window from the default 14 days to a shorter period (e.g., 3-7 days) based on actual business RPO.
- **Why It Saves Money:** Snapshots are billed at $0.05/GB-month. High-churn servers generate massive snapshot chains over 14 days. Reducing the window drastically cuts snapshot storage costs.
- **Implementation Steps:**
  1. Go to the DRS console -> Replication settings.
  2. Edit the replication template or individual server settings.
  3. Locate "Point in time (PIT) policy" and reduce the retention days.
  4. Save settings. Old snapshots will automatically expire.
- **Estimated Savings:** 30-60% of snapshot storage costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Alignment with business RPO/compliance requirements.

#### 4. Use sc1 (Cold HDD) for Staging Storage
- **What:** Switch the staging area EBS volumes from gp3 (SSD) to sc1 (Cold HDD) for source servers with low transactional write rates.
- **Why It Saves Money:** sc1 storage costs ~$0.015/GB-month compared to gp3's ~$0.08/GB-month, reducing staging volume costs by over 80%.
- **Implementation Steps:**
  1. In the DRS console, go to Replication settings.
  2. Under Staging area volume type, select `sc1`.
  3. Monitor replication lag to ensure sc1 throughput can handle the source's write rate.
- **Estimated Savings:** 80% of staging volume costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Source servers must have low to moderate disk churn.

#### 5. Consolidate Replication Servers (Multi-Tenant Staging)
- **What:** Configure DRS to use a single shared replication server (EC2 instance) to handle replication for multiple source servers.
- **Why It Saves Money:** Reduces the number of continuous lightweight EC2 instances (like `t3.small`) running 24/7 in the staging area.
- **Implementation Steps:**
  1. Go to Replication settings.
  2. Adjust the replication server instance type or routing to consolidate multiple disks to fewer replication servers.
- **Estimated Savings:** $10-$20 per month per eliminated replication server.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Combined write throughput must not exceed the replication server's network/EBS limits.

#### 6. Stop Replicating Decommissioned Servers
- **What:** Disconnect and remove source servers from DRS that have been decommissioned or are no longer in service.
- **Why It Saves Money:** Stops the flat $0.028/hr ($20.44/mo) DRS replication fee per server, plus associated staging compute and storage costs.
- **Implementation Steps:**
  1. Identify inactive servers in the DRS console.
  2. Select the server and choose "Disconnect from AWS".
  3. Delete the server from the DRS console to clear staging resources.
- **Estimated Savings:** $35-$100+ per month per decommissioned server.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Confirmation that the source server is permanently retired.

#### 7. Rightsize Launch Template Instances
- **What:** Downgrade the target EC2 instance sizes defined in the DRS Launch Templates if the DR workload does not require full production capacity immediately.
- **Why It Saves Money:** When failovers or drills occur, smaller instances (e.g., m6i.large instead of m6i.2xlarge) incur lower hourly run costs.
- **Implementation Steps:**
  1. Go to DRS -> Source servers -> Launch settings.
  2. Modify the EC2 Launch Template.
  3. Select a more cost-effective instance type for the target environment.
- **Estimated Savings:** 20-50% on drill/failover compute costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of minimum performance requirements during a DR event.

#### 8. Clean Up Orphaned Snapshots
- **What:** Identify and delete EBS snapshots left behind if a source server was improperly removed from DRS without cleaning up the staging area.
- **Why It Saves Money:** Prevents ongoing $0.05/GB-month storage charges for useless, disconnected snapshots.
- **Implementation Steps:**
  1. Go to the EC2 console -> Snapshots.
  2. Filter by tags related to DRS (e.g., `AWSElasticDisasterRecoveryManaged`).
  3. Verify snapshots that are no longer associated with active replication.
  4. Delete orphaned snapshots.
- **Estimated Savings:** $5-$50+ per month per orphaned chain.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Validation that snapshots are truly orphaned.

#### 9. Leverage Savings Plans for Staging Compute
- **What:** Cover the lightweight EC2 instances (t3.small/micro) running continuously in the DRS staging area with Compute Savings Plans.
- **Why It Saves Money:** Staging servers run 24/7/365, making them perfect candidates for 1- or 3-year commitments, yielding discounts up to 60-72%.
- **Implementation Steps:**
  1. Use AWS Cost Explorer to analyze steady-state staging EC2 usage.
  2. Purchase Compute Savings Plans via AWS Cost Management.
- **Estimated Savings:** 30-60% on staging EC2 costs.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Predictable, long-term DR strategy.

#### 10. Tiered DR Architecture (Pilot Light / Cold Standby)
- **What:** Re-architect the DR strategy so only mission-critical databases replicate continuously via DRS, while app/web tiers use AMI backups or Infrastructure as Code (IaC) to deploy on demand.
- **Why It Saves Money:** Eliminates the $20.44/mo per-server fee and staging costs for stateless servers that can simply be rebuilt from templates.
- **Implementation Steps:**
  1. Categorize servers by state (stateful vs stateless).
  2. Remove stateless servers from DRS replication.
  3. Ensure IaC pipelines are robust enough to deploy the stateless tier quickly during an event.
- **Estimated Savings:** 50-70% total DR costs depending on fleet composition.
- **Risk Level:** High
- **Implementation Scope:** Leadership | Engineer/DevOps
- **Prerequisites:** Mature IaC (Terraform/CloudFormation) and automated AMI pipelines.

#### 11. Auto-terminate Drill Instances (EventBridge/Lambda)
- **What:** Implement automation to forcefully stop or terminate DRS drill instances after a fixed time limit (e.g., 12 hours).
- **Why It Saves Money:** Prevents human error from leaving expensive target instances and gp3 volumes running indefinitely after a test.
- **Implementation Steps:**
  1. Create an AWS EventBridge rule triggering on DRS drill launch events.
  2. Trigger a Lambda function that waits or schedules termination of tagged drill instances.
- **Estimated Savings:** Prevents cost spikes; highly variable.
- **Risk Level:** Low
- **Implementation Scope:** DevOps
- **Prerequisites:** Standardized drill windows.

#### 12. Leverage 30-Day Free Trial for Migrations
- **What:** Use the DRS 30-day free trial strategically if using DRS for server migration purposes (though AWS Application Migration Service is generally preferred, some use DRS).
- **Why It Saves Money:** The $0.028/hr replication fee is waived for the first 720 hours.
- **Implementation Steps:**
  1. Plan cutovers within the 30-day window.
  2. Ensure resources are completely disconnected before day 31.
- **Estimated Savings:** $20.44 per server.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Fast migration timelines.

#### 13. Optimize Intra-Region Data Transfer
- **What:** If replicating within AWS (Region A to Region A, across AZs), ensure routing avoids traversing NAT Gateways or public IPs unnecessarily.
- **Why It Saves Money:** Intra-AZ or Inter-AZ data transfer via NAT Gateway incurs processing charges ($0.045/GB). VPC Peering or PrivateLink reduces this.
- **Implementation Steps:**
  1. Use VPC Endpoints for DRS APIs.
  2. Route replication traffic privately over local VPC links or Transit Gateway.
- **Estimated Savings:** $0.02 - $0.045 per GB of replicated data.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps (Networking)
- **Prerequisites:** Network architecture review.

#### 14. Rightsize Replication Servers
- **What:** Ensure the replication servers automatically provisioned by DRS are not oversized for the data churn they are handling.
- **Why It Saves Money:** By default DRS provisions instances based on disk count. Over-provisioned replication servers waste EC2 spend.
- **Implementation Steps:**
  1. Review staging instance CPU and network metrics in CloudWatch.
  2. Adjust replication server settings to enforce smaller instance types if utilization is near zero.
- **Estimated Savings:** $5-$15 per server month.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch metric analysis.

#### 15. Tag-Based Exclusions for Non-Essential Environments
- **What:** Audit your DR environment to ensure Dev, Test, or QA servers were not accidentally included in the DRS replication scope.
- **Why It Saves Money:** Avoids paying replication fees and staging costs for environments that do not require disaster recovery guarantees.
- **Implementation Steps:**
  1. Audit installed AWS Replication Agents across your fleet.
  2. Uninstall the agent and disconnect servers tagged `env:dev` or `env:test`.
- **Estimated Savings:** 100% of DRS costs for non-prod systems.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps | Leadership
- **Prerequisites:** Strict environment tagging policy.
