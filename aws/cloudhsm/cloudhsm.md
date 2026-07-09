# AWS Service Cost Research: AWS CloudHSM

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS CloudHSM is a cloud-based hardware security module (HSM) service that enables users to generate and manage cryptographic keys on dedicated AWS hardware. Single-tenant HSM instances run inside your VPC under strict FIPS 140-2/3 Level 3 compliance. Unlike AWS KMS (which uses shared multitenant HSM hardware), CloudHSM provides dedicated single-tenant hardware ownership. CloudHSM is a major cost driver billed per provisioned HSM instance hour.

---

## 2. Billing Mechanics
AWS CloudHSM billing is structured around dedicated hardware provisioning:
1. **HSM Instance Hourly Fee:** Billed hourly per provisioned HSM instance (prorated by the second).
2. **Empty Cluster Fee:** Creating an empty CloudHSM cluster is **100% Free ($0.00)**. Billing accumulates only when HSM instances are launched inside a cluster.
3. **Data Transfer:** Standard AWS cross-AZ and internet data egress charges apply to client-to-HSM traffic.

---

## 3. Key Cost Dimensions

### A. HSM Instance Hourly Rates (us-east-1)
* **The Rate:** **$1.45 per hour per HSM instance** ($0.0004027 per second).
* **Single HSM Instance 24/7 Baseline:**
  $$\text{Single HSM Cost} = \$1.45 \times 730\text{ hours} = \$1,058.50\text{ / month per HSM}$$
* **Production High Availability (HA) Baseline (2 HSMs across 2 AZs):**
  $$\text{HA Cluster Cost} = 2\text{ HSMs} \times \$1,058.50 = \$2,117.00\text{ / month flat baseline cost}$$

### B. Comparison with AWS KMS
* *AWS KMS Customer Managed Key:* $1.00 per month per key + $0.03/10k requests.
* *AWS CloudHSM Cluster (2 HSMs):* **$2,117.00 per month base** (regardless of key count or request volume).

---

## 4. Detailed Pricing Rates (us-east-1)

| CloudHSM Resource | Billed Unit | Rate (us-east-1) | Monthly Cost per Unit (24/7) |
|-------------------|-------------|------------------|------------------------------|
| **Empty Cluster** | Per cluster | **Free ($0.00)** | $0.00 |
| **Single HSM Instance** | Per HSM-hour | **$1.45** | **$1,058.50** |
| **HA Cluster (2 HSMs)** | Per cluster-month | **$2.90 / hour** | **$2,117.00** |

---

## 5. AWS Free Tier Coverage
* **AWS CloudHSM:** No free tier available. All provisioned HSM instances generate $1.45/hour billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
* **Deploying CloudHSM for Standard Encryption Needs:** Provisioning CloudHSM clusters ($2,117.00/mo base) for standard S3 bucket, EBS volume, or RDS database encryption when **AWS KMS** ($1.00/key-mo or free AWS-managed keys) satisfies security compliance.
* **Orphaned Test HSM Instances:** Leaving HSM instances active in development or sandbox accounts after testing completes, draining **$1,058.50/month** per forgotten HSM.

---

## 7. Actionable Cost Optimization Strategies
1. **Evaluate AWS KMS First (Avoid CloudHSM Unless Mandated):**
   * Use standard **AWS KMS** (which runs on FIPS 140-2 Level 3 HSMs managed multitenantly by AWS).
   * Choose CloudHSM ONLY if corporate governance or regulatory compliance (e.g., PCI PIN, custom PKI root CAs) explicitly mandates dedicated single-tenant hardware.
   * **The Savings:** Saves **$2,117.00+/month** per application stack ($25,404.00/year).
2. **Delete Non-Production HSM Instances When Idle:**
   * CloudHSM clusters with **0 provisioned HSM instances cost $0.00/month**.
   * Save cluster configurations and backup state.
   * Delete HSM instances (`delete-hsm`) when dev/test cycles finish, re-provisioning via script when testing resumes.
   * **The Savings:** Drops non-production hardware fees to $0.00 during idle periods.
3. **Consolidate Applications onto Shared CloudHSM Clusters:**
   * A single CloudHSM cluster supports thousands of keys and multiple client applications.
   * Consolidate team workloads onto a shared enterprise cluster rather than launching separate clusters per application.
