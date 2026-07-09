# Cost-Cutting Playbook: Amazon Pinpoint
> **Companion File:** [pinpoint.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/pinpoint/pinpoint.md)
> **Last Updated:** July 2026

---
## Executive Summary
Amazon Pinpoint is actively undergoing sunset, with AWS ceasing support on October 30, 2026. While short-term optimization involves aggressively pruning stale user endpoints, managing Monthly Targeted Users (MTU) counts, and shifting traffic to cheaper channels like Push Notifications over SMS, the ultimate "cost-cutting" strategy is a complete architectural migration. Workloads must transition to AWS End User Messaging (Push/SMS), Amazon SES (Email), Amazon Connect (Journeys), and Amazon Kinesis (Analytics). This playbook provides 19 strategies to govern costs leading up to the sunset date while facilitating efficient migrations.

## Strategy Categories
### 1. Waste Elimination
- **Prune Inactive Endpoints:** Regularly remove device tokens older than 90 days to avoid MTU charges.
- **Remove Unverified Emails:** Cleanse segments of hard bounces and invalid addresses.
- **Delete Unused Projects:** Purge legacy Pinpoint applications that continue to accumulate endpoint data.
- **Pause Inactive Campaigns:** Stop automated journeys with no active engagement.
- **Clean Up Abandoned Segments:** Remove dynamic segments that are no longer queried.

### 2. Rightsizing
- **Consolidate User IDs:** Ensure users with multiple devices map to a single endpoint to avoid MTU duplication.
- **Rightsize Event Payloads:** Reduce the size and frequency of custom event tracking.
- **Filter Client-Side Events:** Drop non-essential analytics events on the device before transmission to reduce API requests.
- **Optimize Campaign Frequency:** Cap messages per user per month to decrease volume and improve engagement.

### 3. Commitment Discounts
- **Maximize Free Tier:** Leverage the 5,000 free MTUs, 1M Push notifications, and 10k emails per month where applicable.
- *(Note: Pinpoint does not support standard Reserved Instances or Savings Plans)*.

### 4. Architecture Changes
- **Migrate SMS/Push to End User Messaging:** Shift transactional messaging away from Pinpoint ahead of the 2026 sunset.
- **Migrate Emails to SES:** Move raw email volume to standard SES sending infrastructure.
- **Migrate Journeys to Amazon Connect:** Transition marketing automation flows to Amazon Connect Outbound Campaigns.
- **Migrate Analytics to Kinesis:** Transition event pipelines to Amazon Kinesis.

### 5. Scheduling & Auto-Scaling
- **Pre-Billing Cycle Pruning:** Schedule endpoint cleanups before the month rolls over to minimize MTU baseline counts.
- **Smart Sending/Quiet Hours:** Suppress campaign execution during low-engagement periods to reduce wasted deliveries.
- **Batch Event Imports:** Ingest segment data in scheduled batches rather than continuous real-time streams.

### 6. Pricing Model Optimization
- **Shift SMS to Push:** Prioritize Push notifications ($1.00/Million) over SMS ($0.0065+ each) wherever feasible.

### 7. Network & Data Transfer Optimization
- **VPC Endpoints:** Route backend API calls to Pinpoint through VPC Endpoints to eliminate NAT Gateway data processing charges.

---
## Cross-Service Synergies
- **Amazon SES:** Handles the underlying email transport. Optimizing SES deliverability directly benefits Pinpoint email ROI.
- **AWS End User Messaging & Amazon Connect:** The designated successors to Pinpoint. Cost optimization now involves shifting workloads incrementally to these services.
- **Amazon Kinesis & QuickSight:** Replace Pinpoint's native analytics with more flexible and cost-effective data pipelines.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- `lineItem/ProductCode` = `AmazonPinpoint`
- Analyze `UsageType` for `MTU`, `Push`, and `SMS`.

### B. CloudWatch Metrics
- Monitor `MessagesSent`, `CampaignMetrics`, and active `Endpoint` counts.

### C. AWS Config / Trusted Advisor
- Audit for unused Pinpoint projects, applications, and orphaned segments.

### D. Company Policies
- Data retention policies for user endpoints and internal sunset migration timelines.

### E. IaC (Optional)
- Terraform/CloudFormation templates to identify hardcoded Pinpoint API references requiring migration.

---
## Output Schema
### Finding Record (JSON)
```json
[
  {
    "finding_id": "PINPOINT-WST-001",
    "category": "Waste Elimination",
    "name": "Prune Inactive Endpoints",
    "description": "Delete device tokens and endpoints inactive for 90+ days to avoid $1.20/1k MTU charges.",
    "impact": "High",
    "effort": "Low"
  },
  {
    "finding_id": "PINPOINT-WST-002",
    "category": "Waste Elimination",
    "name": "Remove Bounced Emails",
    "description": "Scrub hard bounces and unverified addresses from target segments.",
    "impact": "Medium",
    "effort": "Low"
  },
  {
    "finding_id": "PINPOINT-WST-003",
    "category": "Waste Elimination",
    "name": "Delete Unused Projects",
    "description": "Remove legacy Pinpoint applications that accumulate endpoint data.",
    "impact": "High",
    "effort": "Low"
  },
  {
    "finding_id": "PINPOINT-WST-004",
    "category": "Waste Elimination",
    "name": "Pause Inactive Campaigns",
    "description": "Halt journeys and campaigns with negligible engagement.",
    "impact": "Medium",
    "effort": "Low"
  },
  {
    "finding_id": "PINPOINT-WST-005",
    "category": "Waste Elimination",
    "name": "Clean Up Segments",
    "description": "Delete orphaned or overlapping segments.",
    "impact": "Low",
    "effort": "Low"
  },
  {
    "finding_id": "PINPOINT-RGT-001",
    "category": "Rightsizing",
    "name": "Consolidate User IDs",
    "description": "Map multiple devices to a single user ID to prevent duplicate MTU billing.",
    "impact": "High",
    "effort": "Medium"
  },
  {
    "finding_id": "PINPOINT-RGT-002",
    "category": "Rightsizing",
    "name": "Rightsize Event Payloads",
    "description": "Minimize custom event payload attributes to reduce processing overhead.",
    "impact": "Medium",
    "effort": "Low"
  },
  {
    "finding_id": "PINPOINT-RGT-003",
    "category": "Rightsizing",
    "name": "Filter Client Events",
    "description": "Drop non-essential analytics events on mobile devices before transmission.",
    "impact": "Medium",
    "effort": "Medium"
  },
  {
    "finding_id": "PINPOINT-RGT-004",
    "category": "Rightsizing",
    "name": "Optimize Send Frequency",
    "description": "Cap the frequency of messages per user per month to reduce volume.",
    "impact": "Medium",
    "effort": "Low"
  },
  {
    "finding_id": "PINPOINT-CMT-001",
    "category": "Commitment Discounts",
    "name": "Maximize Free Tier",
    "description": "Ensure usage falls within 5k MTU and 1M push limits where possible.",
    "impact": "Low",
    "effort": "Low"
  },
  {
    "finding_id": "PINPOINT-ARC-001",
    "category": "Architecture Changes",
    "name": "Migrate Push/SMS",
    "description": "Transition transactional messaging to AWS End User Messaging ahead of 2026 sunset.",
    "impact": "High",
    "effort": "High"
  },
  {
    "finding_id": "PINPOINT-ARC-002",
    "category": "Architecture Changes",
    "name": "Migrate Emails to SES",
    "description": "Move email marketing directly to Amazon SES.",
    "impact": "High",
    "effort": "Medium"
  },
  {
    "finding_id": "PINPOINT-ARC-003",
    "category": "Architecture Changes",
    "name": "Migrate Journeys",
    "description": "Transition segment orchestration to Amazon Connect Outbound Campaigns.",
    "impact": "High",
    "effort": "High"
  },
  {
    "finding_id": "PINPOINT-ARC-004",
    "category": "Architecture Changes",
    "name": "Migrate Analytics",
    "description": "Decouple mobile event collection and move to Amazon Kinesis.",
    "impact": "Medium",
    "effort": "High"
  },
  {
    "finding_id": "PINPOINT-SCH-001",
    "category": "Scheduling & Auto-Scaling",
    "name": "Pre-Billing Pruning",
    "description": "Run endpoint cleanup scripts prior to month-end to lower the next month's MTUs.",
    "impact": "High",
    "effort": "Low"
  },
  {
    "finding_id": "PINPOINT-SCH-002",
    "category": "Scheduling & Auto-Scaling",
    "name": "Implement Quiet Hours",
    "description": "Suppress message delivery during off-peak hours to improve engagement ROI.",
    "impact": "Medium",
    "effort": "Low"
  },
  {
    "finding_id": "PINPOINT-SCH-003",
    "category": "Scheduling & Auto-Scaling",
    "name": "Batch Segment Imports",
    "description": "Upload segment data in scheduled batches rather than real-time streams.",
    "impact": "Low",
    "effort": "Medium"
  },
  {
    "finding_id": "PINPOINT-PRC-001",
    "category": "Pricing Model Optimization",
    "name": "Shift SMS to Push",
    "description": "Favor $1/Million Push Notifications over expensive per-message SMS rates.",
    "impact": "High",
    "effort": "Medium"
  },
  {
    "finding_id": "PINPOINT-NET-001",
    "category": "Network & Data Transfer Optimization",
    "name": "Use VPC Endpoints",
    "description": "Use VPC endpoints for backend Pinpoint API calls to bypass NAT Gateway costs.",
    "impact": "Medium",
    "effort": "Low"
  }
]
```

### Summary Report Table
| Finding ID | Category | Strategy Name | Impact | Effort |
|------------|----------|---------------|--------|--------|
| PINPOINT-WST-001 | Waste Elimination | Prune Inactive Endpoints | High | Low |
| PINPOINT-WST-002 | Waste Elimination | Remove Bounced Emails | Medium | Low |
| PINPOINT-WST-003 | Waste Elimination | Delete Unused Projects | High | Low |
| PINPOINT-WST-004 | Waste Elimination | Pause Inactive Campaigns | Medium | Low |
| PINPOINT-WST-005 | Waste Elimination | Clean Up Segments | Low | Low |
| PINPOINT-RGT-001 | Rightsizing | Consolidate User IDs | High | Medium |
| PINPOINT-RGT-002 | Rightsizing | Rightsize Event Payloads | Medium | Low |
| PINPOINT-RGT-003 | Rightsizing | Filter Client Events | Medium | Medium |
| PINPOINT-RGT-004 | Rightsizing | Optimize Send Frequency | Medium | Low |
| PINPOINT-CMT-001 | Commitment Discounts| Maximize Free Tier | Low | Low |
| PINPOINT-ARC-001 | Architecture Changes| Migrate Push/SMS | High | High |
| PINPOINT-ARC-002 | Architecture Changes| Migrate Emails to SES | High | Medium |
| PINPOINT-ARC-003 | Architecture Changes| Migrate Journeys | High | High |
| PINPOINT-ARC-004 | Architecture Changes| Migrate Analytics | Medium | High |
| PINPOINT-SCH-001 | Scheduling & Auto-Scaling| Pre-Billing Pruning | High | Low |
| PINPOINT-SCH-002 | Scheduling & Auto-Scaling| Implement Quiet Hours | Medium | Low |
| PINPOINT-SCH-003 | Scheduling & Auto-Scaling| Batch Segment Imports | Low | Medium |
| PINPOINT-PRC-001 | Pricing Model Optimization| Shift SMS to Push | High | Medium |
| PINPOINT-NET-001 | Network & Data Transfer| Use VPC Endpoints | Medium | Low |
