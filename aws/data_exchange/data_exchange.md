# AWS Service Cost Research: AWS Data Exchange

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Data Exchange makes it easy to find, subscribe to, and use third-party data in the AWS cloud. Data providers publish datasets in various formats (such as files in Amazon S3, tables in Amazon Redshift, or APIs in Amazon API Gateway). Because the marketplace hosts third-party datasets, the primary cost is determined by the data provider's subscription rates, while AWS charges for underlying storage, data copying, and provider transaction fees.

---

## 2. Billing Mechanics
AWS Data Exchange billing is split between subscribers (data consumers) and providers (data publishers):
1.  **Subscription Fees (Subscribers):** Paid directly to data providers based on their listed pricing (flat rate, tier-based, or custom contracts).
2.  **Data Export & Storage (Subscribers):** Standard AWS charges apply when exporting datasets to your own Amazon S3 buckets or querying via Redshift/APIs.
3.  **Data Grant Fees (Providers):** Billed hourly per active data grant to share data securely across accounts.
4.  **AWS Fulfillment Fee (Providers):** A transaction fee collected by AWS on all subscription sales.

---

## 3. Key Cost Dimensions

### A. Subscription Models (Subscribers)
Data providers configure their own pricing, which is billed directly to your AWS invoice:
*   **Free Subscriptions:** Many public datasets are listed for **free ($0.00)**.
*   **Paid Subscriptions:** Ranging from $10.00/month to $100,000.00+/year. Can be billed monthly, annually, or upfront.
*   *Note: Cancelling a subscription stops auto-renewals for the next cycle, but active subscriptions are non-refundable.*

### B. Data Export & Storage Charges (Subscribers)
Subscribing to a dataset does not copy it to your account automatically. When you export or copy the data:
*   **S3 Storage:** You pay standard S3 capacity rates (**$0.023/GB-month**) for the S3 buckets where you download the files.
*   **Request & API Fees:** Billed standard S3 API requests ($0.005 per 1,000 PUTs) during export tasks.
*   **Network Egress:** Data transfer from the provider's bucket to your bucket in the **same AWS region** is **free ($0.00)**. Cross-region copy tasks incur standard inter-region data transfer fees ($0.01 to $0.02 per GB).

### C. Data Provider Fees
If you act as a data provider publishing datasets:
*   **AWS Marketplace Fee:** AWS collects a transaction fee on subscription revenues (typically **1.2% to 5.0%** depending on contract volume).
*   **Data Grants:** Sharing data directly via AWS Data Exchange data grants costs **$0.02 per hour** (~$14.60/month) per active grant (grant is considered active once accepted by the receiver).

---

## 4. Detailed Pricing Rates (us-east-1)

| Participant Role | Billing Component | Rate (us-east-1) | Details |
|------------------|-------------------|------------------|---------|
| **Subscriber** | **Subscription Price**| Set by Data Provider | Billed on AWS Invoice |
| **Subscriber** | **Intra-Region Copy** | **Free ($0.00)** | Same region copy |
| **Subscriber** | **Cross-Region Copy**| **$0.0100 - $0.0200 / GB** | Standard egress rates |
| **Provider** | **Data Grant** | **$0.0200 / hour** | Per active shared grant |
| **Provider** | **AWS Revenue Split**| **1.2% - 5.0%** | Marketplace transaction fee |

### Example Monthly Cost Calculation
*Workload: A subscriber purchases a historical financial dataset in `eu-west-1` for $1,000.00/month. The dataset is 10 TB (10,000 GB). The subscriber copies the dataset to an S3 bucket in `us-east-1` (cross-region) and keeps it in standard storage for one month.*

*   **Data Subscription Fee:** **$1,000.00**
*   **Cross-Region Egress Cost:**
    $$\text{Egress Fee} = 10,000\text{ GB} \times \$0.02/\text{GB} = \$200.00$$
*   **S3 Storage Fee:**
    $$\text{S3 Cost} = 10,000\text{ GB} \times \$0.023/\text{GB-mo} = \$230.00$$
*   **Total Cost:** **$1,430.00/month** (Subscription + copy + storage).

---

## 5. AWS Free Tier Coverage
*   **AWS Data Exchange:** No free tier is available. Accessing paid subscriptions or creating data grants generates standard charges immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Accidental Auto-Renewals on Expensive Datasets:** Subscribing to a paid dataset for a one-off project and forgetting to disable the auto-renew checkbox in the AWS Marketplace console.
*   **Cross-Region Data Copying:** Initiating a data export task from a provider's bucket in another region to your analytics S3 bucket, generating high egress fees.
*   **Storing Duplicate Raw Datasets:** Copying raw files from Data Exchange into multiple developer S3 buckets without compression or lifecycle controls, accumulating redundant S3 storage fees.

---

## 7. Actionable Cost Optimization Strategies
1.  **Disable Auto-Renewal Immediately After Subscribing:**
    *   If you are purchasing a dataset for a single analysis, go to the **Subscriptions** tab in the AWS Data Exchange console immediately after subscribing and **disable the Auto-Renew toggle**.
    *   **The Savings:** Prevents unintended subscription renewals on expensive datasets.
2.  **Align S3 Bucket Regions for Free Transfers:**
    *   Review the metadata page of the dataset to identify the provider's source AWS region.
    *   Create your target export S3 bucket in the **same AWS region** as the provider's source data.
    *   **The Savings:** Drops data transfer copy fees to **$0.00/GB**, saving $0.02/GB on large ingestion runs.
3.  **Evaluate API Datasets over File Exports:**
    *   If you only need access to a small portion of a massive dataset, subscribe to the **API-based dataset** version instead of the full file export. Query only the specific data fields you need via API Gateway to avoid S3 storage costs.
4.  **Enforce S3 Lifecycle Rules on Export Buckets:**
    *   Configure S3 Lifecycle policies to move raw, imported third-party files to S3 Glacier Deep Archive ($0.00099/GB) or delete them once they have been loaded into your analytics tables.
