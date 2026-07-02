# AWS Service Cost Research: AWS Secrets Manager

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Secrets Manager helps you protect secrets needed to access your applications, services, and IT resources. The service enables you to easily rotate, manage, and retrieve database credentials, API keys, and other secrets throughout their lifecycle. While individual secret costs are low ($0.40/mo), application polling habits and improper storage choices can generate high API request bills.

---

## 2. Billing Mechanics
Secrets Manager billing is split into two primary dimensions:
1.  **Secret Storage:** Billed hourly per secret stored ($0.40 per secret per month).
2.  **API Requests:** Billed per 10,000 API calls (`GetSecretValue`, `DescribeSecret`, `PutSecretValue`).
3.  **Automatic Rotation Compute:** Secret rotation uses an underlying AWS Lambda function, billed at standard Lambda execution rates.

---

## 3. Key Cost Dimensions

### A. Storage & Request Pricing (us-east-1)
*   **Secret Storage Fee:** **$0.40 per secret per month** (prorated hourly at ~$0.00055/hour).
    *   *Note:* Secrets marked for deletion (during the recovery window) are **free ($0.00)**.
*   **API Request Fee:** **$0.05 per 10,000 API calls** ($0.000005 per call).
    *   *Price per Million calls:* **$5.00 per Million API requests**.

### B. Secret Rotation Costs
*   When automatic rotation is enabled (e.g. for RDS database passwords), Secrets Manager triggers an AWS Lambda function.
*   *The Charge:* Standard AWS Lambda compute fees ($0.00001667/vCPU-sec) plus 2 to 4 KMS API operations per rotation execution.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Component | Rate (us-east-1) | Price per 1,000 / Million Units | Notes |
|-------------------|------------------|---------------------------------|-------|
| **Secret Storage** | **$0.40 / secret-month** | $400.00 per 1,000 secrets/mo | Prorated hourly |
| **API Requests** | **$0.05 / 10,000 requests** | **$5.00 per 1 Million requests** | `GetSecretValue` calls |
| **Pending Deletion**| **Free ($0.00)** | $0.00 | During recovery window |

---

## 5. AWS Free Tier Coverage
*   **Secrets Manager Free Tier:** 30-day free trial containing 10,000 API requests and 1 free secret per account.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Fetching Secrets on Every Incoming Request (Uncached Polling):**
    *   Calling `GetSecretValue` inside an API Gateway / Lambda handler or Web Server request loop for every single HTTP request.
    *   *Math:* A service handling 100 Million requests per month calling `GetSecretValue` uncached generates:
        $$\frac{100,000,000}{10,000} \times \$0.05 = \$500.00\text{ / month in API fees}$$
*   **Storing Non-Sensitive Configuration Data in Secrets Manager:** Creating Secrets Manager entries ($0.40/secret/mo) for non-sensitive application settings (like `MAX_PAGE_SIZE=50`, `FEATURE_FLAG_X=true`, or public API URLs).
*   **Creating Per-User Secrets:** Provisioning a separate Secrets Manager secret ($0.40/mo) for thousands of individual application users instead of storing encrypted user tokens in DynamoDB or RDS.

---

## 7. Actionable Cost Optimization Strategies
1.  **Use AWS SSM Parameter Store for Non-Sensitive Configs:**
    *   Migrate all non-sensitive configuration settings, feature flags, and environment variables to **AWS Systems Manager (SSM) Parameter Store**.
    *   **Standard SSM Parameters are 100% Free ($0.00)** for storage and standard throughput.
    *   **The Savings:** Saves $0.40/month per parameter and eliminates request fees.
2.  **Cache Secrets in Application Memory (Use Caching Client Libraries):**
    *   Never call `GetSecretValue` directly inside request loops.
    *   Use official **AWS Secrets Manager Caching Libraries** (available for Java, Python, Go, Node.js) or cache secrets in global variables in AWS Lambda.
    *   Set a cache TTL (e.g., 15 minutes).
    *   **The Savings:** Slashes API request volume by **99.9%**, dropping a $500/month API bill down to pennies.
3.  **Consolidate Multi-Key Credentials into a Single JSON Secret:**
    *   Instead of creating separate secrets for `DB_HOST`, `DB_USER`, `DB_PASS`, and `DB_NAME` ($1.60/month for 4 secrets), package them into a single JSON object inside one secret ($0.40/month).
    *   **The Savings:** Reduces storage costs by 75%.
4.  **Purge Stale Secrets:** Audit Secrets Manager console. Delete secrets that are no longer referenced by active applications.
