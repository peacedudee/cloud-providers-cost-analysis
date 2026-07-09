# Cost-Cutting Playbook: Amazon AppStream 2.0
> **Companion File:** [appstream.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/appstream/appstream.md)
> **Last Updated:** July 2026
---
## Executive Summary
Amazon AppStream 2.0 provides scalable desktop application streaming but can quickly accumulate costs through always-on idle fleets, over-provisioned instance types, and unnecessary Microsoft RDS SAL licensing fees. The primary cost drivers are hourly streaming instance fees, stopped instance fees for baseline capacity on On-Demand fleets, and the $4.19 per user/month RDS SAL fee for Windows fleets. This playbook outlines strategies to optimize these costs, primarily by aggressively scaling fleets, shifting to On-Demand or Elastic fleets, adopting Linux where possible, and eliminating unused capacity.

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
- **Amazon S3:** AppStream 2.0 heavily relies on S3 for Home Folders and Application Blocks. Using S3 Gateway Endpoints eliminates NAT Gateway data processing charges.
- **AWS Directory Service:** Efficient fleet utilization reduces the query load and sizing requirements for Managed Active Directory or AD Connectors.
---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Analyze usage by `productCode` `AmazonAppStream` and operations like `RunFleet`, `RunImageBuilder`.
### B. CloudWatch Metrics
- Monitor `CapacityUtilization`, `InUseCapacity`, and `AvailableCapacity` for fleets to inform scaling and rightsizing policies.
### C. AWS Config / Trusted Advisor
- Check for running Image Builders without active sessions or orphaned fleets.
### D. Company Policies
- Determine core business hours for Scheduled Scaling rules and acceptable user startup times for On-Demand vs Always-On decisions.
### E. IaC (Optional)
- Terraform/CloudFormation templates to review fleet configurations, instance types, and auto-scaling rules.
---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "APPSTREAM-001",
  "service": "Amazon AppStream 2.0",
  "category": "Waste Elimination",
  "finding_name": "Stop Unused Image Builders",
  "impact_level": "High",
  "effort_level": "Low"
}
```
### Summary Report Table
| ID | Strategy | Category | Estimated Savings | Risk |
|---|---|---|---|---|
| APPSTREAM-001 | Stop Unused Image Builders | Waste Elimination | 5-15% | Low |
| APPSTREAM-002 | Configure Idle Disconnect Timeouts | Waste Elimination | 10-25% | Low |
| APPSTREAM-003 | Configure Max Session Duration | Waste Elimination | 5-10% | Low |
| APPSTREAM-004 | Terminate Unused Fleets | Waste Elimination | 5-20% | Low |
| APPSTREAM-005 | Rightsize Instance Types | Rightsizing | 20-50% | Medium |
| APPSTREAM-006 | Downgrade Graphics Instances | Rightsizing | 50-70% | High |
| APPSTREAM-007 | Consolidate App Fleets | Rightsizing | 15-30% | Medium |
| APPSTREAM-008 | Enterprise Discount Program (EDP) | Commitment Discounts | 5-15% | Low |
| APPSTREAM-009 | Migrate Always-On to On-Demand | Architecture Changes | 40-70% | Medium |
| APPSTREAM-010 | Migrate to Elastic Fleets | Architecture Changes | 30-80% | High |
| APPSTREAM-011 | Migrate Windows to Linux Fleets | Architecture Changes | 10-20% | High |
| APPSTREAM-012 | Implement Scheduled Scaling | Scheduling & Auto-Scaling | 40-60% | Low |
| APPSTREAM-013 | Implement Target Tracking Scaling | Scheduling & Auto-Scaling | 20-40% | Low |
| APPSTREAM-014 | Optimize Cooldown Periods | Scheduling & Auto-Scaling | 5-10% | Low |
| APPSTREAM-015 | Bring Your Own License (BYOL) | Pricing Model Optimization | 10-15% | Medium |
| APPSTREAM-016 | Use S3 Gateway Endpoints | Network & Data Transfer Optimization | 1-5% | Low |
| APPSTREAM-017 | Co-Locate Backend Services | Network & Data Transfer Optimization | 1-5% | Low |

---
## Detailed Strategies

### 1. Waste Elimination

#### 1. Stop Unused Image Builders
- **What:** Identify and terminate AppStream 2.0 Image Builders that are running but not actively being used to configure new images.
- **Why It Saves Money:** Image Builders are charged hourly at the same rate as running fleet instances while they are in the "Running" state, regardless of whether an administrator is connected.
- **Implementation Steps:**
  1. Go to the AppStream 2.0 Console > Image Builders.
  2. Identify Image Builders in the "Running" state.
  3. Stop the Image Builders when configuration tasks are complete.
  4. Terminate Image Builders that are obsolete.
- **Estimated Savings:** 5-15%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None

#### 2. Configure Idle Disconnect Timeouts
- **What:** Set aggressive idle disconnect timeouts to automatically end user sessions when they are inactive for a specified period.
- **Why It Saves Money:** For On-Demand and Elastic fleets, disconnecting an idle user transitions the instance to a stopped state (saving ~$0.175/hr for standard.medium) or terminates it entirely, reducing active streaming billing.
- **Implementation Steps:**
  1. Navigate to Fleets in the AppStream 2.0 Console.
  2. Select the fleet and click Edit.
  3. Under Session details, reduce the "Idle disconnect timeout" to 10-15 minutes.
  4. Save changes.
- **Estimated Savings:** 10-25%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Alignment with user experience requirements.

#### 3. Configure Max Session Duration
- **What:** Enforce a maximum session duration to prevent sessions from running indefinitely (e.g., users leaving their browsers open overnight).
- **Why It Saves Money:** Ensures that instances do not remain in the active, fully billed state indefinitely for dormant sessions.
- **Implementation Steps:**
  1. Navigate to Fleets in the AppStream 2.0 Console.
  2. Select the fleet and click Edit.
  3. Set "Maximum session duration" to a reasonable workday limit (e.g., 600 minutes / 10 hours).
  4. Save changes.
- **Estimated Savings:** 5-10%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None

#### 4. Terminate Unused Fleets
- **What:** Identify and delete fleets and stacks that were created for POCs, testing, or obsolete applications.
- **Why It Saves Money:** Even On-Demand fleets incur stopped-instance fees ($0.025/hr) for baseline minimum capacity. Always-On fleets incur full running costs. Terminating unused fleets stops all associated hourly charges.
- **Implementation Steps:**
  1. Identify fleets with zero `InUseCapacity` over the last 30 days via CloudWatch.
  2. Disassociate the fleet from its stack.
  3. Stop the fleet.
  4. Delete the fleet and its corresponding stack.
- **Estimated Savings:** 5-20%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Verification of zero usage.

### 2. Rightsizing

#### 5. Rightsize Instance Types
- **What:** Monitor actual CPU and memory usage and downgrade fleet instance sizes (e.g., from `stream.standard.large` to `stream.standard.medium`).
- **Why It Saves Money:** A `stream.standard.large` ($0.40/hr) costs double a `stream.standard.medium` ($0.20/hr).
- **Implementation Steps:**
  1. Use application-level monitoring tools to track OS-level CPU/Memory utilization of active streaming sessions.
  2. Create a new fleet with the smaller instance type using the existing image.
  3. Associate the new fleet with the stack.
  4. Validate performance with users.
- **Estimated Savings:** 20-50%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Load testing to ensure application performance on smaller instances.

#### 6. Downgrade Graphics Instances
- **What:** Move non-3D, non-GPU accelerated applications from `stream.graphics.*` instances to `stream.standard.*` instances.
- **Why It Saves Money:** Graphics instances like `g4dn.xlarge` cost $0.75/hr, while a `standard.large` with equivalent CPU/RAM costs $0.40/hr.
- **Implementation Steps:**
  1. Audit applications running on graphics fleets.
  2. Identify workloads that do not leverage CUDA or DirectX APIs.
  3. Rebuild the image on a standard Image Builder.
  4. Deploy to a standard instance fleet.
- **Estimated Savings:** 50-70%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Testing application compatibility without GPU acceleration.

#### 7. Consolidate App Fleets
- **What:** Combine multiple single-application fleets into a single multi-application fleet.
- **Why It Saves Money:** Each fleet requires a minimum baseline capacity and an associated buffer of idle/stopped instances to handle logins. Consolidating fleets pools this buffer, drastically reducing the total number of idle instances maintained.
- **Implementation Steps:**
  1. Group applications with similar OS, performance, and IAM requirements.
  2. Create a master AppStream image containing all apps in the group.
  3. Create a single consolidated fleet.
  4. Route users to the consolidated stack.
- **Estimated Savings:** 15-30%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Apps must not conflict when installed on the same OS image.

### 3. Commitment Discounts

#### 8. Enterprise Discount Program (EDP)
- **What:** Leverage an AWS EDP if your overall organizational spend is high enough to qualify.
- **Why It Saves Money:** AppStream 2.0 does not offer Savings Plans or Reserved Instances, so an overall EDP discount is the primary mechanism to reduce on-demand unit costs.
- **Implementation Steps:**
  1. Work with AWS Account Manager to assess total AWS spend.
  2. Negotiate an EDP discount (typically 5-15% across all services).
  3. Sign EDP commitment.
- **Estimated Savings:** 5-15%
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** High total AWS annual spend (typically >$1M).

### 4. Architecture Changes

#### 9. Migrate Always-On to On-Demand
- **What:** Switch fleet configurations from Always-On (instances run constantly) to On-Demand (instances are stopped until requested by a user).
- **Why It Saves Money:** Always-On instances charge full rates (e.g., $0.20/hr) 24/7. On-Demand charges full rates only when active, and a reduced stopped fee ($0.025/hr) when idle in the capacity pool.
- **Implementation Steps:**
  1. Go to AppStream 2.0 Console > Fleets.
  2. Create a new On-Demand fleet matching the Always-On configuration.
  3. Update the Stack to use the new On-Demand fleet.
  4. Delete the Always-On fleet.
- **Estimated Savings:** 40-70%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Users must tolerate a 1-2 minute start time for On-Demand sessions.

#### 10. Migrate to Elastic Fleets
- **What:** Use Elastic Fleets instead of Always-On or On-Demand fleets by delivering applications via S3 Application Blocks.
- **Why It Saves Money:** Elastic Fleets have no capacity pools, no minimum capacities, and zero stopped-instance fees. You are billed per second only when users are actively streaming.
- **Implementation Steps:**
  1. Package applications into Virtual Hard Disks (VHDs).
  2. Upload to S3 and create AppStream Application Blocks.
  3. Create an Elastic Fleet.
  4. Assign Application Blocks to the Elastic Fleet.
- **Estimated Savings:** 30-80%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Applications must be portable (run without deep OS installation/reboots) and compatible with Elastic Fleets.

#### 11. Migrate Windows to Linux Fleets
- **What:** Move web-based, Java, or open-source applications from Windows Server AppStream fleets to Amazon Linux 2 AppStream fleets.
- **Why It Saves Money:** Completely eliminates the $4.19 per user per month Microsoft RDS SAL fee, which heavily impacts deployments with thousands of intermittent users.
- **Implementation Steps:**
  1. Identify applications compatible with Linux.
  2. Provision a Linux Image Builder.
  3. Install and configure applications.
  4. Deploy a Linux fleet.
- **Estimated Savings:** 10-20% (Highly dependent on user count)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application compatibility with Amazon Linux 2.

### 5. Scheduling & Auto-Scaling

#### 12. Implement Scheduled Scaling
- **What:** Configure Scheduled Scaling policies to drop fleet Minimum Capacity to zero (or very low) outside of business hours.
- **Why It Saves Money:** Eliminates the stopped-instance fees ($0.025/hr) or running fees ($0.20/hr) for the baseline fleet capacity during nights, weekends, and holidays.
- **Implementation Steps:**
  1. Navigate to Application Auto Scaling via CLI or AppStream console.
  2. Define a schedule (e.g., cron expression for 6 PM Friday to 6 AM Monday).
  3. Set Minimum Capacity to 0 during off-hours, and restore to normal levels during business hours.
- **Estimated Savings:** 40-60%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Predictable user schedules.

#### 13. Implement Target Tracking Scaling
- **What:** Use Target Tracking scaling policies to dynamically adjust fleet size based on actual `CapacityUtilization`.
- **Why It Saves Money:** Automatically scales in (removes unused capacity buffer) during low demand, avoiding paying for unnecessary stopped or running instances in the pool.
- **Implementation Steps:**
  1. Select Fleet > Scaling Policies.
  2. Add Target Tracking policy.
  3. Set target `CapacityUtilization` to 75%.
  4. Configure scale-out and scale-in cooldowns.
- **Estimated Savings:** 20-40%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None

#### 14. Optimize Cooldown Periods
- **What:** Adjust scale-in and scale-out cooldown periods on auto-scaling policies to prevent fleet flapping.
- **Why It Saves Money:** A scale-in cooldown that is too long keeps unnecessary instances running longer than needed after users log off.
- **Implementation Steps:**
  1. Review existing scaling policies.
  2. Reduce scale-in cooldowns to 5-10 minutes to rapidly remove idle buffer instances.
  3. Monitor `CapacityUtilization` closely to ensure users aren't starved of capacity.
- **Estimated Savings:** 5-10%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Auto-scaling must be active.

### 6. Pricing Model Optimization

#### 15. Bring Your Own License (BYOL)
- **What:** Use your existing Microsoft RDS Client Access Licenses (CALs) with active Software Assurance instead of paying the AWS per-user fee.
- **Why It Saves Money:** Eliminates the $4.19 per user per month fee charged by AWS for Microsoft RDS SALs.
- **Implementation Steps:**
  1. Verify RDS CALs have active Software Assurance.
  2. Submit a License Mobility verification form to Microsoft.
  3. Inform AWS Support of your License Mobility status.
- **Estimated Savings:** 10-15% (or exactly $4.19/user/month)
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team / Procurement
- **Prerequisites:** Eligible Microsoft Licensing Agreement.

### 7. Network & Data Transfer Optimization

#### 16. Use S3 Gateway Endpoints
- **What:** Configure S3 Gateway Endpoints in the VPC subnets where AppStream fleets reside.
- **Why It Saves Money:** AppStream heavily uses S3 for Application Blocks, Home Folders, and Application Settings Persistence. If routed through a NAT Gateway, you pay $0.045/GB in data processing fees. S3 Gateway Endpoints are free.
- **Implementation Steps:**
  1. Go to VPC Console > Endpoints.
  2. Create a Gateway Endpoint for S3.
  3. Attach it to the Route Tables associated with AppStream subnets.
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Fleet must reside in a private subnet.

#### 17. Co-Locate Backend Services
- **What:** Ensure backend databases and file servers accessed by AppStream applications reside in the same AWS Region and Availability Zone.
- **Why It Saves Money:** Cross-AZ data transfer costs $0.01/GB, and cross-region costs $0.02/GB+. Co-locating resources eliminates these egress fees.
- **Implementation Steps:**
  1. Analyze VPC Flow Logs to identify AppStream egress traffic destinations.
  2. Relocate backend EC2/RDS instances or adjust AppStream subnets to match AZs.
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Architecture and subnet flexibility.
