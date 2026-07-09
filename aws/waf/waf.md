# AWS Service Cost Research: AWS WAF (Web Application Firewall)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS WAF is a web application firewall that helps protect your web applications or APIs against common web exploits and bots that may affect availability, compromise security, or consume excessive resources. AWS WAF can be attached to CloudFront distributions, Amazon API Gateways, Application Load Balancers (ALBs), AWS App Runner services, and Amazon Cognito user pools. WAF is billed on a pay-as-you-go model based on Web ACLs, rules, request volume, and advanced managed rule groups.

---

## 2. Billing Mechanics
AWS WAF billing is split into core infrastructure metrics and optional managed rule modules:
1.  **Web ACL Fee:** Billed hourly per provisioned Web Access Control List ($5.00 per month).
2.  **Rule Fee:** Billed hourly per rule added to a Web ACL ($1.00 per rule per month).
3.  **Request Inspection:** Billed per 1 Million requests evaluated by a Web ACL ($0.60 per Million).
4.  **Managed Rule Groups (Add-ons):** Subscriptions for specialized rules (e.g., Bot Control, Account Takeover Prevention).

---

## 3. Key Cost Dimensions

### A. Core Infrastructure Pricing (us-east-1)
*   **Web ACL Storage:** **$5.00 per Web ACL per month** (prorated hourly at ~$0.0068/hour).
*   **Rule Storage:** **$1.00 per rule per month** (prorated hourly at ~$0.00137/hour per rule).
    *   *Note:* A rule inside a Managed Rule Group counts as 1 rule ($1.00/mo) for the entire group wrapper, regardless of how many individual inspection statements are inside.
*   **Request Processing:** **$0.60 per 1 Million requests** evaluated ($0.0000006 per request).

### B. Managed Rule Group Surcharges
*   **AWS Managed Rules (Core Rule Set, Known Bad Inputs, SQLi):** Free subscription fees; standard $1.00/mo rule and $0.60/M request fees apply.
*   **Bot Control (Common Bot Protection):**
    *   *Monthly Subscription:* **$10.00 per month** per Web ACL.
    *   *Request Fee:* **$1.00 per 1 Million requests** inspected.
*   **Bot Control (Targeted Bot Protection):**
    *   *Request Fee:* **$10.00 per 1 Million requests** inspected.
*   **Account Takeover Prevention (ATP) / Fraud Control:**
    *   *Monthly Subscription:* **$10.00 per month** per Web ACL.
    *   *Request Inspection Fee:* **$1.00 to $30.00 per 1,000 requests** inspected on auth endpoints.

### C. Advanced Body Inspection Surcharges
*   Default body inspection evaluates up to **16 KB** of HTTP request payloads.
*   Inspecting up to **64 KB** incurs an extra surcharge of **$0.30 per 1 Million requests** per 16 KB block increment.

---

## 4. Detailed Pricing Rates (us-east-1)

| WAF Component | Billing Unit | Rate (us-east-1) | Price per 1,000,000 Requests |
|---------------|--------------|------------------|------------------------------|
| **Web ACL Base** | Per ACL-month | **$5.00** | $5.00 / month |
| **Rule Base** | Per rule-month | **$1.00** | $1.00 / month |
| **Standard Inspection**| Per 1M requests | **$0.60** | **$0.60** |
| **Bot Control Common** | Per 1M requests | **$1.00** | **$1.00** (+$10/mo sub) |
| **Bot Control Targeted**| Per 1M requests | **$10.00** | **$10.00** |

---

## 5. AWS Free Tier Coverage
*   **AWS WAF:** **No free tier** is available. Creating a Web ACL or evaluating requests generates standard charges immediately. (Shield Advanced subscribers receive free WAF Web ACLs and AWS Managed Rules for protected resources).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Attaching Bot Control / Fraud Control to Unscoped Web ACLs:**
    *   Applying Bot Control ($1.00/M requests) or Fraud Control ($10.00/M requests) to an entire Web ACL attached to a CloudFront distribution that serves static assets (images, CSS, JS).
    *   *Math:* Serving 50 Million image requests/month with targeted Bot Control enabled on the entire Web ACL costs:
        $$50 \times \$10.00 = \$500.00\text{ / month in Bot Control fees alone!}$$
*   **Deploying Duplicate Web ACLs per Load Balancer:** Creating a separate Web ACL ($5.00/mo + $1.00/rule) for every individual regional ALB in the same account instead of sharing a single Web ACL across multiple ALBs.
*   **Over-Inspecting Body Payloads (64 KB Inspection):** Enabling 64 KB body inspection across high-throughput file upload endpoints, adding $0.30 to $0.90 per Million request surcharges.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enforce Scope-Down Statements on Bot & Fraud Control Rules:**
    *   Do not evaluate Bot Control or ATP on static assets or general GET requests.
    *   Add a **Scope-Down Statement** in WAF to restrict Bot Control execution strictly to POST requests on sensitive URIs (e.g., `URI Path equals /api/v1/login` or `/checkout`).
    *   **The Savings:** Cuts Bot/Fraud Control request inspection costs by **95–99%** by skipping static image/CSS traffic.
2.  **Attach Web ACLs at CloudFront (Edge Absorption):**
    *   Attach WAF at the CloudFront distribution layer rather than at the regional Application Load Balancer (ALB).
    *   *Why:* Blocking bad traffic at CloudFront prevents malicious requests from reaching your regional ALBs, NAT Gateways, or EC2 instances, saving downstream compute and bandwidth bills.
3.  **Share Web ACLs Across Regional ALBs & API Gateways:**
    *   A single regional WAF Web ACL can be attached to multiple ALBs and API Gateways in the same region.
    *   Consolidate Web ACLs across similar application stacks to save the **$5.00/month Web ACL fee** and **$1.00/rule fees** per endpoint.
4.  **Use Rate-Based Rules for DDoS Mitigation:** Configure free custom **Rate-Based Rules** (e.g. limit clients to 2,000 requests per 5 minutes) to block HTTP flood attacks automatically for $1.00/month instead of relying on expensive third-party rule groups.
