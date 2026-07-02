# AWS Service Cost Research: AWS Control Tower

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Control Tower provides the easiest way to set up and govern a secure, multi-account AWS environment (landing zone). It automates the creation of a well-architected landing zone using AWS Organizations, provisions centralized identity (IAM Identity Center), establishes federated access, centralizes logging (CloudTrail and Config), and enforces governance guardrails (preventive SCPs and detective Config rules). Control Tower itself is provided at **no additional charge**.

---

## 2. Billing Mechanics
1.  **Control Tower Orchestrator:** **100% Free ($0.00)**. No charges for landing zone creation, account factory provisioning, or guardrail policy management.
2.  **Underlying Provisioned Services:** You pay standard rates for the underlying AWS infrastructure Control Tower automatically launches to enforce governance:
    *   **AWS Config:** Config Rules and CIs recorded for detective guardrails ($0.003/CI, $0.001/eval).
    *   **AWS CloudTrail:** Audit logging sent to the central log archive bucket ($2.00/100k events).
    *   **Amazon S3:** Storage of centralized audit logs ($0.023/GB-mo).
    *   **AWS Service Catalog:** Account Factory API provisioning ($0.0007/call).

---

## 3. Key Cost Dimensions

| Service Component | Billing Metric | Rate (us-east-1) | Cost Driver Notes |
|-------------------|----------------|------------------|-------------------|
| **Control Tower Orchestrator**| Per account | **Free ($0.00)** | $0 base charge |
| **AWS Config Rules (Guardrails)**| Per evaluation | $0.0010 / eval | Multiplied across accounts/regions |
| **AWS CloudTrail Logs** | Per 100k events | $2.00 | Data logging volume |
| **S3 Audit Bucket Storage** | Per GB-month | $0.023 / GB-mo | Centralized log retention |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Control Tower Service Fee:** $0.00 per month.
*   **Account Factory Operations:** $0.00.

---

## 5. AWS Free Tier Coverage
*   **AWS Control Tower:** Always 100% free for all AWS accounts with no time or landing zone scale limits.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Multi-Region Config Guardrail Explosion (The 20x Multiplier):**
    *   Control Tower automatically deploys AWS Config rules across **all enabled AWS regions** in every child account.
    *   If Control Tower is configured without region restrictions across 20 regions for 50 child accounts:
        $$\text{Evaluations: } 50\text{ accounts} \times 20\text{ regions} \times \text{Config Rules}$$
    *   Generates thousands of dollars per month in **AWS Config Rule evaluation fees ($0.001/eval)** in regions where you run zero actual resources!

---

## 7. Actionable Cost Optimization Strategies
1.  **Enforce Region Deny Guardrails (De-register Unused Regions):**
    *   In Control Tower settings, enable the **Region Denial Guardrail** (`Deny all actions outside approved regions`).
    *   Limit Control Tower governance strictly to your active operating regions (e.g., `us-east-1` and `us-west-2`).
    *   **The Savings:** Slashes AWS Config rule evaluation fees by **80–90%** by disabling Config recording in 15+ unused global regions.
2.  **Set Lifecycle Rules on Central Log Archive S3 Buckets:**
    *   Control Tower centralizes CloudTrail and Config logs into an S3 bucket in the Log Archive account.
    *   Add S3 Lifecycle rules to transition historical audit logs to **S3 Glacier Instant Retrieval ($0.004/GB-mo)** after 30 days and **Glacier Deep Archive ($0.00099/GB-mo)** after 90 days.
    *   **The Savings:** Slashes audit log storage bills by **95%**.
