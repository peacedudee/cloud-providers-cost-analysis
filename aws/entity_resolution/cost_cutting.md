# Cost-Cutting Playbook: AWS Entity Resolution
> **Companion File:** [entity_resolution.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/entity_resolution/entity_resolution.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Entity Resolution is billed purely on a pay-as-you-go model based on the volume of records processed. The service charges a flat rate of $0.25 per 1,000 records for Rule-Based and Machine Learning matching, and $0.10 per 1,000 records for Data Service Provider matching (excluding provider fees). There are no volume tiers or free tier allowances. Because the pricing scales linearly with data volume, the most impactful cost optimization strategies revolve around data hygiene, reducing processed record counts via local deduplication, and adopting Incremental Matching (delta processing). 

## Strategy Categories

### 1. Waste Elimination

#### ENTITY-01. Pre-Ingestion Local Deduplication
- **What:** Clean input datasets using SQL, Python, or AWS Glue to remove exact duplicates before sending data to Entity Resolution.
- **Why It Saves Money:** Entity Resolution charges $250 per million records regardless of whether rows are obvious duplicates. Removing them locally costs fractions of a cent in compute.
- **Implementation Steps:** 
  1. Analyze raw data sources for duplicate rows.
  2. Apply `DISTINCT` or `GROUP BY` logic on exact match keys (e.g., email, ID) in your data pipeline.
  3. Output the cleaned file to the S3 bucket designated for Entity Resolution.
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Pre-existing ETL/Data preparation pipeline.

#### ENTITY-02. Filter Irrelevant or Inactive Records
- **What:** Remove inactive, deceased, or out-of-scope customer records from the dataset before processing.
- **Why It Saves Money:** Processing records that offer no business value still incurs the $0.25/1,000 record fee. Filtering them out avoids this entirely.
- **Implementation Steps:** 
  1. Identify filtering criteria (e.g., `last_login > 2 years`).
  2. Add `WHERE` clauses in your data extraction queries.
  3. Send only active, relevant records to AWS ER.
- **Estimated Savings:** 10-25%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Clear business logic defining active vs. inactive records.

#### ENTITY-03. Drop Empty and Null Rows
- **What:** Prevent files with empty rows or rows lacking critical matching fields from being processed.
- **Why It Saves Money:** A blank row counts as a record and incurs the standard processing fee, yielding zero matching value.
- **Implementation Steps:** 
  1. Add validation checks to drop rows where essential identifiers (e.g., `firstName`, `lastName`, `email`) are entirely null.
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Data validation steps in the ingestion pipeline.

#### ENTITY-04. S3 Lifecycle Rules on Raw Input Files
- **What:** Transition or delete raw input files from S3 after they have been processed by Entity Resolution.
- **Why It Saves Money:** Storing large uncompressed CSV/JSON datasets in S3 Standard costs $0.023/GB. Moving them to Glacier or deleting them eliminates unnecessary storage fees.
- **Implementation Steps:** 
  1. Configure S3 bucket lifecycle policies on the input buckets.
  2. Expire files older than 14 days or transition them to Glacier Flexible Retrieval.
- **Estimated Savings:** 5-10% of ancillary S3 storage costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** S3 Bucket management access.

#### ENTITY-05. Terminate Unused Matching Workflows
- **What:** Delete matching workflows that are no longer used or were created strictly for one-off testing.
- **Why It Saves Money:** Prevents accidental triggers or scheduled runs on stale data, and allows for the cleanup of associated output S3 buckets.
- **Implementation Steps:** 
  1. Audit AWS ER for inactive workflows.
  2. Delete unused workflows and their corresponding S3 target buckets.
- **Estimated Savings:** Variable
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Visibility into workflow usage metrics.

### 2. Rightsizing

#### ENTITY-06. Pre-Match with Local Rule-Based Logic
- **What:** Perform cheap, deterministic rule-based matching in Athena, EMR, or locally before invoking AWS ER ML matching.
- **Why It Saves Money:** Bypassing AWS ER for simple joins (e.g., matching UserID or SSN) avoids the $0.25/1k ER fee. ER is reserved only for "fuzzy" or complex matches.
- **Implementation Steps:** 
  1. Run a deterministic join on high-confidence keys.
  2. Extract unresolved, remaining rows.
  3. Send only unresolved records to AWS ER ML-based matching.
- **Estimated Savings:** 30-60%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Advanced data engineering pipeline.

#### ENTITY-07. Scope Provider Matching Targets
- **What:** Send only the records that explicitly need third-party enrichment to Data Service Provider Matching.
- **Why It Saves Money:** Provider matching costs $0.10/1,000 records plus external provider fees. Avoid sending high-confidence internal records that don't need external validation.
- **Implementation Steps:** 
  1. Split datasets post-internal-resolution.
  2. Send internally resolved records straight to the database.
  3. Send only unresolved records to Provider Matching.
- **Estimated Savings:** 20-50% on Provider Matching spend
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Logic to segment resolved vs. unresolved entities.

#### ENTITY-08. Optimize Output Data Retention
- **What:** Limit the number of historical matching outputs retained in the S3 target buckets.
- **Why It Saves Money:** Entity Resolution writes comprehensive output files to S3. Retaining every historical run permanently inflates storage costs.
- **Implementation Steps:** 
  1. Apply S3 lifecycle rules to the ER output bucket.
  2. Retain only the latest 2-3 versions or transition older outputs to Glacier Deep Archive.
- **Estimated Savings:** Variable
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** S3 lifecycle management.

### 3. Commitment Discounts

#### ENTITY-09. Negotiate Private Offers via Data Exchange
- **What:** Procure third-party data services (e.g., LiveRamp, TransUnion) via AWS Data Exchange Private Offers.
- **Why It Saves Money:** While the AWS ER fee is a fixed $0.10/1k for provider matching, the actual provider subscription fee can be heavily discounted through custom 1-3 year contracts.
- **Implementation Steps:** 
  1. Identify expected annual volume.
  2. Contact the data provider for a Private Offer.
  3. Accept the offer via AWS Data Exchange to lock in lower rates.
- **Estimated Savings:** 10-30% on Provider Fees
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** High volume of Provider Matching data.

#### ENTITY-10. Enterprise Discount Program (EDP) Coverage
- **What:** Ensure Entity Resolution spend is covered under the company's AWS EDP.
- **Why It Saves Money:** EDPs provide a blanket 5-15% discount across most AWS services in exchange for a committed spend.
- **Implementation Steps:** 
  1. Review the EDP contract with your AWS TAM.
  2. Verify that Entity Resolution is an eligible service for the discount.
- **Estimated Savings:** 5-15%
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** Active EDP agreement with AWS.

### 4. Architecture Changes

#### ENTITY-11. Implement Incremental Matching (Delta Processing)
- **What:** Use the Incremental Machine Learning-based matching feature (released May 2026) to process only new or updated records instead of the full historical dataset.
- **Why It Saves Money:** Total records processed drops drastically. E.g., Reprocessing a 5M record table costs $1,250. Processing only 100k delta additions costs $25, yielding 98% savings.
- **Implementation Steps:** 
  1. Enable Incremental matching in the ER workflow settings.
  2. Adjust data extraction to feed only delta updates into the input S3 bucket.
- **Estimated Savings:** 90-99%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** EKS or S3 integration supporting incremental workflows.

#### ENTITY-12. Centralize Entity Resolution Across Tenants
- **What:** Consolidate multi-tenant or multi-account matching into a single central data pipeline.
- **Why It Saves Money:** Centralization deduplicates across the entire organization, preventing independent accounts from paying multiple times to resolve the exact same identities.
- **Implementation Steps:** 
  1. Aggregate datasets in a central data lake (using AWS Lake Formation).
  2. Run Entity Resolution centrally.
  3. Distribute resolved Entity IDs back to the respective tenants.
- **Estimated Savings:** 15-40%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Cross-account data sharing capabilities.

#### ENTITY-13. Build Custom SageMaker ML for Massive Scale
- **What:** Replace AWS ER with custom ML models on SageMaker when processing multi-billion row datasets.
- **Why It Saves Money:** At $250/million records, processing 10 billion rows costs $2.5M. Running a custom SageMaker Batch Transform job costs only the underlying EC2 compute, saving millions.
- **Implementation Steps:** 
  1. Analyze if ER spend exceeds budget thresholds (e.g., > $100k/mo).
  2. Train a custom Record Linkage model.
  3. Deploy via SageMaker.
- **Estimated Savings:** 50-80% (at massive scale only)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** ML Engineering resources and massive data volume.

#### ENTITY-14. Use AWS Clean Rooms for Partner Matching
- **What:** Utilize AWS Clean Rooms to match data with external partners without duplicating or transferring data.
- **Why It Saves Money:** Avoids the pipeline costs, S3 storage overhead, and data transfer fees associated with moving large datasets purely for third-party matching.
- **Implementation Steps:** 
  1. Set up an AWS Clean Room.
  2. Connect with the partner account.
  3. Execute matching queries in-place.
- **Estimated Savings:** 10-20%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Partner must be willing to use AWS Clean Rooms.

### 5. Scheduling & Auto-Scaling

#### ENTITY-15. Optimize Batch Frequency
- **What:** Run matching workflows weekly or monthly instead of daily if real-time resolution isn't strictly necessary.
- **Why It Saves Money:** Reducing frequency limits how often identical or minimally-changed datasets are re-processed, and saves on orchestration overhead (Step Functions, EventBridge).
- **Implementation Steps:** 
  1. Assess business SLAs for resolved identity freshness.
  2. Adjust EventBridge cron schedules for ER workflows accordingly.
- **Estimated Savings:** 5-15% 
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Alignment with business stakeholders on data freshness.

#### ENTITY-16. Batch Micro-Updates
- **What:** Accumulate real-time micro-updates into a larger daily batch rather than triggering an ER run for a handful of rows continuously.
- **Why It Saves Money:** Batching is highly efficient for S3 API calls (PUT/GET) and prevents fragmented processing and orchestration overhead.
- **Implementation Steps:** 
  1. Queue incoming updates in SQS or Kinesis.
  2. Flush data to S3 in consolidated batches.
  3. Trigger the Entity Resolution workflow on the batched file.
- **Estimated Savings:** 5-10% (Ancillary pipeline costs)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Streaming or queueing architecture.

### 6. Pricing Model Optimization

#### ENTITY-17. Compare Native ML vs Provider Matching 
- **What:** Evaluate if third-party Provider Matching ($0.10/1k + provider fee) offers a lower Total Cost of Ownership than Native ML ($0.25/1k).
- **Why It Saves Money:** If provider subscription fees are sunk costs or highly discounted via an enterprise agreement, routing records through Provider Matching might be cheaper per-record than AWS Native ML.
- **Implementation Steps:** 
  1. Calculate the effective per-record TCO of Native ML vs Provider.
  2. Adjust workflows if the Provider route is cheaper and meets accuracy requirements.
- **Estimated Savings:** 10-30%
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Active provider subscription via AWS Data Exchange.

### 7. Network & Data Transfer Optimization

#### ENTITY-18. Align S3 Bucket and ER Service Regions
- **What:** Ensure that input and output S3 buckets are located in the exact same AWS region as the Entity Resolution workflow (e.g., all in `us-east-1`).
- **Why It Saves Money:** Cross-region data transfer costs $0.02 per GB. Processing terabytes of records across regions adds significant hidden fees to data ingestion.
- **Implementation Steps:** 
  1. Audit bucket regions vs. ER workflow regions.
  2. Migrate buckets or recreate the ER workflow in the local region.
- **Estimated Savings:** 100% of related Data Transfer costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region deployment.

#### ENTITY-19. Use VPC Endpoints for S3
- **What:** Use an S3 Gateway Endpoint to route traffic from EKS/EC2 data preparation pipelines to S3 without passing through a NAT Gateway.
- **Why It Saves Money:** NAT Gateways charge $0.045 per GB of data processed. S3 Gateway Endpoints are entirely free, eliminating network overhead for uploading input datasets.
- **Implementation Steps:** 
  1. Create an S3 Gateway Endpoint in the VPC.
  2. Associate the endpoint with private subnet route tables.
- **Estimated Savings:** $0.045 per GB transferred
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC with Private Subnets.

---
## Cross-Service Synergies
Entity Resolution costs are deeply tied to the surrounding data pipeline. Optimizations must extend to:
- **AWS Glue / Athena:** Performing pre-processing, filtering, and deterministic matching locally before invoking Entity Resolution.
- **Amazon S3:** Utilizing Gateway Endpoints to avoid NAT charges and applying Lifecycle Policies to manage large input/output datasets.
- **AWS Data Exchange:** Managing third-party subscriptions effectively to ensure Provider Matching makes financial sense.

---
## Required Input Data for Real-World Analysis
To accurately find and size these savings, the following data is required:

### A. AWS CUR 2.0
- **Query Targets:** Filter by `line_item_product_code = 'AWSEntityResolution'`. Look at `line_item_usage_type` for Rule-Based vs ML-Based vs Provider records processed. 
- **Timeframes:** Monthly trends to identify full reprocessing spikes vs. incremental processing behavior.

### B. CloudWatch Metrics
- Inspect `RecordsProcessed` and `MatchRate` metrics to identify workflows with massive inputs but low match rates (indicating poor pre-filtering).

### C. AWS Config / Trusted Advisor
- Use AWS Config to track the existence of Entity Resolution workflows and map them to their respective input and output S3 buckets to check for lifecycle policies.

### D. Company Policies
- Data retention policies to dictate how quickly raw S3 input files can be purged. 

### E. IaC (Optional)
- Review Terraform/CloudFormation to ensure S3 Gateway endpoints are configured and that workflows are correctly pointing to regional S3 buckets.

---
## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "ENTITY-001",
  "category": "Waste Elimination",
  "strategy_name": "Pre-Ingestion Local Deduplication",
  "resource_id": "arn:aws:entityresolution:us-east-1:123456789012:matchingworkflow/CustomerMatch",
  "condition": "Duplicate records identified in raw input files prior to processing.",
  "recommended_action": "Implement local DISTINCT query in data pipeline before uploading to S3.",
  "estimated_savings_monthly": 450.00,
  "level_of_effort": "Low"
}
```

### Summary Report Table

| Finding ID | Strategy Name | Estimated Monthly Savings | Effort | Scope |
|------------|---------------|---------------------------|--------|-------|
| ENTITY-01 | Pre-Ingestion Local Deduplication | $XXX | Low | Engineer |
| ENTITY-11 | Implement Incremental Matching | $XXX | Medium | Engineer |
| ENTITY-18 | Align S3 Bucket Regions | $XXX | Low | DevOps |
| ENTITY-09 | Negotiate Private Offers | $XXX | Low | Procurement |
