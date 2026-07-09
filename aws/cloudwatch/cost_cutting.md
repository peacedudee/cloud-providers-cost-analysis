# Cost-Cutting Playbook: Amazon CloudWatch

> **Companion File:** [cloudwatch.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/cloudwatch/cloudwatch.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon CloudWatch is an enterprise monitoring, logging, and observability service billed on a multi-dimensional pay-as-you-go model: Custom Metrics ($0.30/metric-mo baseline), CloudWatch Logs Ingestion ($0.50/GB Standard vs $0.25/GB Infrequent Access), Log Storage ($0.03/GB-mo), Logs Insights queries ($0.005/GB scanned), Alarms ($0.10–$0.30/mo), and Dashboards ($3.00/mo).

Because default log group retention is set to **"Never Expire"** and custom metric dimensions can experience high-cardinality explosions, CloudWatch is a frequent source of runaway monthly billing spikes.

This playbook provides **18 actionable strategies** across six operational categories, delivering an estimated **40–80% reduction in total CloudWatch spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Set Explicit Retention Limits (7/14/30 Days) on ALL Log Groups:** Stops permanent log storage accumulation ($0.03/GB-mo).
2. **Switch Build & Audit Log Groups to Infrequent Access (IA) Class:** Instant **50% price reduction** on log ingestion ($0.25/GB vs $0.50/GB).
3. **Fix High-Cardinality Custom Metric Dimensions:** Removes dynamic values (User ID, IP address) from `PutMetricData`, preventing multi-thousand dollar custom metric explosions.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Enforce Explicit Retention Limits on ALL CloudWatch Log Groups
- **What:** Update all log groups set to default **"Never Expire"** retention to explicit limits (e.g. 7 days for dev/test, 30 days for production).
- **Why It Saves Money:** Un-configured log groups accumulate historical application logs forever, billing **$0.03 per GB-month** indefinitely. Retaining 50 TB of stale logs costs $1,500/month in dead storage.
- **Implementation Steps:**
  1. Find log groups with no retention: `aws logs describe-log-groups --query "logGroups[?retentionInDays==null].logGroupName"`.
  2. Apply retention policy: `aws logs put-retention-policy --log-group-name name --retention-in-days 14`.
  3. Deploy AWS Systems Manager Quick Setup to enforce default retention on all newly created log groups automatically.
- **Estimated Savings:** 70–90% reduction in CloudWatch log storage bills.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compliance log retention audit.

#### 2. Audit and Delete Orphaned Custom Alarms and Dashboards
- **What:** Identify and delete CloudWatch Alarms monitoring terminated EC2 instances or deleted Lambda functions, and remove unused custom dashboards.
- **Why It Saves Money:** Standard alarms cost **$0.10/month**, High-Resolution alarms cost **$0.30/month**, Composite alarms cost **$0.50/month**, and custom dashboards cost **$3.00/month**. 500 orphaned alarms cost $50–$150/month.
- **Implementation Steps:**
  1. List alarms in `INSUFFICIENT_DATA` state: `aws cloudwatch describe-alarms --state-value INSUFFICIENT_DATA`.
  2. Delete orphaned alarms and unused dashboards.
- **Estimated Savings:** 100% of orphaned alarm and dashboard fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Alarm state audit.

---

### 2. Rightsizing & Log Class Optimization

#### 3. Switch Non-Critical Log Groups to Infrequent Access (IA) Log Class
- **What:** Change log group class from `Standard` to `Infrequent Access` for build logs, debug logs, audit trails, and staging environments.
- **Why It Saves Money:** Infrequent Access log class costs **$0.25 per GB** ingested compared to Standard at **$0.50 per GB** — an immediate **50% direct price reduction**.
- **Implementation Steps:**
  1. Modify log group class via AWS CLI or Terraform: `aws logs create-log-group --log-group-name name --log-group-class INFREQUENT_ACCESS`.
- **Estimated Savings:** **50% direct savings** on log ingestion.
- **Risk Level:** Low (IA class excludes metric filters, anomaly detection, and subscription filters, but supports Logs Insights queries).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Confirm log group does not require metric filters or subscription streams.

#### 4. Filter High-Velocity Application Logs at the Source (FluentBit / Log Agent)
- **What:** Update logging agents (FluentBit, CloudWatch Agent) to drop debug statements, 200 OK HTTP access logs, and health check pings (`/healthz`) before sending payload to CloudWatch.
- **Why It Saves Money:** Prevents paying $0.50/GB ingestion fee for useless noise logs. Dropping 100 GB/day of health check logs saves **$1,500.00/month**.
- **Implementation Steps:**
  1. Add regex filter in FluentBit config to exclude `/healthz` and `HTTP 200` lines.
  2. Set production application log levels to `INFO` or `WARN` instead of `DEBUG`.
- **Estimated Savings:** 40–80% reduction in total log ingestion volume.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Logging agent access.

---

### 3. Metric & Dimension Optimization

#### 5. Eliminate High-Cardinality Custom Metric Dimensions
- **What:** Audit custom metric publishing (`PutMetricData` API calls) to ensure dynamic, high-cardinality values (e.g. `User_ID`, `IP_Address`, `Session_ID`, UUIDs) are NOT included as metric dimensions.
- **Why It Saves Money:** CloudWatch bills **$0.30 per unique metric-dimension combination**. Publishing a metric with 10,000 unique User IDs creates 10,000 custom metrics, generating a sudden **$3,000.00/month bill!**
- **Implementation Steps:**
  1. Audit metric names and dimensions via `aws cloudwatch list-metrics`.
  2. Embed high-cardinality metadata in structured JSON log events instead, querying them via Logs Insights when needed.
- **Estimated Savings:** Prevents multi-thousand dollar custom metric billing explosions.
- **Risk Level:** Low to Medium.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Application code metric audit.

#### 6. Optimize Custom Metric Volume Tiers
- **What:** Aggregate custom metric publishing calls to use high-volume metric namespaces across the organization to trigger volume tier discounts.
- **Why It Saves Money:** Custom metric rates drop automatically with volume:
  - First 10,000 metrics: **$0.30/metric-mo**
  - Next 240,000 metrics: **$0.10/metric-mo**
  - Next 750,000 metrics: **$0.05/metric-mo**
  - Over 1,000,000 metrics: **$0.02/metric-mo** (93% discount!).
- **Implementation Steps:**
  1. Consolidate custom metrics into single AWS accounts or shared namespaces.
- **Estimated Savings:** Up to 93% on high-volume metric spend.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team / DevOps
- **Prerequisites:** Metric consolidation strategy.

#### 7. Avoid High-Resolution Custom Metrics (1-Second Resolution) Unless Mandatory
- **What:** Use Standard Resolution (60-second) custom metrics instead of High-Resolution (1-second) metrics.
- **Why It Saves Money:** High-Resolution metrics are billed as 5 custom metrics (**$1.50/metric-month**) compared to Standard Resolution at **$0.30/metric-month**.
- **Implementation Steps:**
  1. Pass `StorageResolution = 60` in `PutMetricData` API calls.
- **Estimated Savings:** **80% savings** per custom metric ($1.20/mo saved per metric).
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Verification that 1-second monitoring is not required.

---

### 4. Query & Analytics Optimization

#### 8. Restrict CloudWatch Logs Insights Queries by Time Window and Log Group
- **What:** Train engineers to select explicit log groups and narrow query timeframes when running CloudWatch Logs Insights queries.
- **Why It Saves Money:** Logs Insights charges **$0.005 per GB of uncompressed data scanned**. Running wildcard queries (`*`) over 30 days across a 10 TB log group costs **$50.00 per single query run**.
- **Implementation Steps:**
  1. Select specific log groups instead of "All Log Groups".
  2. Limit query time range to 1 hour or 1 day.
- **Estimated Savings:** 80–98% reduction in Logs Insights query fees.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Team training.

#### 9. Export Long-Term Compliance Logs to S3 Standard or Glacier
- **What:** Configure CloudWatch Logs Subscription Filters or S3 Export Tasks to stream long-term compliance logs to **Amazon S3** ($0.023/GB-mo) or **S3 Glacier** ($0.004/GB-mo).
- **Why It Saves Money:** Storing logs in S3 Standard is 23% cheaper than CloudWatch log storage ($0.03/GB-mo). Storing in S3 Glacier is **86% cheaper**.
- **Implementation Steps:**
  1. Create Kinesis Data Firehose subscription filter streaming logs to S3.
  2. Set CloudWatch Log Group retention to 7 days.
- **Estimated Savings:** 86% savings on long-term log storage.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** S3 bucket and Kinesis Firehose setup.

---

### 5. Architectural & Event Optimization

#### 10. Replace Custom Metric Polling with Embedded Metric Format (EMF)
- **What:** Publish application metrics using **CloudWatch Embedded Metric Format (EMF)** inside structured JSON logs instead of calling the `PutMetricData` API.
- **Why It Saves Money:** EMF automatically extracts custom metrics from log streams without incurring `PutMetricData` API request charges ($0.01 per 1,000 calls) and allows high-cardinality metadata logging.
- **Implementation Steps:**
  1. Use AWS EMF SDK libraries to output JSON log lines matching `_aws.CloudWatchMetrics` specification.
- **Estimated Savings:** 100% of `PutMetricData` API call costs.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** EMF SDK integration.

#### 11. Consolidate Standard Resolution Alarms (Use 5-Minute Evaluation)
- **What:** Set non-critical alarms (e.g. disk space warning) to 5-minute evaluation periods instead of 1-minute.
- **Why It Saves Money:** Reduces alarm evaluation processing overhead and metric data points queried.
- **Implementation Steps:**
  1. Set `Period = 300` in alarm configuration.
- **Estimated Savings:** Operational efficiency.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SLA review.

#### 12. Evaluate Amazon Managed Prometheus / Grafana for Large Container Clusters
- **What:** For EKS/ECS clusters generating > 50,000 custom metrics, deploy **Amazon Managed Service for Prometheus (AMP)** and **Amazon Managed Grafana (AMG)** instead of CloudWatch Custom Metrics.
- **Why It Saves Money:** AMP metrics cost **$0.09 per 10 million samples ingested**, which is vastly cheaper than CloudWatch custom metrics ($0.30 per metric-month) for high-density container environments.
- **Implementation Steps:**
  1. Deploy Prometheus agent (ADOT collector) in EKS/ECS cluster.
  2. Stream metrics to AMP workspace.
- **Estimated Savings:** 50–80% reduction in observability costs for large container fleets.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AMP/AMG workspace setup.

---

### 6. Pricing Model & Dashboard Optimization

#### 13. Maximize Always-Free Tier Observability Allowances
- **What:** Consolidate development monitoring into accounts to fully consume the **Indefinite Always-Free Tier**:
  - 10 custom metrics and 10 alarms
  - **5 GB of log ingestion** and 5 GB of log scanning
  - 3 custom dashboards
- **Why It Saves Money:** Always-Free Tier applies account-wide per region indefinitely.
- **Implementation Steps:**
  1. Audit free tier utilization in AWS Billing Console.
- **Estimated Savings:** Free baseline monitoring for small accounts.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

#### 14. Prune Unused Custom Dashboards
- **What:** Delete custom CloudWatch Dashboards that have not been viewed in 30 days beyond the 3 free dashboards.
- **Why It Saves Money:** Custom dashboards bill **$3.00 per dashboard per month**. Deleting 20 unused dashboards saves $60/month ($720/year).
- **Implementation Steps:**
  1. List dashboards: `aws cloudwatch list-dashboards`.
  2. Delete stale dashboards: `aws cloudwatch delete-dashboards --dashboard-names name`.
- **Estimated Savings:** $3.00/month saved per deleted dashboard.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Dashboard usage check.

#### 15. Consolidate Composite Alarms
- **What:** Replace individual redundant alarms with Composite Alarms to reduce total alarm count.
- **Why It Saves Money:** Combines multiple metric conditions into a single composite alarm ($0.50/mo), suppressing duplicate alert notifications.
- **Implementation Steps:**
  1. Create Composite Alarm evaluating `ALARM("cpu_high") AND ALARM("memory_high")`.
- **Estimated Savings:** 20–40% alarm management cost reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Composite alarm rule definition.

#### 16. Disable Unnecessary Synthetic Canaries
- **What:** Review CloudWatch Synthetics Canaries; pause or delete non-critical URL ping canaries in staging environments.
- **Why It Saves Money:** Synthetics canaries bill per run ($0.0012/run). A canary running every 1 minute costs **$52.56/month**.
- **Implementation Steps:**
  1. Stop non-prod canaries: `aws synthetictests stop-canary --name canary-name`.
- **Estimated Savings:** $52.56/month saved per stopped 1-minute canary.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod monitoring review.

#### 17. Use CloudWatch RUM (Real User Monitoring) Event Sampling
- **What:** Configure sampling rates on CloudWatch RUM web client snippets (e.g. sample 10% of sessions instead of 100%).
- **Why It Saves Money:** CloudWatch RUM bills **$1.00 per 100,000 RUM events**. High-traffic web apps sampling 100% of sessions generate massive event volumes.
- **Implementation Steps:**
  1. Update RUM web client config: `sessionSampleRate: 0.1`.
- **Estimated Savings:** 90% reduction in RUM event billing.
- **Risk Level:** Low.
- **Implementation Scope:** Front-End Engineer
- **Prerequisites:** RUM client snippet update.

#### 18. Consolidate Cross-Account CloudWatch Observability
- **What:** Enable CloudWatch Cross-Account Observability to monitor multi-account AWS Organizations from a central monitoring account.
- **Why It Saves Money:** Eliminates duplicate dashboard creation ($3/mo each) and centralizes alarm management.
- **Implementation Steps:**
  1. Configure Monitoring Account link and Source Account links.
- **Estimated Savings:** 30–50% dashboard cost reduction across Organizations.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS Organizations setup.

---

## Cross-Service Synergies

```
[ CloudWatch Service ] 
        │
        ├──(Log Ingestion Class)──> [ Infrequent Access Class ] (Saves 50% on log ingestion $0.25/GB)
        │
        ├──(Long-Term Storage)────> [ S3 Glacier ] (Saves 86% vs CloudWatch log storage $0.03/GB)
        │
        └──(Metric Publishing)────> [ Embedded Metric Format (EMF) ] (Eliminates PutMetricData API fees)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `TimedStorage-ByteHrs`, `DataProcessing-Bytes`, `InfrequentAccess-Bytes`, `CustomThreshold-Alarm`, `Dashboards-Month`, `PutMetricData-Requests`.
- `line_item_resource_id`: Log Group Name / Metric Namespace / Alarm Name.

### B. CloudWatch Metrics & Diagnostic API
- `AWS/Logs` Namespace: `IncomingBytes`, `IncomingLogEvents`.
- `AWS/CloudWatch` API: `list-metrics`, `describe-alarms`, `describe-log-groups`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "CW-LOG-001",
  "service": "CloudWatch",
  "category": "Waste Elimination",
  "resource_id": "arn:aws:logs:us-east-1:123456789012:log-group:/aws/ecs/production-application-*",
  "resource_name": "/aws/ecs/production-application-*",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "log_group_class": "STANDARD",
    "retention_setting": "Never Expire",
    "stored_bytes_gb": 4500,
    "monthly_ingestion_gb": 1200,
    "monthly_cost_usd": 735.00
  },
  "recommended_config": {
    "log_group_class": "INFREQUENT_ACCESS",
    "retention_setting": "14 Days",
    "projected_monthly_cost_usd": 300.00
  },
  "financial_impact": {
    "monthly_savings_usd": 435.00,
    "annual_savings_usd": 5220.00,
    "savings_percentage": 59.2
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "14-day retention satisfies operational troubleshooting; IA class supports Logs Insights queries."
  },
  "implementation": {
    "scope": "Engineer/DevOps",
    "effort_estimate": "30 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Waste Elimination (Log Retention)** | 45 | $6,200.00 | $4,960.00 | 80.0% | Low |
| **Log Class (Infrequent Access)** | 28 | $8,400.00 | $4,200.00 | 50.0% | Low |
| **Log Filtering (FluentBit)** | 15 | $5,100.00 | $3,060.00 | 60.0% | Low |
| **Metric Cardinality Fixes** | 4 | $4,500.00 | $4,050.00 | 90.0% | Low |
| **Total** | **92** | **$24,200.00** | **$16,270.00** | **67.2%** | -- |
