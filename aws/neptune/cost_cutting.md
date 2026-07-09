# Cost-Cutting Playbook: Amazon Neptune
> **Companion File:** [neptune.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/neptune/neptune.md)
> **Last Updated:** July 2026
---
## Executive Summary
This playbook details targeted cost-optimization strategies for Amazon Neptune. Due to its cloud-native architecture (separating compute and storage), organizations can realize significant savings by aggressively managing compute footprints (e.g., pausing Neptune Analytics, scheduling non-prod instances), adopting Serverless carefully (setting NCU limits), and managing I/O costs through the I/O-Optimized storage tier for query-intensive workloads.

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
- **Amazon S3:** Used for bulk-loading graph data into Neptune and archiving cold graph data. Using VPC Endpoints for S3 prevents NAT Gateway data processing charges.
- **AWS Lambda/EventBridge:** Essential for orchestrating auto-start/stop schedules for non-prod clusters and pausing Neptune Analytics graphs.
- **Amazon CloudWatch/CloudTrail:** Required to identify unused clusters, inefficient queries causing I/O spikes, and scaling events on Serverless clusters.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
Identify line items where `lineItem/ProductCode` is `AmazonNeptune`. Look at `lineItem/UsageType` to differentiate between compute (`InstanceUsage`, `ServerlessUsage`), storage (`StorageUsage`), I/O (`IOUsage`), and analytics (`AnalyticsUsage`).
### B. CloudWatch Metrics
Monitor `CPUUtilization`, `FreeableMemory`, `VolumeReadIOPs`, `VolumeWriteIOPs`, `ServerlessDatabaseCapacity`, and `GremlinRequestsPerSec` / `SparqlRequestsPerSec` to determine instance rightsizing and identify I/O-heavy workloads.
### C. AWS Config / Trusted Advisor
Check for underutilized Neptune instances, unattached clusters, and instances running on older generation instance types.
### D. Company Policies
Understand retention policies for snapshots, acceptable downtime for dev/test environments, and RI/commitment policies.
### E. IaC (Optional)
Terraform or CloudFormation templates to implement infrastructure adjustments like instance type updates, Serverless max capacity limits, and I/O-Optimized configurations.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "NEPTUNE-001",
  "resource_id": "arn:aws:rds:us-east-1:123456789012:cluster:my-neptune-cluster",
  "strategy_used": "Convert to I/O-Optimized Storage",
  "monthly_savings": 450.00,
  "effort_level": "Low"
}
```
### Summary Report Table
| Strategy | Potential Savings | Risk | Implementation Scope |
|----------|-------------------|------|----------------------|
| Convert to I/O-Optimized | High | Low | FinOps / DevOps |
| Pause Neptune Analytics | High | Low | DevOps |
| Purchase Reserved Nodes | High | Low | FinOps |
| Schedule Non-Prod Clusters | High | Low | DevOps |
| Optimize Graph Queries | High | Medium | Engineer |

---

### 1. Waste Elimination

#### NEPTUNE-001. Delete Unused Manual Snapshots
- **What:** Identify and delete manual Neptune cluster snapshots that are no longer needed for recovery or compliance.
- **Why It Saves Money:** Manual snapshots are billed at $0.095 per GB-month. Keeping old manual snapshots indefinitely accrues compounding storage charges.
- **Implementation Steps:**
  1. List all manual Neptune snapshots using the AWS Console or CLI.
  2. Identify snapshots older than the organization's retention policy (e.g., > 30 days).
  3. Delete the obsolete snapshots.
- **Estimated Savings:** 1-5% of total Neptune spend.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Backup and retention policy defined.

#### NEPTUNE-002. Prune Non-Prod Replicas
- **What:** Configure development, test, and QA Neptune clusters to run with 0 reader nodes (Single-Node configuration).
- **Why It Saves Money:** Replicas cost the same as primary instances. A cluster with one writer and one reader costs twice as much in compute as a single-node cluster.
- **Implementation Steps:**
  1. Identify non-production Neptune clusters using tags or naming conventions.
  2. Review the number of instances attached to the cluster.
  3. Delete all read replica instances in these environments.
- **Estimated Savings:** 50% of compute costs for non-prod environments.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod environments do not require high availability.

#### NEPTUNE-003. Delete Unused Development Clusters
- **What:** Terminate Neptune clusters in dev/sandbox environments that are no longer being actively used.
- **Why It Saves Money:** Eliminates 100% of the compute and storage costs associated with abandoned environments. An idle serverless cluster still costs at least $219/month per node.
- **Implementation Steps:**
  1. Monitor CloudWatch metrics for clusters showing near-zero activity over 14-30 days.
  2. Verify with the owning team that the environment is defunct.
  3. Take a final snapshot (optional) and delete the cluster.
- **Estimated Savings:** 100% of the abandoned cluster's cost.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch metric visibility.

### 2. Rightsizing

#### NEPTUNE-004. Pause Idle Neptune Analytics Graphs
- **What:** Pause in-memory Neptune Analytics graphs when they are not actively executing graph algorithms.
- **Why It Saves Money:** While paused, you are billed at a discounted rate of 10% of the active compute rate—a 90% discount on m-NCU charges.
- **Implementation Steps:**
  1. Identify batch graph analytics jobs.
  2. Automate the pausing of the Neptune Analytics graph immediately after the batch job completes using AWS CLI or SDKs.
  3. Resume the graph prior to the next scheduled job.
- **Estimated Savings:** Up to 90% of Neptune Analytics compute costs for batch workloads.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload is batch-oriented, not serving real-time requests.

#### NEPTUNE-005. Rightsize Provisioned Instances
- **What:** Downsize overprovisioned Neptune compute instances (e.g., scaling from `db.r6g.4xlarge` to `db.r6g.2xlarge`).
- **Why It Saves Money:** Halving the instance size generally cuts the compute cost by 50%.
- **Implementation Steps:**
  1. Analyze CloudWatch `CPUUtilization` and `FreeableMemory` over a 2-4 week period.
  2. Identify instances where peak CPU and memory usage remain consistently below 30-40%.
  3. Modify the instance to a smaller size during a scheduled maintenance window.
- **Estimated Savings:** 50%+ per downsized instance.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Rightsizing analysis tools/dashboards.

#### NEPTUNE-006. Archive Cold Graph Data
- **What:** Identify stale or historical sub-graphs (e.g., old log data, inactive relationships) and offload them to Amazon S3.
- **Why It Saves Money:** Neptune storage costs $0.10/GB-month (Standard), while S3 Standard costs ~$0.023/GB-month.
- **Implementation Steps:**
  1. Identify historical nodes/edges no longer queried in real-time.
  2. Export this data to S3 using Neptune export tools.
  3. Delete the cold data from the Neptune cluster.
- **Estimated Savings:** ~75% reduction on the storage cost for archived data.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Data lifecycle policy.

### 3. Commitment Discounts

#### NEPTUNE-007. Purchase Reserved Nodes for Production
- **What:** Commit to a 1- or 3-year term for production Neptune instances.
- **Why It Saves Money:** Reserved instances can offer up to a 50% discount compared to On-Demand hourly rates.
- **Implementation Steps:**
  1. Ensure production instance families and sizes are stable.
  2. Calculate baseline usage and verify that the instance will be needed for > 1 year.
  3. Purchase Neptune Reserved Nodes via the AWS Billing Console.
- **Estimated Savings:** Up to 50% on provisioned compute costs.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Stable production workload footprint.

### 4. Architecture Changes

#### NEPTUNE-008. Optimize Graph Queries to Reduce I/O
- **What:** Refactor deeply nested Gremlin loops, unbounded openCypher scans, or inefficient SPARQL queries.
- **Why It Saves Money:** On Neptune Standard, I/O requests cost $0.20 per million. Unoptimized graph traversals create "I/O storms" that exponentially increase the storage bill.
- **Implementation Steps:**
  1. Use Neptune `profile` and `explain` endpoints to analyze query execution plans.
  2. Identify queries scanning a large percentage of the graph unnecessarily.
  3. Rewrite queries to use bound limits, targeted indices, and more efficient traversal patterns.
- **Estimated Savings:** 20-80% of I/O request costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Graph query language expertise.

#### NEPTUNE-009. Cache Static Graph Responses
- **What:** Place a caching layer (like Amazon ElastiCache or API Gateway caching) in front of read-heavy, infrequently changing graph queries.
- **Why It Saves Money:** Serves requests from cache instead of querying Neptune, reducing compute load (allowing smaller instances) and cutting I/O request charges.
- **Implementation Steps:**
  1. Identify high-frequency read queries that return static or slowly changing data.
  2. Implement an application-level cache or API Gateway cache.
  3. Set appropriate TTLs to ensure data freshness.
- **Estimated Savings:** Variable (can significantly reduce compute/IO needs).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Read-heavy workload with tolerant data staleness.

#### NEPTUNE-010. Disable Multi-AZ Deployments for Dev/Test
- **What:** Ensure that non-production clusters are not deployed across multiple Availability Zones unnecessarily.
- **Why It Saves Money:** Placing active compute instances (read replicas) in multiple AZs for dev/test doubles compute costs and adds potential cross-AZ data transfer fees.
- **Implementation Steps:**
  1. Audit non-prod clusters to ensure they operate as Single-Node (1 primary, 0 replicas).
  2. Ensure the primary instance is in a single AZ.
- **Estimated Savings:** ~50% on compute costs for dev/test.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Acceptance of lower availability in dev/test.

### 5. Scheduling & Auto-Scaling

#### NEPTUNE-011. Schedule Non-Prod Cluster Start/Stop
- **What:** Automatically stop dev and test Neptune clusters outside of business hours (e.g., nights and weekends).
- **Why It Saves Money:** Stopping a cluster for 14 hours a day and all weekend reduces total active hours from 168/week to 50/week, a ~70% reduction in compute runtime.
- **Implementation Steps:**
  1. Tag all eligible non-prod Neptune clusters.
  2. Deploy an AWS Lambda function triggered by EventBridge rules to run `StopDBCluster` at 6 PM and `StartDBCluster` at 8 AM.
- **Estimated Savings:** ~70% of compute costs for scheduled clusters.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Defined working hours for development teams.

#### NEPTUNE-012. Configure Serverless Max NCU Limits
- **What:** Set a hard ceiling on the maximum Neptune Capacity Units (NCUs) a Serverless cluster can scale to.
- **Why It Saves Money:** A single inefficient query can cause Neptune Serverless to rapidly autoscale to 128 NCUs ($15.36/hour). Setting a cap (e.g., 16 or 32 NCUs) prevents runaway cost spikes.
- **Implementation Steps:**
  1. Review the cluster scaling settings in the Neptune Console.
  2. Analyze typical peak NCU usage via CloudWatch.
  3. Modify the Serverless configuration to set a realistic `MaxCapacity` value.
- **Estimated Savings:** Prevents 10x-50x compute cost spikes during bad query events.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Neptune Serverless deployed.

#### NEPTUNE-013. Migrate Spiky Workloads to Neptune Serverless
- **What:** Transition unpredictable, spiky, or intermittent workloads from provisioned instances to Neptune Serverless.
- **Why It Saves Money:** Instead of overprovisioning a large instance to handle infrequent traffic spikes, Serverless scales down to 2.5 NCUs ($0.30/hr) during idle periods and scales up only when queried.
- **Implementation Steps:**
  1. Identify clusters with highly variable `CPUUtilization`.
  2. Modify the cluster instance type to `db.serverless`.
  3. Set min (2.5) and max NCU boundaries.
- **Estimated Savings:** 30-60% for highly variable workloads.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload profile is intermittent or spiky.

### 6. Pricing Model Optimization

#### NEPTUNE-014. Switch to I/O-Optimized Storage Tier
- **What:** Change the storage configuration of high-traffic Neptune clusters from Standard to I/O-Optimized.
- **Why It Saves Money:** I/O-Optimized eliminates the $0.20 per million I/O request charge. If I/O charges make up more than 25% of the total cluster bill under Standard, this switch yields net savings.
- **Implementation Steps:**
  1. Analyze Cost Explorer to determine the % of spend attributed to Neptune I/O requests for a specific cluster.
  2. If >25%, modify the cluster storage type to I/O-Optimized.
  3. Monitor the bill for the next 7 days to confirm savings.
- **Estimated Savings:** 10-40% on total cluster cost for I/O-heavy workloads.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Bill analysis capability.

#### NEPTUNE-015. Upgrade to Graviton2 (db.r6g) Instances
- **What:** Migrate older generation Neptune instances (e.g., `db.r5`) to AWS Graviton2 instances (`db.r6g`).
- **Why It Saves Money:** Graviton2 instances provide up to 20% better performance-per-dollar compared to equivalent x86-based instances, effectively offering a ~10% cost reduction for identical specs.
- **Implementation Steps:**
  1. Identify any clusters running `db.r4` or `db.r5` instances.
  2. Update infrastructure as code (IaC) to use `db.r6g` equivalents.
  3. Apply the change during a maintenance window.
- **Estimated Savings:** ~10% on provisioned compute rates.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compatibility testing.

### 7. Network & Data Transfer Optimization

#### NEPTUNE-016. Align App and Database in the Same Availability Zone
- **What:** Ensure that the compute instances (EC2, EKS, Lambda) querying Neptune reside in the same AZ as the primary Neptune instance.
- **Why It Saves Money:** Data transferred across AZs within the same region costs $0.01 per GB in each direction. Heavy graph query results traversing AZs can generate hidden network fees.
- **Implementation Steps:**
  1. Use VPC Flow Logs or Cost Explorer to identify `DataTransfer-Regional-Bytes`.
  2. Align the application deployments to favor the AZ where the Neptune primary resides.
- **Estimated Savings:** Eliminates cross-AZ data transfer fees ($0.02/GB round-trip).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Architecture allows AZ-affinity routing.

#### NEPTUNE-017. Use VPC Endpoints for Bulk Loading from S3
- **What:** Configure an S3 Gateway VPC Endpoint when using Neptune's bulk loader to import data from Amazon S3.
- **Why It Saves Money:** If the VPC uses a NAT Gateway to access the internet/S3, pulling large graph datasets through it incurs a $0.045/GB data processing charge. A Gateway VPC Endpoint routes traffic to S3 for free.
- **Implementation Steps:**
  1. Check the VPC route tables associated with the Neptune cluster subnets.
  2. If an S3 Gateway Endpoint does not exist, create one.
  3. Add the endpoint route to the subnets.
- **Estimated Savings:** $0.045 per GB of data loaded.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS VPC configuration access.
