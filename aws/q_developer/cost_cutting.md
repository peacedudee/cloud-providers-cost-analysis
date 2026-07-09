# Cost-Cutting Playbook: Amazon Q Developer
> **Companion File:** [q_developer.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/q_developer/q_developer.md)
> **Last Updated:** July 2026

---

## Executive Summary
Amazon Q Developer (formerly CodeWhisperer) is billed on a per-user licensing model, making cost optimization primarily an exercise in seat management, license utilization tracking, and tier matching. While the Free Tier (Individual) is sufficient for many independent developers and contractors, the Pro Tier ($19.00/user/month) unlocks enterprise SSO, advanced agents, and Code Transformation features. This playbook outlines 20 actionable strategies to eliminate inactive seats, prevent duplicate AI tool subscriptions, and implement just-in-time licensing to ensure you only pay for active, value-generating usage of Amazon Q Developer.

---

## Strategy Categories

### 1. Waste Elimination
*   **[QDEV-WE-001] De-provision Inactive Seats:** Conduct monthly audits to identify and remove Q Developer Pro licenses from users who have not generated a code suggestion or logged into the IDE extension in the last 30 days, saving $19.00/month per user.
*   **[QDEV-WE-002] Eliminate Duplicate Subscriptions:** Audit developers' environments to ensure they are not concurrently licensed for multiple premium AI coding assistants (e.g., both Q Developer Pro and GitHub Copilot), avoiding doubled tooling costs.
*   **[QDEV-WE-003] Restrict Open License Enrollment:** Prevent users from self-provisioning Pro licenses by locking down the IAM Identity Center application. Require management approval to join the designated `Q-Developer-Pro-Users` SSO group.
*   **[QDEV-WE-004] Revoke Licenses on Offboarding:** Integrate IAM Identity Center groups with your HRIS or Identity Provider (IdP) to instantly revoke Q Developer Pro access when an employee leaves the company or transfers to a non-technical role.
*   **[QDEV-WE-005] Enforce Free Tier for External Staff:** Mandate that temporary contractors, interns, or developers working on non-proprietary sandbox projects use their own AWS Builder IDs on the Free Individual Tier.
*   **[QDEV-WE-006] Centralized License Recovery:** Implement automated workflows to periodically ping users with active Pro licenses asking them to confirm continued need; if unconfirmed, automatically remove them from the IAM group.

### 2. Rightsizing
*   **[QDEV-RS-001] Role-Based Tier Matching:** Assign the Pro Tier strictly to core engineers who require enterprise SSO, custom repository scanning, and advanced agents. Assign the Free Tier to adjacent roles (e.g., DevOps, QA, Data Analysts) who only need basic code completions.
*   **[QDEV-RS-002] Selective Departmental Rollout:** Avoid purchasing organization-wide licenses. Instead, provision seats team-by-team based on demonstrated ROI and active development cycles.
*   **[QDEV-RS-003] Downgrade Low-Frequency Users:** Identify users who consistently generate fewer than 10 lines of AI code per week and downgrade them to the Free Tier.

### 3. Commitment Discounts
*   **[QDEV-CD-001] Leverage AWS EDP:** While Q Developer does not offer Reserved Instances or Savings Plans, the monthly $19.00 subscription fee contributes to overall AWS spend. Ensure these charges are included and discounted under your AWS Enterprise Discount Program (EDP).

### 4. Architecture Changes
*   **[QDEV-AC-001] Centralize IAM Identity Center Management:** Ensure Q Developer Pro is managed via a centralized AWS Organization and a single IAM Identity Center instance to prevent fragmented, unmanaged license purchases across multiple isolated AWS accounts.
*   **[QDEV-AC-002] Consolidate Customization Repositories:** When configuring Q Developer Customization to scan internal codebases, point it only to authoritative, high-quality repositories rather than indexing duplicate or legacy code, streamlining the learning process and avoiding administrative overhead.
*   **[QDEV-AC-003] Standardize IDE Extensions:** Standardize the deployment of the Q Developer extension via corporate MDM (Mobile Device Management) to ensure all developers use the same version, reducing IT support costs.

### 5. Scheduling & Auto-Scaling
*   **[QDEV-SAS-001] Just-in-Time Licensing for Code Transformation:** Treat Code Transformation (e.g., Java 8 to Java 17 upgrades) as a project-based activity. Provision Pro licenses for the specific engineering squad during the migration sprint, and revoke them upon completion.
*   **[QDEV-SAS-002] Automated License Reclamation Scripts:** Deploy AWS Lambda functions that periodically pull IAM Identity Center usage reports and automatically remove inactive users from the designated Q Developer provisioning group.
*   **[QDEV-SAS-003] Request-Driven Access Workflows:** Utilize ITSM tools (like ServiceNow or Jira Service Desk) to grant Pro licenses with a built-in expiration date (e.g., 90 days). The license auto-expires unless the developer explicitly requests an extension.

### 6. Pricing Model Optimization
*   **[QDEV-PMO-001] Implement Departmental Chargebacks:** Use AWS Cost Allocation Tags on the IAM Identity Center application to charge the $19.00/month fee directly to the specific cost centers of the engineering teams, incentivizing managers to maintain lean seat counts.
*   **[QDEV-PMO-002] Optimize Onboarding Proration:** Take advantage of subscription proration by provisioning licenses exactly when the developer needs them in the month, rather than bulk provisioning on the 1st of the month if they won't start coding until the 15th.
*   **[QDEV-PMO-003] Conduct Quarterly ROI Analysis:** Track developer velocity and PR acceptance rates against Q Developer Pro costs per team. Defund licenses for teams that show no measurable productivity improvement.

### 7. Network & Data Transfer Optimization
*   **[QDEV-NDT-001] Local Network Extension Caching:** Distribute IDE extension updates for Q Developer via an internal artifact repository (e.g., JFrog Artifactory) to reduce external internet bandwidth consumption across hundreds of developer workstations.

---

## Cross-Service Synergies
*   **IAM Identity Center:** Crucial for enforcing centralized SSO control and preventing rogue Q Developer Pro sign-ups across isolated AWS accounts.
*   **AWS Cost Explorer / AWS Billing:** Use Cost Categories and tagging to separate Q Developer seat charges by department or project.
*   **AWS Lambda / EventBridge:** Use serverless functions to automate the auditing and revocation of inactive Q Developer Pro licenses based on usage metrics.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
*   `lineItem/ProductCode` = `AmazonQDeveloper`
*   `lineItem/Operation` indicating Pro tier subscription charges per user.

### B. CloudWatch Metrics
*   (N/A - Q Developer usage metrics are primarily surfaced through the Q Developer admin console and IAM Identity Center logs, rather than standard CloudWatch metrics).

### C. AWS Config / Trusted Advisor
*   (N/A - Trusted Advisor currently does not have dedicated checks for inactive Q Developer seats).

### D. Company Policies
*   **IT Asset Management (ITAM) Policy:** Standard operating procedures for AI tooling provisioning, duplicate tool prevention, and acceptable use.
*   **Onboarding/Offboarding Checklists:** Rules dictating when and how access is granted to new hires and revoked from departing staff.

### E. IaC (Optional)
*   **Terraform/CloudFormation:** Code managing the IAM Identity Center groups and application assignments that govern Q Developer Pro access.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "QDEV-WE-001",
  "category": "Waste Elimination",
  "service": "Amazon Q Developer",
  "resource_id": "user:jdoe@company.com",
  "issue": "Inactive Q Developer Pro seat (no usage in 30+ days).",
  "recommendation": "Remove user from the Q-Developer-Pro-Users IAM Identity Center group.",
  "estimated_monthly_savings": 19.00,
  "effort_level": "Low"
}
```

### Summary Report Table

| Finding ID | Category | Issue | Recommendation | Est. Monthly Savings | Effort |
|------------|----------|-------|----------------|----------------------|--------|
| QDEV-WE-001 | Waste Elimination | 50 inactive Q Developer Pro seats detected | Revoke licenses from inactive users | $950.00 | Low |
| QDEV-WE-002 | Waste Elimination | 20 users with duplicate Copilot/Q Dev licenses | Standardize on one tool and revoke the other | $380.00 | Low |
| QDEV-SAS-001 | Scheduling | Code Transformation sprint complete for Team A | Revoke Pro licenses from Team A post-migration | $190.00 | Low |
| QDEV-PMO-001 | Pricing Model | Unattributed Q Developer spend | Tag SSO application for departmental chargebacks | $0.00 (Visibility) | Medium |
