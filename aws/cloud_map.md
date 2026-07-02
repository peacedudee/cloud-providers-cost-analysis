# AWS Service Cost Research: AWS Cloud Map

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Cloud Map is a cloud resource discovery service that allows developers to define custom names for application resources (such as databases, queues, microservices, or APIs). The service maintains up-to-date physical locations for dynamically changing resources. Cloud Map operates on a pay-as-you-go billing model based on registered resource count and discovery API calls.

---

## 2. Billing Mechanics
Cloud Map bills monthly across three primary dimensions:
1. **Resource Registrations:** Billed per active registered resource per month ($0.10/resource-mo, prorated daily).
2. **Discovery API Queries:** Billed per million `DiscoverInstances` API requests ($1.00 per million queries).
3. **Route 53 DNS Charges (Optional):** Standard Route 53 hosted zone ($0.50/mo) and query charges ($0.40/M) apply if using DNS-based namespaces.

---

## 3. Key Cost Dimensions

### A. Resource Registrations (us-east-1)
* **Standard Registration Rate:** **$0.10 per registered resource per month** (prorated daily).
* **Amazon ECS Service Discovery Integration:** Resource registrations created automatically via **Amazon ECS Service Discovery** are **100% Free ($0.00)**. You pay only for lookup queries.

### B. Discovery API Queries
* **The Rate:** **$1.00 per million queries** (for `DiscoverInstances` API calls).
* **Mathematical Scale Calculation Example:**
  * *Scenario: 200 microservice tasks querying Cloud Map every 2 seconds to discover downstream endpoints.*
  $$\text{Monthly Queries} = 200\text{ tasks} \times 30\text{ lookups/min} \times 43,800\text{ mins} = 262.8\text{ Million queries / month}$$
  $$\text{Query Cost} = 262.8\text{ Million} \times \$1.00 = \$262.80\text{ / month in API fees}$$

### C. Route 53 DNS Charges
If using a **DNS-based namespace**:
* Route 53 charges **$0.50/month** for the auto-created private hosted zone.
* Route 53 DNS queries are billed at **$0.40 per million** standard DNS queries.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Monthly Base Rate | Usage Rate (per Million) | Notes |
|----------------|-------------------|--------------------------|-------|
| **Registered Resource** | **$0.10 / month** | N/A | Prorated daily |
| **ECS Registered Resource** | **Free ($0.00)** | N/A | Billed only for queries |
| **API Lookup Query** | N/A | **$1.00** | DiscoverInstances API |
| **Route 53 DNS Zone** | $0.50 / month | $0.40 (per M DNS queries) | Auto-created by DNS namespaces |

---

## 5. AWS Free Tier Coverage
* **AWS Cloud Map:** No free tier available. All resource registrations and lookups generate standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
* **High-Frequency Uncached API Polling:** Polling Cloud Map APIs continuously without client-side caching, accumulating hundreds of millions of API query charges ($1.00/M).
* **Orphaned Service Registrations:** Retaining stale microservice endpoints or inactive database locations in the Cloud Map registry ($0.10/resource-mo).

---

## 7. Actionable Cost Optimization Strategies
1. **Implement Client-Side Lookup Caching:**
   * Configure application service discovery clients to cache lookup results locally for a short duration (e.g., 30–60 seconds).
   * **The Savings:** Cuts API lookup queries by **up to 95%**.
2. **Leverage ECS Service Discovery:** Deploy containerized applications via ECS Service Discovery to register task IPs into Cloud Map for **free ($0.00)**, bypassing manual registration charges.
3. **Use HTTP Namespaces for Non-DNS Applications:** For apps querying endpoints via HTTP/gRPC API calls, define **HTTP namespaces** instead of DNS namespaces to bypass the Route 53 Private Hosted Zone base fee ($0.50/mo per zone) and DNS query fees.
