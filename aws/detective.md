# AWS Service Cost Research: Amazon Detective

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Detective makes it easy to analyze, investigate, and quickly identify the root cause of potential security issues or suspicious activities. Detective automatically collects log data from your AWS resources (CloudTrail, VPC Flow Logs, GuardDuty findings, EKS audit logs) and uses machine learning, statistical analysis, and graph theory to build an interactive security investigation timeline. Detective is serverless, billing based on the volume of log data ingested into the service.

---

## 2. Billing Mechanics
Amazon Detective charges are calculated monthly based on data ingestion volume:
1.  **Ingested Data Volume:** Billed per GB of log data ingested from CloudTrail, VPC Flow Logs, GuardDuty, and EKS audit logs.
2.  **No Hourly Base Fees:** There are no fixed monthly fees per graph or per account.
3.  **Tiered Volume Discounts:** Per-GB rates drop as monthly ingestion volume increases across an account or organization.

---

## 3. Key Cost Dimensions

### A. Ingestion Tier Rates (us-east-1)
*   **First 1,000 GB (1 TB) / month:** **$2.00 per GB**.
*   **Next 4,000 GB (1 TB to 5 TB) / month:** **$1.00 per GB**.
*   **Next 5,000 GB (5 TB to 10 TB) / month:** **$0.50 per GB**.
*   **Over 10,000 GB (10 TB+) / month:** **$0.25 per GB**.

### B. Math Example
If your account ingests 500 GB of VPC Flow Logs and CloudTrail data into Detective per month:
$$500\text{ GB} \times \$2.00 = \$1,000.00\text{ / month}$$

---

## 4. Detailed Pricing Rates (us-east-1)

| Ingestion Tier (Monthly Volume) | Rate per GB | Price for 100 GB Ingested | Price for 1 TB Ingested |
|---------------------------------|-------------|---------------------------|-------------------------|
| **0 to 1,000 GB (Tier 1)** | **$2.00** | **$200.00** | **$2,000.00** |
| **1,000 to 5,000 GB (Tier 2)** | **$1.00** | **$100.00** | $1,000.00 |
| **5,000 to 10,000 GB (Tier 3)**| **$0.50** | $50.00 | $500.00 |

---

## 5. AWS Free Tier Coverage
*   **Amazon Detective 30-Day Free Trial:** 30-day free trial for all new accounts.
    *   *Cost Preview:* The Detective console displays an estimated average monthly cost during the trial based on active log volumes before any charges begin.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Enabling Detective Across High-Noise Non-Production Accounts:** Enabling Amazon Detective in development or sandbox accounts containing noisy, high-volume Kubernetes clusters or high-traffic load testing endpoints ($2.00/GB).
*   **Duplicate Ingestion Surcharges for Unused Data:** Retaining Detective graphs in accounts where no security incident investigation is actively taking place.

---

## 7. Actionable Cost Optimization Strategies
1.  **Disable Amazon Detective in Non-Production Accounts:**
    *   Enable Amazon Detective only in production and core security accounts.
    *   **The Savings:** Avoids paying $2.00/GB for dev/test VPC Flow Logs and CloudTrail events.
2.  **Use the 30-Day Free Trial Panel to Audit Volumes:** Check the **Usage** tab in the Amazon Detective console during the 30-day free trial to verify which log source (VPC Flow Logs vs. CloudTrail) is driving the highest volume.
3.  **Manage Data Sources via Organization Controls:** Set up **Amazon Detective Delegated Administrator** in AWS Organizations to selectively enable or disable data source ingestion across member accounts.
