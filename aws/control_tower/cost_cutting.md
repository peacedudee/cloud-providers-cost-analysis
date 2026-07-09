# Cost-Cutting Playbook: AWS Control Tower
> **Companion File:** [control_tower.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/control_tower/control_tower.md)
> **Last Updated:** July 2026

---

## Executive Summary
AWS Control Tower itself is free, but the underlying resources it automatically provisions across every enrolled account (Config, CloudTrail, S3, Service Catalog, VPCs) can rapidly inflate cloud bills if left unmanaged. This playbook outlines actionable strategies to optimize these hidden costs. The most dramatic savings come from reigning in multi-region Config evaluations, modifying default Account Factory VPC networking, and optimizing the central log archive storage.

---

## Strategy Categories

### 1. Waste Elimination

#### CT-01. Enforce Region Deny Guardrails (De-register Unused Regions)
- **What:** Apply the Region Denial Guardrail (`Deny all actions outside approved regions`) to restrict Control Tower governance to active operating regions.
- **Why It Saves Money:** Control Tower deploys AWS Config rules and CloudTrail logging across all enabled regions. By restricting active regions, you avoid thousands of idle Config Rule evaluation fees ($0.001/eval). 50 accounts x 15 unused regions x 20 Config Rules = $1,500/month saved.
- **Implementation Steps:** 
  1. Access the AWS Control Tower console.
  2. Go to Landing zone settings and identify unused regions.
  3. Enable the "Deny all actions outside approved regions" guardrail.
  4. Update the landing zone to de-register unused regions.
- **Estimated Savings:** 80-90% on AWS Config rule fees.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** A clear definition of globally approved operating regions.

#### CT-02. Disable Global Resource Recording in Config for Secondary Regions
- **What:** Ensure that AWS Config only records global resources (like IAM) in a single designated region (e.g., `us-east-1`), rather than in all active regions.
- **Why It Saves Money:** Recording global resources in multiple regions results in redundant Configuration Items (CIs) at $0.003/CI and redundant evaluations.
- **Implementation Steps:** 
  1. Review Config Delivery Channels deployed by Control Tower.
  2. Verify that `IncludeGlobalResourceTypes` is set to `false` for all regions except the primary home region.
  3. Use AWS CLI or CloudFormation StackSets to enforce this setting.
- **Estimated Savings:** 10-20% on AWS Config costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Administrator access to Config settings in child accounts.

#### CT-03. Deploy SCPs to Block Costly Prohibited Resource Types
- **What:** Implement custom Service Control Policies (SCPs) via AWS Organizations (managed by Control Tower) to deny the launch of expensive resources like `.metal` instances or high-end GPU instances in sandbox environments.
- **Why It Saves Money:** Prevents accidental or malicious provisioning of high-cost compute resources, avoiding billing shocks.
- **Implementation Steps:** 
  1. Identify high-cost instance types and services not needed for standard workloads.
  2. Create an SCP that explicitly denies `ec2:RunInstances` for these resource types.
  3. Attach the SCP to the Developer Sandbox Organizational Unit (OU).
- **Estimated Savings:** 5-50% (avoids massive cost spikes).
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Organizational unit structure defined.

#### CT-04. Exclude Extraneous Resource Types from AWS Config
- **What:** Filter the resource types recorded by AWS Config to only include those relevant to your compliance guardrails, rather than recording all supported resource types.
- **Why It Saves Money:** Reduces the volume of Configuration Items (CIs) generated at $0.003/CI. Ephemeral resources (like auto-scaling EC2 instances or ENIs) generate massive CI volumes.
- **Implementation Steps:** 
  1. Analyze CloudWatch metrics for Config to identify the highest volume resource types.
  2. Update the Config Recorder settings in your Control Tower customizations to explicitly list required resource types instead of recording "All".
- **Estimated Savings:** 30-50% on AWS Config costs.
- **Risk Level:** Medium (may impact compliance visibility)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Approval from Security/Compliance teams.

#### CT-05. Remove Default Config Rules Not Relevant to Environment
- **What:** Disable or remove elective Config Rules that do not map to actual business or regulatory compliance needs.
- **Why It Saves Money:** Eliminates $0.001/eval per rule across all resources in all accounts.
- **Implementation Steps:** 
  1. Review active Config Rules in the Control Tower dashboard.
  2. Identify elective rules with zero compliance value.
  3. Deactivate those specific guardrails in the Control Tower console.
- **Estimated Savings:** 5-15% on AWS Config costs.
- **Risk Level:** Low
- **Implementation Scope:** Security | Engineer/DevOps
- **Prerequisites:** Compliance team sign-off.

### 2. Rightsizing

#### CT-06. Consolidate Developer Sandboxes
- **What:** Instead of provisioning a new AWS account for every single developer via Account Factory, consolidate developers into shared sandbox accounts separated by IAM permissions and resource tagging.
- **Why It Saves Money:** Reduces the baseline cost per account (Config, CloudTrail, default VPC infrastructure). Consolidating 20 sandbox accounts into 5 shared accounts eliminates the baseline overhead of 15 accounts.
- **Implementation Steps:** 
  1. Design a shared sandbox architecture with strict IAM namespace boundaries.
  2. Update Account Factory provisioning processes to grant access to shared accounts rather than spinning up new ones.
  3. Terminate unused standalone developer accounts.
- **Estimated Savings:** 20-40% on baseline governance costs.
- **Risk Level:** Medium (requires careful IAM policy design)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Robust IAM permission boundary templates.

#### CT-07. Set Lifecycle Rules on Central Log Archive S3 Buckets
- **What:** Apply S3 Lifecycle rules to transition historical audit logs (CloudTrail and Config) from Standard storage to Glacier classes.
- **Why It Saves Money:** Control Tower centralizes logs into an S3 bucket in the Log Archive account. Transitioning to Glacier Instant Retrieval ($0.004/GB-mo) after 30 days and Glacier Deep Archive ($0.00099/GB-mo) after 90 days slashes the standard $0.023/GB-mo cost.
- **Implementation Steps:** 
  1. Log into the Log Archive account.
  2. Navigate to the S3 console and select the central log bucket.
  3. Create a Lifecycle Rule for prefix `/AWSLogs/`.
  4. Set transition to Glacier Deep Archive after 90 days.
- **Estimated Savings:** Up to 95% on S3 audit log storage.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ensure compliance policies allow long-term storage in Glacier.

#### CT-08. Reduce Config Snapshot Frequency
- **What:** Change the delivery frequency of AWS Config snapshots to the central S3 bucket from the default (daily) to weekly or on-demand, if compliance allows.
- **Why It Saves Money:** Reduces S3 PUT requests and storage volume generated by frequent, large configuration snapshots.
- **Implementation Steps:** 
  1. Update the Config Delivery Channel settings via CloudFormation StackSets.
  2. Modify `deliveryFrequency` to `TwentyFour_Hours` or less frequent if supported by custom scripts.
- **Estimated Savings:** 5-10% on S3 storage and request costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compliance review of snapshot requirements.

### 3. Commitment Discounts

#### CT-09. Centralized Compute Savings Plans via Organization
- **What:** Leverage AWS Organizations (managed by Control Tower) to purchase and share Compute Savings Plans from the payer account, automatically applying discounts to all child accounts.
- **Why It Saves Money:** Savings Plans offer up to 72% discount on EC2, Fargate, and Lambda usage across all accounts in the Organization. Centralized sharing maximizes discount utilization.
- **Implementation Steps:** 
  1. Access AWS Cost Explorer in the management account.
  2. Ensure Savings Plan sharing is enabled in Billing preferences.
  3. Analyze recommendations and purchase a Compute Savings Plan covering the aggregate baseline usage.
- **Estimated Savings:** 20-40% on overall compute costs.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement
- **Prerequisites:** Stable baseline compute usage across the Organization.

### 4. Architecture Changes

#### CT-10. Customize Account Factory Network Settings to Exclude NAT Gateways
- **What:** Do not use the default Account Factory VPC template for developer sandbox or test environments, as it provisions costly NAT Gateways in every AZ.
- **Why It Saves Money:** Eliminates the flat $0.045/hour charge per NAT Gateway. A 3-AZ deployment in a single child account costs $98.55/month for idle NATs. 20 sandboxes = $1,971/month.
- **Implementation Steps:** 
  1. Create a custom VPC template using AWS Service Catalog or Account Factory for Terraform (AFT).
  2. Omit NAT Gateways from the template.
  3. Use VPC Endpoints for required AWS API access, or provide no internet egress for isolated sandboxes.
- **Estimated Savings:** ~$100 per month per sandbox account.
- **Risk Level:** Medium (Sandbox users must adapt to restricted egress)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Custom Service Catalog portfolios or AFT setup.

#### CT-11. Implement Centralized Egress via Transit Gateway
- **What:** Instead of deploying NAT Gateways in every child account VPC, route all outbound internet traffic through a Transit Gateway to a centralized Egress VPC with a shared NAT Gateway cluster.
- **Why It Saves Money:** Consolidates dozens of idle NAT Gateways across the Organization into 2 or 3 highly utilized NAT Gateways in a shared services account.
- **Implementation Steps:** 
  1. Deploy a Transit Gateway using AWS Resource Access Manager (RAM) to share it across the Organization.
  2. Create an Egress VPC in the Network account with NAT Gateways.
  3. Update child account VPC route tables to send `0.0.0.0/0` traffic to the TGW.
- **Estimated Savings:** 50-80% on NAT Gateway hourly baseline costs.
- **Risk Level:** High (Requires network architecture overhaul)
- **Implementation Scope:** Engineer/DevOps (Network Specialists)
- **Prerequisites:** Transit Gateway architecture in place.

#### CT-12. Consolidate AWS IAM Identity Center Directories
- **What:** Use the default IAM Identity Center integration provided by Control Tower rather than deploying separate AWS Managed Microsoft AD or AD Connectors in individual accounts.
- **Why It Saves Money:** AWS Managed Microsoft AD costs ~$86/month minimum, and AD Connectors cost ~$36/month. Centralizing identity removes these distributed directory costs.
- **Implementation Steps:** 
  1. Integrate your corporate Identity Provider (IdP) like Okta or Entra ID directly with the centralized IAM Identity Center in the management account.
  2. Remove legacy AD Connectors from child accounts.
- **Estimated Savings:** $36-$86+ per month per legacy directory eliminated.
- **Risk Level:** Medium
- **Implementation Scope:** Security | Engineer/DevOps
- **Prerequisites:** Compatible corporate IdP supporting SAML/OIDC.

### 5. Scheduling & Auto-Scaling

#### CT-13. Automate Sandbox Account Deletion and Vending Cleanup
- **What:** Implement a lifecycle mechanism (using Step Functions or EventBridge) to automatically suspend and eventually delete sandbox accounts that have exceeded their time-to-live (TTL).
- **Why It Saves Money:** Sandbox accounts accumulate orphaned resources (EBS volumes, idle RDS, Config CIs). Terminating the entire account via AWS Organizations stops all billing for those resources immediately.
- **Implementation Steps:** 
  1. Tag newly vended sandbox accounts with an `ExpirationDate`.
  2. Deploy a Lambda function in the management account scheduled via EventBridge.
  3. The Lambda identifies expired accounts, removes them from Control Tower, and calls `CloseAccount`.
- **Estimated Savings:** 10-30% on overall sandbox infrastructure spend.
- **Risk Level:** Medium (Must ensure data is not prematurely deleted)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Account Factory for Terraform (AFT) or custom automation framework.

### 6. Pricing Model Optimization

#### CT-14. Use S3 Intelligent-Tiering for Central CloudTrail Logs
- **What:** Enable S3 Intelligent-Tiering on the central log archive bucket if strict compliance lifecycles are not mandated, automatically moving infrequently accessed logs to cheaper storage tiers.
- **Why It Saves Money:** S3 Intelligent-Tiering automatically moves data not accessed for 30 days to the Infrequent Access tier ($0.0125/GB-mo) and after 90 days to the Archive Instant Access tier ($0.004/GB-mo), without retrieval fees or complex lifecycle rule management.
- **Implementation Steps:** 
  1. Navigate to the Log Archive account S3 console.
  2. Select the CloudTrail log bucket.
  3. Create a Lifecycle Rule transitioning all objects to S3 Intelligent-Tiering on Day 0.
- **Estimated Savings:** Up to 70% on storage costs for logs.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** None.

### 7. Network & Data Transfer Optimization

#### CT-15. Use VPC Endpoints instead of NAT Gateways for AWS API Access
- **What:** In child accounts that only need to communicate with AWS services (like S3, DynamoDB, or Systems Manager) and do not need general internet access, use VPC Gateway and Interface Endpoints.
- **Why It Saves Money:** Gateway Endpoints for S3 and DynamoDB are completely free. Interface endpoints cost $0.01/hour + data processing, which is significantly cheaper than the $0.045/hour NAT Gateway charge, and traffic doesn't traverse the public internet.
- **Implementation Steps:** 
  1. Identify VPCs in child accounts with no external internet requirements.
  2. Remove NAT Gateways and Internet Gateways from these VPCs.
  3. Deploy Gateway Endpoints for S3. Deploy Interface Endpoints (PrivateLink) for required services like SSM or CloudWatch.
- **Estimated Savings:** Eliminates NAT Gateway hourly fees and NAT Data Processing fees ($0.045/GB).
- **Risk Level:** Medium (Ensure applications don't require external API access)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Network traffic analysis.

---

## Cross-Service Synergies
- **Control Tower + Compute Optimizer:** By enabling AWS Compute Optimizer at the Organization level in Control Tower, you instantly generate rightsizing recommendations across all child accounts in a single dashboard.
- **Control Tower + Cost Explorer:** Utilizing AWS Cost Categories mapped to Control Tower Organizational Units (OUs) allows the FinOps team to seamlessly allocate costs back to specific business units based on the OU structure.
- **Control Tower + Savings Plans:** Centralized landing zones make sharing commitment discounts (Savings Plans, RIs) highly efficient, maximizing the discount float across dynamically vended accounts.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
- Query for `line_item_product_code` matching `AWSConfig`, `AmazonS3`, and `AmazonVPC` (specifically NAT Gateways).
- Group by `line_item_usage_account_id` to identify accounts with disproportionate governance overhead.

### B. CloudWatch Metrics
- Monitor `ConfigurationItemsRecorded` for AWS Config to identify spikes caused by ephemeral resource churn.

### C. AWS Config / Trusted Advisor
- Use Trusted Advisor Organization View to identify underutilized resources across all Control Tower enrolled accounts.

### D. Company Policies
- Determine the required retention periods for CloudTrail and Config logs to accurately set S3 Lifecycle policies.
- Clarify approved operating regions for the Region Deny Guardrail.

### E. IaC (Optional)
- Review AWS Service Catalog products or Account Factory for Terraform (AFT) repositories to analyze default VPC templates.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "CT-01",
  "service": "AWS Control Tower",
  "strategy_category": "Waste Elimination",
  "finding_name": "Enforce Region Deny Guardrails",
  "description": "Control Tower deploys Config rules across all regions. Restrict to active regions.",
  "potential_savings_usd": 1500.00,
  "savings_percentage": 85,
  "risk_level": "Low",
  "effort_level": "Low",
  "action_required": "Enable 'Deny all actions outside approved regions' guardrail."
}
```

### Summary Report Table
| Finding ID | Strategy Name | Category | Risk Level | Est. Savings |
|------------|---------------|----------|------------|--------------|
| CT-01 | Enforce Region Deny Guardrails | Waste Elimination | Low | 80-90% (Config) |
| CT-02 | Disable Global Resource Recording | Waste Elimination | Low | 10-20% (Config) |
| CT-03 | Deploy SCPs for Costly Resources | Waste Elimination | Medium | 5-50% (Avoidance) |
| CT-04 | Exclude Extraneous Resource Types | Waste Elimination | Medium | 30-50% (Config) |
| CT-05 | Remove Default Config Rules | Waste Elimination | Low | 5-15% (Config) |
| CT-06 | Consolidate Developer Sandboxes | Rightsizing | Medium | 20-40% (Base) |
| CT-07 | Lifecycle Rules on Central Log S3 | Rightsizing | Low | 95% (S3) |
| CT-08 | Reduce Config Snapshot Frequency | Rightsizing | Low | 5-10% (S3) |
| CT-09 | Centralized Compute Savings Plans | Commitment Discounts | Low | 20-40% (Compute) |
| CT-10 | Exclude NAT Gateways from Account Factory | Architecture Changes | Medium | $100/mo/account |
| CT-11 | Centralized Egress via Transit Gateway | Architecture Changes | High | 50-80% (NAT) |
| CT-12 | Consolidate IAM Identity Center | Architecture Changes | Medium | $36-$86/mo/dir |
| CT-13 | Automate Sandbox Account Deletion | Scheduling & Auto-Scaling | Medium | 10-30% (Sandbox) |
| CT-14 | Intelligent-Tiering for CloudTrail Logs | Pricing Model Optimization | Low | 70% (S3) |
| CT-15 | Use VPC Endpoints instead of NATs | Network Optimization | Medium | Eliminates NAT |
