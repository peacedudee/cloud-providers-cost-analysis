# AWS Service Cost Research: AWS Supply Chain

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Supply Chain is a cloud-based, machine learning-powered supply chain application that connects data from existing enterprise resource planning (ERP) and supply chain systems (SAP, Oracle, Infor) without requiring system migration. It unifies supply chain data into a contextual data lake, provides real-time ML inventory visibility, generates actionable rebalancing insights, automates demand planning forecasts, and tracks N-tier vendor risks. AWS Supply Chain is billed per active user license and module.

---

## 2. Billing Mechanics
1.  **Base Platform (Supply Chain Insights):** Billed monthly per active user license ($10.00 per user per month).
2.  **AWS Supply Chain Demand Planning Add-on:** Billed monthly per active user ($30.00 per user per month).
3.  **AWS Supply Chain N-Tier Visibility Add-on:** Billed monthly per active user ($20.00 per user per month).
4.  **Data Storage:** Standard S3 storage rates apply ($0.023 per GB-month).

---

## 3. Key Cost Dimensions

| AWS Supply Chain Module | Included Capabilities | Monthly Rate per User | Cost for 100 Users / Month |
|-------------------------|-----------------------|-----------------------|-----------------------------|
| **Supply Chain Insights** | Data Lake, Risk Map | **$10.00 / user-mo** | **$1,000.00 / month** |
| **Demand Planning Add-on**| ML Forecasting | **$30.00 / user-mo** | **$3,000.00 / month** |
| **N-Tier Visibility Add-on**| Vendor Portal | **$20.00 / user-mo** | **$2,000.00 / month** |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Base Insights Rate:** $10.00 per user per month.
*   **Demand Planning Rate:** $30.00 per user per month.
*   **N-Tier Visibility Rate:** $20.00 per user per month.

---

## 5. AWS Free Tier Coverage
*   **AWS Supply Chain Free Tier:** Includes a **60-day free trial** of Supply Chain Insights for up to 10 users.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Assigning All Add-on Licenses to Every Employee:** Assigning full Demand Planning ($30/mo) and N-Tier ($20/mo) licenses to warehouse staff who only need basic inventory visibility ($10/mo).

---

## 7. Actionable Cost Optimization Strategies
1.  **Role-Based License Tier Allocation:**
    *   Assign full Demand Planning ($30/mo) and N-Tier ($20/mo) add-on licenses strictly to specialized inventory planners and procurement managers.
    *   Keep general warehouse operators, logistics coordinators, and executives on Base Insights licenses ($10/mo).
    *   **The Savings:** Slashes per-user licensing overhead by **66–83%**.
2.  **Audit Active User Licenses Monthly:** Remove licenses for former employees or contractors during monthly access reviews to avoid $10–$60/mo per dormant account.
