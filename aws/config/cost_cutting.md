# Cost-Cutting Playbook: AWS Config

> **Companion File:** [config.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/config/config.md)  
> **Last Updated:** July 2026

---

## Executive Summary

AWS Config continuously monitors, records, and audits configuration changes across AWS resources. Billing is driven by **Configuration Items (CI) Recorded** ($0.003/CI for Continuous vs $0.012/CI for Periodic) and **Config Rule Evaluations** ($0.0010/eval).

Key billing traps include:
1. **Continuous Recording of High-Churn Ephemeral Resources:** Recording CIs continuously for short-lived EKS pods, Elastic Network Interfaces (ENIs), or Auto-Scaling EC2 instances that churn hundreds of times per day. 10,000 ENI churn events/day = 300,000 CIs/mo (**$900.00/month in continuous CI fees**).
2. **Redundant Conformance Pack Subscriptions:** Subscribing to CIS Benchmarks, PCI-DSS, NIST 800-53, and AWS Foundational Best Practices concurrently, causing single resources to be evaluated multiple times for identical rules.

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **40–85% reduction in total AWS Config spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Exclude High-Churn Ephemeral Resources (`AWS::EC2::NetworkInterface`, `AWS::EKS::Pod`):** Excludes short-lived ENIs and container pods from Config recording to cut CI recording fees by **60–85%**.
2. **Switch High-Activity Resources to Periodic Recording ($0.012/CI once per 24 hrs):** Caps recording frequency to once per 24 hours for auto-scaling resources, saving **70–90%** vs continuous recording.
3. **Consolidate Redundant Conformance Packs into Security Hub:** Consolidates compliance rules to eliminate duplicate rule evaluation fees ($0.0010/eval).

---

## Strategy Categories

### 1. Resource Type Exclusion & Scope Reduction

#### 1. Exclude High-Churn Ephemeral Resource Types from Config Recording
- **What:** Update AWS Config settings from *Record all current and future resource types* to *Record specific resource types*, explicitly **excluding** high-churn ephemeral resources:
  - `AWS::EC2::NetworkInterface` (ENIs created/deleted by Lambda, ECS, EKS)
  - `AWS::EKS::Pod`
  - `AWS::ECS::TaskSet`
  - `AWS::EC2::SecurityGroup` (if dynamically managed by Kubernetes controllers).
- **Why It Saves Money:** High-churn resources generate a new Configuration Item (CI) on every create, attach, detach, and delete event ($0.003/CI). Recording 100,000 ENI events/month costs **$300.00/month for ENIs alone**. Excluding ENIs drops this cost to **$0.00**.
- **Detailed Implementation Steps:**
  1. Update AWS Config recorder via AWS CLI:
     ```bash
     aws configservice put-configuration-recorder \
       --configuration-recorder '{
         "name": "default",
         "roleARN": "arn:aws:iam::123456789012:role/aws-service-role/config.amazonaws.com/AWSServiceRoleForConfig",
         "recordingGroup": {
           "allSupported": false,
           "includeGlobalResourceTypes": true,
           "resourceTypes": [
             "AWS::EC2::Instance", "AWS::EC2::Volume", "AWS::S3::Bucket",
             "AWS::RDS::DBInstance", "AWS::IAM::Role", "AWS::IAM::Policy"
           ]
         }
       }'
     ```
  2. In Terraform:
     ```hcl
     resource "aws_config_configuration_recorder" "main" {
       name     = "default"
       role_arn = aws_iam_role.config_role.arn
       recording_group {
         all_supported                 = false
         include_global_resource_types = true
         resource_types                = ["AWS::EC2::Instance", "AWS::S3::Bucket", "AWS::RDS::DBInstance"]
       }
     }
     ```
- **Estimated Savings:** **60–85% reduction** in total AWS Config CI recording fees.
- **Risk Level:** Low (verify security policy permits excluding ENIs/pods from Config timeline audit).
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** Compliance team agreement on excluded resource types.

---

### 2. Recording Frequency Tuning

#### 2. Switch High-Activity Resource Types to Periodic Recording
- **What:** Configure recording frequency to **Periodic** (once per 24 hours) for resource types that experience frequent intraday updates but only require daily compliance checks (e.g. `AWS::EC2::Instance`, `AWS::AutoScaling::AutoScalingGroup`).
- **Why It Saves Money:**
  - **Continuous Recording ($0.003/CI):** Billed on every change event. A server updating tags or scaling 10 times/day generates 300 CIs/month = **$0.90/resource-month**.
  - **Periodic Recording ($0.012/CI):** Captures at most 1 snapshot per 24 hours = 30 CIs/month = **$0.36/resource-month (60% savings)**.
- **Detailed Implementation Steps:**
  1. Update recording mode in AWS Config configuration recorder via CLI:
     ```bash
     aws configservice put-configuration-recorder \
       --configuration-recorder '{
         "name": "default",
         "recordingMode": {
           "recordingFrequency": "CONTINUOUS",
           "recordingModeOverrides": [{
             "description": "Daily snapshot for EC2 instances",
             "resourceTypes": ["AWS::EC2::Instance", "AWS::AutoScaling::AutoScalingGroup"],
             "recordingFrequency": "DAILY"
           }]
         }
       }'
     ```
- **Estimated Savings:** **60–90% reduction** in CI recording volume on active auto-scaling resources.
- **Risk Level:** Zero risk (daily snapshot preserves compliance history).
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** Config recorder API support for recording mode overrides.

---

### 3. Rule & Conformance Pack Consolidation

#### 3. Consolidate Redundant Conformance Packs into Security Hub
- **What:** Disable duplicate AWS Config Conformance Packs (e.g. running CIS Benchmark, PCI-DSS, and NIST 800-53 conformance packs concurrently) and consolidate security monitoring under **AWS Security Hub**.
- **Why It Saves Money:** Running 4 conformance packs evaluates identical resources 4 times for the same rule (e.g., `s3-bucket-ssl-requests-only`), billing $0.0010 per evaluation 4 times ($0.0040/resource). Consolidating eliminates 75% of duplicate rule evaluation charges.
- **Detailed Implementation Steps:**
  1. List active conformance packs: `aws configservice describe-conformance-packs`.
  2. Delete duplicate packs: `aws configservice delete-conformance-pack --conformance-pack-name CIS-Pack`.
  3. Enable AWS Foundational Security Best Practices in Security Hub.
- **Estimated Savings:** 50–75% reduction in Config rule evaluation fees ($0.0010/eval).
- **Risk Level:** Zero risk (Security Hub consolidates compliance findings seamlessly).
- **Implementation Scope:** Security Engineer
- **Prerequisites:** Security Hub enabled.

#### 4. Turn Off Non-Essential AWS Config Managed Rules in Dev Accounts
- **What:** Disable heavy compliance rules (such as `desired-instance-type` or `iam-user-unused-credentials-check`) in development and sandbox accounts.
- **Why It Saves Money:** Reclaims rule evaluation fees ($0.0010/eval) on non-production resources.
- **Detailed Implementation Steps:**
  1. Delete non-essential rules: `aws configservice delete-config-rule --config-rule-name dev-rule`.
- **Estimated Savings:** 100% of non-prod rule evaluation fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod compliance policy.

---

### 4. Storage & History Lifecycle Optimization

#### 5. Set Aggressive S3 Lifecycle Expiration on Config History Buckets
- **What:** Configure S3 Lifecycle Policies on the target S3 bucket storing AWS Config configuration history snapshots (`s3://config-bucket-account-region/`).
- **Why It Saves Money:** AWS Config delivers daily configuration history JSON files. Retaining historical JSON files in S3 Standard ($0.023/GB-mo) indefinitely accumulates terabytes of audit logs.
- **Detailed Implementation Steps:**
  1. Configure S3 Lifecycle rule: Transition to Glacier Deep Archive ($0.00099/GB-mo) after 90 days; expire after 365 days.
- **Estimated Savings:** **95% storage discount** on historical Config audit files.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Audit retention policy alignment.

---

### 5. Multi-Account Aggregator Optimization

#### 6. Optimize Multi-Account Aggregator Configurations
- **What:** Ensure AWS Config Aggregators in Security Hub / Organization Management accounts aggregate findings without triggering duplicate local rule executions.
- **Why It Saves Money:** Avoids double-evaluating rules in both child accounts and master aggregator accounts.
- **Detailed Implementation Steps:**
  1. Configure Organization Aggregator with `account_ids` list.
- **Estimated Savings:** 20–40% reduction in multi-account evaluation spend.
- **Risk Level:** Low.
- **Implementation Scope:** Security Architect
- **Prerequisites:** AWS Organizations setup.

---

### 6. Governance & Observability

#### 7. Disable Config Recording in Inactive Regions
- **What:** Turn off AWS Config recording (`aws configservice stop-configuration-recorder`) in unused AWS regions where zero application workloads are deployed.
- **Why It Saves Money:** Stops background global resource recording fees in idle regions.
- **Detailed Implementation Steps:**
  1. Stop recorder in unused regions:
     ```bash
     aws configservice stop-configuration-recorder --configuration-recorder-name default --region eu-north-1
     ```
- **Estimated Savings:** 100% of Config spend in inactive regions.
- **Risk Level:** Low (verify region is restricted by SCP).
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** Regional SCP restrictions.

#### 8. Audit Custom Config Rule Lambda Function Execution Costs
- **What:** Right-size custom Config Rule evaluation Lambda functions to 128 MB RAM and optimize code execution speed.
- **Why It Saves Money:** Reduces auxiliary Lambda compute charges triggered by Config custom rules.
- **Detailed Implementation Steps:**
  1. Set custom rule Lambda memory = 128 MB.
- **Estimated Savings:** 30–50% custom rule Lambda compute savings.
- **Risk Level:** Zero.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Custom Config rule usage.

#### 9. Enforce CloudWatch Alarms for Daily Config CI Volume Spikes
- **What:** Put CloudWatch alarm on `ConfigurationItemsRecorded` (> 10,000 CIs/day).
- **Why It Saves Money:** Instant alert if a runaway automation script creates and deletes thousands of resources.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm targeting `AWS/Config`.
- **Estimated Savings:** Proactive billing risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 10. Standardize Config Delivery Channel Frequency
- **What:** Set Config delivery channel snapshot frequency to `TwentyFour_Hours`.
- **Why It Saves Money:** Reduces S3 delivery API calls.
- **Detailed Implementation Steps:**
  1. Update delivery channel configuration.
- **Estimated Savings:** S3 API fee optimization.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 11. Restrict SNS Notification Topic Subscriptions for Config Events
- **What:** Disable SNS topic notifications on routine Config change events.
- **Why It Saves Money:** Prevents SNS delivery fee clutter ($0.06/100k).
- **Detailed Implementation Steps:**
  1. Detach SNS topic ARN from Config delivery channel.
- **Estimated Savings:** SNS fee elimination.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compliance review.

#### 12. Audit Remediation Action Execution Frequencies
- **What:** Ensure AWS Config Auto-Remediation actions (SSM Automation documents) have max retry limits.
- **Why It Saves Money:** Prevents infinite remediation execution loops.
- **Detailed Implementation Steps:**
  1. Set `MaximumAutomaticAttempts = 3` on remediation configurations.
- **Estimated Savings:** Prevents SSM automation overruns.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** Config auto-remediation setup.

#### 13. Enable IAM Least Privilege Policies on Config Roles
- **What:** Restrict IAM permissions attached to `AWSServiceRoleForConfig`.
- **Why It Saves Money:** Security governance best practice.
- **Detailed Implementation Steps:**
  1. Audit service role policy.
- **Estimated Savings:** Security risk mitigation.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** IAM policy review.

#### 14. Utilize AWS CloudTrail for Administrative Auditing (Where Config Excluded)
- **What:** Use CloudTrail Event History for free administrative change tracking on excluded resource types (e.g. ENIs).
- **Why It Saves Money:** Free administrative audit trail alternative.
- **Detailed Implementation Steps:**
  1. Query CloudTrail event history for change events.
- **Estimated Savings:** Zero-cost audit capability.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** None.

#### 15. Audit Conformance Pack Deployment Sizing
- **What:** Deploy conformance packs across Organization OUs selectively rather than universally across all accounts.
- **Why It Saves Money:** Avoids evaluating compliance rules on sandbox accounts.
- **Detailed Implementation Steps:**
  1. Update Organization Conformance Pack target OU scope.
- **Estimated Savings:** 30–50% conformance pack evaluation savings.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team / Security Architect
- **Prerequisites:** AWS Organizations setup.

#### 16. Enforce Automated Environment Tagging Rules
- **What:** Enforce tagging rules to auto-exclude non-prod resources from compliance audits.
- **Why It Saves Money:** Governance cost optimization.
- **Detailed Implementation Steps:**
  1. Add tag filter in rule logic.
- **Estimated Savings:** Rule evaluation fee optimization.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Tagging compliance.

---

## Cross-Service Synergies

```
[ AWS Environment ] 
        │
        ├──(Resource Exclusion)─> [ Exclude ENIs / Pods ] (Saves 60-85% on continuous CI recording)
        │
        ├──(Recording Mode)─────> [ Periodic Daily Snapshot ($0.012/CI) ] (Caps high-activity resource CIs)
        │
        └──(Rule Governance)────> [ Security Hub Consolidation ] (Eliminates duplicate evaluation fees)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `ConfigurationItemRecorded`, `ConfigRuleEvaluation-Tier1`, `ConfigRuleEvaluation-Tier2`, `ConformancePackEvaluation`.
- `line_item_resource_id`: Config Recorder / Rule Name (`arn:aws:config:us-east-1:123456789012:config-rule/xxx`).

### B. CloudWatch Metrics
- `AWS/Config` Namespace: `ConfigurationItemsRecorded`, `ConfigRuleEvaluations`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "CFG-EXC-001",
  "service": "Config",
  "category": "Resource Type Exclusion & Scope Reduction",
  "resource_id": "arn:aws:config:us-east-1:123456789012:config-recorder/default",
  "resource_name": "default-recorder",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "all_supported_recorded": true,
    "high_churn_enis_recorded": true,
    "monthly_cis_recorded_thousands": 450.0,
    "monthly_cost_usd": 1350.00
  },
  "recommended_config": {
    "all_supported_recorded": false,
    "excluded_resource_types": ["AWS::EC2::NetworkInterface", "AWS::EKS::Pod"],
    "projected_monthly_cis_recorded_thousands": 65.0,
    "projected_monthly_cost_usd": 195.00
  },
  "financial_impact": {
    "monthly_savings_usd": 1155.00,
    "annual_savings_usd": 13860.00,
    "savings_percentage": 85.5
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "Excluding ENIs and EKS pods from Config timeline recording cuts 85% of CI churn while maintaining audit compliance on core compute/storage resources."
  },
  "implementation": {
    "scope": "Security / DevOps",
    "effort_estimate": "30 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **ENI & Pod Resource Exclusion** | 10 | $14,500.00 | $12,397.50 | 85.5% | Low |
| **Periodic Recording Mode Override**| 8 | $6,200.00 | $4,340.00 | 70.0% | Zero |
| **Security Hub Rule Consolidation** | 12 | $4,800.00 | $3,600.00 | 75.0% | Zero |
| **Inactive Region Recorder Shutdown**| 15 | $2,100.00 | $2,100.00 | 100.0% | Low |
| **Total** | **45** | **$27,600.00** | **$22,437.50** | **81.3%** | -- |
