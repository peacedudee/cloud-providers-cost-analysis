# Cost-Cutting Playbook: AWS Clean Rooms
> **Companion File:** [clean_rooms.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/clean_rooms/clean_rooms.md)
> **Last Updated:** July 2026
---
## Executive Summary
This playbook outlines comprehensive strategies for optimizing costs within AWS Clean Rooms. As a serverless collaboration service billed entirely on Clean Rooms Processing Units (CRPU-hours), cost optimization hinges on highly efficient query design, smart data formatting, avoiding strict minimum billing windows, and ensuring strict governance over partner query execution and payment allocations. By implementing these practices, organizations can drastically lower their CRPU burn rates while maintaining secure, multi-party data collaborations.

## Strategy Categories

### 1. Waste Elimination
#### CLEAN-01. Deprovision Inactive Collaborations
- **What:** Delete or suspend collaborations that are no longer actively used by partners.
- **Why It Saves Money:** Prevents accidental or rogue queries from being executed by partners, eliminating unexpected CRPU compute charges.
- **Implementation Steps:** 
  1. Audit CloudWatch `CRPUConsumed` metric grouped by collaboration. 
  2. Identify collaborations with zero usage over the last 90 days. 
  3. Delete or suspend the inactive collaboration to revoke query capabilities.
- **Estimated Savings:** 5-15% (Risk avoidance)
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** CloudWatch metrics access

#### CLEAN-02. Delete Unused Clean Rooms ML Models
- **What:** Remove Clean Rooms ML models that are hosted but no longer actively used for inference.
- **Why It Saves Money:** Hosting active ML models consumes ongoing CRPU-hours. Deleting unused models stops this continuous $0.027/CRPU-hour drip.
- **Implementation Steps:** 
  1. Identify deployed ML models in AWS Clean Rooms. 
  2. Check invocation logs in CloudWatch. 
  3. Delete models that have no recent invocations.
- **Estimated Savings:** 10-25% of ML costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Clean Rooms ML logging enabled

#### CLEAN-03. Prune Orphaned S3 Query Result Tables
- **What:** Apply S3 Lifecycle policies to buckets storing Clean Rooms query results and intermediate tables.
- **Why It Saves Money:** Clean Rooms outputs query results directly to S3. Without lifecycle policies, terabytes of outdated query results accumulate over time, driving up S3 storage costs.
- **Implementation Steps:** 
  1. Identify the S3 buckets configured for Clean Rooms outputs. 
  2. Apply a lifecycle rule to transition to standard-IA or expire/delete objects after 30 days.
- **Estimated Savings:** 5-10% of underlying S3 costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** S3 administrative access

### 2. Rightsizing
#### CLEAN-04. Enforce Strict Query Analysis Rules
- **What:** Configure Clean Rooms analysis rules to limit query complexity (e.g., restricting to list intersections or specific aggregations).
- **Why It Saves Money:** Prevents partners from running unoptimized, full-table multi-billion row joins that consume massive amounts of CRPUs.
- **Implementation Steps:** 
  1. Review current analysis rules on shared tables. 
  2. Restrict allowed SQL operations to explicitly necessary aggregations. 
  3. Block unrestricted cross-joins and full-table scans.
- **Estimated Savings:** 20-40% per query execution
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of partner query requirements

#### CLEAN-05. Optimize Underlying S3 Data to Parquet/ORC
- **What:** Convert JSON/CSV datasets shared in Clean Rooms to columnar formats like Apache Parquet.
- **Why It Saves Money:** Columnar formats reduce the amount of data scanned. Queries run faster and consume fewer CRPU-seconds, directly lowering the compute bill.
- **Implementation Steps:** 
  1. Use AWS Glue to convert existing CSV/JSON data to Parquet. 
  2. Update Clean Rooms table definitions and Glue Data Catalog to point to the new formats.
- **Estimated Savings:** 40-70% per query
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS Glue or EMR for data conversion

#### CLEAN-06. Partition S3 Data to Limit Scan Scope
- **What:** Partition the underlying S3 datasets by date, region, or other high-cardinality fields.
- **Why It Saves Money:** Partition pruning allows Clean Rooms queries to skip irrelevant data, drastically reducing execution time and CRPU consumption.
- **Implementation Steps:** 
  1. Organize data in S3 via partitions (e.g., `s3://bucket/year=2026/month=07/`). 
  2. Define partitions in the AWS Glue Data Catalog used by Clean Rooms.
- **Estimated Savings:** 30-60% per query
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Partitioned S3 data structure

#### CLEAN-07. Avoid Unindexed Multi-Billion Row Joins
- **What:** Ensure all partner tables joined in Clean Rooms queries have appropriate indexing and strict join keys defined in the analysis rules.
- **Why It Saves Money:** Unoptimized massive joins spin up huge CRPU clusters. Enforcing join constraints reduces the CRPU-hours billed.
- **Implementation Steps:** 
  1. Analyze slow-running queries. 
  2. Enforce the use of specific join keys in the table analysis rules. 
  3. Pre-filter data locally before engaging the join.
- **Estimated Savings:** 25-50%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** DBA/Data Engineering expertise

### 3. Commitment Discounts
#### CLEAN-08. Leverage Enterprise Discount Program (EDP)
- **What:** Include AWS Clean Rooms spend in the overall AWS Enterprise Discount Program (EDP) commitment.
- **Why It Saves Money:** Clean Rooms does not offer native Savings Plans or Reserved Instances. An EDP provides a flat percentage discount across all services, including Clean Rooms CRPUs.
- **Implementation Steps:** 
  1. Forecast Clean Rooms usage for the next 1-3 years. 
  2. Work with the AWS account team to bundle it into EDP negotiations.
- **Estimated Savings:** 5-15% across all Clean Rooms spend
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** High overall AWS spend to qualify for EDP

### 4. Architecture Changes
#### CLEAN-09. Utilize Intermediate Tables for Cached Queries
- **What:** Write intermediate query outputs to a Clean Rooms Intermediate Table for multi-step reports.
- **Why It Saves Money:** Downstream analytical queries read the smaller, cached data rather than repeatedly re-executing expensive joins across raw partner data.
- **Implementation Steps:** 
  1. Identify repetitive, complex multi-step queries. 
  2. Configure them to output to an Intermediate Table. 
  3. Point downstream reporting tools to this cached table.
- **Estimated Savings:** 30-50%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Intermediate Tables feature configured

#### CLEAN-10. Shift Short PySpark Jobs to SQL
- **What:** Convert PySpark jobs that take less than 10 minutes to execute into standard SQL queries.
- **Why It Saves Money:** PySpark jobs incur a strict **10-minute minimum** CRPU charge, whereas SQL queries only have a **60-second minimum**. Moving short jobs to SQL avoids paying for 9 idle minutes of compute.
- **Implementation Steps:** 
  1. Audit PySpark execution times. 
  2. Identify jobs taking < 10 mins. 
  3. Rewrite the transformation logic using Clean Rooms SQL.
- **Estimated Savings:** Up to 90% per execution for short jobs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Advanced SQL expertise

#### CLEAN-11. Pre-Aggregate Data Prior to Collaboration
- **What:** Aggregate and summarize data in your own AWS account (via Athena or Glue) before exposing it to Clean Rooms.
- **Why It Saves Money:** Clean Rooms CRPU rates apply to all processing within the collaboration. Pre-aggregating data using cheaper tools (like Athena) reduces the heavy compute load handled by Clean Rooms.
- **Implementation Steps:** 
  1. Build an ETL pipeline to pre-aggregate raw data. 
  2. Expose only the highly aggregated tables to the Clean Rooms collaboration.
- **Estimated Savings:** 20-40%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Internal ETL pipelines

#### CLEAN-12. Consolidate Overlapping Collaborations
- **What:** Merge multiple smaller collaborations that share the same partners and datasets into a single, unified collaboration.
- **Why It Saves Money:** Reduces administrative overhead and avoids duplicate data processing or redundant model hosting for identical partner sets.
- **Implementation Steps:** 
  1. Map out all active collaborations. 
  2. Identify overlapping participants and datasets. 
  3. Consolidate them and unify IAM/Analysis rules.
- **Estimated Savings:** 5-15%
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Partner agreement to consolidate

### 5. Scheduling & Auto-Scaling
#### CLEAN-13. Batch PySpark Analytical Workloads
- **What:** Group multiple PySpark transformations into a single, longer-running job rather than scheduling frequent short jobs.
- **Why It Saves Money:** Frequent short jobs trigger the 10-minute minimum billing penalty every time (e.g., running every 2 minutes multiplies costs 5x). Batching them into a single 30-minute daily job completely bypasses the minimum penalties.
- **Implementation Steps:** 
  1. Review PySpark job schedules. 
  2. Change cron schedules from hourly/minutely to daily. 
  3. Combine scripts to process data in bulk.
- **Estimated Savings:** 50-80% on PySpark costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Tolerance for higher data latency

#### CLEAN-14. Consolidate Small SQL Queries
- **What:** Combine multiple small SQL checks or aggregations into a single, comprehensive SQL query.
- **Why It Saves Money:** Every SQL query incurs a 60-second minimum charge. Running ten 2-second queries costs 600 seconds of CRPU time. Running one 20-second query combining all checks costs exactly 60 seconds.
- **Implementation Steps:** 
  1. Identify rapid, sequential SQL queries triggered by dashboards. 
  2. Rewrite them into a single query using UNIONs or multiple aggregations.
- **Estimated Savings:** Up to 90% on small query costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Query refactoring capabilities

#### CLEAN-15. Batch ML Training and Inference Tasks
- **What:** Group data for Clean Rooms ML into larger batches before running training or inference tasks.
- **Why It Saves Money:** Avoids triggering the 60-second minimum CRPU charge repeatedly for rapid, small ML inference requests.
- **Implementation Steps:** 
  1. Queue inference requests locally or in S3. 
  2. Trigger Clean Rooms ML on the batched dataset on a predefined schedule.
- **Estimated Savings:** 20-50% on ML costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Batch processing architecture

### 6. Pricing Model Optimization
#### CLEAN-16. Configure CloudWatch Budget Alarms on CRPUConsumed
- **What:** Set up alarms to notify teams or trigger Lambda functions to revoke access when CRPU limits are breached.
- **Why It Saves Money:** Provides a hard ceiling on costs, preventing massive accidental cost overruns from poorly written partner queries.
- **Implementation Steps:** 
  1. Create a CloudWatch Alarm on the `CRPUConsumed` metric. 
  2. Set the threshold to the monthly budget limit. 
  3. Route alerts to SNS to notify the FinOps team.
- **Estimated Savings:** 10-100% (Prevents unlimited overruns)
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** CloudWatch configuration access

#### CLEAN-17. Optimize Member Payment Rules (Flex Billing)
- **What:** Ensure collaborations are configured so that the member deriving value from the query pays for the CRPUs, rather than sponsoring all queries unconditionally.
- **Why It Saves Money:** If configured as the "Sponsor", your account pays for all partner queries regardless of efficiency. Shifting to default member billing forces partners to optimize their own queries and bear the cost.
- **Implementation Steps:** 
  1. Audit collaboration payment settings. 
  2. Negotiate billing splits with partners. 
  3. Reconfigure the collaboration to remove unrestricted sponsor billing.
- **Estimated Savings:** Up to 100% of partner-incurred compute costs
- **Risk Level:** High
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** Contractual agreement with partners

### 7. Network & Data Transfer Optimization
#### CLEAN-18. Align S3 Bucket Regions with Collaboration Region
- **What:** Ensure the S3 buckets hosting the raw data are in the same AWS region as the Clean Rooms collaboration.
- **Why It Saves Money:** If Clean Rooms in `us-east-1` queries data in an S3 bucket in `eu-west-1`, you will incur significant cross-region data transfer charges ($0.02/GB). Aligning regions drops this cost to zero.
- **Implementation Steps:** 
  1. Map S3 bucket regions against the Collaboration region. 
  2. Migrate S3 buckets to the matching region if a mismatch exists.
- **Estimated Savings:** 100% of cross-region transfer costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** S3 migration capability

#### CLEAN-19. Deploy VPC Endpoints for S3 and Clean Rooms API
- **What:** Access S3 and Clean Rooms APIs via VPC Gateway/Interface Endpoints rather than traversing a NAT Gateway.
- **Why It Saves Money:** API calls and data movement through a NAT Gateway incur per-GB data processing charges ($0.045/GB). VPC Endpoints keep traffic on the AWS network for free (Gateway Endpoints) or at a much lower cost (Interface Endpoints).
- **Implementation Steps:** 
  1. Create an S3 Gateway Endpoint in your VPC. 
  2. Create an Interface Endpoint for Clean Rooms. 
  3. Update VPC route tables.
- **Estimated Savings:** $0.045 per GB of data transferred
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC administrative access

---
## Cross-Service Synergies
- **AWS S3:** Lifecycle policies and intelligent tiering on query output buckets drastically reduce storage overhead.
- **AWS Glue / Athena:** Pre-aggregating and converting data to Parquet via Glue/Athena before sharing in Clean Rooms yields massive CRPU savings.
- **Amazon CloudWatch:** Using `CRPUConsumed` to trigger budget alarms is the primary defense against run-away partner query costs.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Analyze usage by `productCode` (Clean Rooms) and `operation` (e.g., SQL execution, PySpark execution, ML execution) to identify cost drivers.
### B. CloudWatch Metrics
- Monitor `CRPUConsumed` at the collaboration level to identify inefficient queries.
### C. AWS Config / Trusted Advisor
- Audit for unused or abandoned Clean Rooms collaborations and ML models.
### D. Company Policies
- Review data sharing agreements to validate billing splits (Sponsor vs. Member billing).
### E. IaC (Optional)
- Review Terraform/CloudFormation templates to ensure VPC endpoints and S3 lifecycle rules are consistently applied.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "CLEAN-013",
  "service": "AWS Clean Rooms",
  "strategy": "Batch PySpark Analytical Workloads",
  "description": "Consolidate 10-minute PySpark tasks into a single daily batch to bypass minimum CRPU run penalties.",
  "estimated_savings_percentage": 60,
  "risk_level": "Low",
  "implementation_scope": "Engineer/DevOps"
}
```

### Summary Report Table
| ID | Strategy | Scope | Risk | Est. Savings |
|---|---|---|---|---|
| CLEAN-01 | Deprovision Inactive Collaborations | FinOps Team | Low | 5-15% |
| CLEAN-02 | Delete Unused Clean Rooms ML Models | Engineer/DevOps | Low | 10-25% |
| CLEAN-03 | Prune Orphaned S3 Query Result Tables | Engineer/DevOps | Low | 5-10% |
| CLEAN-04 | Enforce Strict Query Analysis Rules | Engineer/DevOps | Medium | 20-40% |
| CLEAN-05 | Optimize Underlying S3 Data to Parquet/ORC | Engineer/DevOps | Low | 40-70% |
| CLEAN-06 | Partition S3 Data to Limit Scan Scope | Engineer/DevOps | Low | 30-60% |
| CLEAN-07 | Avoid Unindexed Multi-Billion Row Joins | Engineer/DevOps | Medium | 25-50% |
| CLEAN-08 | Leverage Enterprise Discount Program (EDP) | Procurement/Leadership | Low | 5-15% |
| CLEAN-09 | Utilize Intermediate Tables for Cached Queries | Engineer/DevOps | Medium | 30-50% |
| CLEAN-10 | Shift Short PySpark Jobs to SQL | Engineer/DevOps | Medium | Up to 90% |
| CLEAN-11 | Pre-Aggregate Data Prior to Collaboration | Engineer/DevOps | Low | 20-40% |
| CLEAN-12 | Consolidate Overlapping Collaborations | FinOps Team | Medium | 5-15% |
| CLEAN-13 | Batch PySpark Analytical Workloads | Engineer/DevOps | Low | 50-80% |
| CLEAN-14 | Consolidate Small SQL Queries | Engineer/DevOps | Low | Up to 90% |
| CLEAN-15 | Batch ML Training and Inference Tasks | Engineer/DevOps | Low | 20-50% |
| CLEAN-16 | Configure CloudWatch Alarms on CRPUConsumed | FinOps Team | Low | 10-100% |
| CLEAN-17 | Optimize Member Payment Rules (Flex Billing) | Procurement/Leadership | High | Up to 100% |
| CLEAN-18 | Align S3 Bucket Regions with Collaboration Region | Engineer/DevOps | Low | 100% |
| CLEAN-19 | Deploy VPC Endpoints for S3 and Clean Rooms API | Engineer/DevOps | Low | $0.045/GB |
