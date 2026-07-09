# Cost-Cutting Playbook: AWS Fault Injection Simulator
> **Companion File:** [fis.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/fis/fis.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Fault Injection Simulator (FIS) is a fully managed chaos engineering service. Because it bills per action-minute ($0.10) and per summary report ($5.00), costs can spiral quickly if long-running experiments or continuous automated testing pipelines are left unchecked. This playbook provides actionable strategies to minimize FIS costs, focusing on strict duration limits, eliminating unnecessary reports, preventing runaway experiments, and optimizing architectural approaches to chaos testing. 

## Strategy Categories
### 1. Waste Elimination
### 2. Rightsizing
### 3. Commitment Discounts
### 4. Architecture Changes
### 5. Scheduling & Auto-Scaling
### 6. Pricing Model Optimization
### 7. Network & Data Transfer Optimization

---

## 1. Waste Elimination

#### FIS-001. Enforce Strict Action Duration Limits
- **What:** Restrict the maximum duration of FIS actions (e.g., CPU stress, network latency) to 5-15 minutes rather than letting them run indefinitely.
- **Why It Saves Money:** FIS bills $0.10 per action-minute. Reducing a 60-minute test to a 10-minute test saves $5.00 per action.
- **Implementation Steps:**
  1. Audit existing FIS experiment templates.
  2. Locate the duration parameter for each action.
  3. Modify the duration to the minimum time necessary to validate the health check or auto-scaling response (typically 5-15 mins).
- **Estimated Savings:** 70-90% (per modified action)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of the minimum time required for system monitors to trigger.

#### FIS-002. Implement CloudWatch-driven Stop Conditions
- **What:** Configure FIS Stop Conditions using AWS CloudWatch Alarms to automatically halt experiments when a specific failure threshold is reached.
- **Why It Saves Money:** Prevents experiments from running longer than necessary once the hypothesis is validated or disproven, halting action-minute billing.
- **Implementation Steps:**
  1. Define CloudWatch Alarms for critical application metrics (e.g., latency, error rates).
  2. Edit FIS experiment templates and configure the "Stop Conditions" section.
  3. Attach the relevant CloudWatch Alarms.
- **Estimated Savings:** 10-50%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Existing CloudWatch Alarms for the target workload.

#### FIS-003. Eliminate Non-Prod Summary Reports
- **What:** Disable the automatic generation of FIS Experiment Summary Reports for routine development, testing, and CI/CD pipelines.
- **Why It Saves Money:** Each report costs $5.00. Relying on CloudWatch metrics instead of the PDF report eliminates this flat fee.
- **Implementation Steps:**
  1. Review FIS templates in non-production environments.
  2. Locate the "Experiment Options" configuration.
  3. Disable the "Save experiment results to S3" / report generation feature.
- **Estimated Savings:** 50-80% (if reports were heavily used)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch dashboards in place to visualize metrics.

#### FIS-004. Prune Stale Experiment Templates
- **What:** Delete unused, obsolete, or abandoned FIS experiment templates.
- **Why It Saves Money:** While templates themselves are free, leaving them active risks accidental execution by automated scripts or developers, incurring unwanted action-minute costs.
- **Implementation Steps:**
  1. List all FIS templates using AWS CLI.
  2. Identify templates with no execution history in the last 90 days.
  3. Delete the stale templates.
- **Estimated Savings:** 0-5%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** None.

#### FIS-005. Optimize Automated CI/CD Execution Frequency
- **What:** Trigger chaos experiments only on major releases or significant architectural changes, rather than on every minor code commit.
- **Why It Saves Money:** Reduces the overall volume of action-minutes and reports consumed per month by avoiding over-testing.
- **Implementation Steps:**
  1. Review CI/CD pipeline triggers (e.g., Jenkins, GitHub Actions).
  2. Add conditional logic to skip the FIS testing stage for minor patches or doc updates.
- **Estimated Savings:** 20-40%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Advanced CI/CD pipeline routing.

---

## 2. Rightsizing

#### FIS-006. Minimize Concurrent Actions per Experiment
- **What:** Remove overlapping or unnecessary fault actions within a single experiment if one action is sufficient to test the resiliency mechanism.
- **Why It Saves Money:** FIS charges $0.10/min for *each* action. Running 3 actions concurrently costs $0.30/min. Reducing to 1 action saves $0.20/min.
- **Implementation Steps:**
  1. Review the goals of each experiment.
  2. Identify if multiple actions (e.g., CPU stress + memory stress) are genuinely required.
  3. Remove redundant actions from the template.
- **Estimated Savings:** 30-60%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Clear understanding of chaos engineering objectives.

#### FIS-007. Consolidate Multi-Account Targeting
- **What:** Run experiments within a single AWS account where possible, rather than targeting resources across multiple accounts in an AWS Organization.
- **Why It Saves Money:** Multi-account experiments incur the $0.10/min action charge *per targeted account*. Centralizing the test avoids the multiplier.
- **Implementation Steps:**
  1. Identify multi-account experiments in FIS.
  2. Assess if the test can be adequately simulated in a single account.
  3. Reconfigure the target selection to a single account scope.
- **Estimated Savings:** 50%+ (depending on number of accounts targeted)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Cross-account testing requirements review.

#### FIS-008. Limit Target Resource Blast Radius
- **What:** Target specific, limited subsets of resources (e.g., 1-2 EC2 instances) rather than entire fleets.
- **Why It Saves Money:** While FIS action-minutes are fixed regardless of the number of target instances in a single account, reducing the blast radius minimizes secondary recovery costs (e.g., auto-scaling spin-ups, data transfer spikes).
- **Implementation Steps:**
  1. Edit target definitions in FIS templates.
  2. Use specific Resource IDs or strict tag filters to narrow the scope.
- **Estimated Savings:** 5-15% (on secondary infrastructure costs)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Robust tagging strategy.

---

## 3. Commitment Discounts

#### FIS-009. Enterprise Discount Program (EDP) Integration
- **What:** Include AWS FIS spend in the organization's overarching AWS Enterprise Discount Program (EDP) negotiations.
- **Why It Saves Money:** FIS does not offer Reserved Instances or Savings Plans. EDP provides a flat percentage discount across all eligible AWS services, including FIS.
- **Implementation Steps:**
  1. Project annual FIS spend.
  2. Include projected spend in EDP volume commitments during contract renewal.
- **Estimated Savings:** 5-15%
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** AWS EDP eligibility ($1M+ annual spend).

#### FIS-010. AWS Credits for GameDay Events
- **What:** Utilize AWS promotional credits to fund large-scale Chaos Engineering "GameDays."
- **Why It Saves Money:** Offsets the burst in FIS action-minute costs during intensive 24-48 hour company-wide resiliency testing events.
- **Implementation Steps:**
  1. Plan the GameDay event timeline and budget.
  2. Request POC or GameDay credits through your AWS Account Manager.
  3. Apply credits to the testing accounts prior to execution.
- **Estimated Savings:** Up to 100% (for the specific event)
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Strong relationship with AWS account team.

---

## 4. Architecture Changes

#### FIS-011. Offload Simple Faults to AWS Systems Manager (SSM)
- **What:** Execute basic chaos tasks (e.g., running a `stress-ng` CPU script) directly via AWS SSM Run Command instead of using the managed FIS service.
- **Why It Saves Money:** SSM Run Command is largely free or extremely low-cost, completely bypassing the $0.10/min FIS action fee.
- **Implementation Steps:**
  1. Identify simple, script-based FIS actions.
  2. Create SSM Documents containing the fault scripts.
  3. Trigger the SSM Documents directly via CLI or EventBridge.
- **Estimated Savings:** ~100% (for migrated actions)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SSM Agent installed on target instances.

#### FIS-012. Centralized Chaos Engineering Hub
- **What:** Deploy FIS experiments from a single, centralized tooling account rather than allowing disparate teams to run decentralized experiments.
- **Why It Saves Money:** Centralization allows FinOps teams to easily monitor, budget, and enforce cost guardrails on chaos engineering across the organization.
- **Implementation Steps:**
  1. Designate a tooling/DevOps AWS account.
  2. Set up IAM cross-account roles to allow FIS in the tooling account to target workload accounts.
- **Estimated Savings:** 10-20% (via better governance)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Cross-account IAM administration.

#### FIS-013. Optimize Stop Condition CloudWatch Alarms
- **What:** Use standard-resolution CloudWatch metrics (1-minute) instead of high-resolution metrics (1-second) for FIS Stop Conditions unless millisecond precision is strictly required.
- **Why It Saves Money:** High-resolution custom metrics cost significantly more ($0.30 per metric) than standard metrics. 
- **Implementation Steps:**
  1. Review alarms tied to FIS Stop Conditions.
  2. Downgrade custom metrics to standard 60-second resolution if applicable.
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Tolerance for up to 60 seconds of delay in stop conditions.

#### FIS-014. Integrate Open-Source Chaos Tools for EKS
- **What:** Use Kubernetes-native open-source tools like Chaos Mesh or Litmus for EKS workloads instead of FIS.
- **Why It Saves Money:** Bypasses FIS per-minute billing entirely. You only pay for the compute resources running the open-source operator.
- **Implementation Steps:**
  1. Evaluate Chaos Mesh or Litmus.
  2. Deploy the operator into non-production EKS clusters.
  3. Migrate EKS-focused fault templates off FIS.
- **Estimated Savings:** 80-90% (for EKS-specific chaos testing)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Kubernetes administration expertise.

---

## 5. Scheduling & Auto-Scaling

#### FIS-015. Align Chaos Tests with Auto-Scaling Down-cycles
- **What:** Schedule chaos engineering experiments during low-traffic periods when Auto Scaling Groups (ASGs) have scaled down.
- **Why It Saves Money:** Injecting faults into smaller clusters limits the volume of replacement instances that need to be spun up, reducing collateral EC2 costs.
- **Implementation Steps:**
  1. Analyze traffic patterns using CloudWatch.
  2. Schedule FIS execution windows during off-peak hours (e.g., 2 AM local time).
- **Estimated Savings:** 5-10% (on secondary infrastructure costs)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Automated scheduling mechanisms.

#### FIS-016. Event-Driven Experiment Execution
- **What:** Use Amazon EventBridge to trigger FIS experiments strictly based on deployment events, rather than running them on a frequent, fixed cron schedule (e.g., daily).
- **Why It Saves Money:** Eliminates "empty" test runs on days where no codebase or infrastructure changes occurred.
- **Implementation Steps:**
  1. Create an EventBridge rule listening for AWS CodePipeline or GitHub Actions success events.
  2. Set the EventBridge target to trigger the FIS experiment template.
- **Estimated Savings:** 20-50%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** EventBridge integration.

---

## 6. Pricing Model Optimization

#### FIS-017. Consolidate Experiment Reports
- **What:** For teams that require FIS Experiment Reports for compliance, run one large, consolidated experiment (testing multiple hypotheses sequentially) to generate a single report.
- **Why It Saves Money:** Rather than running 5 small experiments and paying $25.00 for 5 reports, run 1 consolidated experiment and pay $5.00 for 1 report.
- **Implementation Steps:**
  1. Group related fault actions into a single timeline in a unified FIS template.
  2. Execute the unified template and generate the single report.
- **Estimated Savings:** Up to 80% (on report costs)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Complex FIS template design.

#### FIS-018. Tag-Based Cost Allocation for Chargeback
- **What:** Enforce strict resource tagging on FIS Experiment Templates to enable cost allocation and chargebacks to specific product teams.
- **Why It Saves Money:** When teams are financially accountable for their own chaos engineering spend, they naturally optimize action durations and report generations.
- **Implementation Steps:**
  1. Enable cost allocation tags in AWS Billing.
  2. Use AWS Tag Policies to mandate a `CostCenter` or `Team` tag on all FIS templates.
  3. Distribute monthly FinOps reports to team leads.
- **Estimated Savings:** 10-20%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Mature tagging strategy.

#### FIS-019. Monitor FIS Spend with AWS Budgets
- **What:** Set up strict AWS Budgets and Anomaly Detection specifically filtered for the "AWS Fault Injection Service".
- **Why It Saves Money:** Catches runaway automated scripts or accidentally prolonged actions before they accumulate hundreds of dollars in charges.
- **Implementation Steps:**
  1. Navigate to AWS Budgets.
  2. Create a cost budget filtered by Service = FIS.
  3. Set an alert threshold (e.g., 80% of $100/month) to notify via SNS/Slack.
- **Estimated Savings:** Variable (Risk Mitigation)
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** AWS Budgets configured.

---

## 7. Network & Data Transfer Optimization

#### FIS-020. Localize Cross-Region Network Fault Tests
- **What:** When injecting network latency or packet loss, simulate the faults locally using FIS rather than physically routing traffic across AWS Regions to test latency resilience.
- **Why It Saves Money:** Physical cross-region traffic incurs AWS Data Transfer out charges ($0.02 - $0.09/GB). Simulating the latency locally via FIS avoids these massive data transfer fees.
- **Implementation Steps:**
  1. Identify testing scenarios simulating cross-region delays.
  2. Use the FIS `aws:network:inject-api-internal-error` or latency actions on local instances.
  3. Halt physical cross-region data transfers during the test.
- **Estimated Savings:** 10-30% (on secondary Data Transfer costs)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of FIS network fault actions.

---

## Cross-Service Synergies
- **AWS CloudWatch:** Crucial for Stop Conditions to limit FIS billing duration and eliminating the need for paid FIS reports.
- **AWS Systems Manager (SSM):** Can replace FIS entirely for simple OS-level fault injections to bypass action-minute billing.
- **Amazon EventBridge:** Optimizes scheduling to prevent wasteful, fixed-interval test executions.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Identify total monthly spend on `Action-Minutes` vs `Experiment-Reports`.
- Locate the most expensive specific FIS templates using Resource IDs.
### B. CloudWatch Metrics
- Verify if Stop Conditions are actually triggering, or if experiments always run to their maximum duration limits.
### C. AWS Config / Trusted Advisor
- Audit target resources to ensure FIS experiments aren't targeting overly broad groups of instances (Blast radius analysis).
### D. Company Policies
- Review testing and compliance mandates to determine if $5.00 Experiment Reports are legally/internally required, or if CloudWatch dashboards suffice.
### E. IaC (Optional)
- Analyze Terraform/CloudFormation code for FIS templates to easily bulk-update action durations to 5-15 minutes.

---

## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "FIS-001",
  "service": "AWS Fault Injection Simulator",
  "strategy_name": "Enforce Strict Action Duration Limits",
  "category": "Waste Elimination",
  "estimated_savings_percentage": "70-90%",
  "risk_level": "Low",
  "description": "Restrict the maximum duration of FIS actions to 5-15 minutes rather than letting them run indefinitely."
}
```

### Summary Report Table
| Finding ID | Strategy Name | Category | Risk Level | Savings Potential |
|------------|---------------|----------|------------|-------------------|
| FIS-001 | Enforce Strict Action Duration Limits | Waste Elimination | Low | High |
| FIS-002 | Implement CloudWatch-driven Stop Conditions | Waste Elimination | Low | Medium |
| FIS-003 | Eliminate Non-Prod Summary Reports | Waste Elimination | Low | High |
| FIS-004 | Prune Stale Experiment Templates | Waste Elimination | Low | Low |
| FIS-005 | Optimize Automated CI/CD Execution Frequency | Waste Elimination | Medium | Medium |
| FIS-006 | Minimize Concurrent Actions per Experiment | Rightsizing | Medium | Medium |
| FIS-007 | Consolidate Multi-Account Targeting | Rightsizing | Medium | High |
| FIS-008 | Limit Target Resource Blast Radius | Rightsizing | Low | Low |
| FIS-009 | Enterprise Discount Program (EDP) Integration | Commitment Discounts | Low | Low |
| FIS-010 | AWS Credits for GameDay Events | Commitment Discounts | Low | High |
| FIS-011 | Offload Simple Faults to AWS SSM | Architecture Changes | Medium | High |
| FIS-012 | Centralized Chaos Engineering Hub | Architecture Changes | High | Medium |
| FIS-013 | Optimize Stop Condition CloudWatch Alarms | Architecture Changes | Low | Low |
| FIS-014 | Integrate Open-Source Chaos Tools for EKS | Architecture Changes | High | High |
| FIS-015 | Align Chaos Tests with Auto-Scaling Down-cycles | Scheduling & Auto-Scaling | Low | Low |
| FIS-016 | Event-Driven Experiment Execution | Scheduling & Auto-Scaling | Low | Medium |
| FIS-017 | Consolidate Experiment Reports | Pricing Model Optimization | Low | High |
| FIS-018 | Tag-Based Cost Allocation for Chargeback | Pricing Model Optimization | Low | Medium |
| FIS-019 | Monitor FIS Spend with AWS Budgets | Pricing Model Optimization | Low | Variable |
| FIS-020 | Localize Cross-Region Network Fault Tests | Network & Data Transfer | Medium | Medium |
