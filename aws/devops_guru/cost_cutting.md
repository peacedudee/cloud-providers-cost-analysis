# Cost-Cutting Playbook: Amazon DevOps Guru
> **Companion File:** [devops_guru.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/devops_guru/devops_guru.md)
> **Last Updated:** July 2026
---
## Executive Summary
Amazon DevOps Guru is a machine-learning powered service designed to identify anomalous behavior in AWS applications. Because it bills per active resource per hour, costs can rapidly spiral if applied indiscriminately across an entire AWS account. This playbook focuses on scope management, pinpointing critical resources, and avoiding charges for ephemeral or non-critical components. By transitioning from account-wide coverage to targeted Resource Groups, organizations can typically reduce their DevOps Guru spend by 60% to 90% without compromising production reliability.

## Strategy Categories

### 1. Waste Elimination

#### 1. Disable Account-Wide Monitoring
- **What:** Disable the "All resources in the current AWS account" setting in DevOps Guru.
- **Why It Saves Money:** Prevents the service from automatically analyzing all sandbox, developer, and legacy testing resources, which bill at ~$2.04 or ~$3.06 per active resource per month.
- **Implementation Steps:**
  1. Navigate to DevOps Guru console > Settings.
  2. Under Monitored resources, edit the configuration.
  3. Deselect "All resources in the current AWS account".
- **Estimated Savings:** 40-80%
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | DevOps Team
- **Prerequisites:** Must have a targeted monitoring strategy ready to replace it.

#### 2. Scope Monitoring using Free AWS Resource Groups
- **What:** Use AWS Resource Groups to define strict coverage for DevOps Guru monitoring based on production tags.
- **Why It Saves Money:** AWS Resource Groups is a 100% free service ($0.00). Using it ensures you only pay to monitor critical, tagged applications, bypassing dev/test resource billing.
- **Implementation Steps:**
  1. Go to AWS Resource Groups and create a group using tag filters (e.g., `Environment=Production`).
  2. In DevOps Guru Settings, choose "Choose later or use AWS CloudFormation / AWS Resource Groups".
  3. Select the specific Resource Group you created.
- **Estimated Savings:** 60-90%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Consistent resource tagging strategy.

#### 3. Disable Coverage on Auto-Scaling Compute Tiers
- **What:** Exclude individual ephemeral container tasks or auto-scaling EC2 nodes from Resource Groups monitored by DevOps Guru.
- **Why It Saves Money:** Prevents accumulating costs from hundreds of auto-scaling tasks that run briefly but generate active resource hours ($0.0042 per hour for Group B).
- **Implementation Steps:**
  1. Adjust Resource Group tags to exclude auto-scaling nodes.
  2. Ensure DevOps Guru monitors the Application Load Balancer (ALB) and RDS database, which provide aggregate anomaly data.
- **Estimated Savings:** 20-50%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Architecture relying on load balancers.

#### 4. Review and Cleanup Stale Resource Groups
- **What:** Periodically audit AWS Resource Groups used by DevOps Guru and delete those tracking deprecated apps.
- **Why It Saves Money:** Stops paying for resources that are "active" (producing logs/metrics) but no longer part of your core business operations.
- **Implementation Steps:**
  1. List all Resource Groups integrated with DevOps Guru.
  2. Cross-reference with active product environments.
  3. Remove legacy Resource Groups from DevOps Guru coverage.
- **Estimated Savings:** 5-15%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Maintain a registry of active services.

#### 5. Tag-Based Exclusion Rules
- **What:** Actively tag resources with `DevOpsGuru=Exclude` and ensure your Resource Group filters them out.
- **Why It Saves Money:** Prevents developers from accidentally opting-in non-critical resources when using broad tags like `App=MyBackend`.
- **Implementation Steps:**
  1. Implement an organizational tagging policy enforcing `DevOpsGuru` tags.
  2. Update Resource Group definitions to exclude resources with the exclusion tag.
- **Estimated Savings:** 10-20%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Enforcement of tagging policies (e.g., via SCPs).

#### 6. Limit Custom API Dashboard Queries
- **What:** Reduce the frequency of API calls made by custom scripts or third-party tools exporting DevOps Guru data.
- **Why It Saves Money:** API requests for custom dashboard queries are billed at $0.00004 per request. Aggressive polling can accumulate costs.
- **Implementation Steps:**
  1. Review scripts polling the DevOps Guru API.
  2. Reduce polling frequency from per-minute to per-hour or event-driven.
- **Estimated Savings:** < 1%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Identify external tools polling the API.

#### 7. Exclude Ephemeral CI/CD Resources
- **What:** Ensure resources spun up purely for automated testing during CI/CD pipelines are excluded from DevOps Guru.
- **Why It Saves Money:** CI/CD resources are highly active for brief periods, incurring Group A/B resource hour charges without providing long-term anomaly value.
- **Implementation Steps:**
  1. Tag CI/CD resources with `Environment=CICD`.
  2. Ensure these tags are strictly excluded from the monitored Resource Groups.
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CI/CD pipeline tagging automation.

### 2. Rightsizing

#### 8. Prune Lightweight Services (Group A) from Monitored Scopes
- **What:** Remove secondary Group A components (e.g., non-critical SQS queues, SNS topics, or S3 buckets) from monitored Resource Groups.
- **Why It Saves Money:** Saves ~$2.04/month per excluded resource. If an application has hundreds of queues, savings scale rapidly.
- **Implementation Steps:**
  1. Analyze the architecture of the monitored application.
  2. Remove tags that include auxiliary SQS/S3 resources.
  3. Keep monitoring on core compute (Lambda) and databases (DynamoDB).
- **Estimated Savings:** 15-35%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Architectural understanding of failure domains.

#### 9. Target High-Value Group B Resources
- **What:** Prioritize monitoring for Group B resources (RDS, ECS, ALB) where downtime is expensive, rather than simple, highly resilient Group A resources.
- **Why It Saves Money:** Optimizes the $3.06/month spend per Group B resource by focusing on the most critical failure points rather than a blanket coverage approach.
- **Implementation Steps:**
  1. Identify stateful, critical infrastructure (databases, load balancers).
  2. Create a dedicated Resource Group for "Critical Infrastructure" and point DevOps Guru there.
- **Estimated Savings:** 20-40%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application risk assessment.

#### 10. Filter by Critical CloudFormation Stacks
- **What:** Scope DevOps Guru monitoring strictly to critical CloudFormation stacks rather than using global tags.
- **Why It Saves Money:** Ensures precise boundaries around what generates resource-hours, eliminating sprawl from manually created untagged resources.
- **Implementation Steps:**
  1. In DevOps Guru, choose coverage by AWS CloudFormation stacks.
  2. Select only the stacks representing Tier-1 production services.
- **Estimated Savings:** 30-60%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Use of CloudFormation for infrastructure as code.

### 3. Commitment Discounts

*Note: Amazon DevOps Guru is a pay-as-you-go serverless service and currently does not support Savings Plans or Reserved Instances. Cost reduction must rely entirely on scope and resource management.*

### 4. Architecture Changes

#### 11. Consolidate Monitored Microservices
- **What:** Combine smaller, related Lambda functions into a single Lambda function if they do not require independent anomaly detection.
- **Why It Saves Money:** Reduces the absolute count of Group A resources monitored, saving $2.04/month per removed Lambda function.
- **Implementation Steps:**
  1. Review Lambda usage for overly fragmented microservices.
  2. Refactor code to use fewer, consolidated functions.
- **Estimated Savings:** 5-15%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Development resources for code refactoring.

#### 12. Centralize DevOps Guru via Delegated Admin Account
- **What:** Use AWS Organizations delegated administrator to centrally manage DevOps Guru rather than enabling it ad-hoc in every account.
- **Why It Saves Money:** Prevents overlapping monitoring or rogue enablement in developer accounts, ensuring FinOps teams have centralized control over what incurs charges.
- **Implementation Steps:**
  1. Register a delegated administrator account for DevOps Guru in AWS Organizations.
  2. Enforce monitoring scopes globally.
- **Estimated Savings:** 10-20%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** AWS Organizations enabled.

#### 13. Leverage EventBridge for Intelligent Alert Routing
- **What:** Route DevOps Guru alerts via Amazon EventBridge to a single centralized logging/alerting system.
- **Why It Saves Money:** Consolidates downstream processing costs, avoiding fanning out anomalies to multiple costly SNS topics or Lambda processing functions.
- **Implementation Steps:**
  1. Configure EventBridge rules to capture DevOps Guru insight events.
  2. Route these events to a single centralized webhook or ticketing system.
- **Estimated Savings:** < 5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Centralized observability platform.

### 5. Scheduling & Auto-Scaling

#### 14. Shut Down Underlying Resources Off-Hours
- **What:** Stop non-production EC2 instances, RDS databases, and ECS tasks outside of business hours.
- **Why It Saves Money:** DevOps Guru only bills for resources that are "active" (producing metrics/events) during the billing hour. Shutting down the resources stops the DevOps Guru billing for them simultaneously.
- **Implementation Steps:**
  1. Implement AWS Instance Scheduler.
  2. Schedule shutdown of dev/test environments at night and on weekends.
- **Estimated Savings:** ~65% (on dev/test environments)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 15. Synchronize DevOps Guru Scope with Scheduled Chaos Testing
- **What:** Temporarily remove Resource Groups from DevOps Guru coverage during planned chaos engineering or load testing.
- **Why It Saves Money:** Chaos testing generates massive amounts of events and metric spikes, which can trigger prolonged analysis and active resource hours for otherwise idle components.
- **Implementation Steps:**
  1. Automate the detachment of Resource Groups from DevOps Guru via API before a chaos test.
  2. Reattach them after the test concludes.
- **Estimated Savings:** 2-5%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Automated testing pipelines.

### 6. Pricing Model Optimization

#### 16. Maximize the 3-Month Free Tier Trial
- **What:** Strategically plan the rollout of DevOps Guru to stay within the 7,200 Group A and 3,600 Group B resource-hours per month during the 3-month trial.
- **Why It Saves Money:** Keeps the initial evaluation and baseline training phase 100% free.
- **Implementation Steps:**
  1. Calculate current resources: 3,600 hours / 730 hours in a month = ~4.9 Group B resources running 24/7.
  2. Scope the trial strictly to 4 core resources (e.g., 2 RDS, 2 ALB).
- **Estimated Savings:** 100% (during trial)
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Must be a new DevOps Guru customer.

#### 17. Evaluate Multi-Account Trial Utilization
- **What:** Run proof-of-concept trials in separate, newly provisioned accounts for different business units.
- **Why It Saves Money:** The DevOps Guru Free Tier is applied at the account level. Testing across multiple accounts maximizes the free tier before committing to a centralized rollout.
- **Implementation Steps:**
  1. Deploy infrastructure to testing accounts.
  2. Enable DevOps Guru in each account to utilize the separate free tiers.
- **Estimated Savings:** Varies
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Multi-account structure.

### 7. Network & Data Transfer Optimization

#### 18. Optimize SNS Alert Configurations
- **What:** Limit the use of Amazon SNS for DevOps Guru notifications, or batch them.
- **Why It Saves Money:** Avoids integrated SNS alerts surcharges. If DevOps Guru detects hundreds of anomalies, sending an SNS for each incurs standard SNS message delivery fees.
- **Implementation Steps:**
  1. Use EventBridge instead of direct SNS integrations.
  2. Aggregate events using a Lambda function before sending a digest via SNS.
- **Estimated Savings:** < 5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** EventBridge and Lambda knowledge.

#### 19. Consolidate Cross-Region Dashboards
- **What:** Avoid building custom dashboards that query the DevOps Guru API across every single AWS region continuously.
- **Why It Saves Money:** API requests are billed per region. Querying inactive regions incurs API charges without providing value.
- **Implementation Steps:**
  1. Restrict API polling to regions where production workloads actively reside.
- **Estimated Savings:** < 1%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region architecture.

---

## Cross-Service Synergies
- **AWS Resource Groups & AWS Tags:** DevOps Guru's cost efficiency is entirely dependent on excellent tagging practices and Resource Group management.
- **Amazon EventBridge:** Using EventBridge to route insights instead of SNS reduces alert fatigue and downstream messaging costs.
- **AWS Cost Explorer / CUR:** Use CUR to track the line items for `ResourceHourGroupA` and `ResourceHourGroupB` to identify exactly which resources are driving up the DevOps Guru bill.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Analyze line items under `Amazon DevOps Guru`.
- Group by `UsageType` to differentiate between `ResourceHourGroupA`, `ResourceHourGroupB`, and `APICalls`.
- Group by `ResourceId` to identify which specific resources are incurring the most active hours.

### B. CloudWatch Metrics
- Monitor the volume of anomalies generated to correlate with billing spikes.
- Check active hours metrics for underlying EC2 and RDS instances.

### C. AWS Config / Trusted Advisor
- Use AWS Config to ensure resources are tagged correctly so they fall into the proper DevOps Guru inclusion/exclusion Resource Groups.

### D. Company Policies
- Determine the definition of "Tier-1 Production" to justify which resources warrant the $2-$3/month monitoring premium.

### E. IaC (Optional)
- Terraform or CloudFormation templates to verify that `AWS::DevOpsGuru::ResourceCollection` is strictly defined and not set to account-wide.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "DEVGURU-001",
  "service": "Amazon DevOps Guru",
  "strategy_category": "Waste Elimination",
  "name": "Disable Account-Wide Monitoring",
  "description": "Turn off 'All resources in current AWS account' and use targeted Resource Groups.",
  "savings_percentage": "40-80%",
  "risk_level": "Medium",
  "implementation_scope": "FinOps Team",
  "prerequisites": ["Targeted monitoring strategy ready"]
}
```

### Summary Report Table

| ID | Strategy Name | Category | Est. Savings | Risk | Scope |
|---|---|---|---|---|---|
| DEVGURU-001 | Disable Account-Wide Monitoring | Waste Elimination | 40-80% | Medium | FinOps Team |
| DEVGURU-002 | Scope Monitoring using Free AWS Resource Groups | Waste Elimination | 60-90% | Low | Engineer/DevOps |
| DEVGURU-003 | Disable Coverage on Auto-Scaling Compute Tiers | Waste Elimination | 20-50% | Medium | Engineer/DevOps |
| DEVGURU-004 | Review and Cleanup Stale Resource Groups | Waste Elimination | 5-15% | Low | FinOps Team |
| DEVGURU-005 | Tag-Based Exclusion Rules | Waste Elimination | 10-20% | Low | Engineer/DevOps |
| DEVGURU-006 | Limit Custom API Dashboard Queries | Waste Elimination | < 1% | Low | Engineer/DevOps |
| DEVGURU-007 | Exclude Ephemeral CI/CD Resources | Waste Elimination | 10-30% | Low | Engineer/DevOps |
| DEVGURU-008 | Prune Lightweight Services (Group A) from Monitored Scopes | Rightsizing | 15-35% | Medium | Engineer/DevOps |
| DEVGURU-009 | Target High-Value Group B Resources | Rightsizing | 20-40% | Medium | Engineer/DevOps |
| DEVGURU-010 | Filter by Critical CloudFormation Stacks | Rightsizing | 30-60% | Low | Engineer/DevOps |
| DEVGURU-011 | Consolidate Monitored Microservices | Architecture Changes | 5-15% | High | Engineer/DevOps |
| DEVGURU-012 | Centralize DevOps Guru via Delegated Admin Account | Architecture Changes | 10-20% | Low | FinOps Team |
| DEVGURU-013 | Leverage EventBridge for Intelligent Alert Routing | Architecture Changes | < 5% | Low | Engineer/DevOps |
| DEVGURU-014 | Shut Down Underlying Resources Off-Hours | Scheduling & Auto-Scaling | ~65% | Low | Engineer/DevOps |
| DEVGURU-015 | Synchronize DevOps Guru Scope with Scheduled Chaos Testing | Scheduling & Auto-Scaling | 2-5% | High | Engineer/DevOps |
| DEVGURU-016 | Maximize the 3-Month Free Tier Trial | Pricing Model Optimization | 100% | Low | FinOps Team |
| DEVGURU-017 | Evaluate Multi-Account Trial Utilization | Pricing Model Optimization | Varies | Low | FinOps Team |
| DEVGURU-018 | Optimize SNS Alert Configurations | Network & Data Transfer Optimization | < 5% | Low | Engineer/DevOps |
| DEVGURU-019 | Consolidate Cross-Region Dashboards | Network & Data Transfer Optimization | < 1% | Low | Engineer/DevOps |
