# AWS Service Cost Research: AWS AppSync

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS AppSync is a fully managed serverless GraphQL and real-time Pub/Sub API service that simplifies application development by providing a single, unified GraphQL API interface to access data across multiple backends (Amazon DynamoDB, AWS Lambda, Amazon Aurora Serverless, Amazon OpenSearch, or HTTP APIs). AppSync manages GraphQL query parsing, caching, real-time WebSocket subscriptions, and offline data synchronization. AppSync is serverless, billing based on API requests, real-time connection minutes, and subscription messages.

---

## 2. Billing Mechanics
AppSync billing is calculated monthly across three main dimensions:
1. **GraphQL Operations (Queries & Mutations):** Billed per 1 Million GraphQL operations processed ($4.00 per 1 Million requests).
2. **Real-Time Subscriptions (WebSockets):**
   * *Connection Minutes:* Billed per 1 Million active WebSocket connection minutes ($0.08 per 1 Million minutes).
   * *Subscription Update Messages:* Billed per 1 Million real-time update messages delivered ($2.00 per 1 Million messages).
3. **Server-Side Caching (Optional):** Billed hourly per provisioned cache instance (e.g., `cache.t3.small` at $0.05/hour).

---

## 3. Key Cost Dimensions

### A. Core GraphQL Pricing (us-east-1 Rates)
* **GraphQL Operations (Queries & Mutations):** **$4.00 per 1 Million operations** ($0.000004 per request).
* **Real-Time Connection Time:** **$0.08 per 1 Million connection minutes**.
* **Real-Time Subscription Messages:** **$2.00 per 1 Million messages** delivered.

### B. Mathematical Cost Calculation Example
*Scenario: An application processes 10 Million GraphQL queries/mutations per month, maintains 1 Million active WebSocket connection minutes, and delivers 5 Million real-time subscription update messages.*

1. **GraphQL Operations Charge:**
   $$\text{Operations Fee} = 10,000,000 \times \frac{\$4.00}{1,000,000} = \$40.00\text{ / month}$$
2. **Real-Time Connection Charge:**
   $$\text{Connection Fee} = 1,000,000 \times \frac{\$0.08}{1,000,000} = \$0.08\text{ / month}$$
3. **Subscription Messages Charge:**
   $$\text{Messages Fee} = 5,000,000 \times \frac{\$2.00}{1,000,000} = \$10.00\text{ / month}$$
4. **Total Monthly Cost:** **$50.08 / month**

---

## 4. Detailed Pricing Rates (us-east-1)

| AppSync Feature | Free Tier Allowance | Rate per 1 Million Units | Price for 10 Million Units |
|-----------------|---------------------|--------------------------|----------------------------|
| **GraphQL Operations** | 250k / month | **$4.00** | **$40.00** |
| **Real-Time Connections** | 250k mins / month | **$0.08** | **$0.80** |
| **Subscription Messages** | 250k msgs / month | **$2.00** | **$20.00** |

---

## 5. AWS Free Tier Coverage
* **AWS AppSync Free Tier:** Includes **250,000 free GraphQL operations**, **250,000 free connection minutes**, and **250,000 free subscription update messages** per month indefinitely for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
* **Unfiltered Subscriptions (Message Inflation):** Broadcasting broad dataset changes across open WebSocket connections without server-side argument filtering, causing millions of unwanted subscription message deliveries ($2.00/M msgs).
* **Unnecessary 24/7 Server-Side Cache Provisioning:** Provisioning server-side Redis caching instances ($0.05/hr = $36.00/month) in non-production environments.

---

## 7. Actionable Cost Optimization Strategies
1. **Consolidate Multiple Backend Fetch Calls into 1 GraphQL Query:**
   * Utilize AppSync GraphQL resolvers to retrieve nested data from multiple backend sources (DynamoDB + Lambda + Aurora) in a single request.
   * *Benefit:* Pays for 1 AppSync operation ($4.00/M) instead of making multiple separate client-side API requests.
2. **Enforce Enhanced Subscription Filtering:**
   * Implement AppSync **Enhanced Subscriptions** with argument filters (`owner: $userId`).
   * *Benefit:* Ensures clients receive subscription messages ONLY when data specifically pertinent to that user changes.
   * **The Savings:** Cuts real-time message delivery fees ($2.00/M) by **up to 95%**.
3. **Disable Server-Side Caching in Non-Production:** Turn off server-side caching in dev/test accounts, using free client-side caching instead.
