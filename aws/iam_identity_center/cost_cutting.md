# Cost-Cutting Playbook: AWS IAM Identity Center
> **Companion File:** [iam_identity_center.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/iam_identity_center/iam_identity_center.md)
> **Last Updated:** July 2026

---

## Executive Summary
AWS IAM Identity Center (formerly AWS SSO) is provided by AWS at **no additional charge**. However, organizations frequently incur unnecessary costs through related infrastructure, such as premium identity provider (IdP) licenses, paid AWS Directory Services for basic SSO, redundant CloudTrail logging, or custom-built federation proxies. This playbook outlines 19 strategies to leverage IAM Identity Center's free capabilities to decommission costly legacy identity architecture, optimize network/logging overhead, and eliminate third-party SaaS "SSO taxes."

## Strategy Categories

### 1. Waste Elimination

#### 1. Replace AWS Managed Microsoft AD for SSO with IAM Identity Center
- **What:** Decommission AWS Managed Microsoft AD if it was deployed solely to provide corporate users with single sign-on (SSO) to the AWS Console/CLI. Migrate users to IAM Identity Center's native directory or free SAML integration with an existing corporate IdP.
- **Why It Saves Money:** AWS Managed AD costs $175.20/month (Standard) or $584.00/month (Enterprise). IAM Identity Center is $0.
- **Implementation Steps:**
  1. Configure IAM Identity Center and connect your corporate IdP via SAML 2.0.
  2. Map users/groups to AWS accounts using Permission Sets.
  3. Validate login flows for all developer teams.
  4. Delete the AWS Managed Microsoft AD directory.
- **Estimated Savings:** 100% of AD costs ($175 - $584/month).
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Existing corporate IdP (Entra ID, Workspace) supporting SAML 2.0.

#### 2. Deprecate AD Connector Used Solely for AWS Access
- **What:** Eliminate AD Connector instances used strictly to federate on-premises Active Directory to AWS. Replace with a direct SAML integration to IAM Identity Center from Entra ID or ADFS.
- **Why It Saves Money:** AD Connector costs $0.05/hour ($36.50/month) for Small and $0.15/hour ($109.50/month) for Large. IAM IC SAML integration is completely free.
- **Implementation Steps:**
  1. Set up SAML 2.0 federation from your primary IdP directly into IAM IC.
  2. Shift users to the new login flow.
  3. Delete the AD Connector directory from AWS Directory Service.
- **Estimated Savings:** 100% of AD Connector costs ($36 - $109/month per connector).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Primary IdP must support standard SAML 2.0.

#### 3. Eliminate Custom IAM Broker Infrastructure
- **What:** Shut down custom-built SAML/SSO federation portals (e.g., bespoke broker apps running on EC2, Fargate, or API Gateway/Lambda).
- **Why It Saves Money:** Removes the EC2 instance compute, load balancer (ALB), and NAT Gateway costs associated with maintaining bespoke SSO infrastructure.
- **Implementation Steps:**
  1. Replicate role mappings as Permission Sets in IAM Identity Center.
  2. Redirect user login bookmarks to the AWS-provided IAM IC access portal.
  3. Terminate legacy EC2 instances, ALBs, and ECS clusters running the custom broker.
- **Estimated Savings:** $50 - $500+/month in compute and networking.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** IAM Identity Center enabled in AWS Organizations.

#### 4. Deduplicate CloudTrail Management Events for SSO Logins
- **What:** Remove duplicate CloudTrail trails that independently log IAM Identity Center authentication events.
- **Why It Saves Money:** The first CloudTrail management event trail is free. Additional trails cost $2.00 per 100,000 events. Frequent CLI logins generate massive event volume, driving up costs on duplicate trails.
- **Implementation Steps:**
  1. Audit AWS CloudTrail configurations across the Organization.
  2. Consolidate into a single organizational trail.
  3. Disable redundant trails logging management events.
- **Estimated Savings:** $10 - $100+/month depending on organization size.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Ensure the single organizational trail meets all security/compliance requirements.

#### 5. Drop Third-Party IdP Licenses for AWS-Only Users
- **What:** Move contractors, vendors, or auditors to IAM Identity Center's native identity store instead of purchasing extra per-user seats in premium IdPs (like Okta or Entra ID Premium).
- **Why It Saves Money:** Premium third-party IdPs charge $2-$10+ per user/month. IAM Identity Center's native store is $0 for unlimited users.
- **Implementation Steps:**
  1. Change IAM IC identity source to "Identity Center directory" (if not strictly using an external IdP globally).
  2. Provision contractors directly in AWS.
  3. Reduce seat count on third-party IdP contract renewal.
- **Estimated Savings:** $5 - $10 per user/month.
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership | IT Admins
- **Prerequisites:** Ability to manage contractor lifecycle natively in AWS without breaking HR compliance.

#### 6. Clean Up Unused Permission Sets to Reduce AWS Config Costs
- **What:** Delete orphaned or unused IAM IC Permission Sets.
- **Why It Saves Money:** AWS Config records configuration changes for `AWS::SSO::PermissionSet`. Every change or evaluation costs $0.003 per item. Bloated, constantly tweaking permission sets lead to unnecessary Config evaluation costs.
- **Implementation Steps:**
  1. Identify Permission Sets with no assigned accounts/groups.
  2. Delete them via the AWS Console or CLI.
- **Estimated Savings:** $5 - $20/month.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

### 2. Rightsizing

#### 7. Rightsize Third-Party SAML Application Integration
- **What:** Use IAM Identity Center to federate third-party SaaS apps (Jira, Salesforce, Slack) directly, rather than paying for a premium identity broker tier (e.g., Okta SSO Add-ons or OneLogin enterprise tiers) just to achieve basic SaaS SSO.
- **Why It Saves Money:** IAM IC provides unlimited SAML 2.0 app integrations out-of-the-box for $0.
- **Implementation Steps:**
  1. Add the SaaS application in the IAM IC Applications console.
  2. Swap the SAML metadata in the SaaS application to point to AWS.
  3. Downgrade your third-party IdP tier.
- **Estimated Savings:** Hundreds to thousands of dollars per month depending on SaaS SSO tax and vendor pricing.
- **Risk Level:** Medium
- **Implementation Scope:** IT Admins | Engineer/DevOps
- **Prerequisites:** Third-party SaaS apps must support standard SAML 2.0.

#### 8. Downgrade AWS Managed AD if Retained for Legacy Workloads
- **What:** If Managed AD cannot be fully eliminated (due to legacy EC2 Windows workloads), but SSO traffic is moved entirely to IAM IC, downgrade the Managed AD tier from Enterprise to Standard.
- **Why It Saves Money:** Enterprise AD ($584/month) supports millions of objects and multi-region routing. Standard AD ($175/month) is sufficient for a few local EC2 servers.
- **Implementation Steps:**
  1. Confirm directory object count is under 30,000.
  2. Deploy a new Standard Managed AD.
  3. Migrate workloads and delete the Enterprise AD. (In-place downgrade is not supported by AWS).
- **Estimated Savings:** $408.80/month per directory.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Object count < 30,000; tolerance for directory migration effort.

### 3. Commitment Discounts

#### 9. Avoid Multi-Year Identity Provider Lock-in for AWS Access
- **What:** Prevent signing multi-year commitment contracts with enterprise IdPs strictly for the purpose of managing AWS infrastructure access.
- **Why It Saves Money:** Avoids expensive 3-year commitments on external IdPs when AWS IAM IC handles AWS authorization, group mapping, and multi-account access entirely for free.
- **Implementation Steps:**
  1. Scope AWS user counts out of enterprise IdP renewals.
  2. Standardize AWS access purely on IAM IC's native directory or a free-tier IdP (e.g., Entra ID Free).
- **Estimated Savings:** Variable (Contract avoidance).
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** Strategic decision required prior to contract renewal cycles.

### 4. Architecture Changes

#### 10. Replace Long-Lived IAM Users to Eliminate Secrets Manager Costs
- **What:** Migrate developers and CI/CD pipelines from long-lived IAM Access Keys to IAM Identity Center short-lived credentials (`aws sso login`).
- **Why It Saves Money:** Eliminates the need to use AWS Secrets Manager ($0.40/secret/month + $0.05/10k API calls) or third-party vaulting tools to manage, rotate, and securely distribute static AWS access keys.
- **Implementation Steps:**
  1. Install AWS CLI v2 for all developers.
  2. Run `aws configure sso` and map profiles.
  3. Delete static IAM Users and secrets from Secrets Manager.
- **Estimated Savings:** $0.40 per user/month + custom rotation infrastructure costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CLI v2 adoption.

#### 11. Automate SCIM Provisioning Natively vs. Custom Compute
- **What:** Use the native IAM Identity Center SCIM endpoint to automatically sync users from your IdP instead of running custom AWS Lambda cron jobs to sync directories.
- **Why It Saves Money:** Removes Lambda compute costs, EventBridge invocation costs, and associated NAT Gateway data transfer charges.
- **Implementation Steps:**
  1. Enable automatic provisioning in IAM IC.
  2. Generate a SCIM token and configure it in your IdP.
  3. Delete custom Lambda sync scripts.
- **Estimated Savings:** $10 - $50/month in compute and data transfer.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** IdP must support SCIM v2.0.

#### 12. Migrate Amazon Managed Grafana Auth to IAM IC
- **What:** Use IAM Identity Center as the native authentication provider for Amazon Managed Grafana workspaces instead of running a custom SAML proxy (e.g., Nginx) on EC2.
- **Why It Saves Money:** Native IAM IC integration with Grafana is free. Running a proxy incurs hourly compute and data transfer costs.
- **Implementation Steps:**
  1. Recreate or reconfigure the Managed Grafana workspace to use IAM Identity Center.
  2. Terminate the proxy EC2 instances.
- **Estimated Savings:** $30 - $100/month per proxy instance.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Grafana workspace in the same region as IAM IC.

#### 13. Unify QuickSight Authentication with IAM IC
- **What:** Use IAM Identity Center for Amazon QuickSight reader/author logins rather than deploying a dedicated AWS Directory Service (Simple AD or Managed AD).
- **Why It Saves Money:** Simple AD costs ~$36/month and Managed AD ~$175/month. IAM IC integration is free and eliminates directory maintenance overhead.
- **Implementation Steps:**
  1. Integrate QuickSight with IAM IC natively.
  2. Decommission the QuickSight-specific AWS Directory Service.
- **Estimated Savings:** $36 - $175/month.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** QuickSight Enterprise edition.

#### 14. Unify Amazon SageMaker Studio Auth with IAM IC
- **What:** Deploy Amazon SageMaker Studio domains using IAM Identity Center authentication rather than managing IAM Users/Roles via custom federated portal scripts.
- **Why It Saves Money:** Reduces engineering maintenance overhead and Lambda/API Gateway execution costs associated with vending presigned SageMaker Studio URLs dynamically.
- **Implementation Steps:**
  1. Set up SageMaker Domain authentication mode to "AWS IAM Identity Center".
  2. Remove custom URL-vending Lambda APIs.
- **Estimated Savings:** Reduces engineering toil; ~$10-$20/month in API costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SageMaker Domain must be configured for IAM IC from creation.

### 5. Scheduling & Auto-Scaling

#### 15. Scale Down AD Connectors During Off-Hours (If Unavoidable)
- **What:** If AD Connector must be kept for legacy SSO and cannot be migrated to direct SAML, script the modification of AD Connector size (Large to Small) or tear it down via IaC during non-business hours in dev/test environments.
- **Why It Saves Money:** AD Connector Large is $0.15/hour; Small is $0.05/hour. Tearing it down over the weekend saves idle costs.
- **Implementation Steps:**
  1. Implement an EventBridge scheduler triggering a Lambda function.
  2. Rebuild AD Connectors on Monday mornings via CloudFormation/Terraform.
- **Estimated Savings:** ~$72/month per connector if torn down on weekends/nights.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Heavy reliance on IaC and tolerance for dev/test login downtime.

### 6. Pricing Model Optimization

#### 16. Use Free IAM IC Native MFA Over Paid Third-Party MFA
- **What:** Enforce MFA using IAM Identity Center's built-in authenticator app (WebAuthn/TOTP) support instead of routing users through premium tier IdPs that charge extra for advanced MFA features.
- **Why It Saves Money:** IAM IC's native MFA is completely free. Third parties charge $3-$6 per user/month for MFA add-ons.
- **Implementation Steps:**
  1. Enable MFA in IAM IC settings.
  2. Require users to register hardware keys (YubiKey) or TOTP apps (Google Authenticator).
  3. Cancel third-party MFA subscriptions.
- **Estimated Savings:** $3 - $6 per user/month.
- **Risk Level:** Low
- **Implementation Scope:** IT Admins | Security
- **Prerequisites:** Standard authenticator apps or hardware tokens.

#### 17. Optimize CloudWatch Logs Storage for IAM IC CloudTrail Events
- **What:** Store IAM IC authentication events in CloudWatch Logs Infrequent Access (IA) instead of Standard, or route directly to S3.
- **Why It Saves Money:** CloudWatch Logs Standard is $0.50/GB. CWL IA is $0.25/GB. S3 is ~$0.023/GB.
- **Implementation Steps:**
  1. Update the CloudTrail delivery configuration to point to a Log Group configured for Infrequent Access.
  2. Alternatively, bypass CloudWatch entirely and deliver logs only to S3.
- **Estimated Savings:** 50% - 95% on log storage costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Logs are only needed for compliance/auditing, not real-time metric alarms.

### 7. Network & Data Transfer Optimization

#### 18. Use VPC Endpoints (PrivateLink) for STS Instead of NAT Gateways
- **What:** When workloads or CI/CD runners in private subnets request temporary credentials via IAM Identity Center (e.g., `AssumeRoleWithSAML`), route traffic through a VPC Endpoint for STS rather than a NAT Gateway.
- **Why It Saves Money:** NAT Gateway data processing costs $0.045/GB. VPC Endpoint data processing is $0.01/GB. Frequent CI/CD token refreshes generate significant TLS traffic.
- **Implementation Steps:**
  1. Create an Interface VPC Endpoint for `com.amazonaws.[region].sts`.
  2. Ensure Security Groups allow inbound port 443 from the private subnets.
- **Estimated Savings:** $0.035/GB of STS token traffic.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Private subnets routing out via NAT Gateway.

#### 19. Centralize Identity Center Regionally to Avoid Cross-Region CloudTrail Data Transfer
- **What:** Deploy IAM IC in the organization's primary operating region to avoid cross-region data transfer fees when CloudTrail global events are aggregated into a central S3 bucket in a different region.
- **Why It Saves Money:** CloudTrail delivers IAM IC global service events from the region where IAM IC is configured (usually `us-east-1`). If your central bucket is in `ap-southeast-2`, you incur cross-region data transfer fees for logs.
- **Implementation Steps:**
  1. If starting fresh, deploy IAM IC in the same region as your centralized Security/Logging S3 bucket.
- **Estimated Savings:** ~$0.02/GB of log data transfer.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Architecture allows primary region deployment.

---

## Cross-Service Synergies
- **AWS Directory Service:** IAM Identity Center drastically reduces or eliminates the need for expensive Managed AD or AD Connector instances.
- **AWS CloudTrail & Config:** Centralized identity reduces the proliferation of IAM Users and Access Keys, simplifying Config tracking and reducing CloudTrail event volume for IAM changes.
- **AWS Secrets Manager:** Short-lived temporary credentials eliminate the need to store and rotate permanent access keys.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query for `product_product_name = 'AWS Directory Service'` to find AD instances that can be eliminated.
- Analyze `product_product_name = 'AWS CloudTrail'` to identify duplicate trail costs.

### B. CloudWatch Metrics
- Check `AWS/DirectoryService` metrics to ensure AD instances targeted for deletion are not serving non-SSO LDAP queries.

### C. AWS Config / Trusted Advisor
- Use Trusted Advisor to find unused IAM Users/Access Keys that can be deleted in favor of IAM IC.

### D. Company Policies
- Review corporate identity policies to confirm if native IAM IC can replace third-party IdP contractor seats.

### E. IaC (Optional)
- Terraform/CloudFormation templates identifying custom IAM broker setups (EC2, ALBs) that can be decommissioned.

---

## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "IAMIC-001",
  "service": "AWS IAM Identity Center",
  "strategy_name": "Replace AWS Managed Microsoft AD for SSO with IAM Identity Center",
  "category": "Waste Elimination",
  "estimated_savings_mrr": 584.00,
  "risk_level": "Medium",
  "effort_level": "Medium",
  "action_type": "Modify",
  "description": "Migrate SSO authentication from Enterprise Managed AD to free IAM Identity Center SAML integration."
}
```

### Summary Report Table
| Finding ID | Strategy Name | Savings Potential | Risk Level | Effort |
|------------|---------------|-------------------|------------|--------|
| IAMIC-001 | Replace AWS Managed Microsoft AD | High ($175-$584/mo) | Medium | Medium |
| IAMIC-002 | Deprecate AD Connector | Low ($36-$109/mo) | Low | Low |
| IAMIC-003 | Eliminate Custom Broker Infra | Medium ($50-$500/mo) | Medium | High |
| IAMIC-010 | Replace Long-Lived IAM Users | Low ($0.40/user) | Low | Medium |
| IAMIC-018 | VPC Endpoints for STS | Low ($0.035/GB) | Low | Low |
