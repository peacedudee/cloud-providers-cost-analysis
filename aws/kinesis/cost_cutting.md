# Cost-Cutting Playbook: Amazon Kinesis (Data Streams & Data Firehose)

> **Companion Files:** [kinesis_data_streams.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/kinesis/kinesis_data_streams.md) | [kinesis_data_firehose.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/kinesis/kinesis_data_firehose.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon Kinesis provides real-time data streaming (**Kinesis Data Streams**) and managed stream delivery (**Amazon Data Firehose**). Cost hotspots arise from two distinct billing traps:
1. **Firehose 5 KB Write Rounding Trap:** Firehose rounds the size of every individual record written to standard destinations up to the nearest **5 KB**. Writing 100-byte raw log records individually multiplies ingestion fees by **50x**!
2. **Kinesis Streams Capacity Mode Misalignment:** Using On-Demand Standard mode ($0.075/GB) for high-volume steady streams instead of Provisioned Shards ($0.015/shard-hr), or using On-Demand Standard instead of account-level **On-Demand Advantage** (~60% lower usage rates).

This playbook provides **18 actionable strategies** across six operational categories, delivering an estimated **30–80% reduction in total Kinesis spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Buffer and Aggregate Records Locally Before Firehose PUT (KPL / Log Agents):** Batch 100-byte logs into 5–10 KB payloads prior to Firehose write, bypassing the 5 KB minimum rounding tax to cut ingestion fees by up to **90%**.
2. **Enable On-Demand Advantage Mode for High Stream Count Accounts:** Eliminates per-stream hourly base fees ($32.85/mo per stream) and drops ingestion fees by ~60% ($0.032/GB vs $0.075/GB).
3. **Switch Steady High-Throughput Streams to Provisioned Shards:** Provisioned shards ($0.015/shard-hr) are up to **10x cheaper** than On-Demand Standard ($0.075/GB) when shard utilization exceeds 15–30%.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Eliminate Unused or Idle Kinesis Data Streams
- **What:** Identify and delete Kinesis Data Streams with zero write/read activity over 14 days.
- **Why It Saves Money:** On-Demand Standard streams bill **$0.045 per stream-hour ($32.85/month flat)** per stream. 20 idle developer streams waste **$657.00/month**.
- **Detailed Implementation Steps:**
  1. Query CloudWatch metric `IncomingBytes = 0` over 14 days across streams.
  2. Delete stream via AWS CLI:
     ```bash
     aws kinesis delete-stream --stream-name dev-test-stream
     ```
- **Estimated Savings:** 100% of idle stream base charges ($32.85/mo per stream).
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Stream activity audit.

#### 2. Reset Extended Stream Retention to 24 Hours (Free Baseline)
- **What:** Reduce data retention periods on Kinesis Data Streams back to the 24-hour baseline.
- **Why It Saves Money:** Extending retention from 24 hours to 7 days adds a surcharge of **$0.020 per shard-hour** ($14.60/month per shard). If downstream consumers (Lambda, Firehose, S3) process messages within minutes, 7-day retention is pure waste.
- **Detailed Implementation Steps:**
  1. Update retention period via CLI:
     ```bash
     aws kinesis decrease-stream-retention-period \
       --stream-name prod-log-stream \
       --retention-period-hours 24
     ```
- **Estimated Savings:** $14.60 per shard per month (100% of extended retention surcharges).
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Confirm downstream consumers process stream within 24 hours.

---

### 2. Payload Aggregation & Buffer Tuning (Firehose 5 KB Trap)

#### 3. Aggregate Small Records Locally Prior to Firehose PUT (Bypass 5 KB Rounding)
- **What:** Update application logging agents (FluentBit, Logstash, Kinesis Producer Library - KPL) to buffer and aggregate small records (e.g. 100-byte logs) into single **5 KB to 10 KB payloads** before writing to Firehose.
- **Why It Saves Money:** Firehose rounds every written record up to **5 KB**. Writing 1,000 individual 100-byte records bills as **5,000 KB (5 MB)** ($0.375). Aggregating 50 records into one 5 KB payload bills as **5 KB** ($0.0075) — a **90% direct ingestion fee reduction**!
- **Detailed Implementation Steps:**
  1. Update FluentBit configuration file to buffer and pack records:
     ```ini
     [OUTPUT]
         Name        kinesis_firehose
         Match       *
         region      us-east-1
         delivery_stream raw-logs-stream
         auto_retry_requests true
         compression gzip
     ```
  2. In KPL, set `AggregationEnabled = true` and `AggregationMaxBytes = 10240`.
- **Estimated Savings:** **70–90% reduction** in Firehose ingestion fees.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Log agent or producer SDK configuration update.

#### 4. Maximize Firehose Buffer Hints (128 MB & 900 Seconds)
- **What:** Configure Firehose Delivery Stream buffer hints to maximum limits: **128 MB buffer size** and **900 seconds (15 minutes) buffer interval**.
- **Why It Saves Money:** Packaging stream data into maximum buffer chunks allows Firehose to write large contiguous objects to S3, reducing total S3 `PUT` request fees ($0.005/1k) by up to 95%.
- **Detailed Implementation Steps:**
  1. Update delivery stream buffer hints:
     ```bash
     aws firehose update-destination \
       --delivery-stream-name prod-s3-stream \
       --current-delivery-stream-version-id 1 \
       --destination-id destination-id-1 \
       --extended-s3-destination-update "BufferIntervalInSeconds=900,BufferSizeInMBs=128"
     ```
- **Estimated Savings:** 80–95% reduction in destination S3 PUT request charges.
- **Risk Level:** Low (increases delivery latency to 15 minutes; ideal for analytical data lakes).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Target SLA tolerates 15-minute delivery latency.

---

### 3. Kinesis Streams Capacity Mode & Rightsizing

#### 5. Enable On-Demand Advantage Mode for High Stream Count Accounts
- **What:** Convert accounts with multiple serverless streams to **On-Demand Advantage Mode**.
- **Why It Saves Money:**
  - **Standard On-Demand:** $0.045/stream-hr ($32.85/mo per stream) + $0.075/GB ingestion.
  - **Advantage Mode:** **$0.00 per stream-hour** (zero base fee) + **$0.032/GB ingestion** (~60% lower usage rate).
  - An account with 50 streams saving $32.85/mo per stream saves **$1,642.50/month in base charges alone**!
- **Detailed Implementation Steps:**
  1. Enable On-Demand Advantage mode at the account level via AWS Console or CLI:
     ```bash
     aws kinesis update-account-configuration --account-configuration '{"StreamMode":"ADVANTAGE"}'
     ```
- **Estimated Savings:** 60% reduction in ingestion fees + 100% elimination of stream base fees.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team / DevOps
- **Prerequisites:** Account-level streaming volume evaluation.

#### 6. Switch Steady-State High-Volume Streams to Provisioned Shards
- **What:** Convert streams with steady, predictable throughput (> 15–30% capacity utilization) from On-Demand to **Provisioned Mode**.
- **Why It Saves Money:**
  - On-Demand Standard charges **$0.075 per GB** ingested. Processing 10 MB/sec steady traffic (25.9 TB/month) on On-Demand costs **$1,944.00/month**.
  - 10 Provisioned Shards (10 MB/sec write capacity) cost **$109.50/month** ($0.015/shard-hr) — a **94% cost reduction**!
- **Detailed Implementation Steps:**
  1. Check stream throughput stability in CloudWatch (`IncomingBytes`).
  2. If steady > 15% shard capacity, switch to Provisioned mode:
     ```bash
     aws kinesis update-stream-mode \
       --stream-arn arn:aws:kinesis:us-east-1:123456789012:stream/prod-telemetry \
       --stream-mode-details StreamMode=PROVISIONED
     ```
  3. Set target shard count: `aws kinesis update-shard-count --stream-name prod-telemetry --target-shard-count 10 --scaling-type UNIFORM_SCALING`.
- **Estimated Savings:** **70–94% savings** for steady high-throughput streams.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Baseline throughput stability verification.

#### 7. Enable Auto-Scaling on Provisioned Kinesis Streams
- **What:** Configure Application Auto Scaling on Provisioned Kinesis Data Streams.
- **Why It Saves Money:** Automatically scales shard counts up during peak ingestion hours and down during off-peak hours (e.g. scaling from 20 shards down to 2 shards overnight).
- **Detailed Implementation Steps:**
  1. Register stream as scalable target:
     ```bash
     aws application-autoscaling register-scalable-target \
       --service-namespace kinesis \
       --resource-id stream/prod-telemetry \
       --scalable-dimension kinesis:stream:TargetShardCount \
       --min-capacity 2 --max-capacity 20
     ```
- **Estimated Savings:** 30–50% reduction in provisioned shard-hour billing.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application Auto Scaling setup.

---

### 4. Architecture & Consumer Optimization

#### 8. Bypass Enhanced Fan-Out (EFO) for Standard Lambda/S3 Consumers
- **What:** Remove Enhanced Fan-Out (EFO) consumer registration on Lambda functions and Firehose delivery streams that can share standard HTTP/2 pull throughput.
- **Why It Saves Money:** EFO charges **$0.015 per consumer-shard hour** ($10.95/mo per shard per consumer) PLUS **$0.015 per GB** read. Registering 5 EFO consumers on a 20-shard stream adds **$1,095.00/month** in connection fees!
- **Detailed Implementation Steps:**
  1. List registered stream consumers:
     ```bash
     aws kinesis list-stream-consumers --stream-arn arn-xxx
     ```
  2. Deregister non-critical EFO consumers:
     ```bash
     aws kinesis deregister-stream-consumer --stream-arn arn-xxx --consumer-name efo-lambda-consumer
     ```
  3. Reconfigure Lambda Event Source Mapping to use standard polling.
- **Estimated Savings:** 100% of EFO consumer-shard fees ($10.95/mo per shard per consumer).
- **Risk Level:** Low (standard consumers share 2 MB/sec per shard throughput limit).
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Read throughput requirements < 2 MB/sec per shard.

#### 9. Evaluate Firehose Format Conversion (JSON to Parquet) Break-Even
- **What:** Audit Firehose Format Conversion (`$0.018/GB`) based on downstream target query engine.
- **Why It Saves Money:** Converting JSON to Parquet in Firehose adds $0.018/GB. If the destination is Amazon Athena, Parquet reduces data scanned by 90% ($5.00/TB scanned), saving far more in Athena fees than Firehose conversion costs. However, if data is simply archived in S3 without Athena querying, disable conversion to save $0.018/GB.
- **Detailed Implementation Steps:**
  1. Check Athena query frequency in Athena CUR.
  2. If data is unqueried archive, disable conversion in Firehose console.
- **Estimated Savings:** $0.018/GB saved on archive-only delivery streams.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Downstream query pattern audit.

---

### 5. Dynamic Partitioning & S3 Data Lake Tuning

#### 10. Restrict Firehose Dynamic Partitioning to Low-Cardinality Keys
- **What:** Restrict Firehose Dynamic Partitioning S3 path expressions to low-cardinality keys (e.g. `year=!{timestamp:yyyy}/month=!{timestamp:MM}/`).
- **Why It Saves Money:** Dynamic Partitioning adds **$0.020/GB**. Partitioning S3 files by high-cardinality keys (e.g. `customer_id` or `device_id`) forces Firehose to create thousands of concurrent S3 write streams, generating millions of tiny 10 KB S3 files that drive up S3 PUT request fees ($0.005/1k).
- **Detailed Implementation Steps:**
  1. Update Dynamic Partitioning S3 prefix expression in Terraform to date-based keys only.
- **Estimated Savings:** 50–80% reduction in S3 request charges and Athena query slowdowns.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** S3 path schema review.

#### 11. Deploy Interface Endpoints for Private VPC Firehose Delivery
- **What:** Configure PrivateLink Interface Endpoints for Firehose inside private VPCs.
- **Why It Saves Money:** Drops VPC delivery processing from NAT Gateway taxes ($0.045/GB) to PrivateLink rates ($0.01/GB).
- **Detailed Implementation Steps:**
  1. Create VPC Endpoint for `kinesis-firehose` service.
- **Estimated Savings:** **78% reduction** in VPC delivery processing fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Private subnet workloads.

---

### 6. Compression & Payload Optimization

#### 12. Enable Gzip / Snappy Compression on Producer Writers
- **What:** Configure Kinesis producers (FluentBit, Logstash, KPL) to compress payload records using Gzip or Snappy before transmitting over the wire.
- **Why It Saves Money:** Compresses payload bytes by 60–80%, directly dropping Kinesis Data Streams ingestion ($0.075/GB) and Firehose processing ($0.075/GB) fees.
- **Detailed Implementation Steps:**
  1. Enable compression in FluentBit output plugin (`compression gzip`).
- **Estimated Savings:** 60–80% reduction in total data volume ingestion billing.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Downstream consumer decompression support (Lambda/Firehose handles natively).

#### 13. Optimize Kinesis Client Library (KCL) Checkpoint Table Sizing
- **What:** Ensure KCL DynamoDB checkpoint tables run in On-Demand mode with small item sizes.
- **Why It Saves Money:** Prevents KCL workers from over-consuming DynamoDB write capacity units ($0.625/M WRUs).
- **Detailed Implementation Steps:**
  1. Update KCL configuration `withCallProcessRecordsEvenForEmptyRecordList(false)`.
- **Estimated Savings:** 20–40% reduction in auxiliary DynamoDB billing.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** KCL worker application update.

#### 14. Batch Lambda Invocations from Kinesis Streams (BatchSize = 500-1000)
- **What:** Configure Kinesis Lambda Event Source Mappings with large batch sizes (`BatchSize = 500` to `1000`) and a batching window (`MaximumBatchingWindowInSeconds = 10`).
- **Why It Saves Money:** Drastically reduces Lambda invocation count ($0.20/M requests) and decreases total execution duration by processing records in bulk.
- **Detailed Implementation Steps:**
  1. Update Event Source Mapping:
     ```bash
     aws lambda update-event-source-mapping \
       --uuid mapping-uuid-123 \
       --batch-size 500 \
       --maximum-batching-window-in-seconds 10
     ```
- **Estimated Savings:** 80–95% reduction in Lambda request charges.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Lambda code iterates payload array.

#### 15. Consolidate Duplicate Regional Streaming Ingestion Pipelines
- **What:** Consolidate multiple parallel stream channels serving identical data into a single shared Kinesis stream.
- **Why It Saves Money:** Eliminates redundant per-stream base hourly fees ($32.85/mo per stream).
- **Detailed Implementation Steps:**
  1. Merge telemetry streams into single topic with attribute routing.
- **Estimated Savings:** $32.85/mo saved per consolidated stream.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Stream topic consolidation.

#### 16. Monitor and Alarm on Kinesis Throttled Records
- **What:** Create CloudWatch alarm on `WriteProvisionedThroughputExceeded` metric.
- **Why It Saves Money:** Identifies misconfigured shard counts before application retries double data transmission fees.
- **Detailed Implementation Steps:**
  1. Put CloudWatch metric alarm on `WriteProvisionedThroughputExceeded`.
- **Estimated Savings:** Operational risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic configured.

#### 17. Use CloudWatch Logs Subscription Filters to Stream Direct to S3
- **What:** Use CloudWatch Logs Subscription Filters directly to S3 via Firehose without intermediate Kinesis Data Streams.
- **Why It Saves Money:** Eliminates Kinesis Data Streams ingestion fees ($0.075/GB) when streaming logs to S3.
- **Detailed Implementation Steps:**
  1. Point CloudWatch Log Group Subscription Filter directly to Firehose stream ARN.
- **Estimated Savings:** Eliminates intermediate Kinesis Data Stream charges.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch Logs setup.

#### 18. Leverage Free Account Allowances across Sandbox Accounts
- **What:** Audit small test streams in developer accounts.
- **Why It Saves Money:** Minimizes unnecessary sandbox stream costs.
- **Detailed Implementation Steps:**
  1. Audit stream usage via AWS Billing Console.
- **Estimated Savings:** Operational hygiene.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Account structure setup.

---

## Cross-Service Synergies

```
[ Application Log Agents ] 
        │
        ├──(Payload Aggregation)──> [ Amazon Data Firehose ] (Bypasses 5 KB minimum rounding tax)
        │
        ├──(Account Mode)─────────> [ On-Demand Advantage ] (Saves 60% ingestion + $0 stream base)
        │
        └──(Data Lake Target)─────> [ S3 + Parquet Conversion ] (Saves 90% on Athena query scan fees)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Kinesis-OnDemand-StreamHour`, `Kinesis-OnDemand-BytesIngested`, `Kinesis-ShardHour`, `Kinesis-PUT-TPU`, `Firehose-Ingested-Bytes`, `Firehose-FormatConversion-Bytes`.
- `line_item_resource_id`: Kinesis Stream ARN / Firehose Delivery Stream ARN.

### B. CloudWatch Metrics
- `AWS/Kinesis` Namespace: `IncomingBytes`, `IncomingRecords`, `WriteProvisionedThroughputExceeded`, `ReadProvisionedThroughputExceeded`.
- `AWS/KinesisFirehose` Namespace: `IncomingBytes`, `IncomingRecords`, `DeliveryToS3.Bytes`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "KIN-FH-001",
  "service": "Kinesis Firehose",
  "category": "Payload Aggregation & Buffer Tuning",
  "resource_id": "arn:aws:firehose:us-east-1:123456789012:deliverystream/app-logs-firehose",
  "resource_name": "app-logs-firehose",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "avg_record_size_bytes": 120,
    "firehose_5kb_rounding_applied": true,
    "monthly_ingested_gb": 4500,
    "effective_billed_gb": 225000,
    "monthly_cost_usd": 16875.00
  },
  "recommended_config": {
    "action": "Enable KPL / Log Agent 5KB Batch Aggregation",
    "projected_monthly_cost_usd": 337.50
  },
  "financial_impact": {
    "monthly_savings_usd": 16537.50,
    "annual_savings_usd": 198450.00,
    "savings_percentage": 98.0
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "FluentBit log agent aggregates 50 logs per 5KB payload before writing to Firehose; no message loss."
  },
  "implementation": {
    "scope": "Software Engineer / DevOps",
    "effort_estimate": "1-2 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Firehose 5 KB Batch Aggregation** | 6 | $24,500.00 | $22,050.00 | 90.0% | Low |
| **On-Demand Advantage Mode** | 12 | $14,200.00 | $8,520.00 | 60.0% | Low |
| **Provisioned Shard Migration** | 5 | $18,500.00 | $14,800.00 | 80.0% | Low |
| **Bypass EFO Consumers** | 8 | $5,400.00 | $4,320.00 | 80.0% | Low |
| **Total** | **31** | **$62,600.00** | **$49,690.00** | **79.3%** | -- |
