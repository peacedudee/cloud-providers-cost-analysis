# AWS Service Cost Research: Amazon EventBridge

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon EventBridge is a serverless event bus service that makes it easy to connect applications together using data from your own applications, integrated Software-as-a-Service (SaaS) applications (like Zendesk, Datadog, Salesforce), and AWS services. EventBridge simplifies event-driven architectures by routing events from sources to targets (Lambda, SQS, Step Functions, Kinesis) using declarative rules. EventBridge includes Event Buses, EventBridge Scheduler, EventBridge Pipes, and Schema Registry.

---

## 2. Billing Mechanics
EventBridge billing is calculated on a monthly cycle based on the following components:
1.  **Custom / Partner SaaS Events Ingested:** Billed per 1 Million events published to custom event buses ($1.00 per 1 Million events).
2.  **AWS Service Events Ingested:** **100% Free ($0.00)** (e.g., S3 event notifications, EC2 state changes, GuardDuty alerts).
3.  **EventBridge Scheduler:** Billed per 1 Million scheduled invocations ($1.25 per 1 Million calls, with **14 Million free invocations/month**).
4.  **EventBridge Pipes:** Billed per 1 Million requests processed ($0.40 per 1 Million requests after filtering).
5.  **Archive and Replay:** 
    *   *Archive Ingestion/Processing:* **$0.10 per GB** of event data archived.
    *   *Archive Storage:* **$0.023 per GB-month** of data stored.
    *   *Event Replay:* **$1.00 per 1 Million events** replayed.

---

## 3. Key Cost Dimensions

### A. Core Event Bus Pricing (us-east-1)
*   **AWS Service Events:** **100% Free ($0.00)**.
*   **Custom / Partner SaaS Events Ingested:** **$1.00 per 1 Million events** (first 1 Million free/month).
*   **Event Cross-Account Delivery:** Billed as a custom event ($1.00/M) in the target account.
*   **The 64 KB Chunking Rule:** Events are metered in blocks of 64 KB. An event of size 128 KB is metered as **2 events** for ingestion, archiving, and replay.

### B. EventBridge Scheduler & Pipes Pricing
*   **EventBridge Scheduler:** **$1.25 per 1 Million invocations** (first **14 Million invocations free/month**).
*   **EventBridge Pipes:** **$0.40 per 1 Million requests** (filtered requests are **free**).

### C. Archive and Replay
*   Allows you to archive events in an event bus and replay them later for debugging or recovery.
*   *Archive Processing:* **$0.10 per GB** processed.
*   *Archive Storage:* **$0.023 per GB-month** (equivalent to S3 Standard storage).
*   *Replay Processing:* Billed at the standard custom event ingestion rate of **$1.00 per 1 Million events**.

---

## 4. Detailed Pricing Rates (us-east-1)

| EventBridge Component | Free Tier Allowance | Rate per 1 Million Events | Price for 10 Million Events |
|-----------------------|---------------------|---------------------------|-----------------------------|
| **AWS Service Events** | Unlimited | **Free ($0.00)** | **$0.00** |
| **Custom / Partner Events**| 1M events / month | **$1.00** | **$9.00** (1st 1M free) |
| **EventBridge Scheduler** | **14M calls / month**| **$1.25** | **$0.00** (under 14M limit!)|
| **EventBridge Pipes** | None | **$0.40** | **$4.00** |
| **Event Replay** | None | **$1.00** | **$10.00** |

---

## 5. AWS Free Tier Coverage
*   **Amazon EventBridge Free Tier:** Includes **1 Million free custom events per month**, 14 Million free EventBridge Scheduler invocations per month, and unlimited free AWS service events indefinitely.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Emitting High-Frequency Custom Heartbeat Events:** Publishing high-frequency telemetry or heartbeat events onto a custom Event Bus.
*   **Filtering Events Inside Target Lambda Code Instead of Event Rules:** Routing all raw events to a Lambda target and running `if` statements inside Python code, incurring unnecessary Lambda execution fees ($0.20/M + duration).
*   **The 64 KB Chunking Trap on Large Payloads:** Emitting large JSON payloads (e.g., 200 KB) as events. A 200 KB event consumes **4 event units** ($4.00 per million instead of $1.00).

---

## 7. Actionable Cost Optimization Strategies
1.  **Leverage 14 Million Free Monthly Scheduler Invocations:**
    *   Use **EventBridge Scheduler** to trigger periodic Lambda functions, ECS tasks, or cron jobs.
    *   **The Savings:** Provides enterprise cron scheduling at $0.00 cost.
2.  **Filter Events at the Event Bus Rule Layer:**
    *   Define precise JSON **Event Patterns** (`source`, `detail-type`, `detail.status`) on EventBridge rules. EventBridge filters out non-matching events at the bus level for free, preventing unneeded target Lambda executions.
    *   **The Savings:** Slashes downstream Lambda compute bills by **80–95%**.
3.  **Use EventBridge Pipes Filtering:**
    *   When using EventBridge Pipes to connect SQS/Kinesis to targets, apply Pipes filtering rules. Filtered requests are 100% free ($0.00), saving on pipe execution charges ($0.40/M).
4.  **Avoid Large Payload Events (The Pointer Pattern):**
    *   Never send large binary or text payloads in EventBridge events. Store the payload in **Amazon S3** and pass the **S3 object URL key** in the event detail block. This keeps event sizes under 64 KB.
5.  **Configure Time-to-Live (TTL) on Event Archives:**
    *   Set retention periods on event archives (e.g., retain for 14 days rather than indefinitely) to control cumulative storage fees ($0.023/GB-month).
