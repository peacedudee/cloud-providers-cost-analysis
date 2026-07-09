# Cost-Cutting Playbook: AWS Audit Manager
> **Companion File:** [audit_manager.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/audit_manager/audit_manager.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Audit Manager was designed to automate evidence collection for compliance standards, but entered maintenance mode and stopped accepting new customers on April 30, 2026. Because it operates on a pay-as-you-go model tied to resource assessment counts ($1.25 per resource-framework-month), the key to cost optimization is aggressive scope reduction and a planned migration to AWS Security Hub. This playbook provides strategies for minimizing costs on legacy Audit Manager deployments by pruning redundant frameworks, limiting scope strictly to production workloads, and transitioning continuous monitoring to more cost-effective alternatives.

## Strategy Categories

### 1. Waste Elimination

#### 1. AUDIT-001: Pause Assessments Post-Audit Completion
- **What:** Pause Audit Manager assessments as soon as the formal audit or reporting period concludes.
- **Why It Saves Money:** Audit Manager charges $1.25 per resource-framework-month continuously. Pausing stops evidence collection and halts the recurring charges immediately.
- **Implementation Steps:** 
  1. Navigate to Audit Manager console.
  2. Select "Assessments".
  3. Identify assessments for completed audits.
  4. Change status from "Active" to "Inactive".
- **Estimated Savings:** 100% of the cost for the paused assessment.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Compliance Team
- **Prerequisites:** Confirmation that the audit period is complete and evidence is exported.

#### 2. AUDIT-002: Deactivate Overlapping Assessment Frameworks
- **What:** Prevent assessing the same resources against multiple overlapping frameworks (e.g., SOC 2, ISO 27001, NIST) simultaneously if not strictly required.
- **Why It Saves Money:** Enabling 4 frameworks on 100 resources costs $500/month (4 * 100 * $1.25). Consolidating or running them sequentially reduces the multiplier effect.
- **Implementation Steps:**
  1. Review active frameworks.
  2. Identify overlapping compliance requirements.
  3. Deactivate redundant frameworks and rely on a primary framework.
- **Estimated Savings:** 50-75% depending on the number of overlapping frameworks.
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Compliance Team
- **Prerequisites:** Understanding of compliance mapping across frameworks.

#### 3. AUDIT-003: Exclude Non-Production Accounts from Scope
- **What:** Ensure that sandbox, development, and testing AWS accounts are not included in the Audit Manager assessment scope.
- **Why It Saves Money:** Prevents the $1.25/resource-month fee from being applied to thousands of irrelevant non-production resources.
- **Implementation Steps:**
  1. Review the AWS accounts targeted in the assessment scope.
  2. Remove non-production account IDs from the assessment configuration.
- **Estimated Savings:** 30-60% of total assessment costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | Compliance Team
- **Prerequisites:** AWS Organizations account structure separating Prod from Non-Prod.

#### 4. AUDIT-004: Exclude Ephemeral Resources via Tags
- **What:** Exclude short-lived resources (e.g., Spot Instances, EMR clusters, temporary ECS tasks) from automated evidence collection.
- **Why It Saves Money:** Avoids incurring a full month's prorated charge or generating excessive evidence for resources that exist only for hours.
- **Implementation Steps:**
  1. Edit the assessment scope.
  2. Apply tag-based filters (e.g., exclude `Lifecycle=Ephemeral` or include only `Environment=Production`).
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Consistent resource tagging strategy.

#### 5. AUDIT-005: Disable Evidence Collection for Out-of-Scope Regions
- **What:** Restrict the assessment to only the AWS regions where in-scope compliance workloads reside.
- **Why It Saves Money:** Prevents Audit Manager from polling and billing for baseline resources (like default VPCs) in unused regions.
- **Implementation Steps:**
  1. Go to Audit Manager settings.
  2. Review the list of enabled regions.
  3. Disable regions that do not host compliance-scoped workloads.
- **Estimated Savings:** 5-20%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | Compliance Team
- **Prerequisites:** Clear definition of regional compliance boundaries.

#### 6. AUDIT-006: Delete Unused Custom Assessment Frameworks
- **What:** Remove custom assessment frameworks that were created for testing or are no longer in use.
- **Why It Saves Money:** Ensures that nobody accidentally activates an obsolete custom framework that would trigger resource assessment charges.
- **Implementation Steps:**
  1. Navigate to Framework library -> Custom frameworks.
  2. Identify custom frameworks with no active assessments.
  3. Delete them.
- **Estimated Savings:** Preventive (Variable)
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Compliance Team
- **Prerequisites:** Verification that the custom framework is obsolete.

#### 7. AUDIT-007: Clean Up Stale Evidence in S3 Buckets
- **What:** Delete Audit Manager evidence reports and raw evidence data stored in S3 that exceeds mandatory retention periods.
- **Why It Saves Money:** Reduces S3 Standard storage costs associated with accumulating JSON and PDF evidence files.
- **Implementation Steps:**
  1. Identify the designated S3 evidence bucket for Audit Manager.
  2. Implement an S3 Lifecycle policy to delete objects older than the required retention period (e.g., 2-3 years).
- **Estimated Savings:** 5-10% of ancillary S3 storage costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Approval from legal/compliance on retention requirements.

### 2. Rightsizing

#### 8. AUDIT-008: Restrict Scope to Specifically Targeted Resource Types
- **What:** Instead of assessing "All resources", configure assessments to only evaluate the specific resource types mandated by the audit (e.g., EC2, RDS, S3).
- **Why It Saves Money:** Eliminates the $1.25/month charge for evaluating low-risk or irrelevant resource types (e.g., API Gateway, SNS topics) that don't need evidence.
- **Implementation Steps:**
  1. Edit the assessment.
  2. Under "AWS Services", select only the specific services in scope.
- **Estimated Savings:** 20-50%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps | Compliance Team
- **Prerequisites:** Detailed mapping of compliance requirements to AWS services.

#### 9. AUDIT-009: Consolidate Similar Frameworks into a Custom Framework
- **What:** Build a single custom framework that merges the requirements of multiple standards (e.g., SOC 2 and ISO 27001) using common controls.
- **Why It Saves Money:** Running 1 custom framework costs $1.25/resource-month. Running 2 standard frameworks costs $2.50/resource-month. Consolidating halves the cost.
- **Implementation Steps:**
  1. Identify overlapping controls between two frameworks.
  2. Create a Custom Framework mapping these common controls.
  3. Launch a single assessment using the Custom Framework.
- **Estimated Savings:** ~50%
- **Risk Level:** Medium
- **Implementation Scope:** Compliance Team
- **Prerequisites:** Compliance expertise to map controls accurately.

#### 10. AUDIT-010: Deregister Inactive Delegated Administrators
- **What:** Remove delegated administrator accounts in AWS Organizations if they are no longer actively managing Audit Manager.
- **Why It Saves Money:** Reduces the risk of rogue or unmonitored assessments being launched in child accounts, which can silently drive up costs.
- **Implementation Steps:**
  1. Go to AWS Organizations -> Services -> Audit Manager.
  2. Deregister stale administrator accounts.
- **Estimated Savings:** Preventive (Variable)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | Security Team
- **Prerequisites:** Administrative access to AWS Organizations.

### 3. Commitment Discounts

#### 11. AUDIT-011: Implement S3 Lifecycle Rules on Evidence Buckets
- **What:** Move Audit Manager evidence data to cheaper S3 storage tiers (Standard-IA or Glacier) for long-term retention.
- **Why It Saves Money:** Standard S3 pricing is ~$0.023/GB, while Glacier Flexible Retrieval is ~$0.0036/GB, representing an 84% savings on long-term storage of audit logs.
- **Implementation Steps:**
  1. Create an S3 lifecycle rule on the Audit Manager evidence bucket.
  2. Transition objects to Standard-IA after 30 days.
  3. Transition to Glacier after 90 days.
- **Estimated Savings:** 80-90% on ancillary storage costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of evidence retrieval SLAs.

### 4. Architecture Changes

#### 12. AUDIT-012: Migrate Continuous Compliance Monitoring to AWS Security Hub
- **What:** Transition away from Audit Manager entirely for continuous compliance monitoring, utilizing AWS Security Hub instead.
- **Why It Saves Money:** Audit Manager costs $1.25 per resource-month. Security Hub charges $0.0010 per control check. Security Hub is roughly 80-90% cheaper for continuous posture monitoring.
- **Implementation Steps:**
  1. Enable AWS Security Hub.
  2. Activate corresponding security standards (e.g., CIS, PCI DSS).
  3. Deactivate continuous assessments in Audit Manager.
  4. Use Security Hub for daily posture and manual exports for point-in-time formal evidence collection.
- **Estimated Savings:** 80-90%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | Security Team
- **Prerequisites:** Security Hub adoption and configuration.

#### 13. AUDIT-013: Centralize Audit Manager via AWS Organizations
- **What:** Use a single delegated administrator account to manage assessments across the organization instead of managing them per-account.
- **Why It Saves Money:** Centralization prevents duplicative assessments from being run by different account owners, reducing redundant $1.25/resource fees.
- **Implementation Steps:**
  1. Enable Audit Manager integration with AWS Organizations.
  2. Designate a single compliance/security account as the delegated admin.
  3. Disable local Audit Manager access in member accounts.
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** Security Team | DevOps
- **Prerequisites:** AWS Organizations enabled with all features.

#### 14. AUDIT-014: Replace Custom Controls with AWS Config Rules
- **What:** For non-standard requirements, use AWS Config Rules instead of Audit Manager Custom Controls.
- **Why It Saves Money:** AWS Config charges per evaluation ($0.001) which is often cheaper than Audit Manager's flat monthly resource fee if the resource doesn't change frequently.
- **Implementation Steps:**
  1. Identify custom controls in Audit Manager.
  2. Replicate the logic using AWS Config custom rules.
  3. Remove the custom controls from Audit Manager.
- **Estimated Savings:** 20-40% for specific custom workflows.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS Config enabled.

#### 15. AUDIT-015: Offload Evidence Storage to Amazon S3 Glacier Deep Archive
- **What:** For compliance standards requiring 7+ years of evidence retention, route historical Audit Manager assessment reports directly to Glacier Deep Archive.
- **Why It Saves Money:** Deep Archive costs ~$0.00099/GB, minimizing the cost of mandatory legal holds on compliance data.
- **Implementation Steps:**
  1. Modify S3 lifecycle policy to target Glacier Deep Archive for objects > 1 year old.
- **Estimated Savings:** 95% on long-term storage costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Acceptance of 12-48 hour retrieval times for ancient evidence.

### 5. Scheduling & Auto-Scaling

#### 16. AUDIT-016: Activate Assessments Only During Formal Audit Windows
- **What:** Run Audit Manager strictly during the official audit observation period (e.g., Q3) rather than 365 days a year.
- **Why It Saves Money:** Running an assessment for 3 months instead of 12 reduces the annual cost of the framework by 75%.
- **Implementation Steps:**
  1. Define the formal audit window.
  2. Change assessment status to "Active" on Day 1 of the window.
  3. Change status to "Inactive" on the last day.
- **Estimated Savings:** 75%
- **Risk Level:** Medium
- **Implementation Scope:** Compliance Team
- **Prerequisites:** Auditor agreement that point-in-time or windowed evidence is acceptable.

#### 17. AUDIT-017: Automate Pausing of Assessments via AWS Lambda
- **What:** Create a Lambda function to automatically pause Audit Manager assessments outside of business hours or formal testing windows.
- **Why It Saves Money:** Because Audit Manager is prorated daily, turning it off on weekends or off-weeks immediately cuts prorated costs.
- **Implementation Steps:**
  1. Write a Lambda function utilizing the `UpdateAssessmentStatus` API.
  2. Trigger via EventBridge cron schedule.
- **Estimated Savings:** 10-30%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** IAM roles permitting Lambda to manage Audit Manager.

### 6. Pricing Model Optimization

#### 18. AUDIT-018: Transition Fully to AWS Security Hub
- **What:** Treat Audit Manager as deprecated and execute a full migration of continuous posture reporting to AWS Security Hub.
- **Why It Saves Money:** Fundamentally changes the pricing model from per-resource-month ($1.25) to per-check ($0.0010), representing massive architectural savings.
- **Implementation Steps:**
  1. Review legacy Audit Manager setup.
  2. Map frameworks to Security Hub standards.
  3. Turn off Audit Manager entirely.
- **Estimated Savings:** >80%
- **Risk Level:** High
- **Implementation Scope:** Security Team | Procurement/Leadership
- **Prerequisites:** Stakeholder alignment on the new tooling.

#### 19. AUDIT-019: Manual Export vs. Continuous Collection
- **What:** For very small or low-risk workloads, manually export AWS Config and CloudTrail data instead of paying Audit Manager to automatically format it.
- **Why It Saves Money:** Bypasses the $1.25/resource fee entirely by using existing operational tools.
- **Implementation Steps:**
  1. Define manual evidence collection runbooks.
  2. Deactivate Audit Manager.
- **Estimated Savings:** 100% of Audit Manager costs.
- **Risk Level:** High (Increases manual toil)
- **Implementation Scope:** Compliance Team
- **Prerequisites:** Willingness to absorb manual labor costs.

### 7. Network & Data Transfer Optimization

#### 20. AUDIT-020: Colocate S3 Evidence Destination Buckets with the Audit Manager Region
- **What:** Ensure that the S3 bucket designated to receive Audit Manager evidence is in the same AWS Region where Audit Manager is operating.
- **Why It Saves Money:** Avoids AWS cross-region data transfer fees ($0.01 - $0.02 per GB) when Audit Manager writes evidence files and assessment reports to S3.
- **Implementation Steps:**
  1. Check Audit Manager settings for the destination S3 bucket.
  2. Verify the bucket's region matches the active Audit Manager region.
  3. Recreate the bucket in the local region if necessary.
- **Estimated Savings:** Variable (depends on evidence volume)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** S3 bucket creation permissions.

---
## Cross-Service Synergies
- **AWS Security Hub:** The primary successor for continuous compliance monitoring. Transitioning here reduces costs dramatically.
- **AWS Config:** Audit Manager relies on Config for underlying evidence. Optimizing Config recording scopes (e.g., avoiding recording ephemeral data) inherently improves Audit Manager efficiency.
- **Amazon S3:** Proper lifecycle management of evidence buckets directly limits ancillary storage costs.
- **AWS Organizations:** Centralized management prevents duplicative shadow-assessments.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query `line_item_product_code` = `AWSAuditManager`
- Look for `line_item_usage_type` containing `ResourceAssessment` to identify high-cost regions and accounts.
### B. CloudWatch Metrics
- Not heavily applicable to Audit Manager pricing, but useful for tracking S3 bucket sizes.
### C. AWS Config / Trusted Advisor
- Use AWS Config to ensure resources are properly tagged (e.g., `Environment=Production`) to enable tight Audit Manager scoping.
### D. Company Policies
- Confirm audit schedules, regulatory frameworks in scope, and mandatory evidence retention periods.
### E. IaC (Optional)
- Terraform/CloudFormation templates defining the Audit Manager assessments to quickly identify and modify scope definitions.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "AUDIT-001",
  "service": "AWS Audit Manager",
  "category": "Waste Elimination",
  "name": "Pause Assessments Post-Audit Completion",
  "description": "Pause Audit Manager assessments as soon as the formal audit or reporting period concludes to halt recurring daily charges.",
  "estimated_savings_percentage": 100,
  "risk_level": "Low"
}
```

### Summary Report Table

| ID | Category | Strategy | Est. Savings | Risk |
|---|---|---|---|---|
| AUDIT-001 | Waste Elimination | Pause Assessments Post-Audit Completion | 100% | Low |
| AUDIT-002 | Waste Elimination | Deactivate Overlapping Assessment Frameworks | 50-75% | Medium |
| AUDIT-003 | Waste Elimination | Exclude Non-Production Accounts from Scope | 30-60% | Low |
| AUDIT-004 | Waste Elimination | Exclude Ephemeral Resources via Tags | 10-30% | Low |
| AUDIT-005 | Waste Elimination | Disable Evidence Collection for Out-of-Scope Regions | 5-20% | Low |
| AUDIT-006 | Waste Elimination | Delete Unused Custom Assessment Frameworks | Variable | Low |
| AUDIT-007 | Waste Elimination | Clean Up Stale Evidence in S3 Buckets | 5-10% | Medium |
| AUDIT-008 | Rightsizing | Restrict Scope to Specifically Targeted Resource Types | 20-50% | Medium |
| AUDIT-009 | Rightsizing | Consolidate Similar Frameworks into a Custom Framework | ~50% | Medium |
| AUDIT-010 | Rightsizing | Deregister Inactive Delegated Administrators | Variable | Low |
| AUDIT-011 | Commitment Discounts | Implement S3 Lifecycle Rules on Evidence Buckets | 80-90% | Low |
| AUDIT-012 | Architecture Changes | Migrate Continuous Compliance Monitoring to AWS Security Hub | 80-90% | High |
| AUDIT-013 | Architecture Changes | Centralize Audit Manager via AWS Organizations | 10-30% | Low |
| AUDIT-014 | Architecture Changes | Replace Custom Controls with AWS Config Rules | 20-40% | Medium |
| AUDIT-015 | Architecture Changes | Offload Evidence Storage to Amazon S3 Glacier Deep Archive | 95% | Low |
| AUDIT-016 | Scheduling & Auto-Scaling | Activate Assessments Only During Formal Audit Windows | 75% | Medium |
| AUDIT-017 | Scheduling & Auto-Scaling | Automate Pausing of Assessments via AWS Lambda | 10-30% | High |
| AUDIT-018 | Pricing Model Optimization | Transition Fully to AWS Security Hub | >80% | High |
| AUDIT-019 | Pricing Model Optimization | Manual Export vs. Continuous Collection | 100% | High |
| AUDIT-020 | Network & Data Transfer | Colocate S3 Evidence Destination Buckets with the Audit Manager Region | Variable | Low |
