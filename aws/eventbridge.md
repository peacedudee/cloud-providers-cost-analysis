# AWS Service Cost Research: Amazon EventBridge

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon EventBridge is a serverless event bus service that makes it easy to connect applications together using data from your own applications, integrated Software-as-a-Service (SaaS) applications (like Zendesk, Datadog, Salesforce), and AWS services. EventBridge simplifies event-driven architectures by routing events from sources to targets (Lambda, SQS, Step Functions, Kinesis) using declarative rules. EventBridge includes Event Buses, EventBridge Scheduler, EventBridge Pipes, and Schema Registry.

---

## 2. Billing Mechanics
1.  **Custom / Partner SaaS Events Ingested:** Billed per 1 Million events published to custom event buses ($1.00 per 1 Million events).
2.  **AWS Service Events Ingested:** **100% Free ($0.00)** (e.g. S3 event notifications, EC2 state changes, GuardDuty alerts).
3.  **EventBridge Scheduler:** Billed per 1 Million scheduled invocations ($1.25 per 1 Million calls, with **14 Million free invocations/month**).
4.  **EventBridge Pipes:** Billed per 1 Million requests processed ($0.40 per 1 Million requests after filtering).

---

## 3. Key Cost Dimensions

### A. Core Event Bus Pricing (us-east-1)
*   **AWS Service Events:** **100% Free ($0.00)**.
*   **Custom / Partner SaaS Events Ingested:** **$1.00 per 1 Million events** (first 1 Million free/month).
*   **Event Cross-Account Delivery:** Billed as a custom event ($1.00/M) in the target account.

### B. EventBridge Scheduler & Pipes Pricing
*   **EventBridge Scheduler:** **$1.25 per 1 Million invocations** (first **14 Million invocations free/month**).
*   **EventBridge Pipes:** **$0.40 per 1 Million requests** (filtered requests are **free**).

---

## 4. Detailed Pricing Rates (us-east-1)

| EventBridge Component | Free Tier Allowance | Rate per 1 Million Events | Price for 10 Million Events |
|-----------------------|---------------------|---------------------------|-----------------------------|
| **AWS Service Events** | Unlimited | **Free ($0.00)** | **$0.00** |
| **Custom / Partner Events**| 1M events / month | **$1.00** | **$9.00** (1st 1M free) |
| **EventBridge Scheduler** | **14M calls / month**| **$1.25** | **$0.00** (under 14M limit!)|
| **EventBridge Pipes** | None | **$0.40** | **$4.00** |

---

## 5. AWS Free Tier Coverage
*   **Amazon EventBridge Free Tier:** Includes **1 Million free custom events per month**, 14 Million free EventBridge Scheduler invocations per month, and unlimited free AWS service events indefinitely.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Emitting High-Frequency Custom Heartbeat Events:**
    *   Publishing high-frequency telemetry or heartbeat events (e.g. 100 Million events/month) onto a custom Event Bus.
    *   *Math:* $1.00 × 100 Million events = **$100.00/month** in EventBus ingestion fees.
*   **Filtering Events Inside Target Lambda Code Instead of Event Rules:** Routing all raw events to a Lambda target and running `if` statements inside Python code, incurring unnecessary Lambda execution fees ($0.20/M + duration).

---

## 7. Actionable Cost Optimization Strategies
1.  **Leverage 14 Million Free Monthly Scheduler Invocations:**
    *   Use **EventBridge Scheduler** to trigger periodic Lambda functions, ECS tasks, or cron jobs.
    *   *Why:* The first **14 Million invocations per month are 100% Free ($0.00)**.
    *   **The Savings:** Provides enterprise cron scheduling at $0.00 cost.
2.  **Filter Events at the Event Bus Rule Layer:**
    *   Define precise JSON **Event Patterns** (`source`, `detail-type`, `detail.status`) on EventBridge rules.
    *   *Why:* EventBridge filters out non-matching events at the bus level for free, preventing unneeded target Lambda executions.
    *   **The Savings:** Slashes downstream Lambda compute bills by **80–95%**.
3.  **Use EventBridge Pipes Filtering:** When using EventBridge Pipes to connect SQS/Kinesis to targets, apply Pipes filtering rules. Filtered requests are 100% free ($0.00), saving on pipe execution charges ($0.40/M).
