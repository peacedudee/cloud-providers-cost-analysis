# AWS Service Cost Research: AWS Key Management Service (KMS)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Key Management Service (KMS) allows you to create, manage, and control cryptographic keys used to encrypt data across AWS services and within your applications. KMS is integrated with over 100 AWS services. While KMS is inexpensive per key, API request volume surcharges—particularly when paired with high-frequency S3 or DynamoDB operations—can create massive hidden monthly bills.

---

## 2. Billing Mechanics
KMS billing is calculated monthly based on three primary dimensions:
1. **Key Storage Fees:** Billed hourly per provisioned Customer Managed Key (CMK) ($1.00/month per key). AWS Managed Keys are 100% free.
2. **API Requests:** Billed per 10,000 API operations (e.g., `Encrypt`, `Decrypt`, `GenerateDataKey`) beyond the 20,000 free monthly requests.
3. **Custom Key Stores (CloudHSM / External KMS):** Hourly fees if keys are backed by dedicated CloudHSM clusters or external key managers.

---

## 3. Key Cost Dimensions

### A. Key Storage Types (us-east-1 Rates)
* **AWS Managed Keys (e.g. `aws/s3`, `aws/ebs`, `aws/rds`):**
  * *Rate:* **Free ($0.00 per month)**. Created automatically by AWS services.
* **Customer Managed Keys (CMK):**
  * *Rate:* **$1.00 per month per key** (prorated hourly at ~$0.00137/hour).
  * *Key Rotation:* Enabling automatic key rotation preserves previous key versions. You pay **$1.00 per month** for each rotated key version kept.
* **Imported Key Material (BYOK):** Billed at **$1.00 per month per key**.
* **Asymmetric Keys (RSA/ECC):** Billed at **$1.00 per month per key**.

### B. API Request Charges
* **Symmetric Key Operations (`GenerateDataKey`, `Decrypt`, `Encrypt`, `ReEncrypt`):**
  * *Rate:* **$0.03 per 10,000 requests** ($0.000003 per request / $3.00 per Million).
* **Asymmetric Key Operations (Sign, Verify, Encrypt, Decrypt with RSA/ECC):**
  * *Rate:* **$0.03 to $0.12 per 10,000 requests**.
* **GenerateRandom API:** **$0.03 per 10,000 requests**.

### C. Custom Key Stores (CloudHSM)
* If you configure KMS to use a Custom Key Store backed by AWS CloudHSM:
  * *CloudHSM Instance:* **$1.45 per hour** per HSM instance (~$1,058.50/month per HSM).
  * *Minimum Requirement:* A production CloudHSM cluster requires at least 2 HSMs in multi-AZ ($2,117.00/month base).

---

## 4. Detailed Pricing Rates (us-east-1)

| KMS Component | Billing Metric | Rate (us-east-1) | Price per 1,000,000 Units |
|---------------|----------------|------------------|---------------------------|
| **AWS Managed Keys** | Per key-month | **Free ($0.00)** | $0.00 |
| **Customer Managed Key**| Per key-month | **$1.00** | $1.00 / month |
| **Symmetric API Calls** | Per 10,000 calls | **$0.03** | **$3.00 per 1M calls** |
| **Asymmetric API Calls**| Per 10,000 calls | **$0.03 - $0.12** | $3.00 - $12.00 per 1M calls |
| **CloudHSM Key Store** | Per HSM-hour | **$1.45** | $1,058.50 / month |

---

## 5. AWS Free Tier Coverage
* **Key Storage:** No free tier for Customer Managed Keys ($1.00/mo applies immediately).
* **API Requests:** **20,000 free API requests** per month across all non-custom key operations.

---

## 6. Common Cost Hotspots & Pitfalls
* **The S3 SSE-KMS API Trap (Missing Bucket Keys):**
  * Encrypting an S3 bucket with SSE-KMS *without* enabling S3 Bucket Keys.
  * Every single object upload or download makes a direct `GenerateDataKey` or `Decrypt` call to KMS ($0.03/10K calls).
  * *Math:* 1 Billion S3 GET/PUT requests per month = $3,000.00/month in KMS API fees alone!
* **Over-Provisioning Customer Managed Keys:** Creating a unique Customer Managed Key ($1.00/mo) for every single developer microservice or S3 bucket when a shared application key or AWS Managed Key would meet compliance requirements.
* **Decryption Polling in Microservice Loops:** Decrypting sensitive config properties inside a high-throughput Lambda or ECS container loop on every incoming HTTP request instead of caching the plaintext data key.

---

## 7. Actionable Cost Optimization Strategies
1. **Enable S3 Bucket Keys (SSE-KMS Bucket Keys):**
   * Go to your S3 bucket settings under **Default Encryption**.
   * Turn on **S3 Bucket Keys**.
   * *How it works:* S3 generates a bucket-level key from KMS and caches it locally within S3 for data operations, decreasing KMS API requests by up to **99%**.
   * **The Savings:** Drops a $3,000/month KMS bill down to **$30/month** on high-traffic S3 buckets.
2. **Use AWS Managed Keys (`aws/s3`, `aws/ebs`) Where Allowed:**
   * If regulatory compliance (like HIPAA/PCI) does not strictly require customer-managed key deletion control or custom key policies, use **AWS Managed Keys** ($0.00/month) instead of Customer Managed Keys ($1.00/month).
3. **Cache Data Keys Locally (AWS Encryption SDK):**
   * In application code, use the **AWS Encryption SDK with Data Key Caching**.
   * Cache data keys in application memory for short TTLs (e.g. 5 minutes).
   * **The Savings:** Slashes KMS `GenerateDataKey` API calls by 95% on high-throughput microservices.
4. **Consolidate Customer Managed Keys:** Audit your KMS key inventory. Delete unused or orphaned CMKs that are no longer associated with active EBS volumes, S3 buckets, or RDS databases to save $1.00/month per key.
