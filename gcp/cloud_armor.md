# Cloud Armor Cost Optimization & Research

Google Cloud Armor is a Web Application Firewall (WAF) and Distributed Denial of Service (DDoS) protection service. It integrates directly with Google Cloud's HTTP(S) Load Balancer, providing rate limiting, SQL injection defense, and IP blocking. While it is a critical security layer, choosing the wrong subscription tier or evaluating unnecessary internal traffic can cause high monthly charges.

---

## 1. Cloud Armor Billing Tiers & Components

Cloud Armor is priced under two models:

### A. Cloud Armor Standard (Pay-As-You-Go)
Best for small-to-medium scale applications.
* **Security Policies:** $5.00 per policy per month.
* **Rules:** $1.00 per rule per month.
* **Requests Evaluated:** $0.75 per million requests.

### B. Cloud Armor Enterprise (Subscription-based)
Designed for large-scale enterprise environments requiring advanced DDoS protection and managed WAF rules.
* **The Cost:** Billed at a flat **$3,000 per month baseline subscription fee** (which includes up to 100 policies and 9,999 rules) plus a per-million request evaluation fee (discounted vs Standard).
* **The Cost Risk:** Extremely High. If turned on in a developer playground or a startup GCP organization, it immediately bills $3,000/month, even if request traffic is zero.

---

## 2. Core Cost-Optimization Levers

### A. Prevent Accidental Cloud Armor Enterprise Enrolment
* **Action:** Never enable Cloud Armor Enterprise (Managed Protection) in development, sandbox, or staging projects. Standard pay-as-you-go billing is more than adequate for testing WAF rule logic.
* **Governance:** Use GCP Organization Policies to restrict the creation of Cloud Armor Enterprise enrollments to production admin projects.

### B. Route Static/Media Traffic Around Cloud Armor via CDN
Every request evaluated by Cloud Armor costs money ($0.75/million on Standard).
* **The Waste:** Evaluating every request for static images, CSS, and JS files.
* **The Solution:** Enable **Cloud CDN** on your Load Balancer backend and use **Backend Security Policies**.
* **The Benefit:** When using Backend Security Policies, requests that hit the CDN cache are served directly at the edge. Because these cache hits do not reach the load balancer backend, they bypass Cloud Armor evaluation entirely, saving 100% of the WAF evaluation charges for that traffic. (Note: If you need to filter traffic *before* the cache, you must use Edge Security Policies, which will incur evaluation charges on all requests).

### C. Consolidate Rules Using IP Ranges (CIDR Blocks)
* **The Waste:** Writing a separate Cloud Armor rule for every single IP address you want to block (e.g. 50 rules for 50 single IPs = $50/month).
* **Action:** Group IP blocks. Consolidate individual rules into single rules using CIDR blocks (e.g. `192.168.1.0/24` or arrays of multiple IPs in a single rule using expressions: `[ "1.1.1.1", "2.2.2.2" ].contains(request.headers['x-forwarded-for'])`).
* **The Benefit:** Reduces the rule count and cuts the $1.00/rule/month fee.

### D. Avoid Redundant Attachments
* **Action:** Only attach Cloud Armor policies to **External Application Load Balancers** that face the public internet. Do not attach them to Internal Load Balancers or internal VPC microservices where traffic is already trusted.

---

## 3. Cloud Armor Audit Checklist

1. [ ] **Enterprise Enrollment Check:** Confirm that no development or test projects have active Cloud Armor Enterprise subscriptions.
2. [ ] **CDN Caching Audit:** Verify that Cloud CDN is enabled in front of static assets to bypass WAF request evaluation.
3. [ ] **Rule Consolidation Sweep:** Audit active policies. Consolidate individual IP block rules into arrays or CIDR blocks.
4. [ ] **Load Balancer Attachment Review:** Ensure Cloud Armor is only attached to public ingress gateways, not internal resources.
