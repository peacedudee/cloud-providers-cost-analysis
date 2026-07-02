# Service Directory Cost Optimization & Research

Google Cloud Service Directory is a managed service registry that provides a single place to register, lookup, and resolve services regardless of where they are running (on-prem, GCP, or other clouds). While it is a lightweight service, high endpoint counts and un-cached lookup requests can generate unnecessary API billing.

---

## 1. Service Directory Billing Mechanics

Service Directory costs are driven by two metrics:
1. **Registered Endpoints:** Billed per endpoint per month (approx. $0.05 per endpoint). An endpoint represents an IP/port target for a service.
2. **Lookup Requests:** Billed per million resolve queries executed (approx. $1.00 per million queries). The first 1 million queries per month are free.

---

## 2. Core Cost-Optimization Levers

### A. Implement Client-Side DNS/API Caching
If your microservices query Service Directory directly or via DNS lookups on every single transaction, lookup charges can escalate.
* **Action:** Enable local query caching in your service lookup library or DNS cache configurations (e.g. configuring a TTL of 30-60 seconds on service resolutions).
* **The Benefit:** Bypasses repeated API lookups for active, stable services, keeping your query count within the free tier or low-volume billing tiers.

### B. Clean Up Stale & Auto-scaled Endpoints
* When running dynamic systems (like GKE clusters with high Pod churn or VMs in autoscaling groups), endpoints are registered dynamically. If the cleanup scripts fail, deleted containers/VMs remain registered as active endpoints.
* **Action:** Automate a script to query Service Directory and prune endpoints whose health check states are failing or whose target VMs no longer exist.
* **The Benefit:** Reclaims the $0.05/endpoint monthly fee.

---

## 3. Service Directory Audit Checklist

1. [ ] **Endpoint Stale Sweep:** Identify and delete registered service endpoints with failing or unreachable IP targets.
2. [ ] **Resolve Request Cache:** Confirm that client libraries and DNS endpoints cache resolution requests rather than querying on every call.
3. [ ] **Registry Retention Rules:** Implement automated cleanup routines in your CI/CD pipelines to delete service configurations when testing namespaces are torn down.
