# AWS Service Cost Research: AWS Well-Architected Tool

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
The AWS Well-Architected Tool (AWS WA Tool) helps you review your state of workloads and compares them to the latest AWS architectural best practices across the 6 pillars of the AWS Well-Architected Framework: Operational Excellence, Security, Reliability, Performance Efficiency, **Cost Optimization**, and Sustainability. The WA Tool provides step-by-step guidance, actionable improvement plans, and tracking for architectural debt. The tool is provided at **no additional charge**.

---

## 2. Billing Mechanics
1.  **Core Service Pricing:** **100% Free ($0.00)**. No setup fees, workload registration fees, framework review fees, or report export fees.
2.  **Custom Lenses:** Free of charge for creating and sharing custom architectural evaluation lenses within your organization.

---

## 3. Key Cost Dimensions

| Service Feature | Billed Metric | Rate (us-east-1) | Cost Impact |
|-----------------|---------------|------------------|-------------|
| **Workload Definition** | Per workload | **Free ($0.00)** | Unlimited workloads |
| **Architectural Reviews**| Per review run | **Free ($0.00)** | Unlimited reviews |
| **PDF Milestone Exports**| Per report | **Free ($0.00)** | Unlimited exports |
| **Custom Lenses** | Per custom lens | **Free ($0.00)** | Unlimited custom rules |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Well-Architected Tool Base Fee:** $0.00 per month.
*   **Workload Assessments:** $0.00.

---

## 5. AWS Free Tier Coverage
*   **AWS Well-Architected Tool:** Always 100% free for all AWS accounts with no workload limits.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Treating Architecture Reviews as One-Time Events:** Conducting a Well-Architected review at initial application launch but failing to run periodic milestone reviews as application scale or AWS service pricing models evolve.

---

## 7. Actionable Cost Optimization Strategies
1.  **Conduct Quarterly Cost Optimization Pillar Audits:**
    *   Use the AWS WA Tool to perform structured reviews focused specifically on the **Cost Optimization Pillar** (Questions COST 1 through COST 11).
    *   Audit key questions: *How do you evaluate new services? How do you select resource types, sizes, and numbers? How do you manage licensing?*
    *   **The Savings:** Systematically identifies architectural misalignments (e.g. over-provisioned EC2 fleets vs. serverless Lambda/Fargate).
2.  **Deploy Custom Lenses for Enterprise Cost Guardrails:**
    *   Create a **Custom Lens** in the AWS WA Tool containing mandatory corporate cost standards (e.g. "Is S3 Lifecycle enabled on all buckets?", "Is Spot used for non-prod?").
    *   Share the custom lens across all engineering teams via AWS RAM.
    *   **The Savings:** Enforces cost-first engineering design before code is deployed to production.
