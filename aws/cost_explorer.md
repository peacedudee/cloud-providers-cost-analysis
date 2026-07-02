# AWS Service Cost Research: AWS Cost Explorer

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Cost Explorer has an easy-to-use interface that lets you visualize, understand, and manage your AWS costs and usage over time. It provides custom reports, cost forecasts, service category breakdowns, tag-based filtering, and automated recommendations for Savings Plans and Reserved Instances. The Cost Explorer web console is **100% free**, while programmatic API queries incur minor request fees.

---

## 2. Billing Mechanics
1.  **Cost Explorer Console UI:** **100% Free ($0.00)**. Unlimited access to view graphs, filter data, and export CSV reports in the AWS Console.
2.  **Cost Explorer API Requests:** Billed per API call made to the Cost Explorer API (`$0.01 per API request`).
3.  **Hourly & Resource-Level Granularity:** Billed at `$0.01 per 1,000 usage records` per month.
4.  **AWS Cost & Usage Reports (CUR 2.0):** **100% Free** generation (S3 storage fees apply).

---

## 3. Key Cost Dimensions

| Service Access Mode | Billing Metric | Rate (us-east-1) | Price for 1,000 Queries / Records |
|---------------------|----------------|------------------|-----------------------------------|
| **Cost Explorer Web UI** | Per console session| **Free ($0.00)** | **$0.00** |
| **Cost Explorer API** | Per API request | **$0.01** | **$10.00** |
| **Hourly Granularity Data**| Per 1,000 records| **$0.01** | $0.01 per 1k records/mo |
| **AWS CUR 2.0 Files** | Per report file | **Free ($0.00)** | $0.00 (S3 storage fees apply) |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Console UI Base Fee:** $0.00 per month.
*   **API Query Rate:** $0.01 per request (`GetCostAndUsage`, `GetReservationCoverage`, `GetSavingsPlansUtilization`).

---

## 5. AWS Free Tier Coverage
*   **AWS Cost Explorer:** Web console access and basic daily/monthly cost reporting are always 100% free for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Automated Polling of the Cost Explorer API:**
    *   Writing internal dashboard scripts or third-party FinOps integrations that query `GetCostAndUsage` every 5 minutes across multiple dimensions.
    *   *Math:* 12 queries/hour × 24 hours × 30 days = 8,640 requests = **$86.40/month in API fees for a basic polling script!**

---

## 7. Actionable Cost Optimization Strategies
1.  **Use AWS Cost & Usage Reports (CUR 2.0) + Athena Over Cost Explorer API:**
    *   Do not query the Cost Explorer API ($0.01/req) for automated internal dashboards or custom reporting tools.
    *   Enable **AWS Cost and Usage Reports (CUR 2.0)** delivered to Amazon S3.
    *   Query CUR data using **Amazon Athena**.
    *   **The Savings:** Slashes automated cost data ingestion fees by **95%** (paying pennies for Athena SQL queries vs $0.01 per API call).
2.  **Leverage Free Savings Plans & RI Recommendations:** Use the free Cost Explorer **Savings Plans Recommendations** engine to evaluate 1-year or 3-year Compute Savings Plans, unlocking up to 66% compute discounts.
3.  **Enable Cost Allocation Tags:** Activate cost allocation tags (e.g. `Environment`, `Owner`, `Project`) in Billing Console to slice Cost Explorer data by business unit.
