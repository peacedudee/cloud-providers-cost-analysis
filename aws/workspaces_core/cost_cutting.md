# Cost-Cutting Playbook: Amazon WorkSpaces Core
> **Companion File:** [workspaces_core.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/workspaces_core/workspaces_core.md)
> **Last Updated:** July 2026
---
## Executive Summary
This playbook provides 20 actionable cost-optimization strategies for Amazon WorkSpaces Core. WorkSpaces Core enables enterprises to run cloud desktops on AWS while utilizing third-party management solutions like Citrix and VMware. Cost savings for WorkSpaces Core focus heavily on licensing optimization, appropriate run-mode selection (AlwaysOn vs. AutoStop), and rigorous lifecycle management via VDI partners.

## Strategy Categories
### 1. Waste Elimination
- **WKSCORE-001:** Terminate unused or orphaned WorkSpaces Core instances assigned to departed employees or inactive contractors.
- **WKSCORE-002:** Delete orphaned EBS volumes (Root/User) left behind by misconfigured instance termination policies.
- **WKSCORE-003:** Clean up outdated, unused custom OS images and bundles to save on associated snapshot storage costs.
- **WKSCORE-004:** Decommission idle AWS Directory Service directories (Simple AD, Managed AD) that are not actively linked to WorkSpaces.

### 2. Rightsizing
- **WKSCORE-005:** Downgrade instance types (e.g., from `Core Power` to `Core Standard`) for users who only perform basic task work.
- **WKSCORE-006:** Reduce oversized Root and User storage volumes to closely match actual storage consumption.
- **WKSCORE-007:** Rightsize the underlying Citrix/VMware connection broker and control plane EC2 instances running in your VPC.

### 3. Commitment Discounts
- **WKSCORE-008:** Leverage existing Citrix or VMware Enterprise Agreements to use WorkSpaces Core instead of full Amazon WorkSpaces (saving ~66% on infrastructure base rates).
- **WKSCORE-009:** Utilize Microsoft Bring Your Own License (BYOL) for Windows 10/11 desktop OS to bypass AWS-provided OS licensing surcharges.

### 4. Architecture Changes
- **WKSCORE-010:** Shift from persistent, dedicated virtual desktops to non-persistent, pooled desktops managed by Citrix/VMware to increase utilization density.
- **WKSCORE-011:** Migrate eligible users from Windows to Amazon Linux 2 or Ubuntu Core instances to eliminate Microsoft OS licensing fees.
- **WKSCORE-012:** Centralize user profiles and shared data on Amazon FSx or EFS instead of provisioning large, underutilized individual EBS User volumes.
- **WKSCORE-013:** Consolidate multiple distributed Active Directories into a single shared AWS Managed Microsoft AD architecture to reduce directory overhead.

### 5. Scheduling & Auto-Scaling
- **WKSCORE-014:** Switch WorkSpaces to AutoStop (Hourly) mode for part-time contractors or shift workers operating less than ~80 hours per month.
- **WKSCORE-015:** Decrease the AutoStop disconnect timeout (e.g., from 4 hours to 1 hour) to power off idle instances sooner and reduce hourly charges.
- **WKSCORE-016:** Implement aggressive scaling policies within Citrix Autoscale or VMware Horizon to aggressively deallocate Core compute outside of business hours.

### 6. Pricing Model Optimization
- **WKSCORE-017:** Switch instances from AutoStop to AlwaysOn (Flat Rate) for power users exceeding 82 hours per month.
- **WKSCORE-018:** Automate monthly run-mode billing optimization using AWS Cost Optimizer scripts to continually toggle AlwaysOn/AutoStop based on historical usage.

### 7. Network & Data Transfer Optimization
- **WKSCORE-019:** Deploy VPC Endpoints (AWS PrivateLink) for VDI control plane traffic to avoid costly NAT Gateway data processing charges.
- **WKSCORE-020:** Restrict internet egress traffic from WorkSpaces instances using security groups or AWS Network Firewall to reduce outbound data transfer fees.

---
## Cross-Service Synergies
- **Amazon EC2:** Rightsizing and scheduling the VDI management plane (Citrix Cloud Connectors, VMware UAGs).
- **Amazon FSx / EFS:** Implementing cheaper shared storage solutions to replace oversized individual desktop volumes.
- **AWS Directory Service:** Optimizing Active Directory deployments for identity brokering.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Identify specific billing line items for Core infrastructure (e.g., `AlwaysOn-Core-Std`, `AutoStop-Core-Std`).
### B. CloudWatch Metrics
- Analyze metrics like `UserConnected`, `SessionDisconnect`, and `CPUUtilization` to determine idle times.
### C. AWS Config / Trusted Advisor
- Checks for unused WorkSpaces and unattached EBS volumes.
### D. Company Policies
- Remote work policies, hardware refresh cycles, and standard software stacks.
### E. IaC (Optional)
- Terraform or CloudFormation templates defining the Core infrastructure and Directory Service setup.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "WKSCORE-001",
  "category": "Waste Elimination",
  "title": "Terminate unused WorkSpaces Core instances",
  "resource_id": "ws-12345abcd",
  "estimated_monthly_savings": 22.00,
  "level_of_effort": "Low",
  "risk_level": "Low"
}
```
### Summary Report Table
| Finding ID | Category | Title | Est. Savings | Effort | Risk |
|---|---|---|---|---|---|
| WKSCORE-001 | Waste Elimination | Terminate unused WorkSpaces Core instances | $1,200 | Low | Low |
| WKSCORE-005 | Rightsizing | Downgrade Core Power to Core Standard | $800 | Medium | Medium |
| WKSCORE-014 | Scheduling & Auto-Scaling | Switch to AutoStop for part-time workers | $550 | Low | Low |
