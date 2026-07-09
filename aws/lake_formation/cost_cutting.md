# Cost-Cutting Playbook: AWS Lake Formation
> **Companion File:** [lake_formation.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/lake_formation/lake_formation.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Lake Formation is a zero-cost governance and security layer that orchestrates access to data lakes. Because the core Lake Formation service itself incurs no charges ($0.00), cost optimization focuses entirely on the underlying data lake infrastructure: Amazon S3 (storage), AWS Glue Data Catalog (metadata), and the analytical compute engines (Amazon Athena, EMR, Redshift, Glue ETL). The deprecation of Governed Tables in early 2025 shifted the architectural standard toward open-source transactional table formats (like Apache Iceberg). This playbook outlines strategies to optimize file sizes, data formats, partition layouts, and underlying infrastructure to drastically reduce the true cost of operating an AWS data lake.

## Strategy Categories
### 1. Waste Elimination
#### 1. Delete Orphaned Data Lake Files
- **What:** Identify and delete S3 objects located within data lake buckets that are not registered in the AWS Glue Data Catalog or utilized by Lake Formation.
- **Why It Saves Money:** Orphaned data consumes standard S3 storage ($0.023/GB-month) while providing no analytical value since it cannot be queried.
- **Implementation Steps:**
  1. Generate an S3 Storage Lens or S3 Inventory report for data lake buckets.
  2. Compare S3 inventory keys against registered table partitions in the Glue Data Catalog.
  3. Flag discrepancies for review.
  4. Implement an automated deletion or archival script for orphaned objects.
- **Estimated Savings:** 5-15% (on S3 storage)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** S3 Inventory enabled, Glue Data Catalog access.

#### 2. Remove Unused Data Catalog Tables and Partitions
- **What:** Clean up deprecated, abandoned, or empty tables and partitions from the AWS Glue Data Catalog.
- **Why It Saves Money:** Glue Data Catalog charges $1.00 per 100,000 objects (tables, partitions) per month beyond the 1 million free tier limit.
- **Implementation Steps:**
  1. Review query logs (Athena/Redshift) via CloudTrail to identify unqueried tables over the last 90 days.
  2. Use Glue APIs to drop unused tables and their associated partitions.
  3. Ensure underlying S3 data is simultaneously archived or deleted if no longer needed.
- **Estimated Savings:** 1-5% (on Glue Metadata fees)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudTrail query logging, Glue Catalog access.

#### 3. Clean Up Temporary and Failed ETL Data
- **What:** Automatically delete temporary files or output from failed AWS Glue / EMR jobs that clutter the data lake S3 buckets.
- **Why It Saves Money:** Leftover temporary data accrues standard S3 storage fees ($0.023/GB-mo).
- **Implementation Steps:**
  1. Identify standard temporary prefixes (e.g., `_temporary`, `_SUCCESS`, or `<job-id>-temp`).
  2. Create an S3 Lifecycle Rule to automatically abort incomplete multipart uploads after 7 days.
  3. Create an S3 Lifecycle Rule to permanently delete files in temporary prefixes after 7 days.
- **Estimated Savings:** 2-10% (on S3 storage)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Standardized ETL output paths.

### 2. Rightsizing
#### 4. Compact Small Files in S3
- **What:** Merge thousands of tiny KB-sized files into optimal 128 MB - 512 MB chunk sizes.
- **Why It Saves Money:** Reduces S3 PUT/GET request fees ($0.005/1,000 PUTs, $0.0004/1,000 GETs) and drastically reduces Athena query costs ($5.00/TB scanned) by minimizing metadata overhead and scanning time.
- **Implementation Steps:**
  1. Identify tables suffering from the "small file problem" using S3 Storage Lens.
  2. Configure AWS Glue or Athena automated compaction jobs to run on the tables.
  3. Verify that average object size in the partition increases to the target range.
- **Estimated Savings:** 30-90% (on Athena query costs and S3 requests)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Glue or Athena setup for data transformations.

#### 5. Convert Data to Columnar Formats (Parquet/ORC)
- **What:** Convert raw data lake files (JSON, CSV, TSV) into highly compressed columnar formats like Apache Parquet or ORC.
- **Why It Saves Money:** Columnar formats compress data by 70-90% (saving S3 storage costs) and allow query engines to read only specific columns (saving Athena TB-scanned costs).
- **Implementation Steps:**
  1. Identify raw format tables in Lake Formation.
  2. Create a Glue ETL job to transform incoming raw data to Snappy-compressed Parquet.
  3. Update Glue Data Catalog schema and Lake Formation permissions to point to the new Parquet tables.
- **Estimated Savings:** 50-80% (on query and storage costs)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Data engineering pipelines to handle transformation.

#### 6. Rightsize Glue ETL Job DPUs
- **What:** Adjust the Data Processing Units (DPUs) allocated to Glue ETL jobs used for data lake ingestion and transformation to match the actual workload.
- **Why It Saves Money:** Glue charges $0.44 per DPU-hour. Over-provisioned jobs pay for idle compute capacity.
- **Implementation Steps:**
  1. Enable AWS Glue Job metrics to monitor CPU and memory utilization.
  2. Analyze the `Maximum capacity` vs `Allocated capacity`.
  3. Reduce the number of DPUs for jobs with low utilization.
- **Estimated Savings:** 20-40% (on ETL costs)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Glue Job metrics enabled.

### 3. Commitment Discounts
#### 7. Commit to Athena Provisioned Capacity
- **What:** Purchase Provisioned Capacity (DPUs) for Amazon Athena if your Lake Formation data is queried constantly at high volumes.
- **Why It Saves Money:** If TB-scanned costs exceed the break-even point for dedicated compute, paying for Provisioned Capacity can provide a lower effective rate for high-concurrency workloads.
- **Implementation Steps:**
  1. Analyze Athena usage in AWS CUR to calculate monthly TB-scanned spend.
  2. Compare against Athena Provisioned Capacity pricing.
  3. Purchase capacity and route Lake Formation queries to the provisioned workgroup.
- **Estimated Savings:** 10-30% (on high-volume query costs)
- **Risk Level:** High (Commitment)
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Highly predictable, high-volume query workloads.

#### 8. Apply Compute Savings Plans to Data Lake Compute
- **What:** Cover Amazon EMR (EC2 instances) and AWS Glue (if applicable via general compute SPs or specific tracking) with AWS Compute Savings Plans.
- **Why It Saves Money:** Yields up to 66% discount on the underlying EC2 instances processing your data lake compared to On-Demand rates.
- **Implementation Steps:**
  1. Analyze stable baseline usage of EMR/EC2 compute used for the data lake.
  2. Purchase a 1-year or 3-year Compute Savings Plan via AWS Cost Explorer.
- **Estimated Savings:** 20-50% (on EC2 compute costs)
- **Risk Level:** High (Commitment)
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Predictable data processing baselines.

### 4. Architecture Changes
#### 9. Adopt Apache Iceberg Open Table Formats
- **What:** Migrate legacy tables and replace deprecated Governed Tables with Apache Iceberg for ACID transactions, schema evolution, and time travel.
- **Why It Saves Money:** Iceberg’s manifest files allow query engines to perform aggressive file pruning, drastically reducing the amount of data scanned ($5.00/TB) in S3.
- **Implementation Steps:**
  1. Identify critical transactional or frequently-updated tables.
  2. Use Athena or Glue to migrate or recreate the tables using the Iceberg format.
  3. Register the tables in the Glue Data Catalog under Lake Formation governance.
- **Estimated Savings:** 20-60% (on query scan costs)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of Apache Iceberg architecture.

#### 10. Implement Robust Data Partitioning
- **What:** Partition S3 data by highly queried dimensions (e.g., year/month/day or department) rather than keeping flat structures.
- **Why It Saves Money:** Partition pruning prevents query engines from scanning the entire dataset. A query restricted to a single day’s partition scans (and pays for) only that fraction of the data.
- **Implementation Steps:**
  1. Identify common query `WHERE` clauses from CloudWatch/CloudTrail logs.
  2. Restructure the S3 bucket paths to follow `/year=YYYY/month=MM/` formats.
  3. Update the Glue Data Catalog schema to recognize partition keys.
- **Estimated Savings:** 50-95% (on query scan costs)
- **Risk Level:** High (Requires data rewrite)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Data rewrite pipeline.

#### 11. Enforce Column-Level Access Control for Query Pruning
- **What:** Use Lake Formation’s column-level permissions to restrict which columns specific roles can access or query.
- **Why It Saves Money:** When users query tables using `SELECT *`, Lake Formation automatically drops unauthorized columns at the source before the query engine processes them, reducing the TB-scanned metric in Athena/Redshift.
- **Implementation Steps:**
  1. Define strict IAM/Lake Formation personas (e.g., Data Analyst, Data Scientist).
  2. Apply column-level inclusions/exclusions for PII or irrelevant wide columns per persona.
  3. Validate that restricted queries scan less data.
- **Estimated Savings:** 10-30% (on query scan costs)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Lake Formation personas mapped out.

#### 12. Implement S3 Lifecycle Policies for Data Lake Archives
- **What:** Automatically transition older data lake partitions to cheaper S3 storage tiers like Glacier Flexible Retrieval or Glacier Deep Archive.
- **Why It Saves Money:** Deep Archive costs $0.00099/GB-month, approximately 95% cheaper than S3 Standard ($0.023/GB-month).
- **Implementation Steps:**
  1. Identify historical partitions (e.g., data older than 1 year) that are rarely queried.
  2. Create an S3 Lifecycle Rule targeting the specific partition prefixes.
  3. Transition to Glacier Deep Archive.
- **Estimated Savings:** 70-95% (on storage costs for archived data)
- **Risk Level:** Medium (Data retrieval takes hours)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Stakeholder agreement on data retention and retrieval times.

#### 13. Leverage Pushdown Predicates for Data Access
- **What:** Ensure that ETL jobs (AWS Glue, EMR) push down filter predicates directly to the S3 data source rather than pulling all data into memory and filtering it later.
- **Why It Saves Money:** Reduces S3 GET requests, lowers network bandwidth usage, and reduces the memory/DPU requirements of the ETL jobs.
- **Implementation Steps:**
  1. Review Glue/PySpark scripts.
  2. Implement `push_down_predicate` in `create_dynamic_frame.from_catalog` methods.
- **Estimated Savings:** 15-40% (on ETL memory and S3 requests)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** PySpark/Glue scripting knowledge.

### 5. Scheduling & Auto-Scaling
#### 14. Enable Auto-Scaling on Data Lake ETL Pipelines
- **What:** Turn on Auto Scaling for AWS Glue 3.0/4.0 jobs managing Lake Formation data.
- **Why It Saves Money:** Auto-scaling dynamically adds and removes DPUs as the workload changes, meaning you only pay for compute exactly when it is actively processing data.
- **Implementation Steps:**
  1. Edit Glue job properties in the AWS Console.
  2. Enable the "Auto Scaling" checkbox and set a maximum DPU limit.
  3. Monitor job execution metrics to verify scale-down behavior.
- **Estimated Savings:** 15-40% (on Glue ETL costs)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Glue version 3.0 or later.

#### 15. Schedule Maintenance (Compaction/Vacuum) During Off-Peak Hours
- **What:** Run heavy data lake maintenance (file compaction, vacuuming deleted Iceberg snapshots) during low-activity periods.
- **Why It Saves Money:** Prevents resource contention with business-critical queries and allows the use of cheaper Spot instances (via EMR) to perform the background maintenance.
- **Implementation Steps:**
  1. Identify the lowest query volume window (e.g., 2 AM - 4 AM weekend).
  2. Schedule AWS Glue Workflows, EventBridge rules, or Airflow DAGs to trigger compaction jobs.
- **Estimated Savings:** Indirect (Improves overall performance and enables Spot usage)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Orchestration tool (EventBridge, MWAA, Glue Workflows).

### 6. Pricing Model Optimization
#### 16. Utilize Spot Instances for EMR / Glue Data Processing
- **What:** Use Amazon EC2 Spot Instances for Amazon EMR clusters or Spot DPUs for AWS Glue jobs that process data lake data.
- **Why It Saves Money:** Spot instances provide up to 90% discount compared to On-Demand EC2 pricing for fault-tolerant, interruptible workloads like data ingestion or ETL.
- **Implementation Steps:**
  1. Identify stateless, retryable data ingestion pipelines.
  2. Configure EMR clusters with Spot Instance fleets for task nodes.
  3. For AWS Glue, enable "Flex" execution class (which uses spare compute capacity).
- **Estimated Savings:** 50-90% (on processing compute)
- **Risk Level:** Medium (Job interruptions)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Fault-tolerant ETL architecture.

#### 17. Transition S3 Data to Intelligent-Tiering
- **What:** Move data lake S3 buckets to the S3 Intelligent-Tiering storage class to automatically optimize storage costs based on access patterns.
- **Why It Saves Money:** S3 Intelligent-Tiering automatically moves unaccessed data to cheaper access tiers (Infrequent, Archive, Deep Archive) without operational overhead or retrieval fees.
- **Implementation Steps:**
  1. Evaluate data lake access patterns; ideal for datasets with unpredictable access.
  2. Create an S3 Lifecycle Rule to transition current data to Intelligent-Tiering.
  3. Ensure default bucket settings use Intelligent-Tiering for new uploads.
- **Estimated Savings:** 10-30% (on S3 storage)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** S3 lifecycle management access.

### 7. Network & Data Transfer Optimization
#### 18. Co-locate Compute and Data Lake in the Same AWS Region
- **What:** Ensure that Lake Formation, Glue Catalog, S3 Buckets, and query engines (Athena/EMR) are all deployed in the exact same AWS Region.
- **Why It Saves Money:** Cross-region data transfer from S3 costs $0.01 - $0.02 per GB. If queries pull TBs of data across regions, transfer costs will rapidly exceed compute costs.
- **Implementation Steps:**
  1. Audit region deployments of EMR clusters, Athena workgroups, and S3 buckets.
  2. Relocate or duplicate necessary resources so compute runs locally against the data.
- **Estimated Savings:** 100% (eliminates cross-region transfer fees)
- **Risk Level:** Medium (Migration effort)
- **Implementation Scope:** Engineer/DevOps | Architect
- **Prerequisites:** Multi-region architecture review.

#### 19. Route Data Lake Traffic via VPC Endpoints for S3/Glue
- **What:** Deploy Gateway VPC Endpoints for Amazon S3 and Interface VPC Endpoints for AWS Glue within private subnets hosting EMR or ETL compute.
- **Why It Saves Money:** Prevents heavy data lake traffic from traversing NAT Gateways, which charge $0.045 per GB processed. Gateway Endpoints for S3 are entirely free.
- **Implementation Steps:**
  1. Identify VPCs running private data lake compute (e.g., EMR, Redshift, private Glue endpoints).
  2. Create a Gateway VPC Endpoint for S3 and update route tables.
  3. Verify NAT Gateway metrics drop.
- **Estimated Savings:** 100% (eliminates NAT data processing fees for S3)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps (Networking)
- **Prerequisites:** VPC routing configuration access.

---
## Cross-Service Synergies
- **Lake Formation + S3 Storage Lens:** Use Storage Lens to visualize the exact scale of the "small file problem" across the data lake, validating the need for compaction (Strategy #4).
- **Lake Formation + Athena:** Fine-grained access control dynamically reduces data scanned by Athena, generating direct financial savings on Athena queries alongside security benefits.
- **Lake Formation + Cost Explorer (Tags):** Enforce strict tagging on S3 buckets and Glue jobs to accurately separate "Data Lake" costs from general enterprise storage and compute.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query `lineItem/ProductCode` for `AmazonS3`, `AWSGlue`, `AmazonAthena`, and `ElasticMapReduce`.
- Look for `UsageType` related to `DataTransfer`, `Requests-Tier1` (PUTs), and `TB-Scanned` to model the impact of partitioning and compaction.
### B. CloudWatch Metrics
- AWS Glue job metrics (`glue.driver.aggregate.bytesRead`, `glue.driver.aggregate.elapsedTime`) to assess ETL efficiency.
### C. AWS Config / Trusted Advisor
- Identify S3 buckets missing lifecycle policies or public access blocks.
### D. Company Policies
- Data retention rules (when to move data to Glacier Deep Archive vs. deleting it).
- Security persona definitions for row/column Lake Formation filtering.
### E. IaC (Optional)
- Terraform/CloudFormation templates to review how S3 buckets, Glue Databases, and ETL jobs are provisioned.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "LAKEFORM-001",
  "strategy_name": "Implement S3 Lifecycle Policies for Data Lake Archives",
  "resource_id": "arn:aws:s3:::corporate-data-lake-raw",
  "current_cost_monthly": 1500.00,
  "projected_cost_monthly": 150.00,
  "savings_monthly": 1350.00,
  "savings_percentage": 90,
  "risk_level": "Medium",
  "implementation_effort": "Low",
  "status": "Proposed"
}
```

### Summary Report Table
| Finding ID | Strategy | Target Resource | Est. Monthly Savings | Risk Level | Effort |
|------------|----------|-----------------|----------------------|------------|--------|
| LAKEFORM-001 | S3 Lifecycle Archive | `s3://data-lake-raw` | $1,350.00 | Medium | Low |
| LAKEFORM-002 | Compact Small Files | `glue_db.clicks` | $800.00 | Low | Medium |
| LAKEFORM-003 | Enable Glue Auto-Scaling | `glue-job-daily` | $250.00 | Low | Low |
| LAKEFORM-004 | VPC Endpoint for S3 | `vpc-1234abcd` | $600.00 | Low | Low |
