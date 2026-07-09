# Cost-Cutting Playbook: Amazon WorkSpaces
> **Companion File:** [workspaces.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/workspaces/workspaces.md)
> **Last Updated:** July 2026
---
## Executive Summary
Amazon WorkSpaces is a fully managed Desktop-as-a-Service (DaaS) solution that enables scalable virtual desktops. Due to its per-user pricing structure, costs can rapidly spiral out of control without active lifecycle management and right-sizing. The most significant savings are realized by deploying the AWS WorkSpaces Cost Optimizer to dynamically switch billing models (AutoStop vs. AlwaysOn) and terminating orphaned WorkSpaces. This playbook provides 20 actionable strategies to optimize WorkSpaces environments.

## Strategy Categories
### 1. Waste Elimination
*   **WKS-WASTE-001: Terminate Orphaned WorkSpaces:** Identify and terminate WorkSpaces assigned to former employees or inactive contractors.
*   **WKS-WASTE-002: Delete Unused Custom Images & Bundles:** Remove obsolete Golden Images and custom bundles to eliminate storage costs.
*   **WKS-WASTE-003: Purge Unattached Snapshots:** Clean up legacy WorkSpaces snapshots that are no longer required for recovery.
*   **WKS-WASTE-004: Deregister Unused Directories:** Remove inactive AWS Managed Microsoft AD or Simple AD directories used solely for deleted WorkSpaces.

### 2. Rightsizing
*   **WKS-RIGHT-001: Downgrade Power to Performance:** Downgrade users who do not consistently require 4 vCPUs / 16 GB RAM to the Performance bundle.
*   **WKS-RIGHT-002: Downgrade Performance to Standard:** Move basic administrative and task workers from Performance (8 GB RAM, $44/mo) to Standard (4 GB RAM, $33/mo).
*   **WKS-RIGHT-003: Right-Size Root and User Volumes:** Prevent over-provisioning of WorkSpaces storage volumes (e.g., allocating 100 GB+ when utilization is < 20%).

### 3. Commitment Discounts
*   **WKS-COM-001: Bring Your Own License (BYOL):** Leverage existing Microsoft Windows 10/11 Desktop enterprise licenses to save $4.00 per user per month.
*   **WKS-COM-002: BYOL for Productivity Suites:** Switch from bundled Microsoft Office to existing Microsoft 365 E3/E5 enterprise licenses.
*   **WKS-COM-003: Volume Discounts via EDP:** Ensure WorkSpaces spend is aggregated into overall AWS Enterprise Discount Program (EDP) negotiations.

### 4. Architecture Changes
*   **WKS-ARCH-001: Migrate to Linux WorkSpaces:** Transition developers or task workers from Windows to Amazon Linux or Ubuntu WorkSpaces to avoid OS licensing premiums.
*   **WKS-ARCH-002: Replace with Amazon AppStream 2.0:** Use AppStream 2.0 for users who only need access to specific legacy applications rather than a persistent full desktop.
*   **WKS-ARCH-003: Utilize Amazon WorkSpaces Web:** Deploy secure, browser-based access for web-only users instead of provisioning full OS virtual desktops.
*   **WKS-ARCH-004: Centralize File Storage:** Use Amazon FSx for Windows File Server for shared data instead of expanding expensive EBS volumes on individual WorkSpaces.

### 5. Scheduling & Auto-Scaling
*   **WKS-SCHED-001: Enforce Aggressive AutoStop Timeouts:** Reduce inactivity disconnect timeouts from 2-3 hours down to 1 hour to minimize hourly billing on AutoStop bundles.
*   **WKS-SCHED-002: Automate Inactive User Termination:** Implement a Lambda function to automatically terminate WorkSpaces that haven't been logged into for over 30 days.
*   **WKS-SCHED-003: Restrict Logon Hours:** Prevent AutoStop users from logging on during weekends or off-hours via Directory Service policies, unless specifically authorized.

### 6. Pricing Model Optimization
*   **WKS-PRICING-001: Deploy AWS WorkSpaces Cost Optimizer:** Automate monthly transitions between AlwaysOn and AutoStop billing models based on the break-even threshold (e.g., ~77 hours/month for Standard).
*   **WKS-PRICING-002: Default to AutoStop for Contractors:** Enforce AutoStop hourly billing as the default provisioning model for part-time, temporary, or contractor accounts.

### 7. Network & Data Transfer Optimization
*   **WKS-NET-001: Optimize WSP/PCoIP Protocols:** Tune WorkSpaces Streaming Protocol (WSP) settings to reduce pixel streaming data transfer out (DTO) over WAN.
*   **WKS-NET-002: Regional Proximity:** Deploy WorkSpaces in regions geographically closest to end-users to minimize latency and potential cross-region networking charges.

---
## Cross-Service Synergies
*   **AWS IAM Identity Center:** Integrate with IAM Identity Center to streamline user lifecycle management and ensure immediate WorkSpaces termination upon employee offboarding.
*   **Amazon FSx / EFS:** Offload user data from local WorkSpaces storage to centralized, deduplicated FSx or EFS volumes.
*   **AWS CloudTrail:** Audit WorkSpaces API calls to track who is provisioning expensive Power or Graphics bundles.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
*   `lineItem/ProductCode` = `AmazonWorkSpaces`
*   `lineItem/UsageType` (Identify Hourly vs. Monthly usage, bundle types)
*   `lineItem/Operation` (Identify Running vs. Stopped, Storage costs)

### B. CloudWatch Metrics
*   `UserConnected` (Track active user sessions and connection duration)
*   `Stopped` (Track duration of stopped state for AutoStop)
*   `CPUUtilization` / `MemoryUtilization` (Determine rightsizing candidates)

### C. AWS Config / Trusted Advisor
*   "Underutilized Amazon WorkSpaces" (Trusted Advisor check for WorkSpaces with low usage)
*   AWS Config rules for standardizing baseline WorkSpaces configurations.

### D. Company Policies
*   HR Offboarding SLAs (How quickly are accounts terminated?).
*   Approved software and OS baselines for BYOL eligibility.

### E. IaC (Optional)
*   Terraform `aws_workspaces_workspace` definitions (to enforce `workspace_properties.running_mode` based on user groups).

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "WKS-PRICING-001",
  "service": "Amazon WorkSpaces",
  "category": "Pricing Model Optimization",
  "title": "Convert Low-Usage AlwaysOn WorkSpaces to AutoStop",
  "description": "User has logged in for less than 77 hours this month on a Standard bundle, making AutoStop more cost-effective.",
  "severity": "High",
  "estimated_monthly_savings_usd": 23.25,
  "action_required": "Switch Running Mode to AutoStop via Cost Optimizer.",
  "resources": [
    "ws-12345abcd"
  ]
}
```

### Summary Report Table
| Finding ID | Title | Impact | Est. Savings | Effort |
|------------|-------|--------|--------------|--------|
| WKS-PRICING-001 | Deploy WorkSpaces Cost Optimizer | High | $$$ | Low |
| WKS-WASTE-001 | Terminate Orphaned WorkSpaces | High | $$$ | Low |
| WKS-RIGHT-002 | Downgrade Performance to Standard | Medium | $$ | Low |
| WKS-COM-001 | Enable BYOL for Windows Desktop | Medium | $$ | Medium |
| WKS-ARCH-001 | Migrate to Amazon Linux WorkSpaces | Low | $ | High |
