# Cost-Cutting Playbook: AWS Secrets Manager

> **Companion File:** [secrets_manager.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/secrets_manager/secrets_manager.md)  
> **Last Updated:** July 2026

---

## Executive Summary

AWS Secrets Manager protects database credentials, API keys, and OAuth tokens, offering automated rotation and centralized compliance auditing. Billing is based on **Secret Storage** ($0.40 per secret per month) and **API Requests** ($0.05 per 10,000 requests = $5.00/M).

Key cost traps include:
1. **Un-cached Polling in API Request Loops:** Calling `GetSecretValue` on every incoming HTTP request ($5.00/M calls) instead of caching secret values in application RAM or using official client caching libraries.
2. **Storing Non-Sensitive Configuration Data:** Storing public URLs, feature flags, or non-sensitive settings in Secrets Manager ($0.40/secret/mo) instead of **AWS Systems Manager (SSM) Parameter Store (100% FREE)**.
3. **Per-User Secret Proliferation:** Creating individual Secrets Manager entries ($0.40/mo each) for thousands of end-users instead of storing encrypted user tokens in DynamoDB or RDS.

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **40–95% reduction in total Secrets Manager spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Migrate Non-Sensitive Configs to AWS SSM Parameter Store:** Standard parameters are **100% FREE ($0.00)** for storage and standard throughput ($0.40/secret/mo saved).
2. **Implement Local Secret Caching (`aws-secretsmanager-caching-python`):** Caches `GetSecretValue` calls in RAM with a 15-minute TTL, cutting API request volume and cost by **up to 99.9%**.
3. **Consolidate Multi-Key Credentials into a Single JSON Secret:** Packages `DB_HOST`, `DB_USER`, `DB_PASS`, and `DB_NAME` into 1 JSON secret ($0.40/mo) instead of 4 individual secrets ($1.60/mo — **75% savings**).

---

## Strategy Categories

### 1. Secret Storage & Service Selection

#### 1. Migrate Non-Sensitive Configurations to AWS SSM Parameter Store
- **What:** Move non-sensitive application settings, public API endpoints, feature flags, and environment configurations from AWS Secrets Manager to **AWS Systems Manager (SSM) Parameter Store**.
- **Why It Saves Money:**
  - **Secrets Manager:** **$0.40 per secret per month** + $5.00 per Million API requests.
  - **SSM Parameter Store (Standard):** **100% FREE ($0.00)** for storage and standard throughput API requests.
  - Migrating 500 non-sensitive config keys saves **$200.00/month ($2,400.00/year)** in storage fees alone!
- **Detailed Implementation Steps:**
  1. Audit secrets to identify non-sensitive parameters.
  2. Re-create parameters in SSM Parameter Store using Terraform:
     ```hcl
     resource "aws_ssm_parameter" "api_endpoint" {
       name  = "/config/prod/api_endpoint"
       type  = "String"
       value = "https://api.company.com/v1"
     }
     ```
  3. Delete secrets in Secrets Manager: `aws secretsmanager delete-secret --secret-id sec-id --recovery-window-in-days 7`.
- **Estimated Savings:** **100% savings** on storage and API fees for migrated parameters ($0.40/mo saved per item).
- **Risk Level:** Zero risk (SSM Parameter Store is a fully managed AWS native configuration store).
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Parameter audit for non-sensitive data classification.

#### 2. Consolidate Related Credentials into Single JSON Secrets
- **What:** Package multiple related database parameters (`DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASS`, `DB_NAME`) into a **Single JSON Secret** object instead of creating 5 individual secret items.
- **Why It Saves Money:** Storing 5 parameters in separate secrets costs 5 × $0.40 = **$2.00/month**. Packaging them in 1 JSON secret costs **$0.40/month** — an immediate **75% storage savings**!
- **Detailed Implementation Steps:**
  1. Format secret value as JSON string:
     ```json
     {
       "engine": "postgres",
       "host": "prod-db.xxxx.us-east-1.rds.amazonaws.com",
       "port": 5432,
       "username": "admin",
       "password": "SuperSecretPassword123!"
     }
     ```
  2. Parse JSON object in application code after `GetSecretValue` retrieval.
- **Estimated Savings:** **75–80% reduction** in secret storage fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Application JSON secret parser.

---

### 2. Application Caching & API Request Reduction

#### 3. Implement Local Secret Caching in Application Client Libraries
- **What:** Integrate official **AWS Secrets Manager Caching Libraries** (`aws-secretsmanager-caching-python`, `aws-secretsmanager-caching-java`, `aws-secretsmanager-caching-go`) or cache secret strings in global Lambda execution memory with a 15-minute TTL.
- **Why It Saves Money:** Calling `GetSecretValue` uncached on every API Gateway / Lambda request (e.g. 50M requests/month) costs **$250.00/month in Secrets Manager API fees**. Caching the secret in memory for 15 minutes drops API calls to 2,880/month (**$0.01/month**) — a **99.99% cost reduction**!
- **Detailed Implementation Steps:**
  1. Add caching client in Python code:
     ```python
     from aws_secretsmanager_caching import SecretCache, SecretCacheConfig
     import boto3

     client = boto3.client('secretsmanager')
     cache_config = SecretCacheConfig(max_age=900)  # 15 minute cache TTL
     cache = SecretCache(config=cache_config, client=client)

     def get_db_credentials():
         secret_string = cache.get_secret_string('prod/db/credentials')
         return json.loads(secret_string)
     ```
- **Estimated Savings:** **99.9% reduction** in Secrets Manager API request fees.
- **Risk Level:** Zero risk (standard AWS SDK caching pattern; cache invalidates automatically on secret rotation).
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Caching SDK dependency added to application build.

---

### 3. Architecture & User Data Offloading

#### 4. Store Per-User Encrypted Tokens in DynamoDB / RDS (Not Secrets Manager)
- **What:** Store user-specific API keys, OAuth refresh tokens, or user secrets in an encrypted **Amazon DynamoDB** table (or RDS database) using field-level KMS encryption, rather than creating a separate Secrets Manager secret for every user.
- **Why It Saves Money:** Provisioning 10,000 user secrets in Secrets Manager costs **$4,000.00/month**. Storing 10,000 encrypted rows in DynamoDB costs **<$2.00/month** total!
- **Detailed Implementation Steps:**
  1. Create DynamoDB table with KMS Server-Side Encryption enabled.
  2. Store user tokens as encrypted attribute items in DynamoDB.
- **Estimated Savings:** **99.9% cost reduction** for user-level secret storage.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / Architect
- **Prerequisites:** User token data model update.

---

### 4. Secret Rotation & Lambda Optimization

#### 5. Right-Size Secret Rotation Lambda Functions
- **What:** Configure the underlying AWS Lambda function responsible for automatic secret rotation (e.g. RDS password rotation) to use **128 MB RAM** and a **15-second execution timeout**.
- **Why It Saves Money:** Prevents over-provisioned 1024 MB rotation Lambdas from incurring unnecessary execution charges during daily/monthly password rotation runs.
- **Detailed Implementation Steps:**
  1. Set memory size = 128 MB on rotation Lambda functions in Terraform.
- **Estimated Savings:** 50–80% rotation Lambda compute fee reduction.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Lambda configuration access.

---

### 5. Cleanup & Lifecycle Management

#### 6. Purge Stale Secrets (Utilize 7-to-30 Day Deletion Recovery Window)
- **What:** Identify and delete stale or abandoned secrets (`aws secretsmanager list-secrets`) that have not been read in > 90 days.
- **Why It Saves Money:** Secrets marked for deletion (`aws secretsmanager delete-secret`) enter a recovery window (7 to 30 days) during which **storage is 100% FREE ($0.00)**.
- **Detailed Implementation Steps:**
  1. List un-accessed secrets via CLI:
     ```bash
     aws secretsmanager list-secrets \
       --query "SecretList[?LastAccessedDate<='2026-04-01'].{Name:Name,ARN:ARN,LastAccess:LastAccessedDate}"
     ```
  2. Schedule secret deletion:
     ```bash
     aws secretsmanager delete-secret --secret-id prod/legacy-api-key --recovery-window-in-days 7
     ```
- **Estimated Savings:** **$0.40 per month saved** per deleted secret.
- **Risk Level:** Low (7-day recovery window allows restoring secret if inadvertently deleted).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudTrail `GetSecretValue` access log audit.

#### 7. Audit Replica Secrets in Unused Secondary Regions
- **What:** Delete secret replication regions (`aws secretsmanager remove-regions-from-replication`) where secondary region DR read access is no longer required.
- **Why It Saves Money:** Replicating a secret to secondary regions bills **$0.40/month per region per secret**.
- **Detailed Implementation Steps:**
  1. Remove secret replication regions via CLI.
- **Estimated Savings:** $0.40/mo saved per secret per removed region.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Disaster recovery architecture audit.

---

### 6. Observability & Governance

#### 8. Utilize Free Trial Allocations for New Accounts
- **What:** Leverage 30-day free trial (10,000 API requests and 1 free secret) for sandbox testing.
- **Why It Saves Money:** Baseline testing savings.
- **Detailed Implementation Steps:**
  1. Review Billing Console free trial status.
- **Estimated Savings:** Free baseline testing.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

#### 9. Enforce CloudWatch Alarms for Uncached `GetSecretValue` Surges
- **What:** Put CloudWatch alarm on `GetSecretValue` API call volume (> 1,000 calls/hour).
- **Why It Saves Money:** Instant alert if a developer deploys code calling `GetSecretValue` uncached in a loop.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm targeting `AWS/SecretsManager`.
- **Estimated Savings:** Proactive billing risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudTrail / CloudWatch integration.

#### 10. Implement Resource Tagging for Cost Allocation
- **What:** Enforce mandatory tagging (`Environment`, `Owner`, `CostCenter`) on all created secrets.
- **Why It Saves Money:** Enables granular cost tracking and team accountability.
- **Detailed Implementation Steps:**
  1. Require tags in Secrets Manager IAM policies.
- **Estimated Savings:** Cost visibility and governance.
- **Risk Level:** Zero.
- **Implementation Scope:** DevOps / FinOps Team
- **Prerequisites:** Tagging policy setup.

#### 11. Restrict Secret Creation Permissions via SCPs
- **What:** Attach SCP denying `secretsmanager:CreateSecret` without approval.
- **Why It Saves Money:** Prevents un-tracked developer secret proliferation ($0.40/mo per secret).
- **Detailed Implementation Steps:**
  1. Attach SCP restricting secret creation in non-prod accounts.
- **Estimated Savings:** Storage cost containment.
- **Risk Level:** Zero.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** SCP configuration access.

#### 12. Standardize Password Generation Parameters
- **What:** Use `aws secretsmanager get-random-password` for password generation during automated deployments.
- **Why It Saves Money:** Free utility API for generating secure random strings.
- **Detailed Implementation Steps:**
  1. Call `get-random-password` in Terraform / CLI deployment scripts.
- **Estimated Savings:** Operational efficiency.
- **Risk Level:** Zero.
- **Implementation Scope:** DevOps Engineer
- **Prerequisites:** None.

#### 13. Audit KMS Customer Managed Key Surcharges for Secrets
- **What:** Use default AWS Managed Key (`aws/secretsmanager`) for encrypting secrets unless custom CMK key policies are required.
- **Why It Saves Money:** Avoids paying $1.00/mo per Customer Managed Key for encrypting small secrets.
- **Detailed Implementation Steps:**
  1. Select default AWS KMS key when creating secrets.
- **Estimated Savings:** $1.00/mo key storage savings.
- **Risk Level:** Low.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** Compliance review.

#### 14. Optimize Secret Rotation Windows (30 to 90 Days)
- **What:** Adjust automatic secret rotation schedule from 7 days to 30 or 90 days for non-critical credentials.
- **Why It Saves Money:** Reduces rotation Lambda execution and KMS API operation fees.
- **Detailed Implementation Steps:**
  1. Update rotation schedule expression to `rate(30 days)`.
- **Estimated Savings:** 75% reduction in rotation execution fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Security rotation policy alignment.

#### 15. Enforce IAM Least Privilege Read Policies
- **What:** Restrict `secretsmanager:GetSecretValue` permissions to authorized application execution roles only.
- **Why It Saves Money:** Prevents unauthorized internal polling scripts from inflating request fees.
- **Detailed Implementation Steps:**
  1. Apply strict resource-based policies on secrets.
- **Estimated Savings:** Security and cost risk mitigation.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** IAM policy setup.

#### 16. Monitor Recovery Window Expiration Status
- **What:** Ensure secrets scheduled for deletion complete their recovery window and are permanently purged.
- **Why It Saves Money:** Guarantees permanent storage billing termination.
- **Detailed Implementation Steps:**
  1. Audit pending deletion secret list via CLI.
- **Estimated Savings:** Administrative cleanliness.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

---

## Cross-Service Synergies

```
[ Application Code ] 
        │
        ├──(Config Selection)───> [ SSM Parameter Store ] (100% FREE for non-sensitive settings - $0.00)
        │
        ├──(API Request Caching)─> [ Caching Client Library ] (Caches secrets in RAM, cuts API calls 99.9%)
        │
        └──(JSON Consolidation)─> [ 1 Multi-Key JSON Secret ] (Packages 5 keys into 1 secret - 75% storage cut)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `SecretStorage-ByteHrs`, `API-Requests`.
- `line_item_resource_id`: Secrets Manager Secret ARN (`arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/db-credentials-12345`).

### B. CloudWatch & CloudTrail Metrics
- `AWS/SecretsManager` Namespace: `GetSecretValue`, `PutSecretValue`, `DescribeSecret`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "SEC-SSM-001",
  "service": "Secrets Manager",
  "category": "Secret Storage & Service Selection",
  "resource_id": "arn:aws:secretsmanager:us-east-1:123456789012:secret:config/public-api-url",
  "resource_name": "config/public-api-url",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "data_type": "NON_SENSITIVE_CONFIG",
    "storage_service": "Secrets Manager",
    "monthly_cost_usd": 0.40
  },
  "recommended_config": {
    "storage_service": "SSM Parameter Store (Standard)",
    "projected_monthly_cost_usd": 0.00
  },
  "financial_impact": {
    "monthly_savings_usd": 0.40,
    "annual_savings_usd": 4.80,
    "savings_percentage": 100.0
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Parameter is non-sensitive configuration data; SSM Parameter Store provides 100% free standard parameter storage."
  },
  "implementation": {
    "scope": "Software Engineer / DevOps",
    "effort_estimate": "15 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **SSM Parameter Store Migration** | 250 | $100.00 | $100.00 | 100.0% | Zero |
| **Local Secret Caching (SDK)** | 15 | $4,500.00 | $4,495.50 | 99.9% | Zero |
| **JSON Secret Consolidation** | 80 | $160.00 | $120.00 | 75.0% | Zero |
| **User Token DynamoDB Offloading** | 5 | $2,000.00 | $1,990.00 | 99.5% | Low |
| **Total** | **350** | **$6,760.00** | **$6,705.50** | **99.2%** | -- |
