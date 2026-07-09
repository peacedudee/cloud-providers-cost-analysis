# Cost-Cutting Playbook: AWS Proton
> **Companion File:** [proton.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/proton/proton.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Proton is a fully managed delivery service for container and serverless applications that standardizes infrastructure-as-code (IaC) provisioning. **Crucially, AWS Proton is slated for deprecation on October 7, 2026.** The service itself is 100% free ($0.00), meaning all associated costs stem from the underlying AWS resources (EC2, ECS, EKS, ALBs, NAT Gateways, etc.) provisioned by Proton templates. 

This playbook outlines strategies to optimize the cost of the underlying resources provisioned by Proton, manage the lifecycle of active environments and services, and prepare for the mandatory migration away from the Proton service. Because platform engineers define the templates, optimizing a single Proton template can yield cost savings across dozens or hundreds of developer-deployed services.

## Strategy Categories
### 1. Waste Elimination
*   **PROTON-WE-01: Delete Orphaned Environments:** Identify and terminate development or test environments provisioned by Proton that are no longer in use, releasing expensive resources like NAT Gateways ($32/mo) and Application Load Balancers ($18/mo).
*   **PROTON-WE-02: Terminate Unused Service Instances:** Periodically audit Proton service instances and delete those tied to deprecated branches, completed experiments, or inactive projects.
*   **PROTON-WE-03: Enforce Environment TTLs:** Use your CI/CD pipelines invoking Proton APIs to automatically tear down ephemeral preview environments after a set Time-To-Live (TTL).
*   **PROTON-WE-04: Consolidate Shared Resources:** Update environment templates to share common resources (like VPCs, Route53 Hosted Zones, or ALBs) rather than provisioning dedicated infrastructure for every microservice.
*   **PROTON-WE-05: Clean Up Obsolete Templates:** Delete outdated Proton template versions and their associated S3 artifacts to reduce minor S3 storage clutter.

### 2. Rightsizing
*   **PROTON-RS-01: Optimize Default Template Sizes:** Reduce the default CPU, memory, and instance sizes in Proton Service and Environment templates to prevent developers from over-provisioning by default.
*   **PROTON-RS-02: Provide "T-Shirt Sizing" Inputs:** Parameterize Proton templates with "Small", "Medium", and "Large" configurations, defaulting to "Small" for lower environments.
*   **PROTON-RS-03: Downsize Underutilized Databases:** If environment templates provision RDS or DynamoDB instances, ensure non-production templates use the smallest available instance classes (e.g., `db.t4g.micro`).
*   **PROTON-RS-04: Rightsize Fargate Tasks:** Analyze CloudWatch metrics for ECS Fargate tasks deployed via Proton and update the service templates to reflect actual resource utilization.

### 3. Commitment Discounts
*   **PROTON-CD-01: Align Templates with Savings Plans:** Ensure compute resources (EC2, ECS, Fargate) provisioned by Proton templates are standardized to instance families and regions covered by your existing Compute Savings Plans.
*   **PROTON-CD-02: Tagging for Cost Allocation:** Enforce mandatory tagging rules in Proton templates (e.g., Cost Center, Environment) to ensure accurate chargebacks and identify compute usage eligible for future Reserved Instance purchases.

### 4. Architecture Changes
*   **PROTON-AC-01: Plan Migration Before Deprecation (Oct 7, 2026):** *Critical Priority.* Migrate environment and service templates to AWS CloudFormation StackSets, AWS CDK, or Terraform modules to ensure continuity and avoid forced, unoptimized migrations.
*   **PROTON-AC-02: Shift to Serverless:** Publish new Proton templates that utilize AWS Lambda or AWS App Runner instead of persistent ECS/EKS clusters for bursty or low-traffic workloads.
*   **PROTON-AC-03: Adopt Graviton Processors:** Update ECS and EC2-based Proton templates to use ARM-based Graviton architectures (`arm64`), which offer up to 40% better price-performance than x86 counterparts.
*   **PROTON-AC-04: Container Image Optimization:** While not explicitly Proton, ensure CI/CD pipelines deploying to Proton environments build minimal container images to reduce ECR storage and cross-AZ data transfer costs.

### 5. Scheduling & Auto-Scaling
*   **PROTON-SAS-01: Scale Non-Production to Zero:** Integrate AWS Instance Scheduler with tags provisioned by Proton environment templates to shut down underlying EC2/RDS instances outside of business hours.
*   **PROTON-SAS-02: Bake Auto-Scaling into Templates:** Require all production-facing Proton Service templates to include AWS Application Auto Scaling configurations (Target Tracking policies) rather than static desired counts.
*   **PROTON-SAS-03: Implement ECS Service Auto Scaling:** For ECS-based templates, configure scaling policies tied to CPU/Memory utilization to allow the service footprint to shrink dynamically during low traffic.

### 6. Pricing Model Optimization
*   **PROTON-PMO-01: Enable Spot Instances in Templates:** For stateless web services or batch processing jobs, update Proton templates to utilize EC2 Spot Instances or Fargate Spot, saving up to 90% on compute costs.
*   **PROTON-PMO-02: Spot Instance Fallback:** Configure ECS Capacity Providers in environment templates to prioritize Spot instances with a fallback to On-Demand for reliability.

### 7. Network & Data Transfer Optimization
*   **PROTON-NDT-01: Standardize VPC Endpoints:** Include VPC Endpoints (Gateway and Interface) in Proton environment templates so internal traffic to S3, ECR, or DynamoDB doesn't incur NAT Gateway data processing fees.
*   **PROTON-NDT-02: Centralize NAT Gateways:** Prevent environment templates from creating individual NAT Gateways; instead, utilize a Transit Gateway or a shared network environment template to route outbound traffic through a centralized, cost-optimized egress point.

---
## Cross-Service Synergies
*   **AWS CloudFormation / CDK:** As Proton relies heavily on CloudFormation under the hood, optimizations to CloudFormation execution and stack management directly apply here. Migration away from Proton will likely shift focus entirely to CDK or native CloudFormation.
*   **Amazon ECS / EKS:** The bulk of costs generated by Proton originate from the container orchestrators it deploys to. Modifying Proton templates to optimize ECS/EKS settings yields exponential savings.
*   **AWS Cost Explorer / AWS Config:** Use tags enforced by Proton templates to track spending accurately in Cost Explorer and use AWS Config to detect untagged or non-compliant infrastructure.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
*   `lineItem/ProductCode`: Filter for underlying services (`AmazonEC2`, `AmazonECS`, `AmazonRDS`).
*   `resourceTags/user:proton:environment`: To calculate total cost per Proton environment.
*   `resourceTags/user:proton:service`: To calculate total cost per Proton service.

### B. CloudWatch Metrics
*   `CPUUtilization` and `MemoryUtilization` for ECS tasks and EC2 instances provisioned by Proton.
*   Load Balancer `RequestCount` to identify idle environments.

### C. AWS Config / Trusted Advisor
*   AWS Config rules to identify environments missing standard Proton tags.
*   Trusted Advisor checks for underutilized EC2 instances and idle Load Balancers.

### D. Company Policies
*   Environment retention policies (e.g., preview environments max age = 7 days).
*   Approved instance types and sizes for development vs. production templates.
*   Mandatory migration deadlines prior to October 7, 2026.

### E. IaC (Optional)
*   The raw Jinja / CloudFormation templates stored in the Proton template sync repositories (GitHub, Bitbucket, S3) to audit default resource sizes, scaling policies, and spot configurations.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "PROTON-WE-01",
  "category": "Waste Elimination",
  "resource_type": "AWS::Proton::Environment",
  "resource_id": "arn:aws:proton:us-east-1:123456789012:environment/dev-env-01",
  "issue": "Proton environment has been idle for 30+ days.",
  "recommendation": "Delete the environment via Proton to terminate the underlying NAT Gateway and ALB.",
  "estimated_monthly_savings": 50.00,
  "effort": "Low"
}
```

### Summary Report Table
| Finding ID | Strategy | Affected Environments/Services | Monthly Savings | Effort |
|------------|----------|--------------------------------|-----------------|--------|
| PROTON-WE-01 | Delete Orphaned Environments | 15 idle environments | $750.00 | Low |
| PROTON-RS-02 | T-Shirt Sizing Inputs | 45 dev services | $1,200.00 | Med |
| PROTON-AC-01 | Migrate from Proton | All Proton Resources | $0.00 (Risk Mitigation) | High |
| PROTON-PMO-01 | Enable Spot Instances | 10 stateless services | $600.00 | Med |
| PROTON-NDT-01 | Standardize VPC Endpoints | 5 core environments | $350.00 | Low |
