# Cost-Cutting Playbook: AWS Systems Manager
> **Companion File:** [systems_manager.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/systems_manager/systems_manager.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Systems Manager (SSM) provides powerful fleet management, configuration, and automation capabilities. While core EC2 management features (Session Manager, Run Command, Patch Manager) are 100% free, costs can quickly accumulate from advanced features, on-premises node management, Parameter Store high throughput, and third-party dependencies. This playbook outlines strategies to leverage SSM's free tier to eliminate external tool costs, optimize API access, and minimize advanced feature billing.

## Strategy Categories
### 1. Waste Elimination

#### SSM-01. Terminate SSH Bastion Host EC2 Instances
- **What:** Replace costly EC2 SSH bastion hosts with AWS Systems Manager Session Manager.
- **Why It Saves Money:** Session Manager is 100% free for EC2 management. Eliminates the compute ($15-$50/mo) and public IPv4 ($3.60/mo) costs of bastion hosts.
- **Implementation Steps:** 
  1. Ensure SSM Agent is installed on target instances. 
  2. Attach IAM role with `AmazonSSMManagedInstanceCore`. 
  3. Access instances via Session Manager. 
  4. Terminate the bastion EC2 instances.
- **Estimated Savings:** $20-$100/month per environment
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SSM Agent installed, appropriate IAM roles configured.

#### SSM-02. Migrate Non-Sensitive Data from Secrets Manager to Parameter Store
- **What:** Move application configuration, feature flags, and non-sensitive API endpoints from AWS Secrets Manager to SSM Standard Parameter Store.
- **Why It Saves Money:** Standard Parameter Store is 100% free, whereas Secrets Manager costs $0.40 per secret per month plus $0.05 per 10,000 API calls.
- **Implementation Steps:** 
  1. Identify non-sensitive configuration currently in Secrets Manager. 
  2. Create equivalent Standard Parameters in SSM. 
  3. Update application code to fetch from SSM. 
  4. Delete the old secrets.
- **Estimated Savings:** 100% of Secrets Manager costs for migrated items
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Identification of strictly non-sensitive configuration data.

#### SSM-03. Disable High Throughput API for Parameter Store Unnecessarily
- **What:** Turn off the High Throughput setting for Parameter Store if the request volume is safely below the standard 40 requests per second limit.
- **Why It Saves Money:** Standard API throughput is free. High throughput costs $5.00 per million requests.
- **Implementation Steps:** 
  1. Review CloudWatch metrics for Parameter Store API usage. 
  2. Identify accounts/regions with High Throughput enabled. 
  3. Disable the setting via SSM console or CLI if usage is well below 40 req/sec.
- **Estimated Savings:** $5.00 per million requests avoided
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** CloudWatch metrics analysis of `GetParameter` calls.

#### SSM-04. Deregister Unused On-Premises Managed Instances
- **What:** Identify and deregister stale or retired on-premises virtual machines and edge devices from Systems Manager.
- **Why It Saves Money:** Advanced On-Premises Managed Instances bill at $0.0069 per instance-hour (~$5.00/mo). Deregistering unused instances stops this billing.
- **Implementation Steps:** 
  1. Go to Fleet Manager in the SSM Console. 
  2. Filter for instances with prefix `mi-` that are offline or unused. 
  3. Deregister the instances from Systems Manager.
- **Estimated Savings:** ~$5.00 per instance/month
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Verification that the on-premise instance is no longer needed.

#### SSM-05. Consolidate Incident Manager Response Plans
- **What:** Clean up unused or duplicate response plans in AWS Systems Manager Incident Manager.
- **Why It Saves Money:** Incident Manager charges $7.00 per response plan per month.
- **Implementation Steps:** 
  1. Audit all Incident Manager response plans across accounts. 
  2. Identify unused or redundant plans. 
  3. Delete unused plans and consolidate similar workflows into single plans.
- **Estimated Savings:** $7.00/month per deleted plan
- **Risk Level:** Low
- **Implementation Scope:** DevOps | Site Reliability Engineering (SRE)
- **Prerequisites:** Review of incident management processes.

#### SSM-06. Downgrade Advanced Parameters to Standard Parameters
- **What:** Revert Advanced Parameters to Standard Parameters if they do not exceed 4 KB in size and do not require parameter policies.
- **Why It Saves Money:** Standard Parameters are free. Advanced Parameters cost $0.05 per parameter per month.
- **Implementation Steps:** 
  1. List all Advanced Parameters. 
  2. Identify those under 4 KB with no expiration policies. 
  3. Recreate them as Standard Parameters and update application references. 
  4. Delete the Advanced Parameters.
- **Estimated Savings:** $0.05/month per parameter
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Parameter sizes must be <= 4 KB.

### 2. Rightsizing

#### SSM-07. Optimize Automation Step-Minutes
- **What:** Refactor and optimize SSM Automation runbooks to execute faster and consume fewer step-minutes.
- **Why It Saves Money:** The first 5,000 step-minutes are free per month. Excess step-minutes are billed at $0.002 per step-minute. Fast-executing runbooks stay within the free tier.
- **Implementation Steps:** 
  1. Review Runbook execution durations in the SSM Console. 
  2. Identify slow-running scripts (e.g., long wait loops). 
  3. Refactor logic to use event-driven triggers rather than polling waits inside the step.
- **Estimated Savings:** 10-30% on Automation costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Familiarity with SSM Automation documents.

#### SSM-08. Consolidate Change Manager Requests
- **What:** Batch multiple operational changes into single Change Manager requests rather than raising individual requests for trivial operations.
- **Why It Saves Money:** Change Manager charges $0.29 per change request. Consolidating reduces the request count.
- **Implementation Steps:** 
  1. Review change management procedures. 
  2. Group routine, related updates into a single runbook/change request execution.
- **Estimated Savings:** Up to 50% on Change Manager billing
- **Risk Level:** Low
- **Implementation Scope:** DevOps | Change Management Team
- **Prerequisites:** Alignment with organizational change policies.

### 3. Commitment Discounts
*Systems Manager does not have direct commitment discounts like Savings Plans or Reserved Instances, as its billing is purely on-demand consumption.*

### 4. Architecture Changes

#### SSM-09. Cache Parameter Store Values in Application Memory
- **What:** Implement in-memory caching for Parameter Store values in Lambda functions, ECS containers, or microservices with a TTL (e.g., 5-15 minutes).
- **Why It Saves Money:** Drastically reduces `GetParameter` API calls, keeping requests safely under the 40 req/sec limit to avoid $5.00/M request High Throughput fees.
- **Implementation Steps:** 
  1. Use AWS Lambda Powertools or custom caching logic in the application layer. 
  2. Set an appropriate TTL based on how frequently config changes. 
  3. Deploy the updated code.
- **Estimated Savings:** 90%+ reduction in Parameter Store API costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Applications must tolerate slight eventual consistency (TTL duration) for config updates.

#### SSM-10. Replace Third-Party Patch Tools with SSM Patch Manager
- **What:** Migrate from expensive third-party patch management systems (e.g., WSUS, SCCM, Ansible Tower) to native SSM Patch Manager.
- **Why It Saves Money:** SSM Patch Manager for EC2 instances is 100% free, eliminating third-party software licensing and infrastructure costs.
- **Implementation Steps:** 
  1. Define Patch Baselines in SSM. 
  2. Configure Maintenance Windows. 
  3. Test patching on non-prod instances. 
  4. Decommission third-party patching infrastructure.
- **Estimated Savings:** 100% of 3rd party licensing and hosting costs
- **Risk Level:** Medium
- **Implementation Scope:** IT Operations | DevOps
- **Prerequisites:** EC2 instances must be managed by SSM.

#### SSM-11. Replace Third-Party Incident Management with Incident Manager
- **What:** For simpler use cases, use SSM Incident Manager instead of dedicated platforms like PagerDuty or Opsgenie.
- **Why It Saves Money:** Eliminates expensive per-seat licenses of third-party tools, relying instead on Incident Manager's flat $7.00/month response plan fee (which includes 100 SMS/voice calls).
- **Implementation Steps:** 
  1. Evaluate Incident Manager features against current third-party tool usage. 
  2. Recreate on-call schedules and escalation paths. 
  3. Migrate alerts from CloudWatch to Incident Manager.
- **Estimated Savings:** $10-$50/user/month in licensing
- **Risk Level:** High
- **Implementation Scope:** SRE | IT Operations
- **Prerequisites:** Assessment of required incident management capabilities.

#### SSM-12. Implement VPC Endpoints (PrivateLink) for SSM
- **What:** Deploy Interface VPC Endpoints for Systems Manager (`ssm`, `ssmmessages`, `ec2messages`) in private subnets instead of routing SSM traffic through NAT Gateways.
- **Why It Saves Money:** NAT Gateways charge $0.045/GB for data processing. VPC Endpoints charge $0.01/GB, cutting data transfer costs for SSM traffic (e.g., Session Manager, Patch Manager) by 75%, or even more if avoiding NAT Gateway entirely.
- **Implementation Steps:** 
  1. Create Interface VPC Endpoints for SSM services. 
  2. Associate them with private subnet route tables. 
  3. Ensure security groups allow port 443 traffic from VPC CIDR.
- **Estimated Savings:** 75% on SSM-related NAT Data Processing
- **Risk Level:** Low
- **Implementation Scope:** Network Engineer | DevOps
- **Prerequisites:** VPC architecture with private subnets.

### 5. Scheduling & Auto-Scaling

#### SSM-13. Automate Non-Production Instance Scheduling with SSM
- **What:** Use SSM Automation and Maintenance Windows (or AWS Instance Scheduler which uses SSM under the hood) to stop EC2/RDS instances outside of working hours.
- **Why It Saves Money:** Halting instances for 12-14 hours a day and on weekends cuts compute costs by up to 70%.
- **Implementation Steps:** 
  1. Tag target non-production instances (e.g., `Schedule: BusinessHours`). 
  2. Set up an SSM Maintenance Window or EventBridge schedule triggering an SSM Automation document (`AWS-StopEC2Instance` and `AWS-StartEC2Instance`).
- **Estimated Savings:** 60-70% on non-prod compute costs
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Consistent tagging strategy.

#### SSM-14. Schedule Patching During Low-Traffic Hours
- **What:** Use SSM Maintenance Windows to schedule patch operations only during off-peak hours when auto-scaling groups are at minimum capacity.
- **Why It Saves Money:** Patching requires compute resources. Patching during low-traffic periods avoids having to scale up extra instances to handle both user traffic and heavy patching loads simultaneously.
- **Implementation Steps:** 
  1. Configure SSM Maintenance Windows for late-night/weekend hours. 
  2. Target specific instance tags.
- **Estimated Savings:** Avoids temporary compute spikes
- **Risk Level:** Low
- **Implementation Scope:** IT Operations | DevOps
- **Prerequisites:** Understanding of application traffic patterns.

### 6. Pricing Model Optimization

#### SSM-15. Default to Standard Parameters for New Projects
- **What:** Establish a company-wide policy and Infrastructure as Code (IaC) guardrails enforcing Standard Parameters by default for all new SSM Parameter Store entries.
- **Why It Saves Money:** Prevents accidental provisioning of Advanced Parameters ($0.05/mo) when Standard Parameters (Free) suffice.
- **Implementation Steps:** 
  1. Update Terraform/CloudFormation modules to default `tier` to `Standard`. 
  2. Implement AWS Config rules to flag non-compliant Advanced Parameters.
- **Estimated Savings:** Cost avoidance of $0.05 per parameter
- **Risk Level:** Low
- **Implementation Scope:** Cloud Architect | DevOps
- **Prerequisites:** CI/CD pipeline and IaC module governance.

### 7. Network & Data Transfer Optimization

#### SSM-16. Deploy SSM Endpoints in the Same AZ as Instances
- **What:** When setting up Interface VPC Endpoints for Systems Manager, ensure the endpoint ENIs are deployed in every Availability Zone where managed instances reside.
- **Why It Saves Money:** Prevents instances from crossing AZ boundaries to communicate with the SSM endpoint, avoiding the $0.01/GB cross-AZ data transfer charge.
- **Implementation Steps:** 
  1. Modify VPC Endpoint configuration. 
  2. Select subnets in all active AZs for the `ssm`, `ssmmessages`, and `ec2messages` endpoints.
- **Estimated Savings:** $0.01 per GB of SSM traffic
- **Risk Level:** Low
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Multi-AZ deployment.

#### SSM-17. Use S3 Gateway Endpoints for Patch Manager Downloads
- **What:** Ensure an S3 Gateway Endpoint is present in the VPC so that instances downloading patch baselines and updates from AWS S3 buckets route traffic privately and free of charge.
- **Why It Saves Money:** Patch downloads (especially Windows updates) can be gigabytes in size. Routing them through NAT Gateways incurs $0.045/GB. S3 Gateway Endpoints are free and data transfer to S3 within the same region is free.
- **Implementation Steps:** 
  1. Go to VPC Console. 
  2. Create a Gateway Endpoint for S3. 
  3. Associate it with the private subnet route tables.
- **Estimated Savings:** $0.045 per GB of patch data
- **Risk Level:** Low
- **Implementation Scope:** Network Engineer | DevOps
- **Prerequisites:** Private subnets requiring internet/S3 access.

---
## Cross-Service Synergies
*   **SSM & EC2/RDS:** SSM Automation powers the scheduling that directly slashes EC2 and RDS compute bills.
*   **SSM & Lambda:** Parameter Store acts as the zero-cost config backend for Lambda functions, provided in-memory caching is used to avoid API fees.
*   **SSM & VPC:** VPC Endpoints ensure SSM management traffic (like Session Manager and Patch Manager) avoids costly NAT Gateway data processing charges.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
*   `lineItem/ProductCode` = `AmazonEC2` (for Systems Manager usage types) or `AWSSystemsManager`.
*   Look for usage types like `GetParameter-HighThroughput` or `Advanced-Parameter-Month`.

### B. CloudWatch Metrics
*   SSM `GetParameter` request volume (to evaluate Standard vs. High Throughput tiers).
*   Runbook execution duration metrics.

### C. AWS Config / Trusted Advisor
*   List of EC2 instances with public IPs and port 22/3389 open (candidates for Session Manager).
*   Inventory of Secrets Manager secrets lacking rotation schedules (candidates for Parameter Store).

### D. Company Policies
*   Patching SLAs and configuration management requirements.
*   Security requirements regarding bastion hosts and SSH access.

### E. IaC (Optional)
*   Terraform modules deploying Secrets Manager, Parameter Store, and EC2 instances.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "SSM-01",
  "strategy_category": "Waste Elimination",
  "resource_type": "EC2 Bastion / SSM Session Manager",
  "recommended_action": "Terminate SSH Bastion Host EC2 Instances",
  "estimated_monthly_savings": 50.00,
  "effort_level": "Low"
}
```

### Summary Report Table
| Finding ID | Strategy | Category | Est. Savings | Effort |
|------------|----------|----------|--------------|--------|
| SSM-01 | Terminate SSH Bastion Hosts | Waste Elimination | $50/env | Low |
| SSM-02 | Migrate to Standard Parameter Store | Waste Elimination | Varies | Low |
| SSM-03 | Disable High Throughput API | Waste Elimination | $5.00/1M reqs | Low |
| SSM-04 | Deregister On-Premises Instances | Waste Elimination | ~$5.00/inst | Low |
| SSM-05 | Consolidate Incident Response Plans | Waste Elimination | $7.00/plan | Low |
| SSM-06 | Downgrade Advanced Parameters | Waste Elimination | $0.05/param | Low |
| SSM-07 | Optimize Automation Step-Minutes | Rightsizing | 10-30% Cost | Medium |
| SSM-08 | Consolidate Change Manager Requests | Rightsizing | Up to 50% | Low |
| SSM-09 | Cache Parameter Store in Memory | Architecture Changes | >90% API Cost | Low |
| SSM-10 | Replace 3rd Party Patching Tools | Architecture Changes | 100% License Cost | Medium |
| SSM-11 | Use Incident Manager for Basic Ops | Architecture Changes | $10-$50/user | High |
| SSM-12 | Implement VPC Endpoints for SSM | Architecture Changes | 75% NAT Cost | Low |
| SSM-13 | Automate Non-Prod Instance Scheduling | Scheduling | 60-70% Compute | Low |
| SSM-14 | Schedule Patching During Low Traffic | Scheduling | Avoid Spikes | Low |
| SSM-15 | Default to Standard Parameters | Pricing Model Opt | $0.05/param | Low |
| SSM-16 | Deploy Endpoints in Same AZ | Network & Data Opt | $0.01/GB | Low |
| SSM-17 | S3 Gateway Endpoints for Patches | Network & Data Opt | $0.045/GB | Low |
