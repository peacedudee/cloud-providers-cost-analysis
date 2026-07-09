# AWS Service Cost Research: AWS Cost Explorer

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Cost Explorer has an easy-to-use interface that lets you visualize, understand, and manage your AWS costs and usage over time. It provides custom reports, cost forecasts, service category breakdowns, tag-based filtering, and automated recommendations for Savings Plans and Reserved Instances. The Cost Explorer web console is **100% free**, while programmatic API queries and advanced granular data collections incur usage fees.

---

## 2. Billing Mechanics
1.  **Cost Explorer Console UI:** **100% Free ($0.00)**. Unlimited access to view graphs, filter data, and export CSV reports in the AWS Console.
2.  **Cost Explorer API Requests:** Billed per API call made to the Cost Explorer API (`$0.01 per request`). For custom billing views that combine multiple data sources, you are charged **$0.01 per source per request**.
3.  **Hourly & Resource-Level Granularity:** Billed at **$0.00000033 per usage record per day** (which equates to approximately **$0.01 per 1,000 usage records monthly**).
4.  **AWS Cost & Usage Reports (CUR 2.0):** **100% Free** generation (S3 storage and transfer fees apply for the report files).

---

## 3. Key Cost Dimensions

| Service Access Mode | Billing Metric | Rate (us-east-1) | Price for 100,000 Records / Queries |
|---------------------|----------------|------------------|-------------------------------------|
| **Cost Explorer Web UI** | Per console session| **Free ($0.00)** | **$0.00** |
| **Cost Explorer API (Standard)**| Per API request | **$0.01000** | **$1,000.00** |
| **Cost Explorer API (Custom View)**| Per source-request| **$0.01000** | Billed per source included |
| **Hourly Granularity Data**| Per usage record-day| **$0.00000033** | **$0.033** (approx. $1.00 per 100k records/mo) |
| **AWS CUR 2.0 Files** | Per report file | **Free ($0.00)** | $0.00 (S3 storage fees apply) |

---

## 4. Detailed Pricing Rates (us-east-1)
*   **Console UI Base Fee:** $0.00 per month.
*   **API Query Rate:** $0.01 per request (`GetCostAndUsage`, `GetReservationCoverage`, `GetSavingsPlansUtilization`).
*   **Custom Billing View Query Rate:** $0.01 multiplied by the number of data sources queried.

---

## 5. AWS Free Tier Coverage
*   **AWS Cost Explorer:** Web console access and basic daily/monthly cost reporting are always 100% free for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Automated Polling of the Cost Explorer API:**
    *   Writing internal dashboard scripts or third-party FinOps integrations that query `GetCostAndUsage` every few minutes across multiple accounts and dimensions.
    *   *Math:* A script queries 10 custom billing views (each combining 5 data sources) every 5 minutes:
        $$\text{Queries/hr: } 12 \times 10\text{ views} = 120\text{ queries/hour}$$
        $$\text{Billed Requests/hr: } 120 \times 5\text{ sources} = 600\text{ request-units/hour}$$
        $$\text{Monthly Cost: } 600 \times \$0.01 \times 24\text{ hours} \times 30\text{ days} = \$4,320.00\text{ / month in API fees!}$$
*   **Enabling Hourly Granularity on High-Scale Fleets:**
    *   Enabling hourly and resource-level granularity in accounts running thousands of short-lived ephemeral tasks (like Kubernetes pods or ECS tasks), generating millions of daily usage records.

---

## 7. Actionable Cost Optimization Strategies
1.  **Use AWS Cost & Usage Reports (CUR 2.0) + Athena Over Cost Explorer API:**
    *   Do not query the Cost Explorer API ($0.01/req) for automated internal dashboards or custom reporting tools.
    *   Enable **AWS Cost and Usage Reports (CUR 2.0)** delivered to Amazon S3.
    *   Query CUR data using **Amazon Athena** or load it into Quicksight.
    *   **The Savings:** Slashes automated cost data ingestion fees by **95%+** (paying standard S3 storage and minor Athena SQL query fees instead of per-API call charges).
2.  **Combine Custom View Sources in Single Queries:**
    *   If you must use custom billing views, design dashboards to fetch pre-aggregated data rather than querying all individual source views dynamically.
3.  **Leverage Free Savings Plans & RI Recommendations:**
    *   Use the free Cost Explorer **Savings Plans Recommendations** engine in the console to evaluate 1-year or 3-year Compute Savings Plans, unlocking up to **66%** compute discounts.
4.  **Enable Cost Allocation Tags:**
    *   Activate cost allocation tags (e.g. `Environment`, `Owner`, `Project`) in the Billing Console to slice Cost Explorer data by business unit without running programmatic custom view filters.
