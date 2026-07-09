# Cost-Cutting Playbook: Amazon EventBridge

> **Companion File:** [eventbridge.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/eventbridge/eventbridge.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon EventBridge provides serverless event bus routing, EventBridge Scheduler, EventBridge Pipes, and Schema Registry. Billing includes **Custom / Partner SaaS Events Ingested** ($1.00/M events), **EventBridge Scheduler** ($1.25/M calls; 14 Million Free calls/mo), and **EventBridge Pipes** ($0.40/M requests). AWS Service Events (S3, EC2, GuardDuty) are **100% FREE ($0.00)**.

Key billing traps include:
1. **The 64 KB Event Chunking Trap:** Emitting large JSON event payloads (> 64 KB) on custom buses, where every 64 KB chunk counts as 1 additional event ($1.00/M per chunk).
2. **Filtering Events Inside Target Code Instead of Rules:** Routing raw events to Lambda targets and executing Python `if` statements, incurring unnecessary Lambda execution fees ($0.20/M + duration) when EventBridge filters rules for free.
3. **Un-capped Event Archives:** Retaining full event archives in EventBridge indefinitely ($0.023/GB-mo).

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **35–85% reduction in total EventBridge spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Leverage 14 Million Free Monthly EventBridge Scheduler Invocations:** Replaces third-party cron services with free native scheduled triggers ($0.00 cost).
2. **Filter Events at the Event Bus Rule Layer (Event Patterns):** Prevents non-matching events from triggering target Lambda functions, cutting downstream compute spend by **80–95%**.
3. **Keep Event Payloads < 64 KB (S3 Pointer Pattern):** Store large payloads in S3 and pass S3 URL keys in event detail blocks to avoid 64 KB event chunking multipliers.

---

## Strategy Categories

### 1. Event Payload & Chunking Optimization

#### 1. Avoid Large Event Payloads (Use the S3 Pointer Pattern)
- **What:** Refactor custom event publishers to store large JSON or binary payloads (> 64 KB) in **Amazon S3** and pass the **S3 Object Key** in the event detail block.
- **Why It Saves Money:** EventBridge meters custom events in **64 KB chunks**. Emitting a 200 KB event consumes **4 event units** ($4.00 per Million events instead of $1.00). Passing an S3 pointer keeps the event size under 64 KB (1 event unit = **75% savings**).
- **Detailed Implementation Steps:**
  1. Store payload in S3 before publishing event:
     ```python
     import boto3, json

     s3 = boto3.client('s3')
     eb = boto3.client('events')

     def publish_event(large_payload):
         key = f"events/{uuid.uuid4()}.json"
         s3.put_object(Bucket="event-payloads-bucket", Key=key, Body=json.dumps(large_payload))
         
         eb.put_events(Entries=[{
             'Source': 'my.application',
             'DetailType': 'OrderCreated',
             'Detail': json.dumps({'s3_bucket': 'event-payloads-bucket', 's3_key': key}),
             'EventBusName': 'custom-bus'
         }])
     ```
- **Estimated Savings:** **50–75% reduction** in custom event ingestion fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** S3 payload bucket deployed.

---

### 2. Rule Filtering & Target Downstream Optimization

#### 2. Filter Events at the Event Bus Rule Layer (Event Patterns)
- **What:** Define explicit **Event Patterns** (`source`, `detail-type`, `detail.status`) on EventBridge Rules rather than forwarding all raw events to downstream target Lambda functions or SQS queues.
- **Why It Saves Money:** EventBridge evaluates event pattern rules at the event bus layer for **100% FREE ($0.00)**. If an application only processes 5% of incoming events, filtering at the rule layer prevents 95% of unneeded Lambda invocations, cutting Lambda request and duration fees by **95%**!
- **Detailed Implementation Steps:**
  1. Add strict event pattern filter in Terraform:
     ```hcl
     resource "aws_cloudwatch_event_rule" "failed_orders" {
       name           = "failed-orders-rule"
       event_bus_name = aws_cloudwatch_event_bus.custom_bus.name
       event_pattern = jsonencode({
         source      = ["my.orders"]
         detail-type = ["OrderStateChange"]
         detail      = { status = ["FAILED"] }
       })
     }
     ```
- **Estimated Savings:** **80–95% reduction** in downstream Lambda/Step Functions compute fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Event pattern attributes published in event payload.

#### 3. Use EventBridge Pipes Filtering (Free Filtered Requests)
- **What:** Add filtering criteria on **EventBridge Pipes** when connecting SQS or Kinesis sources to targets.
- **Why It Saves Money:** EventBridge Pipes processes requests at $0.40/M. **Filtered requests are 100% FREE ($0.00)**, so filtering out 80% of source stream items drops Pipes processing charges by 80%.
- **Detailed Implementation Steps:**
  1. Configure Pipe source filter criteria JSON in EventBridge Pipes console.
- **Estimated Savings:** 80% savings on EventBridge Pipes request charges.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** EventBridge Pipes usage.

---

### 3. Scheduler & Cron Free Tier Maximization

#### 4. Maximize 14 Million Free Monthly EventBridge Scheduler Invocations
- **What:** Migrate all periodic cron jobs, scheduled Lambda triggers, and ECS task launchers to **EventBridge Scheduler**.
- **Why It Saves Money:** EventBridge Scheduler provides **14 Million free scheduled invocations per month** per account. Beyond 14M, invocations cost $1.25 per Million. Replaces third-party scheduling tools with free native AWS infrastructure.
- **Detailed Implementation Steps:**
  1. Create EventBridge Scheduler schedule in Terraform:
     ```hcl
     resource "aws_scheduler_schedule" "hourly_cleanup" {
       name       = "hourly-cleanup-schedule"
       group_name = "default"
       flexible_time_window { mode = "OFF" }
       schedule_expression = "cron(0 * * * ? *)"
       target {
         arn      = aws_lambda_function.cleanup.arn
         role_arn = aws_iam_role.scheduler_role.arn
       }
     }
     ```
- **Estimated Savings:** **100% savings** on scheduled execution triggers up to 14M calls/mo.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Scheduler API configuration access.

---

### 4. Archive & Replay Storage Control

#### 5. Configure Strict Time-to-Live (TTL) Retentions on Event Archives
- **What:** Set explicit retention periods (e.g. 7 to 14 days) on EventBridge Event Archives rather than retaining archives indefinitely.
- **Why It Saves Money:** Event archives bill **$0.023 per GB-month** for storage (equivalent to S3 Standard). Retaining 5 TB of event history indefinitely costs **$115.00/month**. Setting a 14-day retention drops average storage to 100 GB (**$2.30/month**).
- **Detailed Implementation Steps:**
  1. Set retention days on event archive via AWS CLI:
     ```bash
     aws events update-archive \
       --archive-name "orders-event-archive" \
       --retention-days 14
     ```
- **Estimated Savings:** **95%+ reduction** in archive storage fees.
- **Risk Level:** Zero risk (14-day window provides ample debugging window for event replay).
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** Archive compliance review.

---

### 5. Cross-Account Routing Optimization

#### 6. Minimize Cross-Account Event Bus Hops
- **What:** Consolidate multi-account event bus architectures to route events directly from source to target accounts where possible.
- **Why It Saves Money:** Delivering an event across custom event buses in multiple accounts incurs a custom event ingestion fee ($1.00/M) in *each* receiving account.
- **Detailed Implementation Steps:**
  1. Route source events directly to target account bus using explicit cross-account rule targets.
- **Estimated Savings:** 50% reduction in multi-account event processing fees.
- **Risk Level:** Low.
- **Implementation Scope:** Enterprise Architect / DevOps
- **Prerequisites:** Cross-account event bus policies configured.

---

### 6. Governance & Observability

#### 7. Maximize 1 Million Free Custom Events Allowance
- **What:** Utilize 1 Million free monthly custom events across developer accounts.
- **Why It Saves Money:** Free baseline testing.
- **Detailed Implementation Steps:**
  1. Monitor free tier custom event metric in Billing Console.
- **Estimated Savings:** Baseline free testing.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

#### 8. Delete Unused / Abandoned Custom Event Buses
- **What:** Delete custom event buses (`aws events delete-event-bus`) with 0 ingested events over 30 days.
- **Why It Saves Money:** Administrative hygiene.
- **Detailed Implementation Steps:**
  1. Delete custom event bus via CLI.
- **Estimated Savings:** Administrative cleanliness.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** 30-day bus metric audit.

#### 9. Enforce CloudWatch Alarms on `IngestedEvents` Metric Surges
- **What:** Put CloudWatch alarm on `IngestedEvents` metric (> 100,000 events/hour).
- **Why It Saves Money:** Instant alert if a publisher loop starts spamming custom events.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm targeting `AWS/Events`.
- **Estimated Savings:** Proactive billing risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 10. Leverage Unlimited Free AWS Service Events
- **What:** Use native AWS Service Events (S3 Event Notifications, EC2 State Changes) for infrastructure triggers.
- **Why It Saves Money:** AWS Service Events are **100% FREE ($0.00)**.
- **Detailed Implementation Steps:**
  1. Define rules matching `source = ["aws.s3", "aws.ec2"]`.
- **Estimated Savings:** $0.00 ingestion cost.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 11. Optimize Schema Registry Discoverer Auto-Detection
- **What:** Disable Schema Registry Auto-Discoverer on high-volume custom event buses once schemas are finalized.
- **Why It Saves Money:** Prevents continuous schema parsing overhead.
- **Detailed Implementation Steps:**
  1. Stop schema discoverer via CLI.
- **Estimated Savings:** Schema discovery fee optimization.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Schema validation complete.

#### 12. Consolidate Duplicate Event Pattern Rules
- **What:** Combine single-target rules into a multi-target rule (up to 5 targets per rule).
- **Why It Saves Money:** Eliminates duplicate rule evaluation overhead.
- **Detailed Implementation Steps:**
  1. Attach multiple targets to 1 EventBridge rule.
- **Estimated Savings:** Rule evaluation efficiency.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Target ARN array configuration.

#### 13. Audit Event Replay Batch Sizes
- **What:** Specify narrow time windows when initiating Event Replays (`aws events start-replay`).
- **Why It Saves Money:** Replays bill $1.00/M events replayed.
- **Detailed Implementation Steps:**
  1. Set precise `StartStartTime` and `EventEndTime` replay parameters.
- **Estimated Savings:** Avoids replaying unneeded historical event volume.
- **Risk Level:** Zero.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Event replay workflow.

#### 14. Enforce Dead-Letter Queue (DLQ) Setup on Event Targets
- **What:** Attach an SQS DLQ to EventBridge Rule Targets.
- **Why It Saves Money:** Prevents 24-hour target delivery retry loops on failing target endpoints.
- **Detailed Implementation Steps:**
  1. Specify `DeadLetterConfig` in event target definition.
- **Estimated Savings:** Prevents failed retry delivery overruns.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SQS DLQ created.

#### 15. Standardize IAM Resource Policies on Event Buses
- **What:** Restrict `events:PutEvents` access to authorized IAM roles.
- **Why It Saves Money:** Prevents unauthorized external event publishing.
- **Detailed Implementation Steps:**
  1. Apply strict event bus resource policy.
- **Estimated Savings:** Security risk mitigation.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** IAM policy review.

#### 16. Implement Event Batching via Kinesis Firehose Targets
- **What:** Route high-volume telemetry events from EventBridge directly into Kinesis Firehose for S3 data lake storage.
- **Why It Saves Money:** Batches small events into large S3 objects, saving S3 request fees.
- **Detailed Implementation Steps:**
  1. Add Firehose target to EventBridge rule.
- **Estimated Savings:** S3 request fee optimization.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** Firehose delivery stream setup.

---

## Cross-Service Synergies

```
[ Application Publisher ] 
        │
        ├──(Payload Size)──────> [ S3 Pointer Pattern (< 64 KB) ] (Saves 75% on event chunking fees)
        │
        ├──(Rule Filtering)────> [ Event Pattern Filters (JSON) ] (100% FREE - Cuts Lambda calls 95%)
        │
        └──(Cron Scheduling)───> [ EventBridge Scheduler ] (14 Million FREE calls/mo - $0.00 cost)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Event-64KB-Chunk`, `Scheduler-Invocation`, `Pipes-Request-Count`, `Archive-Storage-ByteHrs`.
- `line_item_resource_id`: EventBridge Event Bus Name (`arn:aws:events:us-east-1:123456789012:event-bus/custom-bus`).

### B. CloudWatch Metrics
- `AWS/Events` Namespace: `IngestedEvents`, `MatchedEvents`, `TriggeredRules`, `SuccessfulInvocationCount`, `FailedInvocations`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "EVB-PTR-001",
  "service": "EventBridge",
  "category": "Event Payload & Chunking Optimization",
  "resource_id": "arn:aws:events:us-east-1:123456789012:event-bus/telemetry-bus",
  "resource_name": "telemetry-bus",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "avg_payload_size_kb": 180,
    "metered_chunks_per_event": 3,
    "monthly_events_millions": 50.0,
    "metered_events_millions": 150.0,
    "monthly_cost_usd": 150.00
  },
  "recommended_config": {
    "action": "Implement S3 Pointer Pattern (< 64 KB)",
    "projected_metered_events_millions": 50.0,
    "projected_monthly_cost_usd": 50.00
  },
  "financial_impact": {
    "monthly_savings_usd": 100.00,
    "annual_savings_usd": 1200.00,
    "savings_percentage": 66.6
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Large JSON payloads stored in S3; event bus passes lightweight pointer key."
  },
  "implementation": {
    "scope": "Software Engineer",
    "effort_estimate": "1-2 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **S3 Pointer Pattern (< 64 KB)** | 8 | $3,500.00 | $2,331.00 | 66.6% | Zero |
| **Event Pattern Rule Filtering** | 15 | $12,400.00 | $11,780.00 | 95.0% | Zero |
| **EventBridge Scheduler Free Tier**| 10 | $1,250.00 | $1,250.00 | 100.0% | Zero |
| **Archive TTL Expiration (14 Days)**| 6 | $2,400.00 | $2,280.00 | 95.0% | Zero |
| **Total** | **39** | **$19,550.00** | **$17,641.00** | **90.2%** | -- |
