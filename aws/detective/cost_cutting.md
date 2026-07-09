# Cost-Cutting Playbook: Amazon Detective
> **Companion File:** [detective.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/detective/detective.md)
> **Last Updated:** July 2026
---
## Executive Summary
Amazon Detective is a powerful, serverless security investigation service that automatically ingests and analyzes log data from AWS CloudTrail, Amazon VPC Flow Logs, Amazon GuardDuty findings, Amazon EKS audit logs, and AWS Security Hub findings. Because Detective charges no base fees and relies entirely on a tiered per-GB ingestion pricing model, cost optimization fundamentally depends on managing your deployment footprint and minimizing upstream data noise. This playbook outlines 18 strategies to eliminate unnecessary ingestion, leverage tiered pricing discounts through centralized administration, and reduce the underlying API and network noise that drives up Detective costs.

## Strategy Categories
### 1. Waste Elimination
Focuses on identifying and disabling Amazon Detective in environments, accounts, or regions where active security investigation is not required, avoiding the steep $2.00/GB tier 1 ingestion costs for useless data.

### 2. Rightsizing
Focuses on selectively pruning the scope of the Detective behavior graph by removing high-noise, low-security-value accounts and continuously auditing log source volumes.

### 3. Commitment Discounts
While Detective does not offer standard Savings Plans or Reserved Instances, configuring a centralized Delegated Administrator allows you to aggregate data volume across your organization to unlock steep tiered volume discounts.

### 4. Architecture Changes
Focuses on optimizing upstream systems (like EKS control planes or automated IAM polling) to reduce the underlying API and network noise that Detective automatically ingests.

### 5. Scheduling & Auto-Scaling
Focuses on automating the lifecycle of Detective member accounts and temporarily pausing ingestion during planned, high-noise events that would otherwise inflate costs.

### 6. Pricing Model Optimization
Focuses on establishing robust cost visibility, anomaly detection, and forecasting using AWS Budgets and the built-in Detective Usage dashboard.

### 7. Network & Data Transfer Optimization
Focuses on reducing excessive internal network chattiness, such as aggressive health checks, which indirectly lower the volume of VPC Flow Logs ingested by Detective.

---
## Cross-Service Synergies
- **Amazon GuardDuty:** Detective integrates closely with GuardDuty; optimizing GuardDuty deployment scopes often aligns with where Detective should be enabled.
- **AWS CloudTrail & VPC Flow Logs:** Reducing unnecessary API calls or excessive network traffic not only reduces CloudWatch/CloudTrail/VPC costs but directly reduces Detective ingestion bills.
- **AWS Organizations:** Centralizing Detective via Organizations is critical to pooling data volumes and unlocking the 50-87% discounts found in higher ingestion tiers.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Analyze the `lineItem/ProductCode` for `AmazonDetective`.
- Review `lineItem/UsageType` to identify specific regions driving ingestion costs.
- Review `lineItem/BlendedRate` vs `lineItem/UnblendedRate` to measure the impact of tiered volume pricing.

### B. CloudWatch Metrics
- Not directly applicable for Detective ingestion, but useful for monitoring upstream network traffic (VPC flow rates) or API call rates that influence Detective ingestion.

### C. AWS Config / Trusted Advisor
- Identify which accounts and regions have Amazon Detective enabled.
- Check AWS Organizations configurations for the Detective Delegated Administrator.

### D. Company Policies
- Review Security Operations Center (SOC) runbooks to determine which environments (e.g., Sandbox, Dev) require full interactive timeline investigations versus simple GuardDuty alerts.

### E. IaC (Optional)
- Terraform/CloudFormation templates to ensure Detective is only provisioned in approved, production-grade landing zones.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "DETECTIVE-REQ-001",
  "strategy_category": "Waste Elimination",
  "account_id": "123456789012",
  "region": "us-east-1",
  "resource_id": "arn:aws:detective:us-east-1:123456789012:graph:default",
  "estimated_monthly_savings": 500.00,
  "recommendation": "Disable Detective in Sandbox Account",
  "effort_level": "Low"
}
```
### Summary Report Table

| Strategy | Category | Est. Savings | Risk Level | Scope |
|----------|----------|--------------|------------|-------|
| 1. Disable in Non-Prod | Waste Elimination | 20-40% | Low | FinOps / Security |
| 2. Centralize Admin | Commitment Discounts | 30-50% | Low | Security / FinOps |
| 3. De-register Regions | Waste Elimination | 10-30% | Medium | Security |
| ... | ... | ... | ... | ... |

---
#### 1. Disable Detective in Non-Production Accounts
- **What:** Disable Amazon Detective in development, staging, sandbox, or training accounts.
- **Why It Saves Money:** Non-prod environments often generate high volumes of noisy test traffic (VPC Flow Logs) and API calls. Avoiding the initial $2.00/GB charge for these accounts saves significant money for environments that the SOC doesn't actively investigate.
- **Implementation Steps:**
  1. Identify all non-production AWS accounts via AWS Organizations.
  2. Log into the Detective Delegated Administrator account.
  3. Disassociate the non-prod member accounts from the Detective behavior graph.
  4. Ensure automated provisioning scripts (IaC) do not re-enable it.
- **Estimated Savings:** 20-40% of total Detective bill.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Security Team
- **Prerequisites:** Clear environment tagging.

#### 2. De-register Unused AWS Regions
- **What:** Disable Detective in AWS regions where you have no active workloads.
- **Why It Saves Money:** Detective is a regional service. Enabling it in regions with minimal usage means you pay the highest tier ($2.00/GB) for small amounts of background noise, without ever reaching cheaper tiers.
- **Implementation Steps:**
  1. Review AWS CUR to find regions with less than 50 GB/month of Detective ingestion.
  2. Verify with architecture teams that these regions host no active workloads.
  3. Disable the Detective graph in those regions.
- **Estimated Savings:** 5-15%
- **Risk Level:** Low
- **Implementation Scope:** Security Team
- **Prerequisites:** Global workload visibility.

#### 3. Remove Defunct Accounts from the Behavior Graph
- **What:** Disassociate suspended, pending-closure, or defunct accounts from the Detective graph.
- **Why It Saves Money:** Even inactive accounts generate background API noise (e.g., AWS Config, automated scanners). Detective ingests this noise.
- **Implementation Steps:**
  1. Audit AWS Organizations for accounts in "Suspended" status.
  2. Remove these accounts from the Detective Delegated Administrator.
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Security Team
- **Prerequisites:** Account lifecycle management process.

#### 4. Disable Detective if Not Actively Used by SOC
- **What:** Completely disable Amazon Detective if your security team does not perform manual root-cause analysis or use the interactive graphs.
- **Why It Saves Money:** If your organization relies solely on automated blocking or GuardDuty alerts and never logs into Detective, the ingestion costs ($2.00/GB) provide zero ROI.
- **Implementation Steps:**
  1. Audit SOC runbooks and console login history for the Detective service.
  2. If unused, disable Detective entirely and rely on GuardDuty and Security Hub.
- **Estimated Savings:** 100% of Detective costs.
- **Risk Level:** High (Reduces investigative capabilities)
- **Implementation Scope:** Security Leadership
- **Prerequisites:** SOC approval.

#### 5. Disable Detective in Organization Root Account
- **What:** Prevent Detective from running in the AWS Organization Management (Root) account.
- **Why It Saves Money:** The management account should only be used for billing and SCPs. Enabling Detective here processes unnecessary background logs.
- **Implementation Steps:**
  1. Log into the Detective Admin account.
  2. Verify the Organization Management account is not an enrolled member.
- **Estimated Savings:** <1%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Security Team
- **Prerequisites:** Best-practice AWS Org structure.

#### 6. Exclude High-Noise, Low-Value Accounts (Load Testing)
- **What:** Remove accounts dedicated to load testing, performance testing, or chaos engineering from Detective.
- **Why It Saves Money:** Load testing generates astronomical amounts of VPC Flow Logs and API calls. Detective will blindly ingest these at up to $2.00/GB, causing massive bill spikes for non-malicious traffic.
- **Implementation Steps:**
  1. Identify performance testing accounts.
  2. Temporarily or permanently remove them from the Detective behavior graph.
- **Estimated Savings:** 10-50% (Highly variable based on testing frequency)
- **Risk Level:** Medium
- **Implementation Scope:** DevOps | Security Team
- **Prerequisites:** Segregated testing accounts.

#### 7. Audit Data Volumes via the 30-Day Free Trial Panel
- **What:** Use the free trial period to accurately project costs and identify the loudest data sources before actual billing begins.
- **Why It Saves Money:** Allows you to realize that a specific workload (e.g., a chatty EKS cluster) will cost $10k/month in Detective fees before you are actually charged, giving you time to optimize or opt-out.
- **Implementation Steps:**
  1. Enable Detective.
  2. Check the "Usage" tab in the Detective console daily during the 30-day trial.
  3. Analyze the split between CloudTrail, VPC Flow Logs, and EKS audit logs.
- **Estimated Savings:** Varies (Cost Avoidance)
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Security Team
- **Prerequisites:** Within the first 30 days of activation.

#### 8. Consolidate Detective under a Delegated Administrator
- **What:** Use AWS Organizations to set a single Delegated Administrator account for Detective, enrolling all member accounts into a single behavior graph per region.
- **Why It Saves Money:** Detective pricing is tiered (e.g., $2.00/GB for the first 1TB, down to $0.25/GB after 10TB). If accounts are standalone, every account pays $2.00 for its first 1TB. Centralizing pools the volume, pushing the bulk of your data into the 50-87% cheaper tiers.
- **Implementation Steps:**
  1. In AWS Organizations, designate a Security Tooling account as the Detective Delegated Administrator.
  2. Auto-enroll member accounts into this central graph.
- **Estimated Savings:** 30-60%
- **Risk Level:** Low
- **Implementation Scope:** Security Team | DevOps
- **Prerequisites:** AWS Organizations enabled.

#### 9. Leverage AWS Enterprise Discount Program (EDP)
- **What:** Ensure Detective spend is included in EDP negotiations.
- **Why It Saves Money:** An EDP provides a flat percentage discount across all AWS services, including Detective's ingestion costs.
- **Implementation Steps:**
  1. Work with AWS account manager.
  2. Project future Detective ingestion volumes during EDP renewal.
- **Estimated Savings:** 5-15% (depending on EDP terms)
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** High overall AWS spend.

#### 10. Optimize IAM Roles and Automated API Calls
- **What:** Refactor excessive, automated API polling by third-party tools or aggressive lambda functions.
- **Why It Saves Money:** Detective ingests CloudTrail management events. If an automation script calls `DescribeInstances` 100 times a second, it inflates CloudTrail volume, directly increasing Detective ingestion costs.
- **Implementation Steps:**
  1. Use Athena to query CloudTrail for the highest volume API callers.
  2. Refactor scripts to use event-driven architectures (EventBridge) rather than polling.
- **Estimated Savings:** 5-15%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudTrail analysis capabilities.

#### 11. Reduce EKS Control Plane Noise
- **What:** Optimize Kubernetes controllers and probes to reduce API server chattiness.
- **Why It Saves Money:** Detective ingests EKS audit logs. Highly aggressive custom controllers, overly frequent liveness probes, or excessive configmap updates generate massive EKS audit logs, inflating Detective costs.
- **Implementation Steps:**
  1. Review EKS audit logs for high-frequency user-agents or service accounts.
  2. Adjust controller sync periods and probe intervals.
- **Estimated Savings:** 10-25%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** EKS workloads.

#### 12. Consolidate Microservices to Reduce Network Chattiness
- **What:** Optimize inter-service communication to reduce internal network traffic.
- **Why It Saves Money:** Detective automatically ingests VPC Flow Logs. Highly chatty microservices (e.g., sending millions of tiny packets) generate massive flow log volume, leading to high Detective ingestion costs.
- **Implementation Steps:**
  1. Implement gRPC or connection pooling.
  2. Batch payloads instead of sending continuous micro-requests.
- **Estimated Savings:** 10-20%
- **Risk Level:** High (Requires code changes)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Microservices architecture.

#### 13. Suspend Detective During Massive Data Migrations
- **What:** Temporarily remove target accounts from the Detective graph during known, massive data transfers or database replications.
- **Why It Saves Money:** Petabyte-scale migrations generate extreme VPC Flow Logs. Detective will ingest all of this at standard per-GB rates, causing a massive, unexpected one-time bill spike.
- **Implementation Steps:**
  1. Coordinate with migration teams.
  2. Disassociate the account from Detective prior to the migration.
  3. Re-associate post-migration.
- **Estimated Savings:** Prevents 100-500% monthly bill spikes.
- **Risk Level:** Medium (Blind spot during migration)
- **Implementation Scope:** Security Team | DevOps
- **Prerequisites:** Planned migration windows.

#### 14. Automate Member Account Enrollment Based on Tags
- **What:** Use a Lambda function to automatically add or remove accounts from the Detective graph based on account-level tags (e.g., `SecurityLevel: High`).
- **Why It Saves Money:** Prevents human error where sandbox or test accounts are accidentally enrolled in Detective, generating unnecessary ingestion costs.
- **Implementation Steps:**
  1. Create an EventBridge rule listening for AWS Organizations tag changes.
  2. Trigger a Lambda function that calls `CreateMembers` or `DeleteMembers` on the Detective API.
- **Estimated Savings:** 5-10%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Account vending machine / robust tagging strategy.

#### 15. Set AWS Budgets Alerts Specifically for Detective
- **What:** Create a granular AWS Budget that tracks only `AmazonDetective` product code spend.
- **Why It Saves Money:** Detects runaway ingestion (e.g., a dev spinning up a load test that spikes VPC Flow logs) before the end of the month, allowing for rapid intervention.
- **Implementation Steps:**
  1. Go to AWS Budgets.
  2. Create a cost budget filtered by Service = Amazon Detective.
  3. Set alerts at 80% and 100% of expected monthly spend.
- **Estimated Savings:** Cost Avoidance
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Standard cost baselines.

#### 16. Monitor Detective Usage Dashboard to Forecast Tier Boundaries
- **What:** Regularly check the built-in Detective Usage dashboard to predict when you will cross into cheaper pricing tiers.
- **Why It Saves Money:** Understanding your ingestion trajectory helps you accurately forecast costs and justify consolidating accounts to push volume into the $0.25/GB tier.
- **Implementation Steps:**
  1. Access the Detective console.
  2. Review the Usage dashboard to view 30-day projected volume.
  3. Feed this data into FinOps forecasting models.
- **Estimated Savings:** Visibility enhancement
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

#### 17. Reduce Excessive Health Checks
- **What:** Tune Load Balancer, Route 53, and internal monitoring tool health check frequencies.
- **Why It Saves Money:** Extremely frequent health checks (e.g., every 1 second) across thousands of targets generate massive VPC Flow Log metadata. Detective ingests this metadata at standard rates.
- **Implementation Steps:**
  1. Increase health check intervals from 5 seconds to 30 seconds where acceptable.
  2. Consolidate monitoring tool agents to reduce polling noise.
- **Estimated Savings:** 2-10%
- **Risk Level:** Medium (Slightly slower failure detection)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Network visibility.

#### 18. Optimize DNS and NAT Gateway Traffic Patterns
- **What:** Reduce external DNS lookups or excessive NAT Gateway usage.
- **Why It Saves Money:** Chatty external routing generates significant VPC Flow logs. Keeping traffic internal (using VPC Endpoints) or caching DNS queries reduces the volume of flow logs generated and subsequently ingested by Detective.
- **Implementation Steps:**
  1. Implement local DNS caching (e.g., NodeLocal DNSCache in EKS).
  2. Deploy VPC Endpoints (PrivateLink) for AWS services to streamline traffic routing.
- **Estimated Savings:** 5-15%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Network architecture review.
