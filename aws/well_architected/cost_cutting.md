# Cost-Cutting Playbook: AWS Well-Architected Tool
> **Companion File:** [well_architected.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/well_architected/well_architected.md)
> **Last Updated:** July 2026
---
## Executive Summary
The AWS Well-Architected (WA) Tool is provided at no additional charge, meaning there are no direct costs to cut from the service itself. Instead, the WA Tool acts as a powerful catalyst for organizational cost reduction. By systematically evaluating workloads against the Cost Optimization Pillar and deploying Custom Lenses, teams can uncover architectural debt, prevent over-provisioning, and align engineering practices with enterprise financial guardrails. This playbook outlines 20 actionable strategies to leverage the WA Tool for driving cost efficiency across all AWS environments.

## Strategy Categories
### 1. Waste Elimination
*   **WA-WST-001: Regular Cost Pillar Audits:** Mandate quarterly reviews of the Cost Optimization Pillar (COST 1-11) to systematically identify abandoned environments, unattached volumes, and un-tagged resources.
*   **WA-WST-002: Architectural Debt Tracking:** Use WA Tool Milestones to track the decommissioning of legacy workloads and quantify the cost savings of removing obsolete infrastructure.
*   **WA-WST-003: Custom Lenses for Waste Prevention:** Deploy enterprise-specific Custom Lenses to ensure standards like "Are S3 lifecycle policies applied?" and "Are idle development environments terminated?" are validated before production launch.

### 2. Rightsizing
*   **WA-RSZ-001: Workload Sizing Validation:** Use the WA Tool (COST 3: Match supply and demand) to ensure that instance families and sizes are chosen based on data-driven metrics rather than engineering estimates.
*   **WA-RSZ-002: Rightsizing Milestone Reviews:** Document the adoption of right-sized resources in WA Tool Milestones to measure continuous improvement against Compute Optimizer recommendations over time.

### 3. Commitment Discounts
*   **WA-COM-001: Commitment Coverage Assessments:** Incorporate checks in the WA Tool to verify that baseline, steady-state workload components are fully covered by Compute Savings Plans, EC2 Instance Savings Plans, or Reserved Instances.
*   **WA-COM-002: Storage and Database Commitments:** Create Custom Lenses to mandate that stable databases (RDS/Redshift/Elasticache) utilize Reserved Nodes to maximize discounting.

### 4. Architecture Changes
*   **WA-ARC-001: Evaluate Serverless Migrations:** Leverage the WA Tool Serverless Lens to identify EC2-based workloads that can be modernized to AWS Lambda or Fargate to eliminate idle capacity costs.
*   **WA-ARC-002: Storage Tiering Validations:** Assess data storage architectures through WA reviews to ensure workloads utilize optimal Amazon S3 storage classes (e.g., transitioning from Standard to Intelligent-Tiering or Glacier).
*   **WA-ARC-003: Open-Source Database Adoption:** Use WA reviews to identify commercial database workloads (Oracle, SQL Server) that can be re-architected to Amazon Aurora or RDS for PostgreSQL/MySQL to eliminate licensing fees.
*   **WA-ARC-004: Managed Service Preferences:** Validate the adoption of AWS managed services (e.g., using Amazon MQ instead of self-managed RabbitMQ on EC2) during WA assessments to reduce operational and infrastructure overhead.

### 5. Scheduling & Auto-Scaling
*   **WA-SCL-001: Elasticity Audits:** Audit application elasticity using WA Tool (COST 5: Evaluate cost when selecting services) to ensure workloads scale in automatically during low-demand periods.
*   **WA-SCL-002: Non-Production Scheduling Validation:** Enforce scheduling policies (e.g., turning off Dev/Test environments on weekends) through WA Tool Custom Lens checks.
*   **WA-SCL-003: Event-Driven Scaling Checks:** Use the WA Tool to confirm that Auto Scaling groups are triggered by appropriate CloudWatch metrics (e.g., SQS queue depth rather than just CPU) to match true workload demand.

### 6. Pricing Model Optimization
*   **WA-PRC-001: Spot Instance Adoption Reviews:** Ensure Amazon EC2 Spot Instances are explicitly evaluated for all fault-tolerant, stateless, or batch processing workloads during WA Tool reviews.
*   **WA-PRC-002: Graviton Processor Validations:** Create a Custom Lens rule to check if workloads have evaluated ARM-based AWS Graviton processors, which offer superior price-performance over x86 processors.
*   **WA-PRC-003: Regional Cost Benchmarking:** Use the WA Tool to validate that the most cost-effective AWS Region is being used for the workload, balancing data sovereignty and latency requirements.

### 7. Network & Data Transfer Optimization
*   **WA-NET-001: Data Transfer Route Reviews:** Review data transfer costs (COST 2) to ensure traffic is efficiently routed through Amazon CloudFront or VPC Endpoints (AWS PrivateLink) instead of traversing the public internet.
*   **WA-NET-002: Multi-AZ vs. Multi-Region Optimization:** Validate architecture for high availability configurations to ensure that cross-AZ and cross-region data transfer fees are minimized and justified by business continuity needs.
*   **WA-NET-003: NAT Gateway Usage Assessments:** Assess NAT Gateway architecture during WA reviews to ensure appropriate placement and sizing, minimizing unnecessary data processing charges for internal traffic.
---
## Cross-Service Synergies
*   **AWS Trusted Advisor & Compute Optimizer:** Integrate findings from Trusted Advisor and Compute Optimizer into the WA Tool's notes and improvement plans to create a single pane of glass for architectural cost optimization.
*   **AWS Cost Explorer & CUR:** Correlate High-Risk Issues (HRIs) identified in the WA Tool's Cost Optimization pillar with spending spikes in Cost Explorer to prioritize remediation efforts.
*   **AWS Resource Access Manager (RAM):** Use AWS RAM to centrally share Cost-Optimized Custom Lenses across all AWS accounts in an AWS Organization, ensuring uniform cost governance.
---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
*   `lineItem/ProductCode` to map specific AWS service costs back to the workloads being reviewed in the WA Tool.
*   `lineItem/UsageAccountId` to track cost efficiency improvements at the account level post-WA review.
### B. CloudWatch Metrics
*   CPU, Memory, and Network utilization metrics to answer the WA Tool's rightsizing and elasticity questions accurately.
### C. AWS Config / Trusted Advisor
*   AWS Config rules (e.g., `s3-bucket-lifecycle-enabled`) and Trusted Advisor cost optimization checks to automatically validate answers provided during the WA Tool review.
### D. Company Policies
*   Enterprise tagging strategies, approved instance type lists, and required architectural patterns to encode into Custom Lenses.
### E. IaC (Optional)
*   Terraform or AWS CloudFormation templates to verify that the deployed infrastructure matches the architecture described and reviewed within the WA Tool.
---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "WA-WST-003",
  "service": "AWS Well-Architected Tool",
  "category": "Waste Elimination",
  "name": "Custom Lenses for Waste Prevention",
  "description": "Deploy enterprise-specific Custom Lenses to ensure standards like S3 lifecycle policies and idle environment termination are validated.",
  "effort": "Medium",
  "impact": "High",
  "remediation_steps": [
    "Define organizational cost guardrails.",
    "Author a Custom Lens in JSON format incorporating these guardrails.",
    "Import and publish the Custom Lens in the AWS WA Tool.",
    "Share the Lens across the organization using AWS RAM.",
    "Mandate the Lens for all new workload reviews."
  ]
}
```
### Summary Report Table

| Finding ID | Category | Strategy Name | Impact | Effort |
|------------|----------|---------------|--------|--------|
| WA-WST-001 | Waste Elimination | Regular Cost Pillar Audits | High | Low |
| WA-WST-002 | Waste Elimination | Architectural Debt Tracking | Medium | Low |
| WA-WST-003 | Waste Elimination | Custom Lenses for Waste Prevention | High | Medium |
| WA-RSZ-001 | Rightsizing | Workload Sizing Validation | High | Low |
| WA-RSZ-002 | Rightsizing | Rightsizing Milestone Reviews | Medium | Low |
| WA-COM-001 | Commitment Discounts | Commitment Coverage Assessments | High | Low |
| WA-COM-002 | Commitment Discounts | Storage and Database Commitments | High | Low |
| WA-ARC-001 | Architecture Changes | Evaluate Serverless Migrations | High | High |
| WA-ARC-002 | Architecture Changes | Storage Tiering Validations | High | Medium |
| WA-ARC-003 | Architecture Changes | Open-Source Database Adoption | High | High |
| WA-ARC-004 | Architecture Changes | Managed Service Preferences | Medium | Medium |
| WA-SCL-001 | Scheduling & Auto-Scaling | Elasticity Audits | High | Low |
| WA-SCL-002 | Scheduling & Auto-Scaling | Non-Production Scheduling Validation | High | Medium |
| WA-SCL-003 | Scheduling & Auto-Scaling | Event-Driven Scaling Checks | Medium | Medium |
| WA-PRC-001 | Pricing Model Optimization | Spot Instance Adoption Reviews | High | Medium |
| WA-PRC-002 | Pricing Model Optimization | Graviton Processor Validations | High | Medium |
| WA-PRC-003 | Pricing Model Optimization | Regional Cost Benchmarking | Medium | Low |
| WA-NET-001 | Network & Data Transfer | Data Transfer Route Reviews | High | Medium |
| WA-NET-002 | Network & Data Transfer | Multi-AZ vs. Multi-Region Optimization | Medium | High |
| WA-NET-003 | Network & Data Transfer | NAT Gateway Usage Assessments | High | Medium |
