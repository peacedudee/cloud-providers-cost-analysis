# API Management Cost Optimization & Research

Google Cloud offers three main tiers of API management: **Apigee** (full-lifecycle enterprise API management), **API Gateway** (lightweight, serverless gateway for Cloud Run and Functions), and **Cloud Endpoints** (developer-focused proxy for App Engine and Kubernetes APIs). Choosing the correct API management architecture is a major driver of cost, with monthly spend ranging from $0 (serverless free tier) to thousands of dollars (Apigee Enterprise subscription).

---

## 1. Billing Components & Pricing

* **Apigee:** Available in two main billing options:
  1. **Subscription Model (Standard, Enterprise, Enterprise+):** Flat monthly pricing starting at ~$2,000/month, including a set number of environments and API calls.
  2. **Pay-as-you-go Model:** Billed per API call ($1.00 - $3.00 per 100,000 calls) plus an environment hosting charge per hour per region.
* **API Gateway:** Billed per million API calls processed:
  * First 2 million calls per month: **Free**.
  * Beyond 2 million: $3.00 per million calls.
* **Cloud Endpoints:** Billed per million calls processed (first 2 million free, then $3.00 per million calls). Billed via the Extensible Service Proxy (ESP) metrics.

---

## 2. Core Cost-Optimization Levers

### A. Apply the "API Gateway First" Principle
* **The Cost Trap:** Deploying Apigee ($2,000+/month flat subscription) to manage external traffic for a few simple internal microservices or public webhooks.
* **Action:** Use **API Gateway** (or Cloud Endpoints) for all standard workloads that only require basic routing, API key validation, rate limiting, and CORS handling. API Gateway is serverless, has a generous free tier, and charges zero hosting fees when traffic is idle.
* **Apigee Rule:** Limit Apigee usage strictly to scenarios that require complex enterprise features: API monetization, deep custom Java/Javascript message mediation, developer portals, or advanced WAF/threat protection.

### B. Prune Unused Apigee Environments
* **The Waste:** Under the Apigee pay-as-you-go model, you are billed an hourly rate for every active environment (e.g. `dev`, `test`, `prod`, `sandbox`) deployed in a region, regardless of traffic volume.
* **Action:** Delete inactive or duplicate environment attachments. For testing, consolidate developer work under a single shared `non-prod` environment rather than spawning separate environments for each developer team.

### C. Evaluate Pay-as-you-go (PAYG) vs. Subscription in Apigee
* **Tactic:** Monitor monthly API call volume. If your API volume is highly variable or averages less than 20 million calls per month, the **Apigee Pay-as-you-go** model will result in a lower total cost than a fixed Standard monthly subscription. Move to Subscription models only when traffic is high and predictable.

---

## 3. API Management Audit Checklist

1. [ ] **Apigee Usage Audit:** Identify all active Apigee instances. Migrate simple routing and rate-limiting workloads to the serverless **API Gateway**.
2. [ ] **Apigee PAYG Environment Check:** Delete unused or stale environments under the Apigee PAYG billing model.
3. [ ] **Rate Limiting at Edge:** Implement caching and rate-limiting rules at the Cloud CDN/Load Balancer layer to drop invalid requests before they reach the API gateway, saving API gateway processing fees.
