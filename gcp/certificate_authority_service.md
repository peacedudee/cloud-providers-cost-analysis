# Certificate Authority Service (CAS) Cost Optimization

Google Cloud Certificate Authority Service (CAS) is a highly available, scalable managed service that simplifies the deployment and management of private Certificate Authorities (CAs). CAS allows you to automate the issuing and management of private certificates for your internal servers, devices, and containers. Because active CAs carry high monthly hosting fees, pruning idle CAs and selecting the correct tier is vital.

---

## 1. CAS Billing Components

CAS pricing is determined by two main factors:
1. **CA Instance Hosting (per CA / month):** A flat monthly fee for running each active CA:
   * **DevOps Tier:** $20.00 per CA per month. No SLA, designed for high-volume, short-lived certificates.
   * **Enterprise Tier:** $200.00 per CA per month. Includes SLA, HSM-backed, designed for long-lived certificates.
2. **Certificates Issued:** Billed per certificate created (one-time charge):
   * **DevOps Tier:** $0.30 per cert (0-50k), dropping to $0.03 (50k-100k), and $0.0009 (100k+).
   * **Enterprise Tier:** $0.50 per cert (0-50k), dropping to $0.05 (50k-100k), and $0.001 (100k+).

---

## 2. Core Cost-Optimization Levers

### A. DevOps Tier for Service Meshes and Kubernetes
* **The Cost Trap:** Using the Enterprise Tier to sign short-lived TLS certificates for GKE pods or microservices. If your service mesh rotates certificates daily across 5,000 pods (150,000 certs/month), Enterprise Tier would charge significantly more for certificates.
* **Action:** Enforce the **DevOps Tier** for all high-velocity, short-lived workloads (container identities, test environments, developer client machines). DevOps tier certs are cheaper at volume and the base CA is only $20/month vs $200/month.

### B. Delete Stale or Temporary Private CAs
* **The Waste:** Engineers spin up a private subordinate CA to test a VPN tunnel or an internal service mesh in a staging project, finish the test, and leave the CA running. An idle DevOps CA costs $20/month, and an idle Enterprise CA costs $200/month.
* **Action:** Audit active CAs in your organization. Immediately **Disable** and then **Delete** any CA that has not signed a certificate in the last 30 days.

### C. Consolidate Intermediate CAs
* **Action:** Instead of creating a separate subordinate CA for every single VPC or project, establish a single central Subordinate CA in a Shared Security project and expose it to other projects using IAM roles (`privateca.certificateAuthorities.use`).

---

## 3. CAS Audit Checklist

1. [ ] **DevOps Tier Enforcement:** Verify that high-churn environments (GKE, Service Meshes, IoT endpoints) utilize the DevOps CAS tier.
2. [ ] **Stale CA Sweep:** Locate and delete private CAs that have not signed active certificates in the last 30 days.
3. [ ] **Centralized CA Model:** Consolidate subordinate CAs under a shared services project to minimize hourly instance fees.
