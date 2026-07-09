# Cost-Cutting Playbook: AWS Telco Network Builder
> **Companion File:** [tnb.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/tnb/tnb.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Telco Network Builder (TNB) automates the deployment and management of telecom network functions using standard ETSI descriptors. While TNB simplifies 5G and LTE deployments on AWS, costs can quickly accumulate due to the Managed Network Function Item (MNFI) hourly billing ($0.035 per MNFI-hr, or ~$25.20/month per MNFI), API request overages, and the underlying AWS infrastructure costs (EKS, EC2, ELB). This playbook outlines 20 comprehensive strategies across key categories to optimize and reduce the total cost of ownership (TCO) for telco workloads running on AWS TNB.

## Strategy Categories
### 1. Waste Elimination
*   **TNB-WE01: Teardown Dev/Test Telecom Topologies Automatically**
    *   **Description:** Use CI/CD pipeline hooks to terminate test TNB network package deployments at the end of daily validation runs to avoid continuous $0.035/hr billing per MNFI.
*   **TNB-WE02: Delete Unused Network Packages and Descriptors**
    *   **Description:** Regularly purge obsolete Network Service Descriptors (NSDs), Virtual Network Function Descriptors (VNFDs), and Network Packages (NPs) to reduce clutter and associated storage overhead in underlying S3 buckets.
*   **TNB-WE03: Terminate Orphaned Infrastructure**
    *   **Description:** Implement sweeps to identify and terminate underlying EKS clusters, EC2 instances, or Load Balancers that were left behind if a TNB deployment failed or was manually disrupted without proper teardown.
*   **TNB-WE04: Implement TTL (Time-to-Live) Tags on Non-Prod Deployments**
    *   **Description:** Assign TTL tags to development MNFIs and use Lambda functions to automatically delete the TNB deployments when the TTL expires.
*   **TNB-WE05: Remove Idle Load Balancers**
    *   **Description:** Audit Elastic Load Balancers provisioned by TNB for cellular networks. Remove ELBs associated with inactive network functions to save on hourly load balancer charges.

### 2. Rightsizing
*   **TNB-RS01: Optimize Container Resource Limits in VNFDs**
    *   **Description:** Analyze CloudWatch metrics for deployed network functions and adjust vCPU and Memory requests/limits in the VNFDs (Helm charts) to prevent over-provisioning on EKS worker nodes.
*   **TNB-RS02: Rightsize Underlying EC2 Worker Nodes**
    *   **Description:** Ensure the EC2 instance types provisioned for TNB EKS clusters match the actual performance requirements of the 5G Core or RAN workloads. Use AWS Compute Optimizer.
*   **TNB-RS03: Consolidate Network Functions**
    *   **Description:** Where architecturally feasible, consolidate lightweight network functions to reduce the total number of MNFIs managed by TNB, thereby lowering the $0.035/hr per-MNFI fee.

### 3. Commitment Discounts
*   **TNB-CD01: Purchase Compute Savings Plans for EKS Nodes**
    *   **Description:** For stable, baseline telco workloads running 24/7, purchase Compute Savings Plans to reduce the cost of the underlying EC2 compute capacity by up to 66%.
*   **TNB-CD02: Utilize Reserved Instances for Predictable Telco Workloads**
    *   **Description:** Use Standard or Convertible Reserved Instances for EC2 nodes hosting critical, always-on 5G Core network functions.

### 4. Architecture Changes
*   **TNB-AC01: Transition to Graviton (ARM) EC2 Instances**
    *   **Description:** If the telecom vendor's containerized network functions support ARM64 architecture, update TNB templates to deploy onto AWS Graviton instances for up to 20% lower cost and better performance.
*   **TNB-AC02: Leverage Spot Instances for Stateless Network Functions**
    *   **Description:** For fault-tolerant, stateless, or batch-processing telecom workloads, configure the underlying EKS node groups to use EC2 Spot Instances for steep compute discounts.
*   **TNB-AC03: Optimize S3 Artifact Storage**
    *   **Description:** Configure S3 Lifecycle Policies to transition old TNB artifacts, Helm charts, and telecom descriptors to cheaper storage tiers like S3 Infrequent Access or Glacier.

### 5. Scheduling & Auto-Scaling
*   **TNB-SA01: Automate Dynamic Scale-In of Off-Peak Cellular Network Functions**
    *   **Description:** Configure TNB lifecycle hooks to automatically scale down non-essential subscriber session network functions during off-peak overnight hours (1:00 AM – 5:00 AM). Reduces MNFI billing by 20–30%.
*   **TNB-SA02: Configure Karpenter or Cluster Autoscaler**
    *   **Description:** Ensure the underlying EKS clusters managed by TNB use Karpenter or Cluster Autoscaler to dynamically spin down EC2 worker nodes when MNFIs are scaled down.
*   **TNB-SA03: Scheduled Suspension of Non-Prod Environments**
    *   **Description:** Completely terminate Dev/QA TNB deployments over the weekends using AWS Instance Scheduler or custom automation.

### 6. Pricing Model Optimization
*   **TNB-PM01: Monitor and Alert on TNB API Request Volumes**
    *   **Description:** Set up CloudWatch billing alarms to monitor TNB API calls. The first 45,000 requests are free; excessive polling by CI/CD tools could incur the $0.0025 per request fee. Optimize automation scripts to poll less frequently.
*   **TNB-PM02: Tagging for Cost Allocation**
    *   **Description:** Enforce strict tagging on TNB network functions, EKS clusters, and VPCs. Use AWS Cost Explorer to track profitability per tenant or slice in a 5G network slicing architecture.

### 7. Network & Data Transfer Optimization
*   **TNB-ND01: Keep Telco Network Traffic Within the Same AZ**
    *   **Description:** Deploy communicating network functions within the same Availability Zone where possible to avoid the $0.01/GB cross-AZ data transfer charges, which can be massive for high-throughput 5G data planes (UPF).
*   **TNB-ND02: Optimize NAT Gateway Usage**
    *   **Description:** Use VPC Endpoints (AWS PrivateLink) for TNB and EKS API communications instead of routing traffic through NAT Gateways, avoiding per-GB data processing fees.

---
## Cross-Service Synergies
*   **EKS & EC2:** Since TNB orchestrates EKS and EC2, cost optimization heavily relies on reducing underlying compute costs (Savings Plans, Spot Instances, Graviton, Autoscaling).
*   **CloudWatch:** Crucial for monitoring MNFI utilization to enable rightsizing and dynamic scale-in.
*   **AWS Cost Explorer:** Essential for separating TNB orchestration fees (MNFI/API) from the heavier underlying compute and data transfer costs.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
*   `lineItem/ProductCode` = `AWSTelcoNetworkBuilder`
*   `lineItem/UsageType` mapping for `MNFI-Hour` and `API-Request`.
### B. CloudWatch Metrics
*   EKS node CPU and memory utilization.
*   TNB API request count.
### C. AWS Config / Trusted Advisor
*   Idle EKS clusters or unused Load Balancers provisioned by TNB.
### D. Company Policies
*   Acceptable TTL for Dev/Test TNB environments.
*   Approved EC2 instance families (e.g., Graviton eligibility).
### E. IaC (Optional)
*   VNFD and NSD YAML files / Helm charts to review hardcoded compute resource limits.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "TNB-WE01",
  "category": "Waste Elimination",
  "service": "AWS Telco Network Builder",
  "title": "Teardown Dev/Test Telecom Topologies Automatically",
  "description": "Dev/Test TNB MNFIs are running 24/7. Teardown outside of business hours.",
  "severity": "High",
  "estimated_savings_monthly": 756.00,
  "action_required": "Implement CI/CD pipeline hooks to terminate network packages after validation."
}
```
### Summary Report Table
| Finding ID | Title | Category | Severity | Est. Monthly Savings |
|---|---|---|---|---|
| TNB-WE01 | Teardown Dev/Test Telecom Topologies Automatically | Waste Elimination | High | $756.00 |
| TNB-RS01 | Optimize Container Resource Limits in VNFDs | Rightsizing | Medium | $450.00 |
| TNB-SA01 | Automate Dynamic Scale-In of Off-Peak NFs | Scheduling & Auto-Scaling | High | $1,200.00 |
| TNB-ND01 | Keep Telco Network Traffic Within the Same AZ | Network & Data Transfer | High | $3,500.00 |
