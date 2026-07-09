# Cost-Cutting Playbook: Amazon Kendra
> **Companion File:** [kendra.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/kendra/kendra.md)
> **Last Updated:** July 2026
---
## Executive Summary
Amazon Kendra is an intelligent enterprise search service powered by machine learning. Due to its hourly provisioned capacity model, Kendra can quickly become a significant cost center if indexes are left idle or improperly scaled. More importantly, Amazon Kendra entered **Maintenance Mode on June 30, 2026** and will stop accepting new customers by July 30, 2026. This playbook details strategies for eliminating waste in existing deployments while urgently prioritizing architectural migrations to Amazon Bedrock Knowledge Bases. The strategies encompass waste elimination, rightsizing, architecture shifts, and more, offering a comprehensive framework for optimizing Kendra spend.

## Strategy Categories
### 1. Waste Elimination
#### 1. Delete Idle Developer Indexes
- **What:** Identify and delete Developer Edition indexes that have no active queries but are left provisioned in sandbox environments.
- **Why It Saves Money:** Developer indexes cost $1.125/hour ($810.00/month) simply for being provisioned, even if they handle zero traffic.
- **Implementation Steps:**
  1. Use AWS Cost Explorer or CloudWatch Metrics (`QueryCount`) to find Developer indexes with zero or near-zero queries over 14 days.
  2. Confirm with the index owner if the environment is still in use.
  3. Backup data source configurations and delete the index.
- **Estimated Savings:** Up to 100% per idle index ($810/month).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** CloudWatch Metrics access, Kendra index permissions.

#### 2. Terminate Unused Enterprise Indexes in Non-Prod Environments
- **What:** Prevent the use of Enterprise Edition indexes in staging or development environments.
- **Why It Saves Money:** Enterprise Edition costs $1.40/hour ($1,008.00/month). Non-prod environments rarely need multi-AZ high availability or 100,000 document limits.
- **Implementation Steps:**
  1. Audit all Enterprise Edition indexes using the AWS CLI or Config.
  2. Identify indexes deployed outside of production accounts.
  3. Delete these indexes or replace them temporarily with Developer or GenAI Enterprise editions if testing is strictly required.
- **Estimated Savings:** $1,008/month per terminated index.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Environment mapping, IaC deployment access.

#### 3. Clean Up Abandoned GenAI Enterprise Indexes
- **What:** Decommission GenAI Enterprise Edition indexes that were spun up for Retrieval-Augmented Generation (RAG) proofs of concept and abandoned.
- **Why It Saves Money:** GenAI Enterprise indexes bill at $0.32/hour ($230.40/month) base cost, regardless of usage.
- **Implementation Steps:**
  1. List all active GenAI Enterprise indexes.
  2. Cross-reference with API usage or application access logs.
  3. Tear down the indexes that have been inactive for over 30 days.
- **Estimated Savings:** $230/month per index.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Usage visibility.

#### 4. Remove Unused Data Connectors
- **What:** Delete connectors (e.g., SharePoint, Salesforce, S3) that are no longer syncing active or relevant data into Kendra.
- **Why It Saves Money:** Sync operations incur compute/processing time and potentially API costs on the source systems.
- **Implementation Steps:**
  1. Review the data sources tab for each Kendra index.
  2. Identify connectors with continuous failures or obsolete data repositories.
  3. Delete the connectors to stop scheduled syncs.
- **Estimated Savings:** 1-5% (Indirect savings on API/compute).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of required data sources.

#### 5. Purge Stale/Duplicate Documents to Reduce Storage
- **What:** Implement retention policies to remove outdated or duplicate documents from the search index.
- **Why It Saves Money:** Keeps the total document count below the base tier limits (10,000 or 100,000), preventing the need for costly Storage Capacity Units (SCUs).
- **Implementation Steps:**
  1. Analyze the document freshness in the source repositories.
  2. Implement lifecycle rules or filters on the data connectors to exclude files older than X years.
  3. Force a re-sync to purge stale documents from Kendra.
- **Estimated Savings:** 10-30% (Avoids SCU overages).
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Data compliance and retention policies.

#### 6. Terminate Unutilized Capacity Units (SCUs/QCUs)
- **What:** Remove explicitly provisioned SCUs and QCUs that are no longer required due to decreased document volume or query traffic.
- **Why It Saves Money:** SCUs ($0.25-$0.35/hour) and QCUs ($0.07-$0.14/hour) are billed hourly when attached, regardless of actual consumption.
- **Implementation Steps:**
  1. Check Kendra metrics for document count and query throughput.
  2. If metrics are well below the provisioned capacity, use the AWS Console or CLI to scale down SCUs/QCUs.
- **Estimated Savings:** $50-$250/month per unit.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch monitoring of Kendra usage.

### 2. Rightsizing
#### 7. Consolidate Multiple Indexes Using Metadata Filtering
- **What:** Merge multiple departmental or tenant-specific indexes into a single central index, using metadata to isolate searches.
- **Why It Saves Money:** Instead of paying the base index hourly rate for N indexes, you pay for 1 index plus any required capacity units, heavily diluting the base cost overhead.
- **Implementation Steps:**
  1. Identify separate indexes that serve different departments (e.g., HR, Finance, Engineering).
  2. Migrate the data sources into a single Enterprise or GenAI Enterprise index.
  3. Tag documents with custom metadata (e.g., `Department=HR`).
  4. Modify the application search queries to include metadata filters.
- **Estimated Savings:** 40-80% (Depends on consolidation ratio).
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application code adjustments for query filtering.

#### 8. Downgrade Enterprise to GenAI Enterprise Edition
- **What:** Transition legacy Enterprise Edition indexes to the GenAI Enterprise Edition where feature parity permits.
- **Why It Saves Money:** Enterprise Edition base cost is $1.40/hour, whereas GenAI Enterprise is $0.32/hour—saving nearly $770/month per index.
- **Implementation Steps:**
  1. Evaluate if the current workload is primarily RAG/hybrid search.
  2. Verify that the 10,000 base document limit (with optional SCUs) fits the cost profile better than the 100,000 limit.
  3. Spin up a GenAI Enterprise index, migrate connectors, test, and cut over traffic.
  4. Delete the legacy Enterprise index.
- **Estimated Savings:** ~75% on base index costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Feature compatibility check, cutover window.

#### 9. Rightsize Storage Capacity Units (SCUs)
- **What:** Align provisioned SCUs strictly with the current document count and size requirements.
- **Why It Saves Money:** Over-provisioning SCUs leads to unnecessary hourly charges ($0.25 to $0.35/hr per unit).
- **Implementation Steps:**
  1. Review total indexed documents and size.
  2. Calculate required SCUs based on edition limits.
  3. Update index capacity settings to the minimum required SCUs.
- **Estimated Savings:** $180-$252/month per excess SCU.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Accurate capacity metric analysis.

#### 10. Rightsize Query Capacity Units (QCUs)
- **What:** Align provisioned QCUs strictly with peak daily query volumes.
- **Why It Saves Money:** Excess QCUs bill at $0.07 to $0.14/hr. Many internal search tools do not exceed the base 4,000 or 40,000 queries/day.
- **Implementation Steps:**
  1. Analyze `QueryCount` metrics in CloudWatch over a 30-day period.
  2. Determine the peak query per day rate.
  3. Reduce QCU provisioning to closely match the peak + 10% buffer.
- **Estimated Savings:** $50-$100/month per excess QCU.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch query metrics.

#### 11. Optimize Data Source Sync Frequencies
- **What:** Reduce the frequency of scheduled data syncs (e.g., from hourly to daily) if data volatility is low.
- **Why It Saves Money:** Reduces potential API charges on source systems, limits outbound data transfer costs, and lowers auxiliary processing compute.
- **Implementation Steps:**
  1. Review the sync schedules for all data connectors.
  2. Consult with business stakeholders on acceptable search index latency (e.g., 24 hours vs 1 hour).
  3. Modify the connector sync schedule accordingly.
- **Estimated Savings:** 1-5% (Indirect savings).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Stakeholder alignment on data freshness.

### 3. Commitment Discounts
#### 12. Leverage AWS Enterprise Discount Program (EDP)
- **What:** Include Kendra spend in overall organizational AWS EDP commitments.
- **Why It Saves Money:** While Kendra lacks specific Reserved Instances, EDPs provide a flat percentage discount across total AWS spend.
- **Implementation Steps:**
  1. Work with FinOps/Procurement to forecast Kendra usage prior to its final sunset.
  2. Roll this forecast into overall AWS commitment negotiations.
- **Estimated Savings:** 5-15% (EDP dependent).
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** High total AWS annual spend.

### 4. Architecture Changes
#### 13. Migrate to Amazon Bedrock Knowledge Bases (Urgent)
- **What:** Replace Amazon Kendra entirely with Amazon Bedrock Knowledge Bases backed by OpenSearch Serverless or Amazon RDS (pgvector).
- **Why It Saves Money:** Bypasses Kendra's massive $810-$1,008/month base provisioned costs. Bedrock + RDS can operate efficiently at a fraction of the cost (~$15-$50/month for small vector databases), operating on a pay-per-use or much lower base compute model.
- **Implementation Steps:**
  1. Acknowledge Kendra's Maintenance Mode status (Sunset July 2026).
  2. Export data source mappings and embedding strategies.
  3. Provision Amazon Bedrock Knowledge Bases and an underlying vector store (e.g., RDS pgvector).
  4. Update application search and RAG logic to point to the Bedrock API.
  5. Decommission the Kendra index.
- **Estimated Savings:** 80-95% of search infrastructure costs.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Generative AI architecture redesign, Bedrock access.

#### 14. Implement Query Caching for Popular Searches
- **What:** Place a caching layer (like Amazon ElastiCache for Redis) in front of Kendra for high-frequency identical queries.
- **Why It Saves Money:** Drastically reduces the total `QueryCount` hitting Kendra, potentially eliminating the need for expensive additional Query Capacity Units (QCUs).
- **Implementation Steps:**
  1. Analyze query logs for repetitive queries (e.g., "holiday policy", "IT helpdesk").
  2. Implement an application-level cache with a reasonable TTL (e.g., 24 hours).
  3. Route requests through the cache before querying Kendra.
- **Estimated Savings:** 10-20% (If QCUs are currently provisioned).
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Caching infrastructure, application code changes.

#### 15. Shift Structured Log Search to Amazon OpenSearch
- **What:** Stop using Kendra for heavily structured data or raw log aggregation, migrating those workloads to Amazon OpenSearch.
- **Why It Saves Money:** Kendra is designed and priced for complex natural language understanding on unstructured documents. Using it for simple structured search is highly cost-inefficient compared to OpenSearch.
- **Implementation Steps:**
  1. Identify indexes used primarily for structured data or logs.
  2. Deploy an Amazon OpenSearch cluster.
  3. Reroute data ingestion and search queries to OpenSearch.
  4. Delete the Kendra index.
- **Estimated Savings:** 40-60% (Workload dependent).
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** OpenSearch administration skills.

### 5. Scheduling & Auto-Scaling
#### 16. Schedule Ephemeral PoC Indexes via IaC
- **What:** Automate the creation and destruction of Developer/GenAI Developer indexes for CI/CD pipelines or active testing windows, rather than leaving them running 24/7.
- **Why It Saves Money:** Paying for a Developer index only during a 4-hour test window costs ~$4.50 instead of $810 for the whole month.
- **Implementation Steps:**
  1. Define the Kendra index and data sources using Terraform or AWS CDK.
  2. Integrate the deployment into the CI/CD pipeline.
  3. Ensure a strict `destroy` step runs upon test completion.
- **Estimated Savings:** Up to 95% for testing environments.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Infrastructure as Code maturity.

#### 17. Automate QCU Scaling via AWS Lambda
- **What:** Build a custom auto-scaler using CloudWatch Alarms and AWS Lambda to dynamically add or remove QCUs based on real-time query load.
- **Why It Saves Money:** Instead of provisioning QCUs for peak load 24/7, scale them up only during business hours or traffic spikes, saving $0.07-$0.14/hr during off-peak times.
- **Implementation Steps:**
  1. Create CloudWatch Alarms based on `QueryCount`.
  2. Write a Lambda function that calls `UpdateIndex` to adjust QCUs.
  3. Trigger the Lambda via EventBridge or CloudWatch Alarms.
- **Estimated Savings:** 20-40% of QCU costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Lambda and AWS API experience.

### 6. Pricing Model Optimization
#### 18. Maximize AWS Free Tier for Initial PoCs
- **What:** Utilize the Amazon Kendra Free Tier (750 free hours in the first 30 days) for all new proofs of concept before committing to paid usage.
- **Why It Saves Money:** Avoids the initial $810 to $1,008 costs while evaluating the service or finalizing migration plans.
- **Implementation Steps:**
  1. Track the creation date of the first Kendra index in a new AWS account.
  2. Ensure the PoC completes within the 30-day window.
  3. Terminate the index before the 750 hours/30 days expire.
- **Estimated Savings:** Up to $810-$1,008 in the first month.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** New Kendra usage in the account.

### 7. Network & Data Transfer Optimization
#### 19. Co-Locate Data Sources and Indexes in the Same Region
- **What:** Ensure that S3 buckets, RDS databases, and other AWS data sources are in the same AWS Region as the Kendra index.
- **Why It Saves Money:** Avoids AWS Cross-Region Data Transfer Out costs ($0.02/GB) during the heavy initial syncs and ongoing differential syncs.
- **Implementation Steps:**
  1. Audit data source locations relative to the Kendra index region (e.g., us-east-1).
  2. If a mismatch exists, migrate the data or deploy the Kendra index in the data's region (if available).
- **Estimated Savings:** 1-5% (Depends on data volume).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region architecture visibility.

#### 20. Utilize VPC Endpoints (PrivateLink) to Bypass NAT Gateway Fees
- **What:** Configure VPC Endpoints for S3 and other AWS services if Kendra is syncing massive amounts of data from within a private VPC subnet.
- **Why It Saves Money:** Routing terabytes of document data through a NAT Gateway incurs $0.045/GB data processing charges. Gateway VPC Endpoints for S3 are free, and Interface Endpoints for other services are significantly cheaper.
- **Implementation Steps:**
  1. Analyze VPC Flow Logs and NAT Gateway metrics.
  2. Create a Gateway VPC Endpoint for S3 in the VPC where data sources reside.
  3. Ensure route tables direct S3 traffic through the endpoint.
- **Estimated Savings:** ~$45 per TB of synced data.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC administration access.

---
## Cross-Service Synergies
- **Amazon Bedrock & Amazon RDS pgvector:** The most critical synergy for cost-cutting is moving entirely off Kendra onto these services, eliminating high flat-rate provisioned index costs and embracing a usage-based or cheaper provisioned vector database model.
- **Amazon ElastiCache:** Implementing caching reduces query volume, avoiding expensive Kendra QCU expansions.
- **AWS Lambda & CloudWatch:** Enables custom auto-scaling for Capacity Units, avoiding 24/7 peak-provisioning waste.
- **Amazon OpenSearch:** A far more cost-effective target for structured and log data search capabilities compared to Kendra.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query for `line_item_product_code = 'AmazonKendra'` to capture exact hourly charges for Indexes, SCUs, and QCUs.
### B. CloudWatch Metrics
- `QueryCount`, `DocumentCount`, `IndexSize` to determine utilization and scaling needs.
### C. AWS Config / Trusted Advisor
- Resource tracking for idle indexes and historical configuration changes.
### D. Company Policies
- Data retention and deletion policies to purge stale documents.
### E. IaC (Optional)
- Terraform/CloudFormation state files to identify environment mapping (Prod vs. Non-Prod) and facilitate ephemeral index scheduling.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "KENDRA-001",
  "resource_id": "arn:aws:kendra:us-east-1:123456789012:index/abc-123",
  "strategy": "Delete Idle Developer Indexes",
  "category": "Waste Elimination",
  "current_monthly_cost": 810.00,
  "projected_monthly_cost": 0.00,
  "savings_percentage": 100,
  "effort": "Low",
  "status": "Actionable"
}
```

### Summary Report Table
| Finding ID | Strategy | Resource | Current Cost | Projected Cost | Savings | Effort |
|------------|----------|----------|--------------|----------------|---------|--------|
| KENDRA-001 | Delete Idle Dev Index | index/abc-123 | $810.00 | $0.00 | $810.00 | Low |
| KENDRA-002 | Migrate to Bedrock | index/xyz-987 | $1,008.00 | $50.00 | $958.00 | High |
