# AWS Service Cost Research: AWS Config

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Config continuously monitors, records, and audits configuration changes across your AWS resources. It provides a detailed inventory of AWS resource configurations, historical configuration relationships, and automated evaluation against compliance rules (such as CIS benchmarks, PCI-DSS, or security best practices). AWS Config is a consumption-based service billed per configuration item recorded and per compliance rule evaluation.

---

## 2. Billing Mechanics
AWS Config billing is split into two primary dimensions:
1.  **Configuration Items (CI) Recorded:** Billed whenever a resource is created, updated, or deleted, generating a configuration snapshot item.
2.  **Config Rule Evaluations:** Billed whenever an AWS Config Rule or Conformance Pack evaluates a resource against a compliance rule.

---

## 3. Key Cost Dimensions

### A. Configuration Item (CI) Recording Pricing (us-east-1)
*   **The Rate:** **$0.003 per Configuration Item recorded**.
*   *Math Example:* If an active account records 100,000 resource configuration changes per month:
    $$100,000 \times \$0.003 = \$300.00\text{ / month}$$

### B. Config Rule Evaluation Volume Tiers (us-east-1)
*   **First 100,000 evaluations / month:** **$0.0010 per evaluation**.
*   **Next 400,000 evaluations / month:** **$0.0008 per evaluation**.
*   **Over 500,000 evaluations / month:** **$0.0005 per evaluation**.
*   **Conformance Pack Evaluations:** **$0.0010 per evaluation**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Config Metric | Billed Unit | Rate (us-east-1) | Price for 100,000 Items / Evaluations |
|---------------|-------------|------------------|---------------------------------------|
| **Configuration Item (CI)** | Per item recorded | **$0.0030** | **$300.00** |
| **Config Rule Eval (Tier 1)**| Per evaluation | **$0.0010** | **$100.00** |
| **Config Rule Eval (Tier 2)**| Per evaluation | **$0.0008** | $80.00 |

---

## 5. AWS Free Tier Coverage
*   **AWS Free Tier:** None. AWS Config offers no free tier for CIs or rule evaluations.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Recording High-Churn Ephemeral Resources (The ENI / Pod Trap):**
    *   Recording CIs for short-lived EKS pods, Elastic Network Interfaces (ENIs), or Auto-Scaling EC2 instances that launch and terminate hundreds of times per day.
    *   *Math:* 5,000 ENI churn events/day = 150,000 CIs/mo = **$450.00/month in CI fees alone for temporary network adapters!**
*   **Enabling Multiple Redundant Conformance Packs:**
    *   Enabling CIS Benchmarks, PCI-DSS, NIST 800-53, and AWS Foundational Best Practices simultaneously.
    *   The same S3 bucket or EC2 instance is evaluated 4 separate times for the same rule, quadrupling evaluation charges ($0.001/eval).

---

## 7. Actionable Cost Optimization Strategies
1.  **Exclude High-Churn Ephemeral Resource Types:**
    *   Go to AWS Config Settings -> **Resource Types to Record**.
    *   Switch from *Record all current and future resource types* to *Record specific resource types*.
    *   Explicitly **exclude** high-churn resource types: `AWS::EC2::NetworkInterface` (ENI), `AWS::EKS::Pod`, `AWS::AutoScaling::AutoScalingGroup`, and `AWS::EC2::SecurityGroup`.
    *   **The Savings:** Slashes AWS Config CI recording bills by **60–85%**.
2.  **Switch to Periodic Recording Mode (24-Hour Snapshots):**
    *   Change the recording frequency from Continuous to **Periodic (24-hour)** for non-critical resources.
    *   Instead of recording every minor configuration edit instantly ($0.003/CI), Config records 1 summary snapshot per day.
    *   **The Savings:** Reduces CI volume by **90%** on active development resources.
3.  **Consolidate Compliance Frameworks in Security Hub:** Enable only **AWS Foundational Security Best Practices** in Security Hub instead of subscribing to 4 redundant compliance packs.
