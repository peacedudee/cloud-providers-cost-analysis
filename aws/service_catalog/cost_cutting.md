# Cost-Cutting Playbook: AWS Service Catalog
> **Companion File:** [service_catalog.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/service_catalog/service_catalog.md)
> **Last Updated:** July 2026

---

## Executive Summary
AWS Service Catalog itself is highly inexpensive, billing primarily on a per-API-call basis with a generous free tier of 1,000 calls per month. As a result, the primary financial value of Service Catalog is not in optimizing its own footprint, but in utilizing its governance capabilities to enforce cost-effective behaviors across the broader AWS environment. This playbook outlines 18 strategies to use Service Catalog as a cost-control mechanism—restricting expensive resources, enforcing lifecycle policies, and ensuring default architectures use the most economical pricing models, alongside a few best practices to limit native API costs.

## Strategy Categories

### 1. Waste Elimination
*   **SVCCAT-WST-001: Terminate Abandoned Provisioned Products:** Use Service Catalog TagOptions to mandate ownership tags (e.g., `OwnerEmail`, `CostCenter`). Periodically audit and terminate provisioned products that belong to offboarded employees or have been abandoned.
*   **SVCCAT-WST-002: Restrict Expensive Resource Types:** Modify CloudFormation/Terraform templates to remove high-cost, specialized resources (e.g., `p4d` or `x1e` instances) from standard portfolios, requiring manual approval for out-of-bounds requests.
*   **SVCCAT-WST-003: Standardize TTLs on Development Stacks:** Implement automation that pairs with Service Catalog to automatically tear down Dev/Test provisioned products after a set time-to-live (TTL) (e.g., 7 or 14 days), preventing runaway zombie infrastructure.
*   **SVCCAT-WST-004: Delete Unused Portfolios and Products:** Regularly audit Service Catalog for unused or outdated product versions and portfolios. While they don't incur storage costs, cleaning them up reduces administrative overhead and prevents accidental provisioning of legacy, unoptimized infrastructure.

### 2. Rightsizing
*   **SVCCAT-RGT-001: Mandate Graviton/ARM Templates:** Update standard compute templates provided in Service Catalog to use Graviton instances (e.g., `t4g`, `m7g`) by default, driving a 20%+ price-performance improvement over x86.
*   **SVCCAT-RGT-002: Enforce Default Instance Constraints:** Use Service Catalog template constraints (Template Rules) to limit the maximum allowed instance sizes a developer can select (e.g., restricting selections to `.micro`, `.small`, and `.medium`).
*   **SVCCAT-RGT-003: Optimize Storage Types in Templates:** Standardize on `gp3` instead of `gp2` for all EBS volumes defined within Service Catalog products, and enforce lower default provisioned IOPS/Throughput.
*   **SVCCAT-RGT-004: Implement Auto-scaling by Default:** Ensure that application templates provision Auto Scaling Groups (ASGs) rather than fixed-size EC2 fleets, allowing infrastructure to scale down during low-traffic periods.

### 3. Commitment Discounts
*   **SVCCAT-COM-001: Align Templates with Active Commitments:** Modify Service Catalog constraints to heavily favor instance types, families, and AWS Regions where the organization has high, unutilized active Compute Savings Plans or Reserved Instances.

### 4. Architecture Changes
*   **SVCCAT-ARC-001: Cache API Calls in Custom Portals:** If integrating internal IT portals (e.g., ServiceNow, Jira) with Service Catalog, implement local in-memory caching for API responses (`DescribePortfolio`, `ListLaunchPaths`) to stay within the 1,000 free API calls/month limit and avoid $0.0007/call charges.
*   **SVCCAT-ARC-002: Shift to Serverless Templates:** Create and heavily promote Service Catalog products that deploy serverless architectures (Lambda, DynamoDB, API Gateway) over traditional EC2-based monolithic templates.
*   **SVCCAT-ARC-003: Adopt Terraform Reference Architectures:** For organizations using Terraform with Service Catalog, mandate the use of pre-optimized, FinOps-approved Terraform modules that adhere strictly to cost-saving best practices.

### 5. Scheduling & Auto-Scaling
*   **SVCCAT-SCH-001: Bake Instance Scheduler Tags into Templates:** Automatically append resource tags (e.g., `Schedule=BusinessHours`) to all provisioned products so that the AWS Instance Scheduler automatically turns off non-production resources at night and on weekends.
*   **SVCCAT-SCH-002: Integrate with Systems Manager Runbooks:** Provide cost-saving SSM runbooks via Service Catalog AppRegistry to allow users to quickly stop/start entire groups of related application resources simultaneously.

### 6. Pricing Model Optimization
*   **SVCCAT-PRC-001: Default to Spot Instances for Dev/Test:** Create specific "Dev/Test" or "Batch Processing" Service Catalog products that mandate the use of Amazon EC2 Spot Instances, reducing compute costs by up to 90%.
*   **SVCCAT-PRC-002: Require S3 Intelligent-Tiering:** Ensure any Amazon S3 buckets provisioned through Service Catalog are configured to use the `INTELLIGENT_TIERING` storage class by default to automatically save on storage costs.

### 7. Network & Data Transfer Optimization
*   **SVCCAT-NET-001: Enforce Private Endpoints (VPC Endpoints):** Ensure that standard network/VPC templates mandate the use of VPC endpoints (PrivateLink) instead of NAT Gateways for AWS API interactions to reduce heavy data processing and transfer costs.
*   **SVCCAT-NET-002: Restrict Multi-Region Deployments:** Use Service Catalog portfolio sharing restrictions to limit deployments to specific, low-cost AWS regions (e.g., `us-east-1`, `us-east-2`, `us-west-2`), preventing developers from deploying to expensive regions unless absolutely necessary.

---

## Cross-Service Synergies
*   **AWS CloudFormation / HashiCorp Terraform:** Service Catalog relies entirely on the quality of the underlying IaC templates. Cost optimization in Service Catalog is intrinsically linked to writing highly cost-efficient CloudFormation or Terraform code.
*   **AWS Identity and Access Management (IAM):** Restricting user permissions to strictly deploy via Service Catalog prevents backdoor creation of expensive resources via the AWS Console or CLI.
*   **AWS Organizations & Service Control Policies (SCPs):** Use SCPs to mandate that certain resource types (like standard EC2s or RDS instances) can *only* be created via the Service Catalog assumed role, centralizing cost governance.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
*   Filter by `lineItem/ProductCode` = `AWSServiceCatalog` to monitor API call volume and costs.
*   Analyze `resourceTags/user:CostCenter` to track costs of products provisioned *via* Service Catalog.

### B. CloudWatch Metrics
*   Monitor API call rates to `ServiceCatalog` to identify aggressive polling scripts or CI/CD pipelines that might exceed the free tier.

### C. AWS Config / Trusted Advisor
*   Use AWS Config to track the compliance of provisioned products against organizational rules (e.g., checking if tags match TagOptions).

### D. Company Policies
*   FinOps tagging requirements and approved instance type lists to encode as constraints within Service Catalog.

### E. IaC (Optional)
*   CloudFormation (`.yaml`/`.json`) and Terraform (`.tf`) templates used to define the Service Catalog portfolios, analyzed for cost-efficiency.

---

## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "SVCCAT-WST-002",
  "category": "Waste Elimination",
  "service": "AWS Service Catalog",
  "title": "Restrict Expensive Resource Types",
  "description": "Modify templates to remove high-cost instance types (e.g., p4d) from standard portfolios.",
  "risk_level": "Medium",
  "effort_to_fix": "Low",
  "estimated_savings_percentage": "10-25%",
  "remediation": "Update Service Catalog template constraints to explicitly deny massive instance types."
}
```

### Summary Report Table
| Finding ID | Category | Title | Risk Level | Effort | Estimated Savings |
|------------|----------|-------|------------|--------|-------------------|
| SVCCAT-WST-002 | Waste Elimination | Restrict Expensive Resource Types | Medium | Low | 10-25% |
| SVCCAT-RGT-001 | Rightsizing | Mandate Graviton/ARM Templates | High | Medium | 20% |
| SVCCAT-PRC-001 | Pricing Model Optimization | Default to Spot Instances for Dev/Test | High | Low | up to 90% |
| SVCCAT-ARC-001 | Architecture Changes | Cache API Calls in Custom Portals | Low | Medium | < 5% |
| SVCCAT-SCH-001 | Scheduling & Auto-Scaling | Bake Instance Scheduler Tags into Templates | Medium | Low | 30-70% |
