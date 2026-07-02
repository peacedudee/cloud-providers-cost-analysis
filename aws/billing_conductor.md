# AWS Service Cost Research: AWS Billing Conductor

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Billing Conductor is a customizable billing management service designed for AWS Solution Providers, Managed Service Providers (MSPs), and large Enterprise Organizations. It enables users to customize pro-forma billing data, construct custom pricing structures (e.g., custom tier markups or enterprise discounts), group AWS accounts into custom billing groups, and generate pro-forma Cost and Usage Reports (CUR) for tenant chargebacks. Billing Conductor is billed on a per-account monthly subscription tier.

---

## 2. Billing Mechanics
1. **Account Assignment Billing (Tiered Monthly Fee):** Billed monthly per AWS account assigned to an active Billing Conductor Billing Group.
2. **Billing Transfer Feature:** Billed per AWS Organization using customer-managed pricing plans ($50.00 per month).
3. **Pro-Forma Data Storage:** Generating pro-forma Cost and Usage Reports (CUR) incurs standard Amazon S3 storage fees.

---

## 3. Key Cost Dimensions

### A. Account Billing Group Volume Tiers (us-east-1)
* **First 500 accounts / month:** **$8.25 per account per month**.
* **Next 1,500 accounts (501 to 2,000) / month:** **$6.75 per account per month**.
* **Over 2,000 accounts / month:** **$5.25 per account per month**.

### B. Mathematical Cost Calculation Example
*Scenario: An MSP manages 50 client AWS accounts assigned across custom billing groups in AWS Billing Conductor.*

1. **Account Assignment Charge:**
   $$\text{Monthly Fee} = 50\text{ accounts} \times \$8.25 = \$412.50\text{ / month}$$
2. **Total Monthly Cost:** **$412.50 / month**

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Conductor Feature | Volume Tier | Monthly Rate per Account | Cost for 10 Accounts / Month |
|---------------------------|-------------|--------------------------|------------------------------|
| **Account Assignment** | 0 – 500 accounts | **$8.25** | **$82.50 / month** |
| **Account Assignment** | 501 – 2,000 accounts | **$6.75** | $67.50 / month |
| **Account Assignment** | > 2,000 accounts | **$5.25** | $52.50 / month |
| **Billing Transfer Fee** | Per Organization | **$50.00** | $50.00 flat |

---

## 5. AWS Free Tier Coverage
* **AWS Billing Conductor Free Tier:** Includes a **62-day free trial** for your first billing group created in an AWS account.

---

## 6. Common Cost Hotspots & Pitfalls
* **Assigning Internal Enterprise Accounts to Billing Groups:** Adding internal developer or departmental accounts to Billing Conductor ($8.25/account-mo) when internal showback reporting can be accomplished for **$0.00** using AWS Cost Categories and Cost Allocation Tags.

---

## 7. Actionable Cost Optimization Strategies
1. **Use Free AWS Cost Categories ($0.00) for Internal Showbacks:**
   * For internal corporate showbacks/chargebacks, do not create Billing Conductor groups.
   * Use **AWS Cost Categories** + **Cost Allocation Tags ($0.00)** in the main Billing Console.
   * **The Savings:** Avoids paying $8.25/account-month on internal accounts ($99.00/year per account).
2. **Reserve Billing Conductor for External MSP Invoicing:** Limit AWS Billing Conductor strictly to commercial MSPs / Solution Providers who require custom margin calculations or pro-forma invoices for paying external clients.
3. **Audit and Remove Inactive Accounts Monthly:** Audit billing groups regularly to remove closed or decommissioned member accounts immediately.
