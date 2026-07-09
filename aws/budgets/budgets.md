# AWS Service Cost Research: AWS Budgets

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Budgets enables users to set custom spend and usage budgets that alert when costs or usage exceed (or are forecasted to exceed) budgeted thresholds. It tracks costs, resource usage, Savings Plans utilization, and Reserved Instance (RI) coverage. AWS Budgets also supports automated **Budget Actions** (e.g., applying IAM SCPs or stopping EC2/RDS instances when budget limits are breached). Base budget tracking, monitoring, and alerts are **100% free**.

---

## 2. Billing Mechanics
1. **Budget Creation & Tracking:** **100% Free ($0.00)** for standard cost, usage, RI coverage, and Savings Plans utilization budgets.
2. **Budget Actions Execution:** $0.10 per automated action execution.
3. **Email Budget Reports:** $0.01 per report email delivered.

---

## 3. Key Cost Dimensions

| Service Feature | Billed Metric | Rate (us-east-1) | Cost Impact |
|-----------------|---------------|------------------|-------------|
| **Cost & Usage Budgets** | Per active budget | **Free ($0.00)** | **$0.00** |
| **RI / Savings Plans Budgets** | Per active budget | **Free ($0.00)** | **$0.00** |
| **Budget Action Execution** | Per action run | **$0.10** | $0.10 per trigger |
| **Email Budget Reports** | Per report sent | **$0.01** | $0.01 per email |

---

## 4. Detailed Pricing Rates (us-east-1)

* **Standard Budget Base Fee:** $0.00 per month.
* **Budget Alerts (Email / SNS):** $0.00.
* **Budget Action Execution Fee:** $0.10 per execution.

---

## 5. AWS Free Tier Coverage
* **AWS Budgets:** Creating custom cost, usage, RI, and Savings Plans budgets is always 100% free for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
* **Setting Budgets Without Forecasted Alerts:** Configuring budget alerts based strictly on *actual* spend (e.g., alert at 100% actual spend). By the time actual spend reaches 100%, the budget breach has already occurred.
* **Ignoring Reservation & Savings Plan Utilization:** Failing to monitor RI/SP utilization budgets, allowing purchased compute commitments to drop below 90% utilization unnoticed.

---

## 7. Actionable Cost Optimization Strategies
1. **Configure Forecasted Alerts (Detect Overages BEFORE They Occur):**
   * Set budget alerts using **Forecasted Spend** rather than actual spend.
   * Threshold rule: *Notify team when Forecasted Spend exceeds 100% of monthly budget*.
   * **The Savings:** Provides FinOps teams 10–15 days advance warning to adjust infrastructure before the billing cycle completes.
2. **Set Up Reserved Instance & Savings Plans Utilization Budgets:**
   * Create an **RI/SP Utilization Budget** with a **90% threshold**.
   * *Benefit:* If utilization drops to 85%, purchased commitments are wasting money.
   * **The Savings:** Prompts immediate action to reallocate workloads or modify commitments.
3. **Attach Automated Budget Actions for Developer Sandbox Accounts:**
   * Attach an **AWS Budget Action** to non-production accounts.
   * Rule: *When actual spend hits 100% of monthly budget ($500), execute an IAM policy denying creation of new resources or stopping dev EC2/RDS instances*.
   * **The Savings:** Enforces hard spend caps in non-production environments automatically.
