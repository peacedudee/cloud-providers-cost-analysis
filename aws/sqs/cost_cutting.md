# Cost-Cutting Playbook: Amazon SQS (Simple Queue Service)

> **Companion File:** [sqs.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/sqs/sqs.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon SQS is a serverless message queuing service supporting **Standard Queues** ($0.40/M requests) and **FIFO Queues** ($0.50/M requests). SQS offers a perpetual free tier of **1 Million requests/month**.

Major billing leaks stem from:
1. **Short Polling Empty Queue Burn:** Polling worker threads calling `ReceiveMessage` with `WaitTimeSeconds = 0` on empty queues, generating millions of empty read calls ($200+/mo per worker pool for zero active messages).
2. **Individual vs Batch API Operations:** Calling `SendMessage` or `DeleteMessage` 10 times individually instead of using `SendMessageBatch` or `DeleteMessageBatch` (10x API billing multiplier).
3. **Payload Size Chunking:** Sending uncompressed messages > 64 KB, where every 64 KB block bills as 1 additional API request (up to 4 requests per 256 KB message).

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **40–95% reduction in total SQS spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Enable SQS Long Polling (`WaitTimeSeconds = 20`):** Reduces empty `ReceiveMessage` API calls by up to **98%**, saving $200+/mo per worker pool.
2. **Implement Batch API Operations (`SendMessageBatch` / `DeleteMessageBatch`):** Groups up to 10 messages per API call, cutting request volume and cost by **90%**.
3. **Compress Payloads > 64 KB (Gzip / Snappy):** Keeps message payloads under the 64 KB billing boundary (1 request instead of 2–4 requests per message).

---

## Strategy Categories

### 1. Polling Optimization & Empty Queue Burn Avoidance

#### 1. Enable SQS Long Polling (`WaitTimeSeconds = 20`) on All Queues
- **What:** Configure `ReceiveMessageWaitTimeSeconds = 20` on all SQS queues and in application worker `ReceiveMessage` calls.
- **Why It Saves Money:**
  - **Short Polling (`WaitTimeSeconds = 0`):** SQS immediately returns an empty response if no messages are present. 10 worker threads polling 20 times/sec generate 518 Million requests/month (**$207.36/month spent polling empty queues**).
  - **Long Polling (`WaitTimeSeconds = 20`):** SQS waits up to 20 seconds for a message to arrive before responding. When a queue is empty, request count drops by **98%+**, saving over $200/month per queue!
- **Detailed Implementation Steps:**
  1. Update queue attribute via AWS CLI:
     ```bash
     aws sqs set-queue-attributes \
       --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/prod-orders-queue \
       --attributes ReceiveMessageWaitTimeSeconds=20
     ```
  2. Set long polling in Terraform:
     ```hcl
     resource "aws_sqs_queue" "orders_queue" {
       name                        = "prod-orders-queue"
       receive_wait_time_seconds  = 20
     }
     ```
  3. Specify `WaitTimeSeconds=20` in application SDK consumer loops (Boto3 / Java SDK).
- **Estimated Savings:** **90–98% reduction** in empty read API request charges ($200+/mo saved per queue).
- **Risk Level:** Zero risk (reduces API calls while actually improving message delivery latency).
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Consumer worker SDK update.

#### 2. Terminate Workers on Idle Queues (Scale-to-Zero Consumers)
- **What:** Configure consumer worker auto-scaling (KEDA, Kubernetes HPA, or AWS Auto Scaling) to scale consumer pods/instances to **0** when `ApproximateNumberOfMessagesVisible = 0`.
- **Why It Saves Money:** Stops background polling loops completely during low-traffic periods (e.g. nights and weekends).
- **Detailed Implementation Steps:**
  1. Configure KEDA ScaledObject for SQS queue with `minReplicaCount = 0`.
- **Estimated Savings:** 100% of off-hours polling fees and consumer compute fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** KEDA or custom scaler integration.

---

### 2. API Batching & Payload Chunking Optimization

#### 3. Use Batch API Operations (`SendMessageBatch` & `DeleteMessageBatch`)
- **What:** Refactor producer and consumer applications to use `SendMessageBatch` and `DeleteMessageBatch` (packaging up to 10 messages per single API request).
- **Why It Saves Money:** Sending 10 messages individually costs 10 SQS requests ($0.000004). Sending 10 messages in 1 batch costs 1 SQS request ($0.0000004) — an instant **90% request fee discount**!
- **Detailed Implementation Steps:**
  1. Implement batch publishing in Python Boto3:
     ```python
     entries = [
         {'Id': str(i), 'MessageBody': json.dumps(msg)}
         for i, msg in enumerate(message_list)
     ]
     response = sqs_client.send_message_batch(
         QueueUrl=QUEUE_URL,
         Entries=entries
     )
     ```
  2. Implement batch deletion in consumer worker loops.
- **Estimated Savings:** **90% reduction** in publish and delete API request fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Producer/consumer batch array support.

#### 4. Compress Large Message Payloads (> 64 KB)
- **What:** Apply Gzip or Snappy compression to message payloads in producer applications before calling `SendMessage`.
- **Why It Saves Money:** SQS meters payload size in 64 KB chunks. A 100 KB uncompressed JSON message bills as 2 requests; a 200 KB message bills as 4 requests. Compressing 200 KB down to 30 KB bills as **1 request (75% savings)**.
- **Detailed Implementation Steps:**
  1. Compress payload in backend before dispatching:
     ```python
     import zlib, base64

     compressed_body = base64.b64encode(zlib.compress(json.dumps(payload).encode('utf-8'))).decode('ascii')
     sqs_client.send_message(QueueUrl=QUEUE_URL, MessageBody=compressed_body)
     ```
- **Estimated Savings:** 50–75% request count reduction for large message payloads.
- **Risk Level:** Zero risk (consumer decompresses payload transparently).
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Consumer decompression helper module.

#### 5. Offload Payloads > 256 KB via S3 Extended Client
- **What:** Use the **Amazon S3 Extended Client Library for Java / Python** for messages exceeding the 256 KB maximum SQS payload limit.
- **Why It Saves Money:** Automatically stores the message payload in S3 and passes an S3 pointer in SQS, avoiding custom multi-chunk queue splits.
- **Detailed Implementation Steps:**
  1. Configure `AmazonSQSExtendedClient` with S3 payload bucket.
- **Estimated Savings:** Avoids complex multi-queue payload splitting architectures.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** S3 payload bucket deployed.

---

### 3. Queue Type & Architecture Selection

#### 6. Prefer Standard Queues Over FIFO Queues (Unless Strict Ordering Mandatory)
- **What:** Use **Standard Queues** ($0.40/M requests) instead of **FIFO Queues** ($0.50/M requests) for workloads where strict FIFO ordering and deduplication can be handled at the application layer.
- **Why It Saves Money:** Standard queues are **20% cheaper** per request ($0.40/M vs $0.50/M) and support unlimited request throughput without batching limits.
- **Detailed Implementation Steps:**
  1. Review queue requirements; deploy Standard queue type in Terraform.
- **Estimated Savings:** **20% direct price discount**.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / Architect
- **Prerequisites:** Application tolerates out-of-order delivery.

#### 7. Increase Message Visibility Timeout (`VisibilityTimeout`)
- **What:** Set `VisibilityTimeout` to 6x the average consumer processing duration (e.g. 180 seconds for a 30-second task).
- **Why It Saves Money:** Prevents SQS from prematurely releasing in-flight messages back to the queue while a worker is still processing, avoiding duplicate processing and duplicate `DeleteMessage` API calls.
- **Detailed Implementation Steps:**
  1. Update queue attribute: `aws sqs set-queue-attributes --queue-url URL --attributes VisibilityTimeout=180`.
- **Estimated Savings:** Prevents 50%+ redundant message reprocessing billing.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Task duration benchmark.

---

### 4. Dead-Letter Queue (DLQ) & Error Handling

#### 8. Configure Dead-Letter Queues (DLQ) to Stop Poison Pill Loops
- **What:** Attach a **Dead-Letter Queue (DLQ)** with `maxReceiveCount = 3` to all primary SQS queues.
- **Why It Saves Money:** Prevents a malformed ("poison pill") message from failing continuously 1,000s of times, generating infinite read, error, and visibility timeout API calls.
- **Detailed Implementation Steps:**
  1. Attach DLQ in Terraform:
     ```hcl
     resource "aws_sqs_queue" "dlq" {
       name = "orders-queue-dlq"
     }

     resource "aws_sqs_queue" "main" {
       name = "orders-queue"
       redrive_policy = jsonencode({
         deadLetterTargetArn = aws_sqs_queue.dlq.arn
         maxReceiveCount     = 3
       })
     }
     ```
- **Estimated Savings:** Prevents infinite loop billing overruns.
- **Risk Level:** Zero risk (standard resilient queuing pattern).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** DLQ creation.

---

### 5. VPC Endpoints & Network Routing

#### 9. Deploy PrivateLink Interface VPC Endpoints for Private VPC Subnets
- **What:** Configure Interface VPC Endpoints (`com.amazonaws.us-east-1.sqs`) for private VPC subnets communicating with SQS.
- **Why It Saves Money:** Prevents SQS API traffic from routing through NAT Gateways ($0.045/GB processing tax). PrivateLink processes data at **$0.01/GB** (78% savings).
- **Detailed Implementation Steps:**
  1. Create VPC Endpoint for `sqs` in Terraform.
- **Estimated Savings:** **78% reduction** in network processing fees for high-volume message traffic.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Network Engineer / DevOps
- **Prerequisites:** Private subnet workloads.

---

### 6. Observability & Governance

#### 10. Maximize Perpetual Free Tier (1 Million Requests/Mo)
- **What:** Audit small queues in developer sandbox accounts to stay under the 1,000,000 free monthly request allowance.
- **Why It Saves Money:** Delivers $0.00 SQS bills for small projects.
- **Detailed Implementation Steps:**
  1. Monitor free tier request counters in AWS Billing Console.
- **Estimated Savings:** Baseline free testing.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

#### 11. Delete Unused / Abandoned SQS Queues
- **What:** Identify and delete queues with 0 messages sent/received over 30 days.
- **Why It Saves Money:** Reclaims administrative hygiene.
- **Detailed Implementation Steps:**
  1. Delete queue: `aws sqs delete-queue --queue-url URL`.
- **Estimated Savings:** Administrative cleanliness.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** 30-day CloudWatch audit.

#### 12. Enforce CloudWatch Alarms on Queue Depth & Empty Receive Counts
- **What:** Put CloudWatch alarm on `NumberOfEmptyReceives`.
- **Why It Saves Money:** Instant alert on short-polling worker leaks.
- **Detailed Implementation Steps:**
  1. Create alarm on `AWS/SQS` `NumberOfEmptyReceives`.
- **Estimated Savings:** Proactive billing protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 13. Optimize Lambda Event Source Mapping Batching Window
- **What:** Set `MaximumBatchingWindowInSeconds = 10` on SQS-to-Lambda Event Source Mappings.
- **Why It Saves Money:** Allows Lambda to process full 10-message batches, cutting Lambda invocation request costs by 90%.
- **Detailed Implementation Steps:**
  1. Update event source mapping batching window via CLI.
- **Estimated Savings:** 90% Lambda invocation cost reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Lambda event source mapping.

#### 14. Purge Stale Messages in Test Queues
- **What:** Execute `aws sqs purge-queue` on non-prod test queues before test runs.
- **Why It Saves Money:** Prevents processing outdated test messages.
- **Detailed Implementation Steps:**
  1. Purge queue via CLI.
- **Estimated Savings:** Operational efficiency.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Test queue confirmation.

#### 15. Enforce Message Retention Period Rules
- **What:** Set `MessageRetentionPeriod = 86400` (1 day) on non-prod queues.
- **Why It Saves Money:** Automatically expires stale test messages.
- **Detailed Implementation Steps:**
  1. Set queue retention attribute to 86400 seconds.
- **Estimated Savings:** Storage and processing optimization.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod retention policy.

#### 16. Implement IAM Least Privilege Queue Policies
- **What:** Restrict `sqs:SendMessage` and `sqs:ReceiveMessage` permissions via IAM.
- **Why It Saves Money:** Prevents unauthorized external services from publishing to your queues.
- **Detailed Implementation Steps:**
  1. Restrict queue policy principals.
- **Estimated Savings:** Security billing protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** IAM policy setup.

---

## Cross-Service Synergies

```
[ Producer Application ] 
        │
        ├──(Batching & Long Polling)─> [ SQS Long Polling (WaitTime=20) ] (98% reduction in empty read API calls)
        │
        ├──(Payload Compression)────> [ Compressed Payloads (< 64 KB) ] (75% reduction in request chunking)
        │
        └──(Network Routing)────────> [ PrivateLink VPC Endpoint ] (78% discount vs NAT processing)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Requests-Tier1` (Standard), `Requests-FIFO`.
- `line_item_resource_id`: SQS Queue ARN (`arn:aws:sqs:us-east-1:123456789012:prod-orders-queue`).

### B. CloudWatch Metrics
- `AWS/SQS` Namespace: `NumberOfMessagesSent`, `NumberOfMessagesReceived`, `NumberOfEmptyReceives`, `ApproximateNumberOfMessagesVisible`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "SQS-LNG-001",
  "service": "SQS",
  "category": "Polling Optimization & Empty Queue Burn",
  "resource_id": "arn:aws:sqs:us-east-1:123456789012:dev-worker-queue",
  "resource_name": "dev-worker-queue",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "receive_wait_time_seconds": 0,
    "monthly_empty_receives_millions": 480.0,
    "monthly_cost_usd": 192.00
  },
  "recommended_config": {
    "receive_wait_time_seconds": 20,
    "projected_monthly_empty_receives_millions": 5.0,
    "projected_monthly_cost_usd": 2.00
  },
  "financial_impact": {
    "monthly_savings_usd": 190.00,
    "annual_savings_usd": 2280.00,
    "savings_percentage": 98.9
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Long polling wait time of 20 seconds is standard SQS best practice; improves latency."
  },
  "implementation": {
    "scope": "Software Engineer / DevOps",
    "effort_estimate": "15 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Long Polling (`WaitTimeSeconds=20`)** | 15 | $3,200.00 | $3,136.00 | 98.0% | Zero |
| **Batch API Operations (Batch 10)** | 12 | $4,500.00 | $4,050.00 | 90.0% | Zero |
| **Payload Compression (< 64 KB)** | 8 | $2,400.00 | $1,800.00 | 75.0% | Zero |
| **Dead-Letter Queue Safeguards** | 10 | $1,800.00 | $1,620.00 | 90.0% | Zero |
| **Total** | **45** | **$11,900.00** | **$10,606.00** | **89.1%** | -- |
