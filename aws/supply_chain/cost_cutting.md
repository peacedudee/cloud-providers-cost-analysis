# Cost-Cutting Playbook: AWS Supply Chain
> **Companion File:** [supply_chain.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/supply_chain/supply_chain.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Supply Chain provides ML-powered supply chain visibility, demand planning, and N-tier vendor tracking. Billed primarily on a per-user, per-module basis alongside standard S3 storage costs for its contextual data lake, cost optimization centers around stringent license management, role-based access control, and efficient data ingestion. This playbook outlines 20 strategies to minimize licensing bloat and optimize the underlying data infrastructure.

## Strategy Categories
### 1. Waste Elimination
*   **SUPPLY-WST-001: Deactivate Dormant User Accounts**
    *   **Description:** Audit user activity logs and remove Supply Chain access for users who have not logged in within the last 30-60 days to save $10-$60/month per user.
*   **SUPPLY-WST-002: Automate Offboarding Processes**
    *   **Description:** Integrate AWS IAM/SSO with HR systems to instantly revoke AWS Supply Chain licenses upon employee departure or role change.
*   **SUPPLY-WST-003: Remove Unused Demand Planning Modules**
    *   **Description:** Identify users assigned the $30/mo Demand Planning add-on who only view inventory insights, and revoke their add-on license.
*   **SUPPLY-WST-004: Remove Unused N-Tier Visibility Add-ons**
    *   **Description:** Identify users assigned the $20/mo N-Tier Visibility add-on who do not interact with vendor portals, and downgrade them to the base license.
*   **SUPPLY-WST-005: Clean Up Stale Data Lake Storage**
    *   **Description:** Delete obsolete, test, or duplicate datasets from the underlying S3 buckets that power the Supply Chain data lake to reduce storage fees.

### 2. Rightsizing
*   **SUPPLY-RSZ-001: Role-Based License Tier Allocation**
    *   **Description:** Assign full add-ons (Demand Planning and N-Tier) strictly to specialized inventory planners and procurement managers, keeping general warehouse operators on Base Insights ($10/mo).
*   **SUPPLY-RSZ-002: Downgrade Executive Observer Licenses**
    *   **Description:** Ensure executives and stakeholders who only need high-level dashboard visibility are assigned the $10/mo Base Insights license, not full operational add-ons.
*   **SUPPLY-RSZ-003: Distribute Exported Reports to Non-Users**
    *   **Description:** Instead of provisioning licenses for stakeholders who only read monthly summaries, export reports from AWS Supply Chain and distribute them via email, QuickSight, or an intranet portal.
*   **SUPPLY-RSZ-004: Rightsize Ingestion Frequency**
    *   **Description:** Reduce the frequency of data updates from ERP systems (e.g., from hourly to daily) if real-time visibility is not strictly required, reducing related ETL compute and S3 PUT request costs.

### 3. Commitment Discounts
*   **SUPPLY-CMT-001: S3 Intelligent-Tiering for Data Lake**
    *   **Description:** Apply S3 Intelligent-Tiering lifecycle policies to the underlying Supply Chain S3 buckets to automatically move infrequently accessed historical supply chain data to cheaper storage tiers.
*   **SUPPLY-CMT-002: Enterprise Discount Program (EDP)**
    *   **Description:** Ensure that AWS Supply Chain licensing and associated storage costs are factored into overall EDP negotiations for flat percentage discounts across the board.

### 4. Architecture Changes
*   **SUPPLY-ARC-001: Delta Data Ingestion**
    *   **Description:** Configure ERP data connectors to only send delta (changed) records rather than full historical dumps to minimize S3 storage, processing overhead, and API calls.
*   **SUPPLY-ARC-002: Compress Ingested Data**
    *   **Description:** Ensure data extracted from systems like SAP or Oracle is compressed (e.g., using Parquet, GZIP) before landing in the AWS Supply Chain S3 data lake.
*   **SUPPLY-ARC-003: Filter Irrelevant SKUs/Locations**
    *   **Description:** Exclude discontinued products, inactive locations, or non-supply-chain-relevant inventory from ingestion pipelines to keep the data lake lean and performant.
*   **SUPPLY-ARC-004: Consolidate Supply Chain Instances**
    *   **Description:** If multiple distinct instances were spun up for different regions, consolidate them into a single global instance (using data segregation features) to avoid duplicate base license provisioning for users needing global views.

### 5. Scheduling & Auto-Scaling
*   **SUPPLY-SCH-001: Off-Peak Batch Processing**
    *   **Description:** Schedule heavy ERP data extractions and transformations using AWS Glue or Step Functions during off-peak hours to take advantage of lower spot compute rates for the ETL pipeline feeding the application.
*   **SUPPLY-SCH-002: Auto-Suspend Dependent ETL Pipelines**
    *   **Description:** Implement logic to pause ingestion pipelines feeding the Supply Chain data lake during scheduled ERP maintenance windows to prevent failed retry loops and wasted compute charges.

### 6. Pricing Model Optimization
*   **SUPPLY-PRC-001: Leverage the 60-Day Free Trial**
    *   **Description:** Utilize the 60-day free trial for up to 10 users to thoroughly validate Demand Planning and N-Tier Visibility modules via a Proof of Concept before committing to paid licenses for the wider team.
*   **SUPPLY-PRC-002: Evaluate Add-on ROI**
    *   **Description:** Periodically assess whether the improved forecasting accuracy generated by the $30/mo Demand Planning module justifies the cost compared to legacy or alternative forecasting tools.

### 7. Network & Data Transfer Optimization
*   **SUPPLY-NET-001: Regional Co-location**
    *   **Description:** Deploy the AWS Supply Chain instance in the same AWS region as the primary ERP database or data warehouse to eliminate cross-region data transfer fees during ingestion.
*   **SUPPLY-NET-002: Optimize On-Premise Data Ingress**
    *   **Description:** Use AWS Direct Connect or PrivateLink to securely and cost-effectively route supply chain data from on-premise ERPs to AWS without incurring high internet-bound NAT Gateway or egress charges.

---
## Cross-Service Synergies
*   **AWS IAM / SSO:** Streamlines license management by automating the provisioning and de-provisioning of Supply Chain users based on corporate Active Directory groups.
*   **Amazon S3:** The foundation of the Supply Chain contextual data lake; implementing S3 lifecycle policies directly impacts the total cost of ownership.
*   **AWS CloudTrail:** Used to audit user login events to identify dormant accounts that can be stripped of paid licenses.
*   **AWS Glue / Lambda:** Often used to prepare and ingest data; optimizing these ETL pipelines reduces the hidden costs of running AWS Supply Chain.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
*   `lineItem/ProductCode` (e.g., `AWSSupplyChain`)
*   `lineItem/Operation` (e.g., `SupplyChainInsights-User`, `DemandPlanning-User`)
*   `lineItem/UsageType` (e.g., User-Months, S3 storage metrics)
*   `lineItem/UnblendedCost`

### B. CloudWatch Metrics
*   *Note: AWS Supply Chain usage is primarily license-based; CloudWatch would track custom ETL pipeline metrics (e.g., Lambda invocations, Glue DPU hours) rather than the SaaS application users itself.*

### C. AWS Config / Trusted Advisor
*   AWS Config: Track configurations of underlying S3 buckets (e.g., checking if lifecycle policies are attached and data is encrypted).

### D. Company Policies
*   **Access Control Policy:** Defines which roles (e.g., Warehouse Staff vs. Planner vs. Exec) require which Supply Chain add-on modules.
*   **Offboarding Policy:** SLAs for how quickly user access must be revoked after termination.

### E. IaC (Optional)
*   Terraform/CloudFormation scripts for the automated ETL pipelines, IAM roles, and S3 buckets feeding the Supply Chain data lake.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "SUPPLY-RSZ-001",
  "category": "Rightsizing",
  "resource_id": "user-group-warehouse-ops",
  "condition": "Users assigned full $60/mo suite but only require $10/mo base insights",
  "recommendation": "Downgrade 50 warehouse operators to Base Supply Chain Insights",
  "estimated_monthly_savings": 2500,
  "level_of_effort": "Low"
}
```

### Summary Report Table
| Finding ID | Category | Resource / Group | Current Monthly Cost | Projected Monthly Cost | Est. Savings | Effort |
|------------|----------|------------------|----------------------|------------------------|--------------|--------|
| SUPPLY-RSZ-001 | Rightsizing | Warehouse Ops | $3,000 | $500 | $2,500 | Low |
| SUPPLY-WST-001 | Waste Elimination | Dormant Users | $600 | $0 | $600 | Low |
| SUPPLY-ARC-001 | Architecture Changes | ERP Data Pipeline | $200 | $50 | $150 | Medium |
