# Free and Low-Cost Security Services (IAM, IAP, Binary Authorization, Assured Workloads, Web Security Scanner, BeyondCorp)

Google Cloud provides several core security and identity services that carry **no direct service fee** or operate on low-cost per-user licensing. Leveraging these free services (like IAM and IAP) is a key architectural practice to improve security posture without adding any software licensing costs.

---

## 1. Directory of Free Security Services ($0 Billing)

### A. Identity and Access Management (IAM)
* **Billing Model:** **100% Free**. There are no charges for creating users, groups, service accounts, or assigning policies, regardless of size or volume.
* **Optimization/Governance:**
  * While IAM is free, service accounts are often used by automated scripts. Audit and delete inactive service accounts and keys to prevent credential leaks.
  * Enforce the Principle of Least Privilege to prevent unauthorized resource provisioning by users (which *does* cost money).

### B. Identity-Aware Proxy (IAP)
* **Billing Model:** **100% Free** for protecting resources hosted on Google Cloud (App Engine, Compute Engine VMs behind Load Balancers, GKE).
* **Cost Advantage:** IAP provides zero-trust remote access without requiring a VPN gateway. By replacing VPN gateways with IAP, you eliminate the hourly VPN tunnel fees and VPN public IP costs entirely.

### C. Binary Authorization
* **Billing Model:** **100% Free**. There is no charge for defining, managing, or enforcing container deployment policies in GKE or Cloud Run.
* **Benefit:** Ensures only signed, validated container images are deployed, blocking malicious or untested software.

### D. Assured Workloads
* **Billing Model:** **100% Free** for standard compliance configurations (creating folders with strict compliance policies like FedRAMP, HIPAA, or IL4).
* **Cost Note:** While the tool is free, running resources within compliant folders may limit you to specific compliant regions (which might have higher VM/storage base rates than standard regions).

### E. Web Security Scanner
* **Billing Model:** **100% Free** for scanning public-facing App Engine, Compute Engine, and GKE endpoints.
* **Benefit:** Automates OWASP Top 10 vulnerability checks without needing paid third-party scanning software.

---

## 2. BeyondCorp Enterprise (Paid)

BeyondCorp Enterprise extends free IAP with advanced threat protection, data loss prevention (DLP), and chrome-based device verification.
* **Billing Model:** Billed per user per month (subscription model).
* **Cost-Optimization Levers:**
  * Only license users who strictly require enterprise DLP and device verification (e.g. external contractors using personal laptops or high-risk administrative teams).
  * Standard employees accessing internal apps can use the free **Identity-Aware Proxy (IAP)**, saving subscription seat costs.

---

## 3. Audit Checklist for Identity & Governance

1. [ ] **VPN to IAP Migration:** Identify internal applications currently accessed via Cloud VPN. Migrate them to IAP to decommission expensive VPN tunnels.
2. [ ] **Service Account Key Sweep:** Delete unused or expired service account keys.
3. [ ] **BeyondCorp License Alignment:** Verify that BeyondCorp Enterprise seat counts match the list of contractors or device-enforced users, removing inactive seats.
4. [ ] **Compliance Region Review:** If using Assured Workloads, confirm that workloads are deployed in the cheapest compliant region allowed by the policy template.
