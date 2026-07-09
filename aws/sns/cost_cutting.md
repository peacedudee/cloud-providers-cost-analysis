# Cost-Cutting Playbook: Amazon SNS (Simple Notification Service)

> **Companion File:** [sns.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/sns/sns.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon SNS is a managed pub/sub messaging service supporting fanout to SQS queues, Lambda functions, HTTP/S webhooks, Mobile Push notifications (APNS/FCM), Email, and SMS text messages. SNS is billed per API publish request ($0.50/M) and per delivery protocol type.

Massive price variances exist across delivery protocols:
- **Mobile Push (APNS/FCM):** $0.50 per 1 Million deliveries
- **HTTP / SQS Fanout:** $0.60 per 1 Million deliveries ($0.06/100k)
- **Email / Email-JSON:** $200.00 per 1 Million deliveries ($2.00/100k)
- **SMS Text Messages:** $6,500.00 to $15,000.00+ per 1 Million deliveries ($0.0065 to $0.05+/SMS)

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **40–99% reduction in total SNS spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Migrate User Notifications from SMS to Mobile Push Notifications (APNS/FCM):** Slashes notification costs from $6,500/M down to **$0.50 per 1 Million deliveries (99.99% discount)**.
2. **Fanout System Topics to SQS Queues ($0.60/M) Instead of Email ($200/M):** Slashes microservice integration delivery costs by **99.7%**.
3. **Apply SNS Subscription Filter Policies:** Ensures endpoints receive ONLY matching event payloads, eliminating 80%+ of unnecessary downstream delivery charges.

---

## Strategy Categories

### 1. Delivery Protocol Optimization (The Protocol Price Gap)

#### 1. Migrate User Notifications from SMS to Mobile Push (APNS / FCM)
- **What:** Re-architect mobile application user notifications to use **Mobile Push Notifications (APNS for iOS, FCM for Android)** instead of SMS text messages.
- **Why It Saves Money:**
  - **SMS Text Messages (US):** ~$0.0065 per SMS = **$6,500.00 per 1 Million deliveries**.
  - **Mobile Push (APNS/FCM):** **$0.50 per 1 Million deliveries**.
  - Sending 1 Million mobile push notifications saves **$6,499.50/month (a 99.99% cost reduction)**!
- **Detailed Implementation Steps:**
  1. Register APNS/FCM platform application in SNS console or via CLI:
     ```bash
     aws sns create-platform-application \
       --name "MobileAppIOS" \
       --platform "APNS" \
       --attributes PlatformCredential="APNS_PRIVATE_KEY"
     ```
  2. Register client device token to obtain `EndpointArn`, then publish to `EndpointArn`.
- **Estimated Savings:** **99.99% reduction** in notification delivery costs ($6,499.50/M saved).
- **Risk Level:** Low.
- **Implementation Scope:** Mobile Developer / Software Engineer
- **Prerequisites:** Mobile app supports push notifications.

#### 2. Fanout System Topics to SQS Queues ($0.60/M) Instead of Email ($200/M)
- **What:** Replace raw developer/system email subscriptions on high-volume operational topics with **SQS queue subscriptions**.
- **Why It Saves Money:**
  - **Email Subscriptions:** **$2.00 per 100,000 deliveries ($200.00 per 1 Million)**.
  - **SQS Subscriptions:** **$0.06 per 100,000 deliveries ($0.60 per 1 Million)**.
  - Delivering 1 Million automated log/alert messages via SQS instead of Email saves **$199.40/month (a 99.7% discount)**. Worker tools can aggregate SQS messages into single digest emails.
- **Detailed Implementation Steps:**
  1. Subscribe SQS queue to SNS topic in Terraform:
     ```hcl
     resource "aws_sns_topic_subscription" "sqs_target" {
       topic_arn = aws_sns_topic.alerts.arn
       protocol  = "sqs"
       endpoint  = aws_sqs_queue.alerts_queue.arn
     }
     ```
- **Estimated Savings:** **99.7% reduction** in automated system notification spend.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SQS consumer app or digest script.

---

### 2. Message Filtering & Payload Optimization

#### 3. Implement SNS Subscription Filter Policies (Filter at Topic)
- **What:** Configure **Subscription Filter Policies** on SNS topic subscriptions so subscribers receive *only* the specific messages matching their filter criteria.
- **Why It Saves Money:** Without filter policies, SNS delivers *every published message* to *every subscriber*. If an SQS worker queue only cares about `event_type = ORDER_FAILED`, filtering at the topic level prevents delivering the other 90% of `ORDER_CREATED` messages, cutting delivery fees by **90%**.
- **Detailed Implementation Steps:**
  1. Set subscription filter policy in Terraform:
     ```hcl
     resource "aws_sns_topic_subscription" "failed_orders_sub" {
       topic_arn = aws_sns_topic.orders.arn
       protocol  = "sqs"
       endpoint  = aws_sqs_queue.failed_orders.arn
       filter_policy = jsonencode({
         event_type = ["ORDER_FAILED"]
       })
     }
     ```
- **Estimated Savings:** 50–90% reduction in unnecessary endpoint delivery fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Message attributes published on SNS messages.

#### 4. Compress Message Payloads < 64 KB
- **What:** Ensure SNS message payloads are kept under 64 KB (or apply Gzip compression when publishing to SQS targets).
- **Why It Saves Money:** SNS meters payload size in 64 KB chunks. A 100 KB message counts as 2 requests for publishing and 2 requests per delivery.
- **Detailed Implementation Steps:**
  1. Store large payloads in S3 and pass S3 pointer in SNS message.
- **Estimated Savings:** 50% request/delivery count reduction on large payloads.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** S3 payload pointer support.

---

### 3. Topic Architecture & Queue Batching

#### 5. Batch SNS Message Processing via SQS Batching
- **What:** Route SNS messages into an SQS queue, allowing worker Lambda functions or EC2 nodes to read messages in **batches of 10**.
- **Why It Saves Money:** SQS worker reads 10 SNS messages in 1 SQS read call ($0.0000004), cutting downstream processing calls by **90%**.
- **Detailed Implementation Steps:**
  1. Connect SNS -> SQS -> Lambda with `BatchSize = 10`.
- **Estimated Savings:** 90% processing fee reduction.
- **Risk Level:** Zero.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** SNS-to-SQS fanout pattern.

#### 6. Use Standard SNS Topics Over FIFO SNS Topics (Unless Mandated)
- **What:** Use Standard SNS Topics ($0.50/M publish) instead of FIFO SNS Topics ($0.60/M publish).
- **Why It Saves Money:** Standard topics are **16.6% cheaper** and support unlimited throughput.
- **Detailed Implementation Steps:**
  1. Select Standard topic type during topic creation.
- **Estimated Savings:** 16.6% direct publish fee savings.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Application handles out-of-order messages.

---

### 4. SMS Telecom Governance & Spend Caps

#### 7. Set Hard Monthly SMS Spend Limits in Amazon SNS Console
- **What:** Configure a hard monthly account-level SMS spend limit (e.g. `MonthlySpendLimit = 100`) in SNS Text Messaging (SMS) settings.
- **Why It Saves Money:** Prevents buggy application retry loops or SMS phishing/pumping attacks from generating tens of thousands of dollars in telecom billing overruns.
- **Detailed Implementation Steps:**
  1. Set monthly SMS spend limit via AWS CLI:
     ```bash
     aws sns set-sms-attributes --attributes MonthlySpendLimit=100
     ```
- **Estimated Savings:** 100% protection against runaway SMS billing spikes.
- **Risk Level:** Zero risk (protects financial budget).
- **Implementation Scope:** Security / FinOps Team
- **Prerequisites:** Monthly SMS budget alignment.

#### 8. Restrict Outbound SMS Destination Countries (Geofencing)
- **What:** Configure SNS SMS Protection settings to block outbound SMS delivery to high-cost international destinations (e.g., countries with > $0.10/SMS rates).
- **Why It Saves Money:** Blocks SMS fraud pumping scripts targeting premium international numbers.
- **Detailed Implementation Steps:**
  1. Enable Account-Level SMS Destination Country Allow List in SNS console.
- **Estimated Savings:** Prevents $1,000s in SMS toll-fraud attacks.
- **Risk Level:** Low.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** Target user country list.

---

### 5. Network Routing & Private Privacy

#### 9. Deploy PrivateLink Interface Endpoints for VPC SNS Access
- **What:** Configure Interface VPC Endpoints (`com.amazonaws.us-east-1.sns`) for private VPC subnets publishing to SNS topics.
- **Why It Saves Money:** Drops VPC data transfer from NAT Gateway taxes ($0.045/GB) down to PrivateLink rates ($0.01/GB — **78% savings**).
- **Detailed Implementation Steps:**
  1. Create VPC Endpoint for `sns` in Terraform.
- **Estimated Savings:** 78% network processing savings.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Private subnet workloads.

---

### 6. Observability & Governance

#### 10. Maximize Perpetual Free Tier Allowances
- **What:** Utilize 1 Million free monthly API requests, 100k HTTP deliveries, 100k SQS deliveries, and 1M Mobile Push deliveries.
- **Why It Saves Money:** Delivers $0.00 SNS bills for small workloads.
- **Detailed Implementation Steps:**
  1. Monitor free tier metrics in Billing Console.
- **Estimated Savings:** Free baseline operations.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

#### 11. Delete Unused / Abandoned SNS Topics
- **What:** Identify and delete SNS topics with 0 publish requests over 30 days.
- **Why It Saves Money:** Administrative hygiene.
- **Detailed Implementation Steps:**
  1. Delete topic: `aws sns delete-topic --topic-arn ARN`.
- **Estimated Savings:** Administrative cleanliness.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** 30-day topic metric audit.

#### 12. Prune Dead / Unconfirmed Subscriptions
- **What:** Delete subscriptions stuck in `PendingConfirmation` status for > 7 days.
- **Why It Saves Money:** Reclaims topic configuration overhead.
- **Detailed Implementation Steps:**
  1. Delete unconfirmed subscriptions via CLI.
- **Estimated Savings:** Governance efficiency.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 13. Enable CloudWatch Alarms on `NumberOfNotificationsFailed`
- **What:** Put CloudWatch alarm on `NumberOfNotificationsFailed`.
- **Why It Saves Money:** Alerts on failed webhook endpoint retries.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm for `AWS/SNS` metric.
- **Estimated Savings:** Operational risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 14. Optimize Custom Delivery Retry Policies
- **What:** Set `numRetries = 3` on HTTP/S delivery retry policies instead of the default 50 retries over 24 hours.
- **Why It Saves Money:** Prevents endless retry attempts to dead HTTP webhooks.
- **Detailed Implementation Steps:**
  1. Update subscription delivery policy JSON.
- **Estimated Savings:** 80% reduction in failed retry delivery fees.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Subscription delivery policy configuration.

#### 15. Standardize IAM Topic Access Policies
- **What:** Restrict `sns:Publish` access to authorized IAM roles only.
- **Why It Saves Money:** Prevents unapproved cross-account publishing.
- **Detailed Implementation Steps:**
  1. Restrict topic IAM policy principals.
- **Estimated Savings:** Security risk mitigation.
- **Risk Level:** Zero.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** IAM policy setup.

#### 16. Audit Log Delivery to CloudWatch Logs
- **What:** Configure SNS Status Logging to log ONLY `FailureFeedback` instead of `SuccessFeedback`.
- **Why It Saves Money:** Prevents CloudWatch log ingestion fees ($0.50/GB) on successful notification deliveries.
- **Detailed Implementation Steps:**
  1. Set `SuccessFeedbackSampleRate = 0` in topic attributes.
- **Estimated Savings:** 90–99% CloudWatch logging cost reduction for SNS.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Topic attribute configuration.

---

## Cross-Service Synergies

```
[ Application Publisher ] 
        │
        ├──(Protocol Selection)─> [ Mobile Push APNS/FCM ] ($0.50/M vs SMS $6,500/M - 99.99% discount)
        │
        ├──(System Delivery)────> [ SQS Queue Subscriptions ] ($0.60/M vs Email $200/M - 99.7% discount)
        │
        └──(Topic Filtering)────> [ Subscription Filter Policies ] (Saves 80%+ on unneeded endpoint calls)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `PublishSize-64KB`, `HTTP-Delivery`, `SQS-Delivery`, `MobilePush-Delivery`, `Email-Delivery`, `SNS:DirectPublishSMS`.
- `line_item_resource_id`: SNS Topic ARN (`arn:aws:sns:us-east-1:123456789012:prod-alerts`).

### B. CloudWatch Metrics
- `AWS/SNS` Namespace: `NumberOfMessagesPublished`, `NumberOfNotificationsDelivered`, `NumberOfNotificationsFailed`, `SMSMonthToDateSpentUSD`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "SNS-PRT-001",
  "service": "SNS",
  "category": "Delivery Protocol Optimization",
  "resource_id": "arn:aws:sns:us-east-1:123456789012:customer-alerts-topic",
  "resource_name": "customer-alerts-topic",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "delivery_protocol": "SMS_TEXT_MESSAGE",
    "monthly_deliveries_millions": 2.5,
    "avg_sms_rate_usd": 0.008,
    "monthly_cost_usd": 20000.00
  },
  "recommended_config": {
    "delivery_protocol": "MOBILE_PUSH_APNS_FCM",
    "projected_monthly_cost_usd": 1.25
  },
  "financial_impact": {
    "monthly_savings_usd": 19998.75,
    "annual_savings_usd": 239985.00,
    "savings_percentage": 99.99
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "Migrating to Mobile Push improves user experience with rich notifications while cutting delivery cost to near zero."
  },
  "implementation": {
    "scope": "Mobile Developer / Software Engineer",
    "effort_estimate": "1-2 days",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Mobile Push vs SMS Migration** | 5 | $25,000.00 | $24,998.00 | 99.99% | Low |
| **SQS Fanout vs Email Migration** | 10 | $8,500.00 | $8,474.50 | 99.7% | Zero |
| **Subscription Filter Policies** | 12 | $6,200.00 | $4,960.00 | 80.0% | Zero |
| **SMS Monthly Spend Limit Caps** | 8 | $4,000.00 | $3,000.00 | 75.0% | Zero |
| **Total** | **35** | **$43,700.00** | **$41,432.50** | **94.8%** | -- |
