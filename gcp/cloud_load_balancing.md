# Cloud Load Balancing Cost Optimization & Research

Google Cloud Load Balancing is a fully distributed, software-defined managed service. It supports global Anycast IP routing, HTTP(S) path routing, and high availability. Because load balancers run 24/7, base hourly charges for forwarding rules combined with high data-processing fees can quickly inflate your networking bill. This file outlines key cost-saving practices for Cloud Load Balancing.

---

## 1. Load Balancing Billing Mechanics

Cloud Load Balancing charges are computed using three primary dimensions:
1. **Forwarding Rules (Hourly):**
   * GCP charges a base fee for the first 5 forwarding rules in a project (approx. $0.025/hour total, or ~$18/month).
   * Each additional forwarding rule beyond the first 5 incurs an hourly charge (approx. $0.01/hour per rule).
2. **Data Processed (per GB):** Billed for data moving through the load balancer (approx. $0.008 per GB).
3. **Operations & Security Add-ons:** Billed if integrated with **Cloud Armor** WAF, **Cloud CDN**, or advanced SSL policies.

---

## 2. Core Cost-Reduction Tactics

### A. Consolidate Forwarding Rules via Host/Path Routing (URL Maps)
* **The Waste:** Creating a completely separate External HTTP(S) Load Balancer for every microservice or static site. If you deploy 15 small microservices and give each its own load balancer, you will pay for 15 separate forwarding rules.
* **The Solution:** Consolidate traffic under a single **Global External Application Load Balancer** using **URL Maps**.
* **Action:**
  * Route requests to different backend services based on the HTTP Host header (e.g., `api.example.com` vs. `app.example.com`) or request path (e.g., `example.com/images/*` vs. `example.com/checkout/*`).
  * This consolidation reduces your active forwarding rules down to 1, saving hundreds of dollars in idle rule fees.

### B. Adjust Load Balancer Log Sampling Rates
For high-traffic applications, Cloud Load Balancer access logs can become a major driver of Cloud Logging ingestion costs (billed at the vended network log rate of **$0.25/GB**).
* **The Trap:** By default, when you enable logging on a Load Balancer Backend Service, GCP logs **100% of all requests (sampling rate = 1.0)**. A site receiving 1,000 requests per second will generate hundreds of gigabytes of logs, running up thousands of dollars in logging charges.
* **The Solution:** Adjust the log sampling rate.
* **Action:** Change the sampling rate from `1.0` to a lower value (e.g., `0.05` or `0.1` for 5% to 10% sampling) in non-critical environments or high-volume static asset servers. Keep the log volume small while still having enough data for traffic analysis.

### C. Offload Data Processing to Cloud CDN
* Every gigabyte processed by the load balancer incurs a data-processing fee ($0.008/GB).
* **Action:** Enable **Cloud CDN** on the load balancer backend services that serve static content (e.g., images, CSS, JavaScript, video).
* **The Benefit:** Requests that hit the CDN cache are served directly from Google's edge POPs, bypassing the load balancer entirely. You avoid the Load Balancer data-processing fee and pay cheaper CDN cache egress rates.

### D. Delete Idle Forwarding Rules and Balancers
* In Kubernetes/GKE, deleting a service of type `LoadBalancer` sometimes fails to clean up the underlying GCP resource due to finalizer bugs, leaving "orphan" forwarding rules billing indefinitely.
* **Action:** Run a gcloud sweep to identify forwarding rules with no active backends and delete them.
  ```bash
  gcloud compute forwarding-rules list --filter="target:-"
  ```

---

## 3. Cloud Load Balancing Audit Checklist

1. [ ] **Rule Consolidation:** Audit your GCP projects to find duplicate external HTTP(S) load balancers. Build a plan to consolidate them under a single URL map.
2. [ ] **Log Sampling Review:** Check the sampling rate configuration on all active Backend Services. Set high-throughput endpoints to < 0.1 sampling.
3. [ ] **Orphan Rule Sweep:** Run CLI scripts to find and delete forwarding rules that do not point to any target proxy or pool.
4. [ ] **CDN Enablement:** Verify that Cloud CDN is enabled on all backend services serving static public media.
5. [ ] **Standard Tier Evaluation:** For single-region regional external load balancers, evaluate if migrating from Premium Tier to Standard Tier is viable to save on egress fees.
