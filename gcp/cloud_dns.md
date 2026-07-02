# Cloud DNS Cost Optimization & Research

Google Cloud DNS is a high-performance, resilient managed Domain Name System (DNS) service. While DNS costs are generally low, high query volumes from intensive internal architectures and zone sprawl (creating separate managed zones for every microservice/sub-environment) can accumulate noticeable monthly charges.

---

## 1. Cloud DNS Billing Mechanics

Cloud DNS charges are calculated based on two metrics:
1. **Managed Zones (per zone / month):**
   * First 25 zones: $0.20 per zone per month.
   * Next 10,000 zones: $0.10 per zone per month.
2. **DNS Queries (per million queries / month):**
   * Standard queries: $0.40 per million queries.
   * Geo-routing/health-checked queries: $0.60 per million queries.

---

## 2. Core Cost-Optimization Levers

### A. Consolidate Managed DNS Zones
* **The Waste:** Creating a separate DNS zone for every subdomain or environment (e.g., creating `dev-api.domain.com`, `test-api.domain.com`, and `staging-api.domain.com` as individual managed zones).
* **The Solution:** Use wildcard or parent zones.
* **Action:** Consolidate records under a single parent zone (e.g. `domain.com` or `dev.domain.com`) and add CNAME/A records within that single zone, rather than provisioning separate zones.

### B. Optimize Time-To-Live (TTL) Settings
* **The Cost Trap:** Setting extremely short TTLs (e.g., 5 seconds or 10 seconds) on static records. This forces client applications, load balancers, and external DNS resolvers to query Cloud DNS continuously on every transaction, inflating query count billing.
* **Action:**
  * For stable, persistent endpoints (database hostnames, main API domains), set TTLs to **300 seconds (5 minutes)** or **3,000 seconds (50 minutes)**.
  * Reduce TTLs to 5-10 seconds only during active maintenance windows or DNS migrations, and restore them to longer values once complete.
* **The Benefit:** Drastically reduces the number of queried lookups hitting Google's DNS servers.

### C. Restrict DNS Queries inside Kubernetes (CoreDNS Tuning)
* In GKE, pods perform external DNS lookups frequently due to the default `ndots:5` search path configuration. If a pod looks up `api.partner.com`, CoreDNS will try searching `api.partner.com.default.svc.cluster.local`, then `api.partner.com.svc.cluster.local`, and so on, before trying the external lookup. Every step triggers a local search and often cascades out to Cloud DNS.
* **Action:**
  * Configure GKE deployments with `ndots:1` in the pod's `dnsConfig` if the pod only accesses external domains.
  * Optimize CoreDNS caching in GKE to resolve queries locally within the cluster rather than forwarding them to Cloud DNS.

---

## 3. Cloud DNS Audit Checklist

1. [ ] **Zone Sprawl Clean Up:** Audit active DNS zones. Delete unused test zones and consolidate subdomains under parent zones.
2. [ ] **TTL Verification:** Inspect TTL configurations on all active zones. Confirm stable hostnames have TTLs >= 300s.
3. [ ] **GKE ndots Optimization:** Review GKE deployment manifests for appropriate `dnsConfig` properties to limit redundant search path lookups.
4. [ ] **Query Volume Monitoring:** Monitor DNS query rates in Cloud Monitoring to identify and investigate applications generating excessive query rates.
