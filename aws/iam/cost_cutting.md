# Cost-Cutting Playbook: AWS IAM
> **Companion File:** [iam.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/iam/iam.md)
> **Last Updated:** July 2026

---

## Executive Summary
While core AWS Identity and Access Management (IAM) components are 100% free, managing access at scale introduces notable indirect costs. Implementing advanced security features like IAM Access Analyzer (Unused Access and Custom Policy Checks) and managing Secure Token Service (STS) network endpoints can generate surprise billing spikes in large environments. This playbook outlines actionable strategies to optimize these paid IAM features, reduce STS network overhead, and streamline identity architecture to achieve zero-waste security governance.

## Strategy Categories

### 1. Waste Elimination

#### 1. Delete Unused IAM Roles
- **What:** Identify and terminate IAM roles that have not been used in over 90 days.
- **Why It Saves Money:** Unused Access Analyzer bills $0.20 per role monitored per month. Removing 5,000 unused legacy roles saves $1,000/month.
- **Implementation Steps:** 
  1. Use AWS IAM credential reports or AWS Config to identify unused roles.
  2. Review role usage with engineering owners.
  3. Delete roles that are obsolete.
- **Estimated Savings:** 10-30% of IAM Access Analyzer costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS Config or IAM Access Analyzer enabled

#### 2. Delete Unused IAM Users
- **What:** Terminate inactive IAM Users across all AWS accounts.
- **Why It Saves Money:** Similar to roles, Unused Access Analyzer charges $0.20 per IAM user analyzed per month. Eliminating 500 inactive user accounts saves $100/month.
- **Implementation Steps:**
  1. Generate an IAM Credential Report.
  2. Filter users with no console or programmatic access in 90+ days.
  3. Deactivate access keys, then delete the users.
- **Estimated Savings:** 10-20% of IAM Access Analyzer costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** IAM Credential Reports

#### 3. Exclude Non-Production Roles from Unused Access Analyzer
- **What:** Use tags to exclude development, test, or highly dynamic CI/CD roles from Unused Access Analyzer scans.
- **Why It Saves Money:** Avoids paying $0.20/month for roles that are inherently short-lived or meant for sandbox testing.
- **Implementation Steps:**
  1. Tag relevant roles with an exclusion tag (e.g., `AnalyzerExclude: true`).
  2. Configure IAM Access Analyzer archive rules to exclude these tagged roles.
- **Estimated Savings:** 40-60% of IAM Access Analyzer costs in large dev accounts
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Consistent resource tagging strategy

#### 4. Disable Unused Access Analyzer in Sandbox Accounts
- **What:** Turn off the paid Unused Access Analyzer feature entirely in non-critical sandbox and playground AWS accounts.
- **Why It Saves Money:** Saves the flat rate of $0.20 per identity for environments that do not hold sensitive data or require strict compliance monitoring.
- **Implementation Steps:**
  1. Identify sandbox AWS accounts via AWS Organizations.
  2. Navigate to IAM Access Analyzer settings in those accounts.
  3. Disable the analyzer for unused access.
- **Estimated Savings:** 100% of IAM Access Analyzer costs in targeted accounts
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Multi-account architecture with clear environment separation

#### 5. Automate Identity Lifecycle Offboarding
- **What:** Integrate HR systems directly with AWS SSO/IAM to automatically deprovision access when employees leave.
- **Why It Saves Money:** Prevents paying $0.20/month indefinitely for orphaned identities and significantly reduces the financial risk of data breaches.
- **Implementation Steps:**
  1. Connect HRIS (e.g., Workday) to your Identity Provider (IdP).
  2. Sync IdP with AWS Identity Center.
  3. Ensure offboarding scripts automatically remove any direct IAM Users.
- **Estimated Savings:** 5-10% of IAM Access Analyzer costs
- **Risk Level:** Low
- **Implementation Scope:** IT/Security | Engineer/DevOps
- **Prerequisites:** Centralized Identity Provider (Okta, Entra ID)

### 2. Rightsizing

#### 6. Scope Internal Access Analyzer to Critical Resources Only
- **What:** Limit the IAM Internal Access Analyzer to monitor only highly critical resources (e.g., Production RDS databases or S3 buckets with PII).
- **Why It Saves Money:** Internal Access Analyzer costs a steep $9.00 per resource per month. Limiting this to 50 critical resources instead of 5,000 general resources saves $44,550/month.
- **Implementation Steps:**
  1. Identify tier-1 mission-critical resources.
  2. Configure Internal Access Analyzer to target only those specific ARNs.
- **Estimated Savings:** 80-90% of Internal Access Analyzer costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Data classification and critical asset inventory

#### 7. Optimize Custom Policy Check CI/CD Triggers
- **What:** Configure CI/CD pipelines to execute IAM Custom Policy Checks ONLY when IAM-related files are modified.
- **Why It Saves Money:** Custom Policy Checks cost $0.0020 per API call. If a pipeline runs 1,000 times a day for UI changes, you waste money. Triggering only on IAM changes eliminates redundant checks.
- **Implementation Steps:**
  1. Update CI/CD pipeline configuration (e.g., GitHub Actions path filters).
  2. Restrict the custom policy check step to run only on paths like `**/*.iam.json` or `**/iam.tf`.
- **Estimated Savings:** 70-90% of Custom Policy Check costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CI/CD pipeline managed infrastructure

#### 8. Batch Custom Policy Checks
- **What:** Consolidate multiple policy validations into fewer API calls or run them on merged branches rather than every individual commit.
- **Why It Saves Money:** Reduces the total volume of billable API calls ($0.0020/call) by validating the final state rather than intermediate states.
- **Implementation Steps:**
  1. Shift policy checks from per-commit hooks to pull-request creation or merge events.
- **Estimated Savings:** 30-50% of Custom Policy Check costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Git workflow supporting PR-level checks

### 3. Commitment Discounts

#### 9. Centralize Identity for Organizational Volume Discounts
- **What:** While IAM itself has no Savings Plans or Reserved Instances, centralizing identity into a primary payer account allows for aggregated billing of Access Analyzer features, simplifying cost visibility and enabling better EDP (Enterprise Discount Program) negotiations.
- **Why It Saves Money:** Aggregated spend contributes to overall AWS EDP thresholds, potentially unlocking higher discount tiers across all AWS services.
- **Implementation Steps:**
  1. Deploy IAM Identity Center in the Organization's management or dedicated security account.
  2. Include IAM Access Analyzer spend in EDP sizing calculations.
- **Estimated Savings:** 1-5% indirect savings via EDP
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership | FinOps Team
- **Prerequisites:** AWS Organizations, EDP eligibility

### 4. Architecture Changes

#### 10. Enforce Regional STS Endpoints
- **What:** Configure the AWS CLI, SDKs, and applications to use regional STS endpoints (e.g., `sts.us-east-1.amazonaws.com`) rather than the global endpoint.
- **Why It Saves Money:** Global STS endpoints can cause cross-region data transfer egress ($0.01 - $0.02 / GB) if the workload is in a different region. Regional endpoints keep traffic local and free.
- **Implementation Steps:**
  1. Set the environment variable `AWS_STS_REGIONAL_ENDPOINTS=regional` in compute environments.
  2. Update AWS SDK configurations to explicitly state the local region.
- **Estimated Savings:** 100% of STS-related cross-region data transfer costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Applications using AWS SDKs or CLI

#### 11. Consolidate STS VPC Endpoints
- **What:** Route STS traffic from private subnets through a shared Central VPC or Transit Gateway endpoint rather than placing a dedicated STS VPC endpoint in every VPC.
- **Why It Saves Money:** Dedicated STS VPC endpoints cost ~$7.20/month per Availability Zone. Removing redundant endpoints across 50 VPCs (2 AZs each) saves ~$720/month.
- **Implementation Steps:**
  1. Deploy a shared STS VPC Endpoint in a centralized networking VPC.
  2. Route STS DNS requests from spoke VPCs to the central endpoint using Route 53 Resolver endpoints.
- **Estimated Savings:** 80-95% of STS VPC Endpoint costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS Transit Gateway or VPC Peering hub-and-spoke architecture

#### 12. Migrate from IAM Users to Identity Center (SSO)
- **What:** Transition away from standalone IAM Users per account to AWS IAM Identity Center (SSO) using a central Identity Provider.
- **Why It Saves Money:** Reduces the sheer volume of IAM User entities that Unused Access Analyzer scans ($0.20 per identity) because users assume temporary roles instead of having permanent IAM User objects in every account.
- **Implementation Steps:**
  1. Deploy AWS IAM Identity Center.
  2. Map IdP groups to permission sets.
  3. Delete legacy IAM Users across workload accounts.
- **Estimated Savings:** 50-80% of identity-related Access Analyzer costs
- **Risk Level:** Medium
- **Implementation Scope:** IT/Security | Engineer/DevOps
- **Prerequisites:** Compatible IdP (Active Directory, Okta, etc.)

#### 13. Replace Customer IAM Users with Amazon Cognito
- **What:** Use Amazon Cognito for external customer or client authentication instead of creating individual AWS IAM Users for them.
- **Why It Saves Money:** Cognito provides a generous free tier (50,000 MAUs) and charges based on active usage, whereas managing thousands of customer IAM users incurs massive Unused Access Analyzer fees ($0.20 per user/month) and creates severe administrative overhead.
- **Implementation Steps:**
  1. Provision an Amazon Cognito User Pool.
  2. Update applications to authenticate via Cognito and exchange tokens for temporary STS credentials.
  3. Delete external customer IAM Users.
- **Estimated Savings:** 99% of customer-related IAM identity costs
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application development capacity for auth refactoring

#### 14. Cache STS Credentials at the Application Layer
- **What:** Modify applications to cache temporary STS credentials (which default to 1-hour expiration) until they are near expiration, instead of calling `AssumeRole` on every single transaction.
- **Why It Saves Money:** Reduces API throttling and minimizes network outbound traffic to STS endpoints, saving VPC endpoint data processing charges ($0.01/GB).
- **Implementation Steps:**
  1. Implement credential caching in custom application code.
  2. Ensure the cache refreshes 5 minutes before token expiration.
- **Estimated Savings:** 10-30% of STS data processing costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Custom application source code access

### 5. Scheduling & Auto-Scaling

#### 15. Schedule Cleanup of Temporary Deployment Roles
- **What:** Implement automation to delete temporary cross-account roles or CloudFormation execution roles immediately after a deployment window closes.
- **Why It Saves Money:** Prevents temporary roles from lingering and being billed by the Unused Access Analyzer at the end of the month.
- **Implementation Steps:**
  1. Use EventBridge and Lambda to trigger role deletion based on CloudFormation deployment completion events.
  2. Alternatively, use a scheduled Lambda job to clean up roles prefixed with `temp-deploy-` nightly.
- **Estimated Savings:** 5-15% of IAM Access Analyzer costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ephemeral deployment architecture

### 6. Pricing Model Optimization

#### 16. Shift Dev/Test Policy Validation to Open-Source Tools
- **What:** Use free open-source policy linters (e.g., Checkov, Open Policy Agent, AWS CloudFormation Guard) in lower environments instead of paying for AWS Custom Policy Checks.
- **Why It Saves Money:** Replaces paid AWS API calls ($0.0020 per check) with free local or CI runner compute execution.
- **Implementation Steps:**
  1. Integrate Checkov or `cfn-guard` into developer workstations and Dev/QA CI pipelines.
  2. Reserve AWS Custom Policy Checks exclusively for Production deployments.
- **Estimated Savings:** 60-80% of Custom Policy Check costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Open-source tooling expertise

#### 17. Standardize IAM Policies via Managed Boundaries
- **What:** Use AWS Managed Policies and standardized Permissions Boundaries instead of writing unique inline policies for every single role.
- **Why It Saves Money:** High standardization reduces the need to run Custom Policy Checks on every deployment, as the core security boundaries are already validated and static.
- **Implementation Steps:**
  1. Create organizational standard permission boundaries.
  2. Enforce developers to attach these boundaries rather than crafting wild-card custom policies.
- **Estimated Savings:** 20-40% of Custom Policy Check costs
- **Risk Level:** Medium
- **Implementation Scope:** IT/Security | Engineer/DevOps
- **Prerequisites:** AWS Organizations

### 7. Network & Data Transfer Optimization

#### 18. Avoid Cross-Region STS Calls
- **What:** Ensure global applications assume roles in the region they are operating in.
- **Why It Saves Money:** Prevents $0.01 - $0.02 / GB egress charges that occur when workloads in `eu-west-1` authenticate against `sts.us-east-1.amazonaws.com`.
- **Implementation Steps:**
  1. Audit CloudTrail for STS `AssumeRole` events originating from foreign IP addresses.
  2. Update application endpoints to regional configurations.
- **Estimated Savings:** 100% of inter-region STS data transfer
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region deployment

---

## Cross-Service Synergies
- **AWS Organizations:** Centralized management enables SCPs (Service Control Policies) which can completely block the creation of paid Access Analyzer resources in unauthorized accounts, preventing shadow IT spend.
- **AWS Config:** Using AWS Config to track IAM changes can reduce reliance on paid continuous polling mechanisms, optimizing overall governance costs.
- **Amazon VPC:** Shared central VPCs with Transit Gateway drastically reduce the need for duplicated STS VPC endpoints across spoke networks.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query for `product_productName = 'AWS Identity and Access Management'` and usage types containing `AccessAnalyzer` or `STS`.
### B. CloudWatch Metrics
- Review STS API call volumes and error rates for throttling related to non-cached credential requests.
### C. AWS Config / Trusted Advisor
- Use Trusted Advisor's "IAM Access Key Rotation" and "IAM Password Policy" checks to identify stale users.
### D. Company Policies
- Determine the required frequency of Access Analyzer scans (e.g., continuous vs. weekly) based on compliance requirements (e.g., SOC2, PCI-DSS).
### E. IaC (Optional)
- Terraform state files can reveal the sheer count of provisioned IAM roles to estimate baseline Unused Access Analyzer costs before enabling the service.

---

## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "IAM-001",
  "category": "Waste Elimination",
  "resource_id": "arn:aws:iam::123456789012:role/LegacyAppRole",
  "strategy_name": "Delete Unused IAM Roles",
  "estimated_monthly_savings": 0.20,
  "confidence_score": 0.95,
  "action_script": "aws iam delete-role --role-name LegacyAppRole"
}
```

### Summary Report Table
| Strategy | Category | Estimated Monthly Savings | Implementation Effort | Risk Level |
|----------|----------|---------------------------|-----------------------|------------|
| Delete Unused IAM Roles | Waste Elimination | $0.20 per role | Low | Medium |
| Scope Internal Access Analyzer | Rightsizing | $9.00 per excluded resource | Medium | Medium |
| Optimize CI/CD Policy Checks | Rightsizing | ~$100+ per active pipeline | Low | Low |
| Consolidate STS VPC Endpoints | Architecture Changes | $21.60 per VPC | High | Medium |
| Shift Dev Policy Validation to OSS | Pricing Model Optimization | ~$50+ per pipeline | Medium | Low |
