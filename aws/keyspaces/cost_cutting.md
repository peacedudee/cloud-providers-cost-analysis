# Cost-Cutting Playbook: Amazon Keyspaces
> **Companion File:** [keyspaces.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/keyspaces/keyspaces.md)
> **Last Updated:** July 2026

---

## Executive Summary
Amazon Keyspaces is a fully managed, serverless Apache Cassandra-compatible database service. While it eliminates the overhead of managing Cassandra nodes, it introduces unique billing dimensions based on Read/Write Request Units, Storage, Point-in-Time Recovery (PITR), and TTL Deletes. Cost optimization in Keyspaces requires a deep understanding of Cassandra query patterns, data modeling, and capacity management. This playbook outlines actionable strategies to minimize Keyspaces expenditures by transitioning to optimal capacity modes, refining read consistencies, right-sizing row payloads, and leveraging caching mechanisms.

## Strategy Categories

### 1. Waste Elimination

#### KEYSPACES-001. Disable Point-in-Time Recovery (PITR) on Non-Production Tables
- **What:** Turn off continuous backups (Point-in-Time Recovery) for tables running in development, testing, or staging environments.
- **Why It Saves Money:** PITR backup storage is billed at $0.228 per GB-month. Disabling this for non-critical data eliminates this surcharge entirely, reducing storage costs by nearly half (standard storage is $0.25/GB-month).
- **Implementation Steps:**
  1. Audit all Keyspaces tables across AWS accounts to identify non-production environments.
  2. Use AWS CLI or IaC (Terraform/CloudFormation) to update the table settings.
  3. Set `pointInTimeRecovery` status to `DISABLED`.
- **Estimated Savings:** 40-47% of total storage costs per affected table
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Environment tagging or naming conventions to safely identify non-prod tables.

#### KEYSPACES-002. Implement TTL (Time-to-Live) for Ephemeral Data
- **What:** Automatically expire and delete old, stale, or operational data (like session logs or time-series events) using Cassandra's Time to Live (TTL) feature.
- **Why It Saves Money:** Standard table storage costs $0.25/GB-month. TTL deletes cost only $0.275 per million 1 KB operations (a 75% reduction introduced in Nov 2024). Expiring data prevents infinite storage growth, exchanging expensive recurring storage fees for a very cheap, one-time delete operation.
- **Implementation Steps:**
  1. Identify tables storing time-series, logging, or session data.
  2. Determine the acceptable retention period for the data.
  3. Modify the `INSERT` or `UPDATE` CQL queries to include the `USING TTL <seconds>` clause.
  4. Alternatively, set a default TTL on the table schema using `ALTER TABLE ... WITH default_time_to_live = <seconds>;`.
- **Estimated Savings:** 20-80% on long-term storage costs depending on data churn
- **Risk Level:** Medium (Data is permanently deleted after TTL)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Business alignment on data retention policies.

#### KEYSPACES-003. Deprovision Orphaned or Unused Tables
- **What:** Identify and delete Keyspaces tables that are no longer actively queried or written to by applications.
- **Why It Saves Money:** Unused tables accumulate standard storage charges ($0.25/GB-month), PITR charges ($0.228/GB-month), and if using Provisioned Capacity with a minimum WCU/RCU, continuous hourly compute charges.
- **Implementation Steps:**
  1. Use CloudWatch to monitor `ConsumedReadCapacityUnits` and `ConsumedWriteCapacityUnits` over a 30-day period.
  2. Identify tables with zero or near-zero consumption.
  3. Verify with application owners that the tables are obsolete.
  4. Export necessary data to S3 (if archival is required) and execute a `DROP TABLE` command.
- **Estimated Savings:** 100% of the orphaned table's cost
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** 30+ days of CloudWatch metric history.

### 2. Rightsizing

#### KEYSPACES-004. Offload Large Payloads to Amazon S3
- **What:** Stop storing large JSON blobs, files, or massive text fields directly in Keyspaces columns. Instead, upload the payloads to S3 and store the S3 URI/Object Key in Keyspaces.
- **Why It Saves Money:** Keyspaces writes are metered per 1 KB block ($1.45/million WRUs). Writing a 50 KB row consumes 50 WRUs. Keyspaces storage is $0.25/GB-month. By moving that 50 KB payload to S3, you consume only 1 WRU in Keyspaces (for the metadata). S3 storage costs only ~$0.023/GB-month (10x cheaper than Keyspaces storage).
- **Implementation Steps:**
  1. Identify tables with large average row sizes (e.g., >2-4 KB).
  2. Refactor the application code to write the large payload to an S3 bucket first.
  3. Retrieve the S3 Object URL/Key.
  4. Write the structured metadata and the S3 Key to Keyspaces.
  5. Refactor the read path to fetch the S3 object asynchronously when the application needs the payload.
- **Estimated Savings:** 50-90% on Write Request Units (WRUs) and 80%+ on Storage
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application code refactoring, S3 bucket provisioning.

#### KEYSPACES-005. Optimize Partition Key Design to Prevent Hot Partitions
- **What:** Redesign Cassandra data models to ensure read and write traffic is evenly distributed across partition keys.
- **Why It Saves Money:** "Hot partitions" (where one partition key receives the majority of traffic) force you to over-provision WCUs/RCUs in Provisioned Mode just to handle the skewed load, resulting in massive idle capacity across the rest of the table. In On-Demand mode, it can lead to throttling if it exceeds partition-level throughput limits, forcing unnecessary retries.
- **Implementation Steps:**
  1. Monitor CloudWatch for `ReadThrottleEvents` and `WriteThrottleEvents`.
  2. Analyze query access patterns to identify partition keys with low cardinality or skewed access.
  3. Introduce a composite partition key or "sharding" technique (e.g., appending a random number or date bucket to the partition key).
  4. Migrate data to the new table structure.
- **Estimated Savings:** 20-50% on Provisioned Capacity costs
- **Risk Level:** High (Requires data migration and application logic changes)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Deep understanding of Cassandra data modeling.

#### KEYSPACES-006. Rightsize Row Data to Optimize 1KB/4KB Metering Blocks
- **What:** Ensure that your Cassandra rows fit neatly into the 1 KB write and 4 KB read metering increments.
- **Why It Saves Money:** Keyspaces rounds up to the nearest KB block. Writing 1.1 KB of data is billed as 2 WRUs (double the cost of a 1 KB write). Compressing data or removing unnecessary columns to bring a row from 1.1 KB down to 0.9 KB cuts write costs in half.
- **Implementation Steps:**
  1. Analyze average row sizes in your highest-volume tables.
  2. Identify rows that are slightly over the 1 KB, 2 KB, or 4 KB thresholds.
  3. Remove legacy/unused columns, use shorter column names (in Cassandra, column names are often stored with each cell), or use more efficient data types.
  4. Apply data modeling changes.
- **Estimated Savings:** 30-50% on WRUs/RRUs for borderline row sizes
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Profiling of average row sizes in production.

#### KEYSPACES-007. Consolidate and Minimize Secondary Indexes
- **What:** Remove unnecessary secondary indexes on Keyspaces tables.
- **Why It Saves Money:** Every write to a table with a secondary index incurs additional write costs because the index itself must be updated (consuming WRUs/WCUs). Furthermore, querying via secondary indexes often results in scatter-gather read operations, consuming massively more RRUs/RCUs than a direct partition key lookup.
- **Implementation Steps:**
  1. Review all secondary indexes (`INDEX` definitions) in the schema.
  2. Identify indexes that are rarely queried or can be replaced by a materialized view or a redesigned base table.
  3. Drop unused or inefficient secondary indexes using `DROP INDEX`.
  4. Rewrite queries to rely on the primary/partition key.
- **Estimated Savings:** 10-30% on Write throughput, up to 80% on Read throughput for unoptimized queries
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Query pattern analysis.

### 3. Commitment Discounts

#### KEYSPACES-008. Leverage AWS Enterprise Discount Program (EDP)
- **What:** Include Amazon Keyspaces spend in your overall AWS EDP commitment.
- **Why It Saves Money:** Keyspaces does not currently offer native Reserved Capacity or Savings Plans like DynamoDB. The primary way to get a discount on standard Keyspaces rates is through an aggregate AWS EDP, which provides a flat percentage discount across all AWS services based on high annual spend commitments.
- **Implementation Steps:**
  1. Forecast total AWS and Keyspaces spend for the next 1-3 years.
  2. Engage your AWS Account Manager to negotiate an EDP if total spend exceeds AWS minimum thresholds (typically $1M+/year).
  3. Apply the global EDP discount to the billing account.
- **Estimated Savings:** 5-20% (depending on EDP tier)
- **Risk Level:** Low (Financial risk if commitments aren't met)
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** High total AWS spend and an active AWS account team.

#### KEYSPACES-009. Utilize AWS Private Pricing Agreements for Massive Scale
- **What:** Negotiate custom Private Pricing Agreements (PPAs) specifically for Amazon Keyspaces.
- **Why It Saves Money:** If your organization processes petabytes of data or spends tens of thousands of dollars purely on Keyspaces, AWS may offer a custom PPA to lower the unit cost of WRUs/RRUs or storage beyond standard EDP rates, ensuring you don't migrate to self-hosted Cassandra.
- **Implementation Steps:**
  1. Aggregate all Keyspaces usage metrics (WRUs, RRUs, Storage) using CUR data.
  2. Build a financial model showing the cost of Keyspaces vs. self-managed EC2 Cassandra.
  3. Present the business case to AWS to negotiate a service-specific PPA.
- **Estimated Savings:** 10-30% off list prices
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Massive Keyspaces-specific spend (e.g., $50k+/month).

### 4. Architecture Changes

#### KEYSPACES-010. Enforce `LOCAL_ONE` Consistency for Read-Heavy Endpoints
- **What:** Change the Cassandra read consistency level from `LOCAL_QUORUM` (strongly consistent) to `LOCAL_ONE` (eventually consistent) for queries that do not require immediate strict accuracy.
- **Why It Saves Money:** `LOCAL_ONE` and `ONE` consistency reads are billed at a 50% discount (0.5 RRUs or RCUs per 4 KB read) compared to standard `LOCAL_QUORUM` reads.
- **Implementation Steps:**
  1. Review application source code and Cassandra drivers.
  2. Identify read operations where eventual consistency is acceptable (e.g., rendering profile info, historical logs).
  3. Update the query execution configuration in the code to specify `ConsistencyLevel.LOCAL_ONE`.
- **Estimated Savings:** 50% on Read Request Units (RRUs/RCUs)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Business validation that eventual consistency is acceptable.

#### KEYSPACES-011. Implement Read Caching with Amazon ElastiCache
- **What:** Place an in-memory caching layer (like Amazon ElastiCache for Redis or Memcached) in front of Amazon Keyspaces.
- **Why It Saves Money:** Repeatedly reading the same data (like configuration tables, user profiles, or popular products) from Keyspaces consumes RRUs every time. By caching the results in ElastiCache, you offload 80-95% of read traffic. ElastiCache is billed by instance size, not per-request, making it drastically cheaper for read-heavy workloads.
- **Implementation Steps:**
  1. Identify high-frequency, read-heavy query patterns (e.g., lookups for top 100 items).
  2. Provision an Amazon ElastiCache cluster.
  3. Modify the application to implement a cache-aside or look-aside pattern (check Redis first, if miss, query Keyspaces and write to Redis).
- **Estimated Savings:** 50-90% on Read throughput costs
- **Risk Level:** Medium (Introduces cache invalidation complexity)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Identifiable read-heavy, low-cardinality access patterns.

#### KEYSPACES-012. Implement Client-Side Payload Compression
- **What:** Compress large text or JSON strings within the application code before writing them to Keyspaces, and decompress them upon reading.
- **Why It Saves Money:** Writes are billed per 1 KB ($1.45/million WRUs). Compressing a 3 KB JSON payload down to 900 bytes reduces the WRU cost from 3 WRUs to 1 WRU (a 66% savings on writes) and also reduces underlying storage costs.
- **Implementation Steps:**
  1. Identify columns storing compressible data (JSON, XML, long text).
  2. Implement compression (e.g., GZIP, LZ4, Zstandard) in the application layer prior to the Keyspaces `INSERT` statement.
  3. Store the data as a `blob` type in Keyspaces.
  4. Decompress the `blob` in the application layer upon `SELECT`.
- **Estimated Savings:** 30-70% on WRUs and Storage
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application code modification; slight CPU overhead on clients.

#### KEYSPACES-013. Avoid `ALLOW FILTERING` in Queries
- **What:** Eliminate the use of the `ALLOW FILTERING` directive in CQL queries.
- **Why It Saves Money:** `ALLOW FILTERING` forces Keyspaces to scan potentially millions of rows across multiple partitions, discarding the ones that don't match the filter. You are billed for *every row read* during the scan, not just the rows returned. This can cause astronomical, unexpected RRU/RCU spikes.
- **Implementation Steps:**
  1. Audit application code and logs for any CQL queries containing `ALLOW FILTERING`.
  2. Prevent developers from using this clause via code reviews or static analysis.
  3. Redesign the data model or create a secondary index to support the query natively without filtering.
- **Estimated Savings:** 90%+ on anomalous read spikes
- **Risk Level:** Low (Highly recommended best practice)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Refactoring of inefficient queries.

### 5. Scheduling & Auto-Scaling

#### KEYSPACES-014. Configure Auto-Scaling with Aggressive Target Utilization (70%+)
- **What:** For tables in Provisioned Capacity Mode, enable auto-scaling with a high target utilization (e.g., 70-80%) and low minimum boundaries.
- **Why It Saves Money:** Provisioned capacity is billed hourly regardless of usage. Auto-scaling ensures you only pay for what you need by adjusting capacity based on actual traffic. Setting target utilization to 70% keeps the buffer tight, preventing over-payment for idle capacity. Setting low minimums (e.g., 1 or 5 units) allows the table to scale down to pennies during the night.
- **Implementation Steps:**
  1. Navigate to Keyspaces in the AWS Console or use IaC.
  2. Select the Provisioned Capacity table.
  3. Enable Auto-Scaling for both Read and Write capacity.
  4. Set target utilization to 70%.
  5. Set the Minimum capacity as low as the workload allows.
- **Estimated Savings:** 30-60% vs. static provisioned capacity
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Table must be in Provisioned Mode.

#### KEYSPACES-015. Pre-Warm Capacity for Scheduled Peak Events
- **What:** Use AWS Application Auto Scaling schedules to manually scale up Provisioned Capacity right before known traffic spikes, and scale down afterward.
- **Why It Saves Money:** Standard reactive auto-scaling can take several minutes to respond to sudden spikes, leading to throttling. To avoid this, engineers often leave capacity permanently over-provisioned. Scheduled scaling allows you to keep capacity at the absolute minimum, preemptively scaling up only when necessary (e.g., Black Friday, nightly batch jobs), eliminating 24/7 over-provisioning waste.
- **Implementation Steps:**
  1. Identify predictable traffic events (e.g., a daily data sync at 2:00 AM).
  2. Configure a Scheduled Action in AWS Application Auto Scaling for the Keyspaces table.
  3. Set the schedule to scale up 15 minutes before the event, and restore minimums after the event concludes.
- **Estimated Savings:** 20-50% on Provisioned Capacity
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Predictable workload schedules.

### 6. Pricing Model Optimization

#### KEYSPACES-016. Switch from On-Demand to Provisioned Mode at >15% Utilization
- **What:** Transition tables from On-Demand Capacity Mode to Provisioned Capacity Mode if their average utilization is consistently above 15%.
- **Why It Saves Money:** On-Demand capacity is significantly more expensive per unit ($1.45/million WRUs vs ~$0.47/WCU-month). The break-even point is roughly 15% utilization. If a table has predictable, steady traffic that utilizes more than 15% of an equivalent provisioned baseline, Provisioned Mode is mathematically cheaper.
- **Implementation Steps:**
  1. Analyze CloudWatch `ConsumedWriteCapacityUnits` and `ConsumedReadCapacityUnits` for On-Demand tables.
  2. Calculate the equivalent WCU/RCU needed to sustain that traffic.
  3. Compare the monthly On-Demand cost vs. Provisioned cost.
  4. Update table properties to `Provisioned` mode and enable auto-scaling.
- **Estimated Savings:** Up to 70% for steady-state workloads
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Steady, predictable traffic patterns.

#### KEYSPACES-017. Toggle to Provisioned Mode Temporarily for Bulk Migrations/ETL
- **What:** Switch an On-Demand table to Provisioned Capacity specifically for the duration of a massive data import, ETL job, or migration, then switch back.
- **Why It Saves Money:** Writing millions of rows using On-Demand at $1.45/million WRUs is up to 3x more expensive than scaling up Provisioned WCU capacity to handle the exact throughput needed for a few hours.
- **Implementation Steps:**
  1. Before initiating a data migration, switch the table to Provisioned Capacity.
  2. Set the WCUs high enough to ingest the data quickly.
  3. Execute the migration job.
  4. Immediately switch the table back to On-Demand capacity once the job completes. (Note: AWS allows switching capacity modes a limited number of times per day).
- **Estimated Savings:** 60-70% on one-time bulk ingestion costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Scheduled downtime or maintenance windows for ETL jobs.

### 7. Network & Data Transfer Optimization

#### KEYSPACES-018. Deploy VPC Endpoints (PrivateLink) for Keyspaces
- **What:** Use AWS PrivateLink (Interface VPC Endpoints) to route traffic from your VPC directly to Keyspaces without traversing the public internet.
- **Why It Saves Money:** If your resources (EC2, Lambda) in private subnets communicate with Keyspaces over the public endpoint, traffic routes through a NAT Gateway. NAT Gateways charge $0.045 per GB of data processed. A VPC endpoint keeps traffic on the AWS network, bypassing NAT Gateway data processing fees (VPC Endpoint data processing is cheaper at ~$0.01/GB).
- **Implementation Steps:**
  1. Open the Amazon VPC Console.
  2. Create an Interface VPC Endpoint for the Keyspaces service (`cassandra.<region>.amazonaws.com`).
  3. Associate the endpoint with your private subnets and update Security Groups.
  4. Ensure applications use the VPC endpoint for connectivity.
- **Estimated Savings:** ~75% reduction in data transfer/processing costs for Keyspaces traffic
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC architecture with private subnets.

#### KEYSPACES-019. Co-Locate Client Compute in the Same AWS Region
- **What:** Ensure that the applications (EC2, EKS, Lambda) querying Keyspaces are located in the exact same AWS Region as the Keyspaces table.
- **Why It Saves Money:** Data transfer between AWS Regions costs $0.02 per GB. If your application in `us-west-2` queries a Keyspaces table in `us-east-1`, you pay data transfer charges on every megabyte returned. Co-locating resources ensures data transfer is free (or vastly reduced).
- **Implementation Steps:**
  1. Use AWS Cost Explorer or CUR to identify Cross-Region Data Transfer costs associated with Keyspaces.
  2. Verify the region of the Keyspaces table and the compute clients.
  3. Migrate compute resources to the table's region, or replicate the Keyspaces data to the local region if multi-region architecture is strictly required.
- **Estimated Savings:** 100% of inter-region data transfer costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps | Architect
- **Prerequisites:** Application deployment flexibility.

---

## Cross-Service Synergies
- **Amazon S3:** Offload large payloads to S3 to reduce Keyspaces WRU, RRU, and Storage costs.
- **Amazon ElastiCache:** Cache frequent Keyspaces queries in Redis/Memcached to slash read throughput requirements.
- **AWS Application Auto Scaling:** Schedule Provisioned Capacity adjustments to align with known traffic peaks, minimizing idle buffer costs.
- **AWS VPC (PrivateLink):** Bypass expensive NAT Gateways by routing Keyspaces traffic through VPC Endpoints.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
- Identify the billing breakdown by usage type (`WriteRequestUnits`, `ReadRequestUnits`, `Storage`, `TTLDeletes`, `PITRStorage`).
- Determine if the majority of spend is concentrated on Storage or Compute (Throughput).

### B. CloudWatch Metrics
- `ConsumedReadCapacityUnits` / `ConsumedWriteCapacityUnits`: Identifies true utilization vs. provisioned limits.
- `ReadThrottleEvents` / `WriteThrottleEvents`: Highlights hot partitions or under-provisioned tables.

### C. AWS Config / Trusted Advisor
- Audit tables for `pointInTimeRecovery` status on non-prod environments.
- Verify capacity mode (On-Demand vs. Provisioned) settings across all tables.

### D. Company Policies
- Data retention rules (critical for setting TTLs).
- RPO/RTO requirements (critical for determining PITR necessity).

### E. IaC (Optional)
- Terraform/CloudFormation templates to quickly audit table schemas, secondary indexes, and auto-scaling configurations.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "KEYSPACES-001",
  "service": "Amazon Keyspaces",
  "strategy_name": "Disable Point-in-Time Recovery (PITR) on Non-Production Tables",
  "category": "Waste Elimination",
  "estimated_savings_percentage": "40-47%",
  "risk_level": "Low",
  "implementation_scope": "Engineer/DevOps",
  "actionable": true
}
```

### Summary Report Table

| ID | Strategy | Category | Savings | Risk | Scope |
|---|---|---|---|---|---|
| KEYSPACES-001 | Disable PITR on Non-Prod | Waste Elimination | 40-47% | Low | Engineer/DevOps |
| KEYSPACES-002 | Implement TTL for Ephemeral Data | Waste Elimination | 20-80% | Medium | Engineer/DevOps |
| KEYSPACES-003 | Deprovision Orphaned Tables | Waste Elimination | 100% | Medium | Engineer/DevOps |
| KEYSPACES-004 | Offload Large Payloads to S3 | Rightsizing | 50-90% | Medium | Engineer/DevOps |
| KEYSPACES-005 | Optimize Partition Key Design | Rightsizing | 20-50% | High | Engineer/DevOps |
| KEYSPACES-006 | Rightsize Row Data (1KB/4KB blocks) | Rightsizing | 30-50% | Low | Engineer/DevOps |
| KEYSPACES-007 | Minimize Secondary Indexes | Rightsizing | 10-80% | Medium | Engineer/DevOps |
| KEYSPACES-008 | Leverage AWS EDP | Commitment Discounts | 5-20% | Low | Procurement |
| KEYSPACES-009 | Utilize Private Pricing Agreements | Commitment Discounts | 10-30% | Low | FinOps/Procurement |
| KEYSPACES-010 | Enforce LOCAL_ONE Consistency | Architecture Changes | 50% | Low | Engineer/DevOps |
| KEYSPACES-011 | Implement ElastiCache Read Caching | Architecture Changes | 50-90% | Medium | Engineer/DevOps |
| KEYSPACES-012 | Implement Client-Side Compression | Architecture Changes | 30-70% | Low | Engineer/DevOps |
| KEYSPACES-013 | Avoid ALLOW FILTERING Queries | Architecture Changes | 90%+ | Low | Engineer/DevOps |
| KEYSPACES-014 | Auto-Scaling with 70%+ Target | Scheduling & Auto-Scaling | 30-60% | Low | Engineer/DevOps |
| KEYSPACES-015 | Pre-Warm Capacity for Peak Events | Scheduling & Auto-Scaling | 20-50% | Low | Engineer/DevOps |
| KEYSPACES-016 | Switch to Provisioned Mode (>15% util) | Pricing Model Opt | 70% | Low | FinOps/Eng |
| KEYSPACES-017 | Toggle Provisioned for Bulk ETL | Pricing Model Opt | 60-70% | Low | Engineer/DevOps |
| KEYSPACES-018 | Deploy VPC Endpoints (PrivateLink) | Network & Data Transfer | 75% | Low | Engineer/DevOps |
| KEYSPACES-019 | Co-Locate Client Compute in Same Region | Network & Data Transfer | 100% | Medium | Engineer/DevOps |
