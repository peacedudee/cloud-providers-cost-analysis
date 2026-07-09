# Cost-Cutting Playbook: AWS Inspector
> **Companion File:** [inspector.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/inspector/inspector.md)
> **Last Updated:** July 2026
---

## Executive Summary
Amazon Inspector is a fully managed vulnerability management service that automatically discovers and scans Amazon EC2 instances, ECR container images, and Lambda functions. While its serverless nature provides effortless security visibility, careless enablement can lead to significant cost overruns. The most common cost traps involve scanning highly ephemeral CI/CD container images ($0.09 per push), running Agentless EC2 scanning without SSM agents installed (incurring a 40% premium), and enabling deep Lambda Code Scanning on thousands of non-production or experimental functions. This playbook details 17 targeted strategies to optimize Amazon Inspector deployments, emphasizing surgical scope reduction, environment-based filtering, and architectural right-sizing without compromising organizational security posture.

## Strategy Categories

### 1. Waste Elimination

#### 1. [INSP-01] Filter ECR Repository Scanning for Production Only
- **What:** Configure Inspector ECR scanning rules to exclusively target production repositories (e.g., `prod/*`) or release tags.
- **Why It Saves Money:** Inspector charges $0.09 per initial container image scan on push. Scanning thousands of temporary CI/CD builds for feature branches wastes thousands of dollars monthly.
- **Implementation Steps:** 1. Navigate to Amazon Inspector console. 2. Go to Settings > ECR Repository Filtering. 3. Create inclusion rules based on repository naming conventions or image tags.
- **Estimated Savings:** 80-95% of ECR scan costs
- **Risk Level:** Low
- **Implementation Scope:** DevOps / SecOps
- **Prerequisites:** Consistent ECR repository naming or image tagging conventions.

#### 2. [INSP-02] Implement Aggressive ECR Lifecycle Policies
- **What:** Automatically expire and delete untagged or temporary container images after a short period (e.g., 3 days).
- **Why It Saves Money:** Inspector charges $0.01 per automated continuous rescan when new CVEs are added to the database. Old, unused images in ECR will continually accrue rescan fees if not deleted.
- **Implementation Steps:** 1. Open ECR Console. 2. Select target repositories. 3. Add Lifecycle rules to expire untagged or dev-branch images.
- **Estimated Savings:** 10-30% of ECR scan costs over time
- **Risk Level:** Low
- **Implementation Scope:** DevOps
- **Prerequisites:** None

#### 3. [INSP-03] Disable Lambda Code Scanning in Non-Prod Environments
- **What:** Disable deep custom application code scanning for Lambda functions in developer and sandbox accounts.
- **Why It Saves Money:** Lambda Code Scanning costs $0.60 per function-month. Applying this to experimental or highly transient dev functions provides minimal security value for the high cost.
- **Implementation Steps:** 1. Access Inspector via Delegated Administrator account. 2. Go to Account Management. 3. Disable Lambda Code Scanning for non-production AWS accounts.
- **Estimated Savings:** 100% of Lambda code scan costs in lower environments
- **Risk Level:** Low
- **Implementation Scope:** SecOps / FinOps
- **Prerequisites:** Multi-account architecture (AWS Organizations).

#### 4. [INSP-04] Disable Lambda Standard Scanning in Sandbox Accounts
- **What:** Disable Lambda standard package dependency scanning in isolated sandbox/experimentation accounts.
- **Why It Saves Money:** Lambda Standard Scanning costs $0.30 per function-month. Eliminating this in sandbox accounts saves money where security compliance is not enforced.
- **Implementation Steps:** 1. Access Inspector settings. 2. Disable Lambda standard scanning for specific sandbox account IDs.
- **Estimated Savings:** 100% of Lambda standard scan costs in sandboxes
- **Risk Level:** Low
- **Implementation Scope:** SecOps
- **Prerequisites:** Multi-account setup with clear sandbox definitions.

#### 5. [INSP-05] Deprovision Orphaned or Unused EC2 Instances
- **What:** Identify and terminate EC2 instances with extremely low utilization (<5% CPU) over 30 days.
- **Why It Saves Money:** Inspector charges $1.25-$1.75 per instance-month. Removing the instance eliminates both the underlying Compute cost and the continuous Inspector monitoring fee.
- **Implementation Steps:** 1. Use AWS Compute Optimizer to find idle instances. 2. Validate with application owners. 3. Terminate instances.
- **Estimated Savings:** 100% of the instance's Inspector (and EC2) cost
- **Risk Level:** Medium
- **Implementation Scope:** Engineer / DevOps
- **Prerequisites:** Proper monitoring and instance ownership tagging.

#### 6. [INSP-06] Ensure Migration from Inspector Classic
- **What:** Validate that Amazon Inspector Classic has been fully disabled and assessment targets removed.
- **Why It Saves Money:** Inspector Classic reached end-of-support in May 2026. Running both v2 continuous scanning and legacy Classic assessments simultaneously results in duplicate billing.
- **Implementation Steps:** 1. Check Inspector Classic console for active assessment runs. 2. Delete all legacy assessment targets and templates.
- **Estimated Savings:** Variable (eliminates double-billing)
- **Risk Level:** Low
- **Implementation Scope:** SecOps / DevOps
- **Prerequisites:** Successful deployment of Inspector v2.

### 2. Rightsizing

#### 7. [INSP-07] Mandate Agent-Based EC2 Scanning (SSM Agent)
- **What:** Standardize deployment of the AWS Systems Manager (SSM) Agent to all EC2 instances to use Agent-based scanning instead of Agentless.
- **Why It Saves Money:** Agent-based scanning costs $1.25/month. Agentless scanning costs $1.75/month (a 40% cost premium).
- **Implementation Steps:** 1. Ensure SSM Agent is baked into AMIs or installed via UserData. 2. Attach the `AmazonSSMManagedInstanceCore` IAM role to EC2 instances.
- **Estimated Savings:** 28% of EC2 scanning costs
- **Risk Level:** Low
- **Implementation Scope:** DevOps / Platform Engineering
- **Prerequisites:** SSM Agent compatible operating systems.

#### 8. [INSP-08] Selectively Scope CIS Benchmark Assessments
- **What:** Only execute CIS Benchmark Assessments on EC2 instances that fall under strict compliance boundaries (e.g., PCI-DSS, HIPAA).
- **Why It Saves Money:** Running CIS checks blindly across all instances costs $0.03 per assessment per instance. Restricting scope eliminates unnecessary compliance overhead.
- **Implementation Steps:** 1. Tag instances requiring compliance (e.g., `Compliance: PCI`). 2. Target only those specific tags for CIS scans in Inspector.
- **Estimated Savings:** 50-90% of CIS assessment costs
- **Risk Level:** Medium
- **Implementation Scope:** SecOps
- **Prerequisites:** Robust instance tagging taxonomy.

#### 9. [INSP-09] Limit EC2 CIS Scan Frequency
- **What:** Reduce the frequency of CIS Benchmark assessments from daily to weekly, monthly, or quarterly, depending on actual audit requirements.
- **Why It Saves Money:** At $0.03 per run per instance, reducing frequency from daily to monthly reduces CIS assessment costs by roughly 97%.
- **Implementation Steps:** 1. Review compliance audit requirements. 2. Modify the Inspector assessment schedule accordingly.
- **Estimated Savings:** Up to 97% of CIS assessment costs
- **Risk Level:** Medium
- **Implementation Scope:** SecOps / Compliance
- **Prerequisites:** Approval from GRC/Compliance teams.

### 3. Commitment Discounts

#### 10. [INSP-10] Enterprise Discount Program (EDP) Coverage
- **What:** Ensure Amazon Inspector usage is forecasted and rolled into your overarching AWS EDP commit.
- **Why It Saves Money:** EDPs provide a blanket discount (typically 9-15%) across most AWS services, which automatically applies to Inspector usage.
- **Implementation Steps:** 1. Forecast annual Inspector spend. 2. Include this forecast in EDP negotiations or renewals with AWS.
- **Estimated Savings:** 9-15% of total Inspector spend
- **Risk Level:** Low
- **Implementation Scope:** FinOps / Procurement
- **Prerequisites:** High overall AWS annual spend.

### 4. Architecture Changes

#### 11. [INSP-11] Shift-Left Vulnerability Scanning in CI/CD
- **What:** Implement open-source vulnerability scanners (like Trivy or Grype) directly in the CI/CD pipeline to block vulnerable container images *before* they are pushed to ECR.
- **Why It Saves Money:** Catching vulnerabilities locally prevents pushing multiple broken iterations to ECR, avoiding the $0.09/image initial scan fee on every failed push.
- **Implementation Steps:** 1. Add Trivy/Grype steps to GitHub Actions or GitLab CI. 2. Fail the pipeline on High/Critical CVEs. 3. Push only the finalized image to ECR.
- **Estimated Savings:** 40-60% of ECR scan costs
- **Risk Level:** Low
- **Implementation Scope:** DevOps / SecOps
- **Prerequisites:** CI/CD Pipeline integration.

#### 12. [INSP-12] Use Minimalist Base Container Images
- **What:** Standardize application deployments on minimal container base images (e.g., Alpine Linux, Distroless).
- **Why It Saves Money:** Minimalist images contain far fewer OS packages. Fewer packages equal fewer vulnerabilities, which drastically reduces the frequency of Inspector continuous rescan triggers ($0.01/rescan).
- **Implementation Steps:** 1. Refactor Dockerfiles to use `alpine` or `gcr.io/distroless`. 2. Remove unnecessary OS utilities and shell binaries.
- **Estimated Savings:** 10-20% of ECR continuous rescan costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineering
- **Prerequisites:** Application compatibility testing with minimalist OSs.

#### 13. [INSP-13] Consolidate Monolithic Lambdas
- **What:** Refactor heavily fragmented, micro-Lambdas into slightly larger, logically grouped functions where architecturally appropriate.
- **Why It Saves Money:** Inspector charges per *function*-month ($0.30-$0.90). Having 100 functions doing tiny tasks costs 10x more to scan than 10 well-organized functions handling the same workload.
- **Implementation Steps:** 1. Identify tightly coupled, low-traffic Lambdas. 2. Combine them using internal router patterns or GraphQL resolvers.
- **Estimated Savings:** Proportional to the reduction in total function count
- **Risk Level:** High
- **Implementation Scope:** Engineering
- **Prerequisites:** Serverless architectural review and refactoring capacity.

### 5. Scheduling & Auto-Scaling

#### 14. [INSP-14] Turn Off Non-Prod EC2 Instances Outside Business Hours
- **What:** Use AWS Instance Scheduler to automatically stop dev and test EC2 instances during nights and weekends.
- **Why It Saves Money:** Inspector EC2 continuous scanning is prorated hourly ($0.0017/hour). Stopping instances for 14 hours a day saves ~60% of the Inspector monitoring cost, alongside massive EC2 compute savings.
- **Implementation Steps:** 1. Tag instances with schedule-defining tags. 2. Deploy AWS Instance Scheduler solution.
- **Estimated Savings:** ~60% of dev/test EC2 scanning costs
- **Risk Level:** Low
- **Implementation Scope:** DevOps
- **Prerequisites:** Workloads that can tolerate scheduled downtime.

#### 15. [INSP-15] Ephemeral CI/CD Runners
- **What:** Utilize short-lived, auto-scaling instances for CI/CD runners (e.g., Jenkins, GitLab Runners) rather than always-on EC2 instances.
- **Why It Saves Money:** Always-on runners incur a full $1.25/mo Inspector charge. Ephemeral runners that live for a few minutes will incur a fraction of a cent due to hourly prorating.
- **Implementation Steps:** 1. Configure CI/CD orchestrators to spin up EC2 instances on demand. 2. Ensure they terminate immediately after the pipeline finishes.
- **Estimated Savings:** 70-90% of runner scanning costs
- **Risk Level:** Low
- **Implementation Scope:** DevOps
- **Prerequisites:** Self-hosted CI/CD infrastructure capable of autoscaling.

### 6. Pricing Model Optimization

#### 16. [INSP-16] Centralize Inspector Management via AWS Organizations
- **What:** Delegate Amazon Inspector administration to a central Security account using AWS Organizations.
- **Why It Saves Money:** Centralized visibility prevents unbudgeted spend spikes caused by rogue developers enabling expensive scanning features (like Lambda Code Scanning) across disparate sub-accounts.
- **Implementation Steps:** 1. Assign a security account as the Delegated Administrator for Inspector. 2. Apply Service Control Policies (SCPs) to restrict local enablement.
- **Estimated Savings:** Avoidance of unbudgeted wildcard spend
- **Risk Level:** Low
- **Implementation Scope:** SecOps / Cloud Admin
- **Prerequisites:** AWS Organizations enabled.

### 7. Network & Data Transfer Optimization

#### 17. [INSP-17] Use VPC Endpoints for SSM Agent Traffic
- **What:** Route SSM Agent heartbeat and Inspector data traffic through VPC Interface Endpoints rather than public NAT Gateways.
- **Why It Saves Money:** Agent-based Inspector scanning relies on the SSM Agent. Sending continuous telemetry from thousands of instances through a NAT Gateway incurs $0.045/GB data processing charges. VPC Endpoints localize this traffic, mitigating NAT tax.
- **Implementation Steps:** 1. Create interface VPC Endpoints for `ssm`, `ssmmessages`, and `ec2messages`. 2. Attach security groups allowing VPC traffic.
- **Estimated Savings:** Variable (mitigates hidden NAT Gateway data processing costs)
- **Risk Level:** Low
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Workloads residing in Private Subnets.

---

## Cross-Service Synergies
- **AWS Systems Manager (SSM):** Fully integrating SSM allows you to utilize cheaper Agent-based scanning and automatically remediate Inspector findings (e.g., patching instances via SSM Patch Manager).
- **Amazon ECR:** Combining ECR Lifecycle Policies with Inspector scanning prevents continuous billing for stagnant, unused images.
- **AWS Security Hub:** Feed Inspector findings into Security Hub to consolidate vulnerability data with other AWS security services, reducing the need for expensive third-party SIEM data ingestion.
- **AWS Compute Optimizer:** Using Compute Optimizer to identify and terminate idle EC2 instances directly reduces the Inspector monitoring footprint.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query `line_item_product_code = 'AmazonInspectorV2'` to isolate Inspector charges.
- Group by `line_item_operation` (e.g., `EC2InstanceScanned`, `ECRImageScanned`, `LambdaFunctionScanned`) to pinpoint cost drivers.
### B. CloudWatch Metrics
- Evaluate Lambda invocation frequency and EC2 CPU utilization to identify idle resources that are incurring passive scanning costs.
### C. AWS Config / Trusted Advisor
- Audit for SSM Agent presence on all EC2 instances to identify instances falling back to the 40% more expensive Agentless scanning model.
- Audit ECR repositories lacking Lifecycle Policies.
### D. Company Policies
- Determine strict compliance requirements (PCI, HIPAA) to accurately scope CIS Benchmark scans.
### E. IaC (Optional)
- Terraform/CloudFormation templates to verify that ECR scanning filters are explicitly codified.

---

## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "INSP-01",
  "service": "Amazon Inspector",
  "category": "Waste Elimination",
  "strategy_name": "Filter ECR Repository Scanning for Production Only",
  "potential_savings_percentage": "80-95",
  "risk_level": "Low",
  "effort_level": "Low",
  "description": "Configure Inspector ECR scanning rules to exclusively target production repositories."
}
```

### Summary Report Table

| ID | Strategy Name | Category | Savings | Risk | Effort |
|---|---|---|---|---|---|
| INSP-01 | Filter ECR Repository Scanning for Production Only | Waste Elimination | 80-95% | Low | Low |
| INSP-02 | Implement Aggressive ECR Lifecycle Policies | Waste Elimination | 10-30% | Low | Low |
| INSP-03 | Disable Lambda Code Scanning in Non-Prod Environments | Waste Elimination | 100% (Non-Prod) | Low | Low |
| INSP-04 | Disable Lambda Standard Scanning in Sandbox Accounts | Waste Elimination | 100% (Sandbox) | Low | Low |
| INSP-05 | Deprovision Orphaned or Unused EC2 Instances | Waste Elimination | 100% (per instance) | Medium | Medium |
| INSP-06 | Ensure Migration from Inspector Classic | Waste Elimination | Variable | Low | Low |
| INSP-07 | Mandate Agent-Based EC2 Scanning (SSM Agent) | Rightsizing | 28% | Low | Medium |
| INSP-08 | Selectively Scope CIS Benchmark Assessments | Rightsizing | 50-90% | Medium | Low |
| INSP-09 | Limit EC2 CIS Scan Frequency | Rightsizing | Up to 97% | Medium | Low |
| INSP-10 | Enterprise Discount Program (EDP) Coverage | Commitment Discounts | 9-15% | Low | High |
| INSP-11 | Shift-Left Vulnerability Scanning in CI/CD | Architecture Changes | 40-60% | Low | Medium |
| INSP-12 | Use Minimalist Base Container Images | Architecture Changes | 10-20% | Medium | High |
| INSP-13 | Consolidate Monolithic Lambdas | Architecture Changes | Variable | High | High |
| INSP-14 | Turn Off Non-Prod EC2 Instances Outside Business Hours | Scheduling & Auto-Scaling | ~60% | Low | Medium |
| INSP-15 | Ephemeral CI/CD Runners | Scheduling & Auto-Scaling | 70-90% | Low | Medium |
| INSP-16 | Centralize Inspector Management via AWS Organizations | Pricing Model Optimization | Variable | Low | Low |
| INSP-17 | Use VPC Endpoints for SSM Agent Traffic | Network Optimization | Variable | Low | Medium |
