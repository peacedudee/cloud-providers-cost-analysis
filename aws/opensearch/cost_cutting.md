# Cost-Cutting Playbook: Amazon OpenSearch Service

> **Companion File:** [opensearch.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/opensearch/opensearch.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon OpenSearch Service (formerly Elasticsearch Service) supports real-time log analytics, application monitoring, and full-text search. Deployed as Managed Clusters (with dedicated master nodes, data nodes, and EBS volumes) or OpenSearch Serverless (billed per OCU-hour). Because search workloads keep large indexes in hot storage and require cluster redundancy, improper storage tiering and Serverless minimum idle charges represent major cost leaks.

This playbook provides **18 actionable strategies** across six operational categories, delivering an estimated **30–70% reduction in total OpenSearch spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Automate Index State Management (ISM) Tiering:** Migrate historical logs from Hot EBS ($0.122/GB-mo) to UltraWarm S3 ($0.024/GB-mo) for an **80% direct storage discount**.
2. **Migrate EBS Storage Volumes from `gp2` to `gp3`:** Instant 9.6% storage price reduction with independent IOPS scaling.
3. **Bypass OpenSearch Serverless for Small Workloads:** Eliminates the $350/mo (Dev) or $700/mo (Prod) flat idle OCU charge by deploying small managed nodes (`t3.medium.search` at $26/mo).

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Decommission Unused OpenSearch Development Domains
- **What:** Identify and delete development or staging OpenSearch domains with zero index writes and query requests over 14 days.
- **Why It Saves Money:** A small 3-node `r6g.large.search` dev cluster costs **$409.50/month** in compute alone.
- **Implementation Steps:**
  1. Check CloudWatch metrics `SearchLatency` and `IndexingLatency` across all domain endpoints.
  2. Take protective snapshot.
  3. Delete domain: `aws opensearch delete-domain --domain-name name`.
- **Estimated Savings:** 100% of idle domain spend.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Domain usage audit.

#### 2. Remove Dedicated Master Nodes in Non-Production Environments
- **What:** Reconfigure development and staging OpenSearch clusters to run without Dedicated Master Nodes (allowing Data Nodes to handle master responsibilities).
- **Why It Saves Money:** Production clusters require 3 Dedicated Master Nodes for quorum stability. In dev/test, running 3 `r6g.large.search` master nodes adds **$409.50/month** of unnecessary redundancy.
- **Implementation Steps:**
  1. Update domain configuration: Set `DedicatedMasterEnabled = false`.
- **Estimated Savings:** 30–50% reduction in cluster compute nodes.
- **Risk Level:** Low (for non-production environments).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod SLA alignment.

---

### 2. Rightsizing & Sizing Optimization

#### 3. Right-Size Data Node Instance Types and Counts
- **What:** Downsize data node instance types (e.g. from `r6g.xlarge.search` to `r6g.large.search`) where CPU utilization is < 30% and RAM (JVM heap) is < 50%.
- **Why It Saves Money:** Halving data node capacity reduces compute costs by 50% ($136.50/mo saved per node).
- **Implementation Steps:**
  1. Review CloudWatch metrics `CPUUtilization` and `JVMMemoryPressure` (keep JVM heap < 75% to prevent GC pauses).
  2. Modify cluster instance types via blue/green deployment.
- **Estimated Savings:** 30–50% compute node savings.
- **Risk Level:** Low (OpenSearch executes blue/green node replacement with zero downtime).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** JVM Memory Pressure monitoring.

#### 4. Evaluate Serverless vs Managed Cluster Break-Even
- **What:** Avoid deploying OpenSearch Serverless for small or low-traffic internal applications.
- **Why It Saves Money:** OpenSearch Serverless requires a minimum of **2 OCUs (Dev = $350.40/mo)** or **4 OCUs (Prod = $700.80/mo)** flat idle fee regardless of traffic. A small managed `t3.medium.search` cluster costs **$26.28/month**.
- **Implementation Steps:**
  1. Compare workload query volume vs Serverless OCU minimums.
  2. Deploy small Managed Clusters for steady-state low-traffic search.
- **Estimated Savings:** 80–90% savings for small search applications.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Cost break-even model.

---

### 3. Commitment Discounts

#### 5. Apply Database Savings Plans to OpenSearch Compute
- **What:** Purchase 1-year or 3-year Database Savings Plans covering Managed Cluster nodes and Serverless OCUs.
- **Why It Saves Money:** Database Savings Plans provide discounts up to **30–50%** off On-Demand compute rates.
- **Implementation Steps:**
  1. Analyze historical minimum hourly OpenSearch spend in Cost Explorer.
  2. Purchase Database Savings Plan commitment.
- **Estimated Savings:** 30–50% compute savings.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** 1-year cluster stability.

---

### 4. Storage & Data Tiering (ISM Policies)

#### 6. Automate Index State Management (ISM) Lifecycle Tiering
- **What:** Define ISM policies to automatically transition historical log indexes across storage tiers:
  - Days 1–7: **Hot Storage (gp3 EBS)** — $0.122/GB-mo
  - Days 8–30: **UltraWarm Storage (S3-Backed)** — $0.024/GB-mo (**80% cheaper**)
  - Days 31+: **Cold Storage (Archival)** — $0.0135/GB-mo (**89% cheaper**)
- **Why It Saves Money:** Keeping 5 TB of historical logs in Hot EBS costs **$610.00/month**. Moving days 8–30 to UltraWarm costs **$120.00/month** ($490/mo saved).
- **Implementation Steps:**
  1. Create ISM policy in OpenSearch Dashboards.
  2. Add rollover, warm transition, and cold/delete actions.
  3. Apply policy to index patterns (`logs-*`).
- **Estimated Savings:** **70–80% storage cost reduction**.
- **Risk Level:** Zero risk (UltraWarm supports active SQL/PPL queries).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** UltraWarm nodes enabled on domain.

#### 7. Migrate EBS Storage Volumes from `gp2` to `gp3`
- **What:** Update data node attached EBS volumes from `gp2` to `gp3`.
- **Why It Saves Money:** `gp3` storage costs **$0.122/GB-month** in `us-east-1` (9.6% cheaper than `gp2`) and allows scaling baseline IOPS and throughput independently without over-provisioning volume capacity.
- **Implementation Steps:**
  1. Update domain configuration storage type to `gp3`.
  2. AWS executes seamless online volume migration.
- **Estimated Savings:** 9.6% direct storage savings + uncoupled IOPS efficiency.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

---

### 5. Architecture & Index Optimization

#### 8. Enable Index Compression (`best_compression`)
- **What:** Configure new index templates to use LZ4 `best_compression` codec (`index.codec: best_compression`).
- **Why It Saves Money:** Compresses index storage footprint on disk by **20–30%**, reducing hot EBS storage and UltraWarm storage costs.
- **Implementation Steps:**
  1. Add `"settings": { "index.codec": "best_compression" }` to index template DDL.
- **Estimated Savings:** 20–30% storage volume reduction.
- **Risk Level:** Low (slight CPU overhead during indexing).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Index template access.

#### 9. Optimize Shard Allocations and Prevent Over-Sharding
- **What:** Adjust index template shard settings so target primary shard sizes remain between **30 GB and 50 GB** for log analytics (or 10–30 GB for search).
- **Why It Saves Money:** Having thousands of tiny 500 MB shards creates severe JVM heap overhead, forcing deployment of larger data nodes to handle heap pressure.
- **Implementation Steps:**
  1. Audit shard sizes: `GET /_cat/shards`.
  2. Reduce `number_of_shards` in index templates.
  3. Use Index Rollover API based on size (`max_primary_shard_size: 40gb`).
- **Estimated Savings:** 30–50% memory/node count reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Shard size audit.

#### 10. Reduce Replica Count in Non-Production Domains
- **What:** Set `number_of_replicas = 0` for development and staging OpenSearch indexes.
- **Why It Saves Money:** Setting 1 replica doubles (2x) storage and indexing compute requirements. Dev environments do not require high-availability index mirroring.
- **Implementation Steps:**
  1. Update non-prod index settings: `PUT /_settings { "number_of_replicas": 0 }`.
- **Estimated Savings:** **50% storage and node capacity reduction** in non-prod.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod SLA alignment.

#### 11. Implement Field Mapping Pruning (Disable `_all` and Unused Fields)
- **What:** Disable indexing on raw string fields (`"index": false`) that are never queried or searched.
- **Why It Saves Money:** Reduces inverted index size on disk and speeds up indexing throughput.
- **Implementation Steps:**
  1. Review mapping definitions; set `"index": false` on heavy metadata fields.
- **Estimated Savings:** 15–30% storage savings.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Field query usage audit.

---

### 6. Network & Access Optimization

#### 12. Deploy OpenSearch inside Private VPC Subnets
- **What:** Place OpenSearch domains inside private VPC subnets and access via private VPC endpoints.
- **Why It Saves Money:** Keeps search traffic internal, avoiding public internet egress fees ($0.09/GB).
- **Implementation Steps:**
  1. Configure domain endpoint as `VPC Access`.
- **Estimated Savings:** Network egress savings.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC configuration.

#### 13. Optimize Bulk Indexing API Batch Sizes
- **What:** Configure indexing clients (Logstash, FluentBit, custom code) to send bulk indexing payloads in 5 MB–15 MB batches.
- **Why It Saves Money:** Reduces HTTP request overhead and CPU indexing load on data nodes.
- **Implementation Steps:**
  1. Update FluentBit `Buffer_Size` and `Batch_Size` settings.
- **Estimated Savings:** 10–20% indexing CPU efficiency gains.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ingestion pipeline access.

#### 14. Enforce Read-Only Tiers for Historical Data
- **What:** Mark historical indexes as read-only (`index.blocks.write: true`) before moving to UltraWarm.
- **Why It Saves Money:** Freezable read-only indexes optimize memory allocation and allow background segment merging.
- **Implementation Steps:**
  1. Add `read_only` step to ISM policy prior to warm transition.
- **Estimated Savings:** Memory optimization.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** ISM policy update.

#### 15. Consolidate Multiple Small Domains into Shared Multi-Tenant Clusters
- **What:** Combine multiple isolated single-tenant OpenSearch domains into a single shared multi-tenant cluster using Index-Level Access Control (Fine-Grained Access Control).
- **Why It Saves Money:** Amortizes master node overhead ($400/mo) and base node capacity across multiple business units.
- **Implementation Steps:**
  1. Enable Fine-Grained Access Control (FGAC).
  2. Map IAM roles to specific index patterns (`tenant-a-*`).
- **Estimated Savings:** 40–60% cluster consolidation savings.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps & Security
- **Prerequisites:** Multi-tenant security architecture review.

#### 16. Monitor and Reduce Search Slow Logs
- **What:** Enable OpenSearch Slow Logs to identify and optimize queries taking > 2 seconds.
- **Why It Saves Money:** Unoptimized wildcard queries (`*foo*`) consume 100% CPU on data nodes, forcing domain scaling.
- **Implementation Steps:**
  1. Set slow log thresholds: `PUT /_settings { "index.search.slowlog.threshold.query.warn": "2s" }`.
  2. Refactor slow queries.
- **Estimated Savings:** 20–40% compute efficiency gains.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Slow log configuration.

#### 17. Use Graviton-Based Node Instances (`r6g.search`, `c6g.search`)
- **What:** Update data node instance types to Graviton-based families.
- **Why It Saves Money:** Graviton search nodes are **~15–20% cheaper** than x86 equivalents (`r6i.search`).
- **Implementation Steps:**
  1. Modify domain instance type to `r6g.large.search`.
- **Estimated Savings:** 15–20% compute savings.
- **Risk Level:** Zero risk (seamless blue/green migration).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 18. Set Up Automated Snapshot Deletion via ISM
- **What:** Configure automated snapshot policies to prune automated OpenSearch snapshots beyond compliance requirements.
- **Why It Saves Money:** Stops snapshot storage accumulation in S3 ($0.024/GB-mo).
- **Implementation Steps:**
  1. Configure Snapshot Management (SM) policy in OpenSearch.
- **Estimated Savings:** Snapshot storage optimization.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Snapshot policy definition.

---

## Cross-Service Synergies

```
[ OpenSearch Managed Cluster ] 
        │
        ├──(Storage Tiering)──> [ UltraWarm S3 Storage ] (Saves 80% on storage $0.024 vs $0.122/GB)
        │
        ├──(EBS Volume Upgrade)─> [ gp3 Volumes ] (Instant 9.6% storage savings)
        │
        └──(Compute Savings)───> [ Database Savings Plans ] (30-50% compute discount)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `ES:r6g.large.search`, `ES:UltraWarm:Storage`, `ES:DirectDownload:ColdStorage`, `ES:OCU-Hours`.
- `line_item_resource_id`: OpenSearch Domain Name / ARN.

### B. CloudWatch & Cluster Metrics
- CloudWatch `AWS/ES` Namespace: `CPUUtilization`, `JVMMemoryPressure`, `FreeStorageSpace`, `SearchLatency`, `IndexingLatency`, `ClusterStatus.red`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "OS-ISM-001",
  "service": "OpenSearch",
  "category": "Storage & Data Tiering",
  "resource_id": "arn:aws:es:us-east-1:123456789012:domain/production-logs-cluster",
  "resource_name": "production-logs-cluster",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "hot_ebs_storage_gb": 8000,
    "storage_type": "gp2",
    "ultrawarm_enabled": false,
    "monthly_storage_cost_usd": 1080.00
  },
  "recommended_config": {
    "hot_ebs_storage_gb": 2000,
    "storage_type": "gp3",
    "ultrawarm_storage_gb": 6000,
    "ism_policy": "Migrate to UltraWarm after 7 days",
    "projected_monthly_storage_cost_usd": 388.00
  },
  "financial_impact": {
    "monthly_savings_usd": 692.00,
    "annual_savings_usd": 8304.00,
    "savings_percentage": 64.1
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "UltraWarm maintains active PPL/SQL query capability for historical logs."
  },
  "implementation": {
    "scope": "Engineer/DevOps",
    "effort_estimate": "1-2 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Storage Tiering (ISM UltraWarm)** | 8 | $14,500.00 | $9,280.00 | 64.0% | Low |
| **Serverless vs Managed Sizing** | 4 | $4,200.00 | $3,360.00 | 80.0% | Low |
| **Rightsizing Data/Master Nodes** | 10 | $12,800.00 | $5,120.00 | 40.0% | Low |
| **Commitment Discounts** | 2 | $16,000.00 | $5,600.00 | 35.0% | Low |
| **Total** | **24** | **$47,500.00** | **$23,360.00** | **49.2%** | -- |
