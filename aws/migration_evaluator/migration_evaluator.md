# AWS Service Cost Research: AWS Migration Evaluator

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Migration Evaluator (formerly TSO Logic) is a complimentary assessment service provided by AWS to assist enterprise customers in creating data-driven business cases and Total Cost of Ownership (TCO) models for migrating on-premises workloads to AWS. It deploys lightweight agentless collectors or ingests existing inventory exports (from VMware vCenter, RVTools, or BMC Helix) to analyze CPU/RAM utilization rates, server configurations, and software licensing (Microsoft Windows Server, SQL Server, Oracle). Migration Evaluator generates comprehensive cost comparison reports detailing right-sized EC2, EBS, and licensing options.

---

## 2. Billing Mechanics
1. **Assessment & Collector Software:** **100% Free ($0.00)**. No service fees, per-server charges, or assessment software costs.
2. **AWS Business Case Delivery:** AWS solution architects and financial analysts provide the assessment reports and directional TCO analysis at zero cost.

---

## 3. Key Cost Dimensions

| Service Feature | Billed Metric | Rate (us-east-1) | Price |
|-----------------|---------------|------------------|-------|
| **Inventory Collection Agent**| Per server | **Free ($0.00)** | **$0.00** |
| **TCO & Business Case Report**| Per assessment| **Free ($0.00)** | **$0.00** |

---

## 4. Detailed Pricing Rates (us-east-1)

* **Service Rate:** $0.00 per month.

---

## 5. AWS Free Tier Coverage
* **AWS Migration Evaluator:** Always 100% free for all AWS customers indefinitely.

---

## 6. Common Cost Hotspots & Pitfalls
* **Migrating On-Premises Core Counts 1-to-1 Without Right-Sizing:** Migrating over-provisioned on-premises physical servers (e.g. 64-vCPU database server running at 5% CPU utilization) to AWS without analyzing actual utilization metrics via Migration Evaluator, overspending by 400% on EC2 compute and database licensing.

---

## 7. Actionable Cost Optimization Strategies
1. **Run Migration Evaluator Prior to Migration Architecture:**
   * Deploy the agentless collector 30 to 60 days before migration design.
   * *Why:* Migration Evaluator measures real CPU, memory, and disk IOPS utilization to model **right-sized EC2 instances** rather than 1-to-1 physical core mapping.
   * **The Savings:** Typically reduces projected AWS cloud compute and licensing costs by **30–50%** before the first server is migrated.
2. **Optimize Microsoft & Oracle License Modeling:**
   * Use Migration Evaluator's licensing module to compare **License Included (LI)** vs. **Bring Your Own License (BYOL)** with Dedicated Hosts.
   * **The Savings:** Prevents purchasing unnecessary SQL Server or Oracle cloud licenses.
