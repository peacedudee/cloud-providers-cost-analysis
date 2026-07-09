# AWS Service Cost Research: AWS Config

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Config continuously monitors, records, and audits configuration changes across your AWS resources. It provides a detailed inventory of AWS resource configurations, historical configuration relationships, and automated evaluation against compliance rules (such as CIS benchmarks, PCI-DSS, or security best practices). AWS Config is a consumption-based service billed per configuration item (CI) recorded and per compliance rule evaluation.

---

## 2. Billing Mechanics
AWS Config billing is split into three primary dimensions:
1.  **Configuration Items (CI) Recorded:** Billed whenever a resource is created, updated, or deleted, generating a configuration snapshot item. You can configure recording frequency as **Continuous** (changes recorded immediately) or **Periodic** (changes recorded once per 24 hours).
2.  **Config Rule Evaluations:** Billed whenever an AWS Config Rule or Conformance Pack evaluates a resource against a compliance rule.
3.  **Conformance Pack Evaluations:** Billed per evaluation for rules grouped into a conformance pack.

---

## 3. Key Cost Dimensions

### A. Configuration Item (CI) Recording (us-east-1)
*   **Continuous Recording:** **$0.0030 per Configuration Item recorded**. Billed instantly upon change detection.
*   **Periodic Recording:** **$0.0120 per Configuration Item recorded**. Captures at most one snapshot per 24 hours.

### B. Config Rule Evaluation Volume Tiers (us-east-1)
*   **First 100,000 evaluations / month:** **$0.0010 per evaluation**.
*   **Next 400,000 evaluations / month:** **$0.0008 per evaluation**.
*   **Over 500,000 evaluations / month:** **$0.0005 per evaluation**.
*   **Conformance Pack Evaluations:** **$0.0010 per evaluation**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Config Metric | Recording Type / Tier | Rate (us-east-1) | Price for 100,000 Items / Evaluations |
|---------------|-----------------------|------------------|---------------------------------------|
| **Configuration Item (CI)** | Continuous | **$0.00300** | **$300.00** |
| **Configuration Item (CI)** | Periodic | **$0.01200** | **$1,200.00** |
| **Config Rule Eval (Tier 1)**| Per evaluation | **$0.00100** | **$100.00** |
| **Config Rule Eval (Tier 2)**| Per evaluation | **$0.00080** | $80.00 |
| **Config Rule Eval (Tier 3)**| Per evaluation | **$0.00050** | $50.00 |

### Cost Trade-off: Continuous vs. Periodic Recording
*   **Continuous Recording ($0.003/CI):** Best for stable resources that rarely change (e.g., S3 Buckets, KMS Keys, IAM Roles).
*   **Periodic Recording ($0.012/CI):** Best for high-churn resources that change frequently throughout the day (e.g., Auto Scaling Groups, ENIs, EC2 instances).
*   *Math Example:* An Elastic Network Interface (ENI) churns 10 times a day due to container scaling.
    *   *Continuous:* $$10\text{ changes/day} \times 30\text{ days} \times \$0.003 = \$0.90\text{ / resource-month}$$
    *   *Periodic:* $$1\text{ change/day} \times 30\text{ days} \times \$0.012 = \$0.36\text{ / resource-month}$$
    *   *Result:* Switching to periodic recording saves **60%** on high-churn resources, despite the 4x higher unit rate.

---

## 5. AWS Free Tier Coverage
*   **AWS Config:** No free tier coverage exists for configuration item recording or compliance evaluations.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Recording High-Churn Ephemeral Resources:** Recording CIs continuously for short-lived EKS pods, Elastic Network Interfaces (ENIs), or Auto-Scaling EC2 instances that launch and terminate hundreds of times per day.
    *   *Math:* 10,000 ENI churn events/day = 300,000 CIs/mo = **$900.00/month in continuous CI fees!**
*   **Enabling Multiple Redundant Conformance Packs:** Subscribing to CIS Benchmarks, PCI-DSS, NIST 800-53, and AWS Foundational Best Practices concurrently. A single resource is evaluated multiple times for identical rules, multiplying evaluation costs.
*   **Integrated Storage/Delivery Fees:** Hidden costs from integrated AWS services, such as S3 storage for holding configuration history snapshots and SNS fees for config change alerts.

---

## 7. Actionable Cost Optimization Strategies
1.  **Switch High-Churn Resource Types to Periodic Recording:**
    *   For resource types that experience continuous intraday modifications but only require daily compliance checks (like `AWS::EC2::Instance`, `AWS::AutoScaling::AutoScalingGroup`), change the recording frequency from Continuous to **Periodic**.
    *   **The Savings:** Limits recording to once per 24 hours, cutting CI volume by **70–90%** on active development/autoscaling resources.
2.  **Exclude Ephemeral and Short-Lived Resource Types Entirely:**
    *   Go to AWS Config Settings -> **Resource Types to Record**.
    *   Switch from *Record all current and future resource types* to *Record specific resource types*.
    *   Explicitly **exclude** high-churn resource types: `AWS::EC2::NetworkInterface` (ENI), `AWS::EKS::Pod`, `AWS::ECS::TaskSet`, and `AWS::EC2::SecurityGroup` if they are managed by EKS/ECS orchestrators.
    *   **The Savings:** Slashes AWS Config CI recording bills by **60–85%**.
3.  **Consolidate Compliance Frameworks in Security Hub:**
    *   Enable only **AWS Foundational Security Best Practices** in Security Hub instead of subscribing to multiple redundant conformance packs in Config.
    *   **The Savings:** Reduces rule evaluation volumes by avoiding duplicate tests.
4.  **Set aggressive Lifecycle Policies on S3 Config Buckets:**
    *   Configure lifecycle rules on the target S3 bucket storing AWS Config history files to transition items to Glacier Flexible or Deep Archive after 90 days, or delete them if no historical audits are required.
