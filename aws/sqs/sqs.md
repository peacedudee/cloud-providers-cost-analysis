# AWS Service Cost Research: Amazon SQS (Simple Queue Service)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon SQS (Simple Queue Service) is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications. SQS eliminates the complexity and overhead associated with managing message-oriented middleware. SQS offers two queue types: **Standard Queues** (maximum throughput, at-least-once delivery) and **FIFO Queues** (first-in-first-out, exactly-once processing). SQS is billed per API request and payload size.

---

## 2. Billing Mechanics
1.  **API Requests:** Billed per API call (`SendMessage`, `ReceiveMessage`, `DeleteMessage`, `ChangeMessageVisibility`).
2.  **Payload Sizing:** Every **64 KB chunk** of message payload counts as 1 request.
    *   *Examples:* A 10 KB message = 1 request. A 100 KB message = 2 requests. A 256 KB message (max limit) = 4 requests.
3.  **Free Tier:** First **1 Million requests per month** are **100% Free ($0.00)**.

---

## 3. Key Cost Dimensions

### A. Standard Queue Volume Tiers (us-east-1)
*   **First 1 Million requests / month:** **Free ($0.00)**.
*   **1 Million to 100 Billion requests / month:** **$0.40 per 1 Million requests** ($0.0000004 per request).
*   **100 Billion to 200 Billion requests / month:** **$0.30 per 1 Million requests**.
*   **Over 200 Billion requests / month:** **$0.24 per 1 Million requests**.

### B. FIFO Queue Volume Tiers (us-east-1)
*   **First 1 Million requests / month:** **Free ($0.00)**.
*   **1 Million to 100 Billion requests / month:** **$0.50 per 1 Million requests** (**25% higher** than Standard).

---

## 4. Detailed Pricing Rates (us-east-1)

| Queue Type | Monthly Request Tier | Rate per 1 Million Requests | Price for 100 Million Requests |
|------------|----------------------|-----------------------------|--------------------------------|
| **Standard Queue** | 1M - 100 Billion | **$0.40** | **$40.00** |
| **FIFO Queue** | 1M - 100 Billion | **$0.50** | **$50.00** |

---

## 5. AWS Free Tier Coverage
*   **Amazon SQS Free Tier:** Includes **1 Million free requests per month** indefinitely for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **The Short Polling Empty Queue Burn ($200+/mo for nothing):**
    *   Configuring worker applications to poll empty SQS queues with `WaitTimeSeconds = 0` (Short Polling).
    *   Worker threads continuously issue `ReceiveMessage` calls 20 times/second per worker, getting empty responses.
    *   *Math:* 10 worker threads polling 20 req/sec = 200 req/sec = 518 Million requests/month = **$207.36/month spent polling empty queues!**
*   **Sending Single Messages Instead of Batches:** Calling `SendMessage` 10 times individually instead of issuing 1 `SendMessageBatch` request.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enable SQS Long Polling (`WaitTimeSeconds = 20`):**
    *   Set `ReceiveMessageWaitTimeSeconds = 20` on your SQS queue configurations and API read calls.
    *   *How it works:* SQS waits up to 20 seconds for a message to arrive before returning a response, rather than immediately returning an empty response.
    *   **The Savings:** Reduces empty read API calls by **98%**, dropping monthly polling bills from $200+ down to near $0.00.
2.  **Use Batch API Operations (`SendMessageBatch` & `DeleteMessageBatch`):**
    *   Group up to 10 messages (up to 256 KB total) into single `SendMessageBatch` and `DeleteMessageBatch` calls.
    *   **The Savings:** Cuts SQS API request volume and billing by **90%**.
3.  **Prefer Standard Queues Over FIFO When Strict Ordering Isn't Mandatory:** Standard queues ($0.40/M) are **20% cheaper** than FIFO queues ($0.50/M) and support nearly unlimited throughput.
4.  **Compress Message Payloads:** Use GZIP compression on large JSON payloads before sending to keep message sizes under 64 KB (1 request instead of 4 requests).
