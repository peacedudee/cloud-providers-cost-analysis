# Cost-Cutting Playbook: AWS IoT Analytics
> **Companion File:** [iot_analytics.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/iot_analytics/iot_analytics.md)
> **Last Updated:** July 2026

---

## Executive Summary
AWS IoT Analytics is a legacy, fully managed service for collecting, processing, and analyzing IoT telemetry data. With new customer access closed as of July 2024, optimization efforts must focus on maintaining legacy workloads efficiently or migrating off the service entirely. Costs are driven by four primary volumetric vectors: Channel Data Ingestion ($0.20/GB), Pipeline Data Processing ($0.20/GB), Datastore Storage ($0.03/GB-month), and SQL Query Execution ($5.00/TB scanned). The most significant cost-cutting strategy involves filtering data *before* it enters IoT Analytics and heavily leveraging datastore partitioning to reduce query scan sizes. Ultimately, an architectural migration to a modern serverless data lake (Kinesis Firehose, S3, and Athena) provides the largest structural savings (up to 90%).

## Strategy Categories
### 1. Waste Elimination
### 2. Rightsizing
### 3. Commitment Discounts
### 4. Architecture Changes
### 5. Scheduling & Auto-Scaling
### 6. Pricing Model Optimization
### 7. Network & Data Transfer Optimization

---

## Cross-Service Synergies
- **AWS IoT Core:** Filtering rules in IoT Core are significantly cheaper than allowing raw data to enter IoT Analytics for downstream dropping.
- **Amazon Kinesis Data Firehose & Amazon S3:** The recommended modern replacements for IoT Analytics Channels and Datastores, offering vastly superior pricing per GB.
- **Amazon Athena:** Powers IoT Analytics querying under the hood. Migrating datastores to S3 and querying via native Athena provides better compression support and lower scanning costs.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- **`lineItem/ProductCode`**: `AWSIoTAnalytics`
- **`lineItem/UsageType`**: Look for `DataIngested`, `DataProcessed`, `DataStored`, and `DataScanned`.
- **`lineItem/UnblendedCost`**: To pinpoint exactly which of the 4 pipeline phases costs the most.

### B. CloudWatch Metrics
- **IoT Analytics Metrics:** `IncomingMessages`, `PipelineActivityExecution`, `DatastoreSize`, `DatasetQueryScannedBytes`.
- **IoT Core Metrics:** `RuleExecutions`, `PublishIn`.

### C. AWS Config / Trusted Advisor
- Inventory of existing Datastores, Pipelines, Channels, and Datasets.
- Analysis of orphaned datastores not linked to any active datasets.

### D. Company Policies
- Data retention requirements (how long raw telemetry must be kept).
- SLAs for reporting and dashboard refresh rates.

### E. IaC (Optional)
- Terraform (`aws_iotanalytics_datastore`, `aws_iotanalytics_pipeline`) or CloudFormation templates to identify misconfigured retention periods and partitioning logic.

---

## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "IOTA-001",
  "service": "AWS IoT Analytics",
  "strategy_category": "Rightsizing",
  "resource_id": "arn:aws:iotanalytics:us-east-1:123456789012:datastore/sensor_data",
  "issue": "Datastore missing time-based partitioning",
  "recommendation": "Configure datastore partitioning by timestamp to reduce dataset query scan volumes.",
  "estimated_monthly_savings": 250.00,
  "effort_level": "Medium"
}
```

### Summary Report Table
| Finding ID | Strategy Name | Estimated Savings | Risk Level | Scope |
|------------|---------------|-------------------|------------|-------|
| IOTA-001 | Pre-filter Data using AWS IoT Core Rules Engine | 70-90% | Low | Engineer/DevOps |
| IOTA-002 | Partition Datastores by Time | 70-95% | Low | Engineer/DevOps |
| IOTA-003 | Migrate to Kinesis Data Firehose | 85% | High | Engineer/DevOps |

---

## Strategies

### 1. Waste Elimination

#### 1. Filter Raw Telemetry at the Edge
- **What:** Modify device firmware or AWS IoT Greengrass logic to drop redundant data (like unchanged temperature readings) or heartbeat messages before network transmission.
- **Why It Saves Money:** Stops data from ever reaching AWS. Prevents $0.20/GB ingestion and $0.20/GB processing fees from ever occurring in IoT Analytics.
- **Implementation Steps:** 
  1. Audit device payload generation frequency.
  2. Implement local delta-threshold logic (only send if value changes by > 1%).
  3. Deploy firmware/Greengrass updates.
- **Estimated Savings:** 20-50% (of total IoT Analytics bill)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Edge compute capabilities and over-the-air update mechanisms.

#### 2. Pre-filter Data using AWS IoT Core Rules Engine
- **What:** Apply SQL `WHERE` clauses in IoT Core Rules to drop unchanging or junk sensor values before routing the traffic to IoT Analytics channels.
- **Why It Saves Money:** Bypasses $0.20/GB ingestion and $0.20/GB processing fees entirely. The IoT Rules cost ($0.15 per million rules executed) is significantly cheaper than volumetric GB billing for frequent, small payloads.
- **Implementation Steps:** 
  1. Identify the IoT Core rule routing to the IoT Analytics channel.
  2. Modify the SQL statement to filter out unneeded attributes or specific state payloads.
  3. Validate downstream dashboards still function.
- **Estimated Savings:** 70-90%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Access to AWS IoT Core Rules.

#### 3. Terminate Unused IoT Analytics Datastores
- **What:** Identify and permanently delete datastores that have not been queried or updated recently.
- **Why It Saves Money:** Eliminates the $0.03/GB-month storage fee for orphaned processed data that provides no business value.
- **Implementation Steps:** 
  1. Review CloudWatch metrics for datastores with zero dataset queries over the last 90 days.
  2. Snapshot the datastore to S3 Glacier if archival is required.
  3. Delete the datastore resource.
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Datastore access and CloudWatch monitoring.

#### 4. Drop Junk Attributes in Pipeline Activities
- **What:** Use the `removeAttributes` activity in IOTA pipelines to strip unnecessary JSON keys (e.g., verbose debug strings, redundant nested objects) early in the pipeline.
- **Why It Saves Money:** Reduces the Datastore storage size ($0.03/GB-mo) and minimizes the GB scanned ($5.00/TB) during subsequent Dataset SQL executions.
- **Implementation Steps:** 
  1. Edit the IoT Analytics pipeline configuration.
  2. Add a `removeAttributes` activity right after ingestion.
  3. Specify the keys to drop.
- **Estimated Savings:** 10-15%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of required payload schema.

#### 5. Delete Obsolete Datasets and Triggers
- **What:** Remove automatically executing SQL datasets that feed unused dashboards, reports, or deprecated ML models.
- **Why It Saves Money:** Prevents the recurring $5.00/TB data scan charges for queries that are executing but whose outputs are ignored.
- **Implementation Steps:** 
  1. Audit QuickSight usage or custom dashboard access logs.
  2. Identify datasets with no active downstream consumers.
  3. Delete or disable the dataset schedule triggers.
- **Estimated Savings:** 20-40%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Visualization tool access logs.

### 2. Rightsizing

#### 6. Implement Aggressive Data Retention Policies on Datastores
- **What:** Configure datastores to automatically expire data after a set period (e.g., 30 days) rather than keeping it indefinitely.
- **Why It Saves Money:** Caps the compounding $0.03/GB-month storage costs. Old data is purged automatically without manual intervention.
- **Implementation Steps:** 
  1. Navigate to Datastore settings in the IoT Analytics console.
  2. Edit the data retention period from "Indefinitely" to a specific number of days.
- **Estimated Savings:** 40-60%
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Business alignment on data retention compliance.

#### 7. Optimize Channel Data Retention
- **What:** Reduce the retention period for raw channel data. Channels store raw incoming data for pipeline reprocessing, but keeping it for a long time is rarely necessary.
- **Why It Saves Money:** Reduces unnecessary S3-backed storage charges for raw data that will never actually be reprocessed.
- **Implementation Steps:** 
  1. Go to the Channel configuration.
  2. Set retention to 7 or 14 days (just enough for emergency reprocessing of failed pipeline logic).
- **Estimated Savings:** 10-20%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 8. Partition Datastores by Time to Minimize Query Scans
- **What:** Configure datastore partitioning by timestamp attributes (year/month/day).
- **Why It Saves Money:** Without partitions, Dataset SQL queries scan the entire historical table. With partitions, queries only scan relevant time windows, drastically reducing the $5.00/TB scan fee.
- **Implementation Steps:** 
  1. Create a new datastore (partitions cannot be added to existing datastores).
  2. Define the partition keys (e.g., `timestamp`).
  3. Repoint pipelines to the new datastore and update datasets to include partition filters in the `WHERE` clause.
- **Estimated Savings:** 70-95% (especially for multi-year historical datastores)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Datastore migration effort.

#### 9. Optimize SQL Queries to Select Specific Columns
- **What:** Rewrite `SELECT *` queries in IoT Analytics datasets to only select the exact required columns for reporting.
- **Why It Saves Money:** IoT Analytics Dataset queries are powered by an Athena-like columnar engine. Selecting fewer columns significantly reduces the TB scanned metric ($5.00/TB).
- **Implementation Steps:** 
  1. Review all dataset SQL queries.
  2. Replace `SELECT *` with `SELECT sensor_id, temperature, timestamp`.
- **Estimated Savings:** 30-50%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Basic SQL proficiency.

#### 10. Reduce Frequency of Recurring Dataset Execution
- **What:** Change schedule-based datasets to run less often (e.g., generate reports daily instead of hourly).
- **Why It Saves Money:** Proportionally reduces the $5.00/TB query scanning costs by decreasing the absolute number of runs per month.
- **Implementation Steps:** 
  1. Edit the Dataset trigger schedule.
  2. Change cron expressions from hourly to daily or weekly based on actual business needs.
- **Estimated Savings:** 50-80% (on query costs)
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Stakeholder approval for increased data latency.

### 3. Commitment Discounts

#### 11. Leverage AWS Enterprise Discount Program (EDP)
- **What:** Negotiate a custom Enterprise Discount Program (EDP) contract with AWS.
- **Why It Saves Money:** There are no Reserved Instances or Savings Plans for IoT Analytics. An EDP applies a blanket discount (usually 5-15%) across all services, including legacy IoT Analytics resources.
- **Implementation Steps:** 
  1. Work with AWS Account Manager to assess total annual organizational spend.
  2. Negotiate EDP terms and commit to an annual spend threshold.
- **Estimated Savings:** 5-15%
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** Large AWS annual spend commitment.

### 4. Architecture Changes

#### 12. Migrate Ingestion to Amazon Kinesis Data Firehose
- **What:** Route AWS IoT Core Rules directly to Kinesis Data Firehose instead of IoT Analytics Channels. (This is the official AWS recommendation for new workloads).
- **Why It Saves Money:** Firehose ingestion ($0.029/GB) is roughly 85% cheaper than IoT Analytics Channel ingestion ($0.20/GB).
- **Implementation Steps:** 
  1. Create a Kinesis Data Firehose delivery stream.
  2. Update IoT Core Rules to route traffic to the Firehose stream.
- **Estimated Savings:** ~85% (on ingestion)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Architecture redesign.

#### 13. Migrate Storage to Amazon S3 using Parquet
- **What:** Output the Kinesis Firehose data directly into Amazon S3, converting it to the Parquet columnar format.
- **Why It Saves Money:** S3 Standard ($0.023/GB-mo) is cheaper than IOTA Datastores ($0.03/GB-mo), and Parquet compression dramatically reduces the total storage footprint, multiplying the savings.
- **Implementation Steps:** 
  1. Configure Firehose to convert JSON to Parquet using AWS Glue.
  2. Set the destination to an S3 bucket.
- **Estimated Savings:** 40-70% (on storage)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS Glue Data Catalog schema setup.

#### 14. Migrate Querying to Native Amazon Athena
- **What:** Run reporting queries directly against the new S3 Parquet data using native Amazon Athena, bypassing IoT Analytics Datasets.
- **Why It Saves Money:** While Athena also costs $5.00/TB scanned, querying heavily compressed Parquet data natively on S3 scans a fraction of the data compared to querying unoptimized JSON payloads in legacy IOTA datastores.
- **Implementation Steps:** 
  1. Point Amazon Athena to the Glue Data Catalog connected to the S3 bucket.
  2. Repoint QuickSight or BI tools to Athena.
- **Estimated Savings:** 60-90% (on querying)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Athena and S3 configuration.

#### 15. Move Data Enrichment to AWS Lambda via IoT Core Rules
- **What:** Replace expensive IoT Analytics Pipeline Lambda enrichment activities with native Lambda functions triggered directly by IoT Core rules.
- **Why It Saves Money:** Avoids the flat volumetric IoT Analytics pipeline processing fee ($0.20/GB), shifting instead to AWS Lambda's highly optimized per-millisecond execution billing model.
- **Implementation Steps:** 
  1. Extract enrichment logic from IOTA pipeline Lambda.
  2. Deploy as a standalone Lambda function.
  3. Route IoT Core rules to Lambda before sending to Kinesis/S3.
- **Estimated Savings:** 50-70% (on processing)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Lambda function refactoring.

### 5. Scheduling & Auto-Scaling

#### 16. Suspend Dataset Execution for Inactive Dashboards
- **What:** Implement automation to detect when a QuickSight or custom dashboard has not been viewed recently, and pause the underlying IOTA dataset schedule.
- **Why It Saves Money:** Automatically stops paying the $5.00/TB scan fee for recurring dataset reports that nobody is actively looking at.
- **Implementation Steps:** 
  1. Write a Lambda function to parse CloudTrail logs for QuickSight dashboard `DescribeDashboard` API calls.
  2. If unaccessed for 14 days, use `UpdateDataset` API to remove the IOTA trigger schedule.
- **Estimated Savings:** 20-100% (for abandoned reporting flows)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudTrail API logging enabled.

### 6. Pricing Model Optimization

#### 17. Leverage Legacy Free Tier for Staging Environments
- **What:** Ensure staging, testing, and development environments are segregated into accounts that can utilize the 12-month free tier (100GB ingest, 100GB process, 10GB store, 10GB scan).
- **Why It Saves Money:** Eliminates costs entirely for low-volume testing workloads over the first year of account life.
- **Implementation Steps:** 
  1. Audit AWS Organizations for newer linked accounts.
  2. Deploy dev/staging infrastructure into those fresh accounts.
- **Estimated Savings:** 100% (for dev/staging environments under the limits)
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Legacy account creation date verification.

### 7. Network & Data Transfer Optimization

#### 18. Consolidate IoT Core and IoT Analytics into a Single Region
- **What:** Ensure devices connect to IoT Core endpoints in the exact same AWS Region where the IoT Analytics channels are deployed.
- **Why It Saves Money:** Avoids the $0.01-$0.02/GB cross-region data transfer fees for high-volume telemetry streams crossing AWS regions.
- **Implementation Steps:** 
  1. Audit region configurations in IaC.
  2. Migrate IoT Analytics resources to match the IoT Core endpoint region.
- **Estimated Savings:** $0.01 - $0.02 per GB transferred
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region architecture review.

#### 19. Prevent Cross-Region Dashboard Egress from Datasets
- **What:** Deploy BI tools (like Amazon QuickSight) in the same AWS region as the IoT Analytics datasets they consume.
- **Why It Saves Money:** Prevents $0.09/GB internet egress or cross-region transfer fees when BI tools pull large dataset result sets across regional boundaries.
- **Implementation Steps:** 
  1. Check QuickSight SPICE dataset ingest origins.
  2. Redeploy BI infrastructure locally if cross-region egress is detected.
- **Estimated Savings:** $0.09 per GB egressed
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** QuickSight administration access.
