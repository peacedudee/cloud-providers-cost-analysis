# Cost-Cutting Playbook: AWS Snowball
> **Companion File:** [snowball.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/snowball/snowball.md)
> **Last Updated:** July 2026

---

## Executive Summary
AWS Snowball Edge is a physical data migration and edge computing device designed for petabyte-scale data transfers and offline edge processing. Pricing consists of a flat one-time service fee per job, daily extension fees (applied after the first 10 free onsite days), shipping costs, and data egress fees (if exporting data out of AWS). The primary focus of cost optimization for Snowball is operational efficiency: minimizing the time the device spends onsite by pre-staging data, rightsizing the device type (Storage vs. Compute optimized), consolidating migration jobs to save on service fees, and comparing the $0.03/GB egress rate against alternative transfer methods like AWS DataSync or Direct Connect.

## Strategy Categories

### 1. Waste Elimination
- **SNOWBALL-WST-001: Pre-stage Local Data:** Ensure all on-premises data is fully pre-staged, accessible, and ready for transfer before ordering the device to prevent it from sitting idle and accumulating daily extension fees after the 10 free days.
- **SNOWBALL-WST-002: Immediate Device Return:** Ship the device back to AWS immediately upon data copy completion. Do not hold the device for prolonged secondary verifications if they can be avoided.
- **SNOWBALL-WST-003: Avoid Lost/Damaged Fees:** Ensure the device is properly handled, packed in its original ruggedized container, and the e-ink shipping label is preserved to avoid physical replacement penalties (up to $20,000+).
- **SNOWBALL-WST-004: Cancel Unused Jobs:** Regularly audit pending Snowball orders and cancel jobs that are no longer necessary before they are provisioned and shipped to avoid the $300 one-time job fee.

### 2. Rightsizing
- **SNOWBALL-RSZ-001: Consolidate Migration Jobs:** Group smaller data migrations into a single Snowball Edge job (up to 80TB or 210TB) rather than ordering multiple devices, saving $300 per avoided job plus shipping costs.
- **SNOWBALL-RSZ-002: Avoid Unnecessary Compute-Optimized Devices:** If you are only migrating data into S3, order the Storage-Optimized device. Compute-Optimized devices have higher daily extension fees ($40/day vs $30/day).
- **SNOWBALL-RSZ-003: Downgrade from 210TB to 80TB:** If your dataset is well under 80TB, order the 80TB device. While the upfront fee is the same ($300), the 210TB device incurs a higher daily extension fee ($50/day vs $30/day).
- **SNOWBALL-RSZ-004: Avoid GPU Devices if Unneeded:** Do not select the Compute-Optimized with GPU option unless you explicitly need local GPU processing at the edge, as it incurs the highest daily extension fee ($70/day).
- **SNOWBALL-RSZ-005: Rightsize to AWS Snowcone:** For small edge data transfer needs (under 8TB), use AWS Snowcone instead of Snowball Edge to reduce upfront service fees and shipping logistics.

### 3. Commitment Discounts
- **SNOWBALL-COM-001: Long-Term Edge Commitments:** If you are deploying Snowball Edge devices for permanent, continuous edge computing workloads (not just one-off data migrations), purchase 1-year or 3-year upfront commitments to significantly discount the daily rental rates.

### 4. Architecture Changes
- **SNOWBALL-ARC-001: Shift Small Migrations to DataSync:** For datasets under 10-20TB where high-bandwidth internet or Direct Connect is available, use AWS DataSync instead of Snowball to eliminate physical logistics, shipping, and $300 job fees.
- **SNOWBALL-ARC-002: Upgrade On-Premises Network:** Ensure local networking infrastructure supports 10G or 25G speeds (SFP+ or RJ45) before the Snowball arrives to maximize transfer speed and reduce billable onsite days.
- **SNOWBALL-ARC-003: Edge Data Compression:** For edge computing workloads, compress or filter raw telemetry/IoT data locally on the Snowball Edge before returning it to AWS. This maximizes the device's storage capacity and reduces subsequent S3 storage costs.
- **SNOWBALL-ARC-004: Archive Small Files:** Millions of small files cause significant transfer overhead. Archive (TAR/ZIP) small files into larger chunks before transferring them to the Snowball to dramatically speed up copy times and prevent daily extension fees.

### 5. Scheduling & Auto-Scaling
- **SNOWBALL-SCH-001: Align Delivery with IT Availability:** Schedule Snowball deliveries so they do not arrive right before long weekends, holidays, or during staff shortages, which wastes the 10 free onsite days.
- **SNOWBALL-SCH-002: Automate 24/7 Transfers:** Use automated, robust scripts (like `rsync`, AWS CLI, or AWS OpsHub) that run continuously overnight and over weekends to ensure the data transfer completes within the 10 free days.

### 6. Pricing Model Optimization
- **SNOWBALL-PRC-001: Evaluate Data Export (Egress) Costs:** When exporting data *out* of AWS, Snowball incurs a $0.03/GB egress fee. Calculate if this fee (e.g., $3,000 for 100TB) is cheaper than using AWS DataSync over existing internet or Direct Connect lines.
- **SNOWBALL-PRC-002: Negotiate Bulk Migration Pricing:** For massive multi-petabyte migrations requiring dozens of Snowball Edge devices, engage your AWS Account Manager to negotiate a bulk migration discount or explore AWS Snowmobile.

### 7. Network & Data Transfer Optimization
- **SNOWBALL-NET-001: Parallelize Checksum Validation:** Run local checksum validation scripts in parallel with the data transfer rather than waiting until the entire copy is finished sequentially to save precious onsite days.
- **SNOWBALL-NET-002: Optimize Transfer Concurrency:** Use multi-part uploads and highly concurrent copy mechanisms to saturate the Snowball's network interface, minimizing total transfer time.

---

## Cross-Service Synergies
- **Amazon S3:** The ultimate destination or source for most Snowball jobs. Storage costs apply once data lands in S3.
- **AWS DataSync:** The primary online alternative to Snowball for migrations where network bandwidth is sufficient.
- **AWS Direct Connect:** If high-capacity dedicated networking is available, Snowball might be unnecessary for both ingress and egress.
- **Amazon EC2 / AWS Lambda:** Edge computing components that run locally on Compute-Optimized Snowball devices.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
- `lineItem/ProductCode` (e.g., `AWSSnowball`)
- `lineItem/UsageType` (to identify job fees, extension days, and egress fees)
- `lineItem/UnblendedCost`
- `resourceId` (to identify specific Snowball jobs)

### B. CloudWatch Metrics
- *N/A directly for device physical tracking*, but edge compute metrics may be available if running EC2 on Snowball Edge.

### C. AWS Config / Trusted Advisor
- *N/A for physical Snowball device tracking.*

### D. Company Policies
- Data migration deadlines and bandwidth availability assessments.
- Local IT staff scheduling and holiday calendars.
- Hardware handling and shipping protocols.

### E. IaC (Optional)
- Terraform/CloudFormation templates that provision the S3 buckets acting as the destination for the Snowball data, ensuring proper lifecycle policies are applied immediately upon data ingestion.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "SNOWBALL-RSZ-001",
  "resource_id": "job-0a1b2c3d4e5f",
  "account_id": "123456789012",
  "region": "us-east-1",
  "finding_type": "Rightsizing",
  "severity": "Medium",
  "description": "Multiple small Snowball Edge jobs ordered within the same month instead of consolidating into a single 80TB device.",
  "recommendation": "Consolidate migrations to avoid multiple $300 job fees and shipping costs.",
  "estimated_monthly_savings": 300.00,
  "effort_level": "Low"
}
```

### Summary Report Table
| Finding ID | Category | Description | Resource ID | Estimated Savings | Effort Level |
|------------|----------|-------------|-------------|-------------------|--------------|
| SNOWBALL-WST-001 | Waste Elimination | Pre-stage data to avoid onsite extension fees ($30/day). | job-* | $300/job | Medium |
| SNOWBALL-RSZ-001 | Rightsizing | Consolidate multiple small migrations. | N/A | $300/job | Low |
| SNOWBALL-ARC-001 | Architecture Changes | Use AWS DataSync for < 20TB migrations. | N/A | Varies | Medium |
| SNOWBALL-SCH-001 | Scheduling | Avoid deliveries before holidays/weekends. | N/A | $60-$90/job | Low |
