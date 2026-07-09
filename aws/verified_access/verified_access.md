# AWS Service Cost Research: AWS Verified Access

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Verified Access is a managed service that provides secure, corporate application access based on Zero Trust principles. It evaluates every request in real time using identity (OIDC/SAML) and device posture (MDM) signals before granting access. Verified Access acts as a modern, clientless alternative to traditional corporate VPNs. However, because it bills per application-hour, deploying it across many individual internal microservices can result in high base charges.

---

## 2. Billing Mechanics
Verified Access is billed on a monthly cycle based on two dimensions:
1.  **Application Hours:** Billed per hour for each application associated with an active Verified Access endpoint.
2.  **Data Processed:** Billed per GB of data processed by Verified Access.

---

## 3. Key Cost Dimensions

### A. Application Hourly Fees (us-east-1 Tiers)
*   **The Model:** Billed hourly per application. This fee is tiered based on the total cumulative application-hours across your account in a month:
    *   *First 9,999 application-hours:* **$0.27 per application-hour** (~$197.10/month per application).
    *   *Next 64,400 application-hours:* **$0.23 per application-hour**.
    *   *Beyond 74,400 application-hours:* **$0.20 per application-hour**.
*   *Scale Impact:* If you protect 10 internal applications continuously (24/7), your baseline fee is:
    $$10\text{ applications} \times 730\text{ hours} \times \$0.27 = \$1,971.00\text{ / month flat}$$
    This is billed regardless of whether any employees log in.

### B. Data Processing Fee
*   **The Rate:** **$0.02 per GB** of data processed.
*   *Egress charges:* Standard AWS data egress rates apply to outbound traffic routed back to user clients.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cumulative Application-Hours / Month | Application Hourly Rate | Data Processing Rate (/GB) |
|--------------------------------------|-------------------------|----------------------------|
| **0 to 9,999 hours** | **$0.2700** | **$0.0200** |
| **10,000 to 74,399 hours** | **$0.2300** | **$0.0200** |
| **74,400+ hours** | **$0.2000** | **$0.0200** |

---

## 5. AWS Free Tier Coverage
*   **AWS Verified Access:** No free tier is available. All endpoints and application associations generate standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Mapping Every Sub-Service Individually:** Associating separate Verified Access endpoints for every minor microservice panel (e.g. `kibana.internal`, `grafana.internal`, `admin.internal`). If 5 sub-panels are configured as 5 separate applications, they bill at 5x the flat rate (**$985.50/month**).
*   **Idle Sandbox Environments:** Leaving Verified Access active on test or sandbox environments when developers are not working, accumulating $0.27/hour per endpoint.
*   **Routing High-Volume File Transfers:** Passing large database dumps or backups through Verified Access, incurring the $0.02/GB processing fee.

---

## 7. Actionable Cost Optimization Strategies
1.  **Consolidate Applications under Shared Endpoints (Wildcards):**
    *   Instead of creating separate Verified Access endpoints for every individual sub-tool (e.g., kibana, grafana, prometheus):
    *   Deploy a **single consolidated proxy or portal** (like a reverse-proxy server or ingress controller) behind **one Verified Access endpoint** (e.g., `portal.company.com`).
    *   Handle sub-routing locally using path-based rules (e.g., `portal.company.com/kibana`) on the backend server.
    *   **The Savings:** Consolidating 5 tools under 1 endpoint saves **$788.40/month** in base application fees.
2.  **Delete Associations in Non-Production Outside Office Hours:** Set up automation scripts (using the AWS CLI/SDK) to disassociate applications from dev/test Verified Access endpoints at 7 PM and re-associate them at 7 AM.
3.  **Evaluate Client VPN for High-Endpoint Scenarios:**
    *   If you have 40 separate internal microservices that must remain isolated:
    *   Protecting them individually with Verified Access costs:
        $$40\text{ apps} \times 730\text{ hours} \times \$0.27 = \$7,884.00\text{ / month}$$
    *   Deploying a centralized **AWS Client VPN** instead costs only **$146.00/month** flat (base subnet associations) plus user connection hours, representing a massive cost reduction for high-endpoint corporate access.
4.  **Bypass Verified Access for Public Data:** Ensure that public-facing pages, public APIs, and asset-heavy assets (CSS/JS) bypass Verified Access to avoid both application-hour and data processing fees.
