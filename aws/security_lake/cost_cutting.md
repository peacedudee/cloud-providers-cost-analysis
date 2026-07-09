# Cost-Cutting Playbook: Amazon Security Lake
> **Companion File:** [security_lake.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/security_lake/security_lake.md)
> **Last Updated:** July 2026
---
## Executive Summary
Amazon Security Lake centralizes and normalizes security logs into the OCSF format for investigation and analysis. Since pricing is primarily driven by log ingestion volume and subsequent S3 storage and Athena query fees, cost optimization requires strict control over *which* logs are ingested, *how long* they are stored in expensive tiers, and *how efficiently* they are queried. The most impactful savings come from disabling high-volume network logs (VPC Flow Logs, DNS logs) in non-production environments and aggressively tiering S3 storage to Glacier classes.

## Strategy Categories
### 1. Waste Elimination
*   **SECLAKE-WST-001: Disable Ingestion for Non-Production Accounts.** Exclude dev, test, and sandbox AWS accounts from Security Lake data collection entirely to eliminate ingestion costs for non-critical environments.
*   **SECLAKE-WST-002: Disable High-Volume Network Logs in Dev/Test.** Disable VPC Flow Logs and Route 53 DNS Query Logs in Security Lake for non-production workloads. These logs are high volume ($0.25/GB) and rarely necessary for standard SOC monitoring in dev environments.
*   **SECLAKE-WST-003: Remove Stale Custom Log Source Integrations.** Audit and remove third-party security tool integrations that are no longer actively used by the SOC to halt unnecessary ingestion fees.
*   **SECLAKE-WST-004: Clean Up Orphaned Security Lake S3 Buckets.** When disabling Security Lake in regions or accounts, ensure the underlying S3 buckets containing historical OCSF data are deleted if the retention policy allows.
*   **SECLAKE-WST-005: Remove Inactive Data Subscribers.** Identify and remove stale Security Lake subscribers (e.g., old IAM roles or external SIEM tools) that might be polling data or triggering unnecessary data transfer/processing costs.

### 2. Rightsizing
*   **SECLAKE-RGT-001: Ingest CloudTrail Management Events Only.** Avoid ingesting CloudTrail Data Events (e.g., S3 object-level logging) into Security Lake unless explicitly required for a specific compliance mandate, as they generate massive log volumes. Stick to Management Events ($0.75/GB).
*   **SECLAKE-RGT-002: Filter Custom Log Sources Pre-Ingestion.** When bringing custom data sources into Security Lake, filter the logs at the source to send only actionable security events rather than raw, unfiltered event streams.
*   **SECLAKE-RGT-003: Limit EKS Audit Log Ingestion.** Limit the ingestion of Amazon EKS audit logs to critical production clusters, as Kubernetes audit trails can be extremely verbose and expensive to ingest.

### 3. Commitment Discounts
*   **SECLAKE-COM-001: Leverage AWS Enterprise Discount Program (EDP).** While Security Lake lacks native reserved pricing, high-volume ingestions contribute to overall AWS spend and can be discounted under a consolidated EDP contract.

### 4. Architecture Changes
*   **SECLAKE-ARC-001: Automate S3 Storage Tiering (Lifecycle Rules).** Implement aggressive S3 Lifecycle policies to transition OCSF parquet files from S3 Standard to Glacier Instant Retrieval (after 30-60 days) and Glacier Deep Archive (after 90 days), cutting storage costs by up to 95%.
*   **SECLAKE-ARC-002: Enforce Partition Pruning in Athena.** Train security analysts and configure automated queries to strictly use date (`eventday`) and region partition filters in Amazon Athena. This prevents $5.00/TB full table scans across years of historical data.
*   **SECLAKE-ARC-003: Implement Athena Workgroup Query Limits.** Create specific Athena Workgroups for Security Lake investigations and apply data-scanned limits (e.g., 500GB per query) to prevent analysts from accidentally running unbounded runaway queries.
*   **SECLAKE-ARC-004: Shift Long-Term Retention to Security Lake.** Instead of storing 1-3 years of logs in an expensive third-party SIEM, use the SIEM for 30-day hot analytics and use Security Lake's S3 backend for cheap, long-term compliance retention.

### 5. Scheduling & Auto-Scaling
*   **SECLAKE-SCH-001: Batch Custom Source Uploads.** For custom log sources, batch logs into larger files before sending them to Security Lake, rather than streaming micro-events in real-time, to optimize S3 PUT request costs and Lambda processing overhead.

### 6. Pricing Model Optimization
*   **SECLAKE-PRC-001: Monitor Volume Tiers.** Track ingestion volumes for VPC/DNS/EKS logs. Because prices drop at 10TB, 30TB, and 50TB ($0.25 -> $0.15 -> $0.075), consolidating organizational ingestion into a central Security Lake administrator account maximizes volume discounts.
*   **SECLAKE-PRC-002: Optimize S3 Storage Class Defaults.** If querying historical data is rare, evaluate moving data to S3 Standard-IA much earlier in the lifecycle, balancing the lower storage cost against retrieval fees during active incident response.

### 7. Network & Data Transfer Optimization
*   **SECLAKE-NET-001: Optimize Cross-Region Rollup Configurations.** When configuring Security Lake rollup regions, only roll up critical logs (like CloudTrail or GuardDuty) to the central SOC region. Leave high-volume logs (like VPC Flow Logs) in their local regions to avoid massive cross-region data transfer out (DTO) costs.
*   **SECLAKE-NET-002: Use VPC Endpoints for Third-Party SIEM Subscriptions.** Ensure that third-party SIEMs pulling data from Security Lake S3 buckets within AWS are using VPC Endpoints (AWS PrivateLink) to avoid routing traffic through NAT Gateways or the public internet.
*   **SECLAKE-NET-003: Avoid Cross-Region Athena Queries.** Instruct analysts to query Security Lake data from an Athena instance deployed in the *same region* as the target data to avoid cross-region data transfer fees incurred during query execution.
*   **SECLAKE-NET-004: Compress Custom Log Sources.** Ensure any custom log sources being transmitted to Security Lake over the network are heavily compressed (e.g., GZIP) before transit to reduce data transfer bandwidth costs.
---
## Cross-Service Synergies
*   **Amazon Athena:** Properly partitioned Security Lake S3 buckets drastically reduce Athena query costs.
*   **Amazon S3:** Lifecycle rules are critical to managing the long-term cost of the data lake created by Security Lake.
*   **Amazon CloudWatch/VPC Flow Logs:** Disabling flow logs at the source saves both CloudWatch processing costs and Security Lake ingestion costs.
*   **AWS Security Hub / Amazon GuardDuty:** Using Security Lake to centralize findings from these services provides a cheaper long-term storage medium than keeping them active in the native services' hot storage indefinitely.
---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
*   `line_item_usage_type` like `%SecurityLake%` or `%DataScannedInBytes%` (Athena).
*   `pricing_term` (to identify ingestion vs storage).
*   `line_item_resource_id` (to map costs to specific Security Lake subscriptions/regions).
### B. CloudWatch Metrics
*   S3 Bucket Size metrics (`BucketSizeBytes`, `NumberOfObjects`) for Security Lake buckets.
*   Athena `DataScannedInBytes` metrics for the Security Lake workgroup.
### C. AWS Config / Trusted Advisor
*   Configurations showing which log sources (CloudTrail, VPC Flow, Route53) are enabled in Security Lake per account.
*   S3 Bucket Lifecycle policy configurations on the Security Lake data buckets.
*   Security Lake Rollup Region configurations.
### D. Company Policies
*   Data retention requirements (e.g., 90 days vs 7 years) to guide S3 lifecycle policies.
*   Log collection mandates (e.g., are VPC Flow logs strictly required for dev environments?).
### E. IaC (Optional)
*   Terraform or CloudFormation scripts defining Security Lake integrations, S3 lifecycle rules, and Athena Workgroup query limits.
---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "SECLAKE-ARC-001",
  "category": "Architecture Changes",
  "service": "Amazon Security Lake",
  "title": "Missing S3 Lifecycle Rules on Security Lake Buckets",
  "description": "Security Lake S3 buckets in us-east-1 are holding 50TB of OCSF data older than 90 days in Standard storage.",
  "severity": "High",
  "estimated_monthly_savings_usd": 1100.00,
  "level_of_effort": "Low",
  "remediation_steps": [
    "Navigate to Amazon S3 console.",
    "Select the Security Lake target bucket.",
    "Create a Lifecycle Rule to transition objects to Glacier Deep Archive after 90 days."
  ]
}
```
### Summary Report Table
| Finding ID | Category | Title | Severity | Est. Savings | Effort |
|------------|----------|-------|----------|--------------|--------|
| SECLAKE-WST-002 | Waste Elimination | Disable VPC Flow Logs in Dev/Test | High | $$$ | Low |
| SECLAKE-ARC-001 | Architecture Changes | Implement S3 Lifecycle Rules | High | $$$ | Low |
| SECLAKE-ARC-002 | Architecture Changes | Enforce Athena Partition Pruning | High | $$ | Medium |
| SECLAKE-NET-001 | Network Optimization | Optimize Cross-Region Rollups | Medium | $$ | Low |
| SECLAKE-PRC-001 | Pricing Optimization | Monitor Volume Tiers | Low | $ | Low |
