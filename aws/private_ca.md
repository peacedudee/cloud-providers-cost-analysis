# AWS Service Cost Research: AWS Private CA (Private Certificate Authority)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Private CA is a managed private certificate authority service for issuing internal SSL/TLS certificates for microservices (mTLS), corporate VPNs, IoT devices, and internal web servers. It allows you to build private CA hierarchies (Root and Subordinate CAs) without operating on-premises CA software. Private CA is a high-cost driver billed per active CA instance plus volume fees per certificate issued.

---

## 2. Billing Mechanics
1.  **Private CA Monthly Base Fee:** Billed hourly per provisioned Private CA instance ($400.00 per month).
2.  **Certificate Issuance Fee:** Billed per private certificate issued, with different pricing modes for standard long-lived certificates vs. short-lived certificates.
3.  **Cross-Account Sharing:** A single Private CA can be shared across multiple AWS accounts using AWS Resource Access Manager (RAM) at no extra base cost.

---

## 3. Key Cost Dimensions

### A. CA Base Operation Fee (us-east-1)
*   **The Rate:** **$400.00 per month per active Private CA** (prorated hourly at ~$0.548/hour).
*   *Applies to:* Both Root CAs and Subordinate CAs.
*   *Math:* A 2-tier CA hierarchy (1 Root CA + 1 Subordinate CA) costs:
    $$2\text{ CAs} \times \$400.00 = \$800.00\text{ / month base}$$

### B. Certificate Issuance Pricing Modes

#### Mode 1: General Purpose Mode (Standard Certificates)
Ideal for long-lived certificates (e.g. valid for 1 to 3 years).
*   **First 1,000 certificates / month:** **$0.75 per certificate**.
*   **Next 9,000 certificates / month:** **$0.35 per certificate**.
*   **Over 10,000 certificates / month:** **$0.001 per certificate**.

#### Mode 2: Short-Lived Certificate Mode (Microservices & mTLS)
Ideal for ephemeral container mTLS mesh where certificates are valid for **7 days or less**.
*   **Flat Rate:** **$0.05 per certificate** (a **93% direct discount** over standard mode).

---

## 4. Detailed Pricing Rates (us-east-1)

| Private CA Component | Mode / Tier | Rate (us-east-1) | Price for 1,000 Certs / Month |
|----------------------|-------------|------------------|-------------------------------|
| **Private CA Base** | Per CA instance | **$400.00 / month** | $400.00 base |
| **General Certs** | 0 - 1,000 certs | **$0.75 / cert** | **$750.00** |
| **General Certs** | 1,000 - 10,000 | **$0.35 / cert** | $350.00 |
| **Short-Lived Certs**| Valid <= 7 days | **$0.05 / cert** | **$50.00** |

---

## 5. AWS Free Tier Coverage
*   **AWS Private CA Free Trial:** 30-day free trial for the first Private CA created in an account and region.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Deploying Private CAs in Development Accounts:** Launching dedicated Private CAs ($400.00/mo) in non-production environments to issue test certificates.
*   **Using General Purpose Mode for Ephemeral Kubernetes Pods:** Issuing thousands of short-lived mTLS pod certificates using General Purpose mode ($0.75/cert) instead of Short-Lived mode ($0.05/cert), resulting in massive issuance bills.

---

## 7. Actionable Cost Optimization Strategies
1.  **Switch to Short-Lived Mode for Container mTLS:**
    *   Configure your Private CA or service mesh (e.g. Istio, AWS App Mesh) to issue short-lived certificates valid for <= 7 days.
    *   Set the CA mode to **Short-Lived Certificate Mode ($0.05/cert)**.
    *   **The Savings:** Slashes certificate issuance costs by **93%** ($50 vs $750 per 1,000 certs).
2.  **Share Private CAs Across Accounts via AWS RAM:**
    *   Do not launch separate Private CAs in child accounts.
    *   Deploy 1 central Subordinate Private CA in a central Security account.
    *   Share access with child accounts using **AWS Resource Access Manager (RAM)**.
    *   **The Savings:** Saves $400.00/month per child account.
3.  **Use Open-Source CAs for Non-Production:** Use free open-source tools (like **HashiCorp Vault**, **step-ca**, or **cert-manager**) in development environments to bypass the $400.00/mo Private CA base fee.
4.  **Delete Inactive Private CAs Immediately:** Delete completed testing CAs in the console to stop the $400.00/month recurring hourly base charge.
