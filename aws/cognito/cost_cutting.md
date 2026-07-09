# Cost-Cutting Playbook: Amazon Cognito

> **Companion File:** [cognito.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/cognito/cognito.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon Cognito provides customer identity management, authentication, multi-factor authentication (MFA), and single sign-on (SSO). Billing is multi-dimensional based on **Monthly Active Users (MAUs)** under selected feature plans (Lite vs Essentials vs Plus), **Enterprise SAML/OIDC Federation**, **Machine-to-Machine (M2M) Token Requests**, and **Outbound SMS Verification Surcharges**.

Key cost pitfalls include:
1. **Outbound SMS Telecom Hazard:** Using SMS OTP for MFA or account verifications. Telecom carrier surcharges range from **$0.02 to $0.15+ per SMS**, costing **$2,000.00 to $15,000.00/month** for 100,000 verification messages!
2. **Feature Plan Over-Provisioning in Dev/Test:** Leaving non-prod user pools on Essentials ($0.015/MAU) or Plus ($0.020/MAU) feature plans instead of the **Lite Plan** (10,000 Free MAUs; $0.0025 to $0.0055/MAU).
3. **Un-cached Machine-to-Machine (M2M) Tokens:** Microservices requesting a new OAuth 2.0 client credentials token on *every single service-to-service API call* ($2.25/1k requests) instead of caching tokens locally.

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **35–85% reduction in total Cognito spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Migrate MFA from SMS OTP to Email OTP / TOTP Authenticator Apps:** Completely eliminates SMS telecom carrier surcharges ($0.02–$0.15/SMS), saving thousands of dollars per month.
2. **Implement Local M2M Token Caching (In-Memory / Redis):** Re-uses valid OAuth 2.0 tokens until expiration (1 hour), cutting M2M token request billing by up to **99.9%**.
3. **Downgrade Dev/Test User Pools to the Lite Feature Plan:** Maximizes the 10,000 free MAU allowance and cuts billable active user costs by up to **63%**.

---

## Strategy Categories

### 1. Feature Plan & MAU Optimization

#### 1. Downgrade Non-Production User Pools to the Lite Feature Plan
- **What:** Configure development, testing, and staging Cognito User Pools to use the **Lite Feature Plan**.
- **Why It Saves Money:**
  - **Plus Plan:** Flat **$0.0200 per MAU** (**No Free Tier**).
  - **Essentials Plan:** Flat **$0.0150 per MAU** (First 10,000 Free).
  - **Lite Plan:** **First 10,000 MAUs FREE ($0.00)**; next 40,000 MAUs at **$0.0055 per MAU** (**63.3% cheaper** than Essentials).
  - Leaving 10 non-prod user pools with 5,000 active testing accounts on Plus costs **$1,000.00/month**. On Lite, it costs **$0.00**!
- **Detailed Implementation Steps:**
  1. Audit user pool feature plans via AWS CLI:
     ```bash
     aws cognito-idp describe-user-pool \
       --user-pool-id us-east-1_123456789 \
       --query "UserPool.UserPoolTier"
     ```
  2. Downgrade non-prod user pools to Lite tier:
     ```bash
     aws cognito-idp update-user-pool \
       --user-pool-id us-east-1_123456789 \
       --user-pool-tier LITE
     ```
- **Estimated Savings:** **63–100% reduction** in non-production user pool MAU billing.
- **Risk Level:** Zero risk (Lite tier supports standard signup, signin, and token verification).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** User pool does not require Plus/Essentials features (Managed Login styling, risk analytics).

#### 2. Automate Cleanup of Unconfirmed & Inactive User Accounts
- **What:** Deploy a scheduled AWS Lambda function to identify and delete user accounts stuck in `UNCONFIRMED` or `ARCHIVED` status for > 30 days.
- **Why It Saves Money:** Prevents fake bot registrations and abandoned signups from completing token refreshes that trigger billable Monthly Active User (MAU) charges.
- **Detailed Implementation Steps:**
  1. Run Python Lambda script querying list_users:
     ```python
     import boto3
     from datetime import datetime, timedelta

     client = boto3.client('cognito-idp')

     def lambda_handler(event, context):
         users = client.list_users(UserPoolId='us-east-1_123456789', Filter='userLastModifiedDate < "2026-06-01"')
         for u in users['Users']:
             if u['UserStatus'] == 'UNCONFIRMED':
                 client.admin_delete_user(UserPoolId='us-east-1_123456789', Username=u['Username'])
     ```
- **Estimated Savings:** 10–30% reduction in billed MAU volume.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Account lifecycle cleanup policy approval.

---

### 2. SMS Telecom Hazard Avoidance

#### 3. Migrate 2FA / MFA from SMS OTP to Email OTP or TOTP Authenticator Apps
- **What:** Reconfigure User Pool MFA settings to require **TOTP Authenticator Apps** (Google Authenticator, Authy, 1Password) or **Email OTP** (sent free via Amazon SES free tier).
- **Why It Saves Money:** SMS verification uses Amazon SNS telecom delivery rates (~$0.0065/SMS in US; **$0.02 to $0.15/SMS internationally**). Delivering 100,000 international MFA codes via SMS costs **$2,000.00 to $15,000.00/month**. Email and TOTP verification cost **$0.00**!
- **Detailed Implementation Steps:**
  1. Enable Software Token (TOTP) MFA in User Pool configuration via AWS CLI:
     ```bash
     aws cognito-idp set-user-pool-mfa-config \
       --user-pool-id us-east-1_123456789 \
       --mfa-configuration ON \
       --software-token-mfa-configuration Enabled=true
     ```
  2. Disable SMS MFA in User Pool settings.
- **Estimated Savings:** **100% elimination** of SMS telecom carrier surcharges ($1,000s/month saved).
- **Risk Level:** Low (significantly improves user authentication security over vulnerable SMS interception).
- **Implementation Scope:** Software Engineer / Product Manager
- **Prerequisites:** Client UI support for TOTP QR-code scanning or Email OTP inputs.

---

### 3. Machine-to-Machine (M2M) Token Optimization

#### 4. Implement Local M2M Token Caching in Client Microservices
- **What:** Configure backend microservices requesting OAuth 2.0 client credentials tokens to cache access tokens in memory (or Redis) and reuse them until expiration (default TTL = 1 hour).
- **Why It Saves Money:** M2M token requests bill **$2.25 per 1,000 requests** (first 250k). Requesting a new token on every API call (10M calls/month) costs **$15,187.50/month**. Caching tokens for 1 hour drops token requests to 730/month (**$1.64/month**) — a **99.99% cost reduction**!
- **Detailed Implementation Steps:**
  1. Add token caching decorator in Python backend:
     ```python
     import functools, time

     token_cache = {}

     def get_m2m_token():
         now = time.time()
         if 'token' in token_cache and token_cache['expires'] > now + 60:
             return token_cache['token']
         
         # Fetch new token from Cognito token endpoint
         res = requests.post(COGNITO_TOKEN_URL, data={'grant_type': 'client_credentials', ...})
         data = res.json()
         token_cache['token'] = data['access_token']
         token_cache['expires'] = now + data['expires_in']
         return token_cache['token']
     ```
- **Estimated Savings:** **99.9% reduction** in M2M token request billing ($1,000s/mo saved).
- **Risk Level:** Zero risk (standard OAuth 2.0 RFC 6749 token caching practice).
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Microservice backend code update.

---

### 4. Federated Identity & Guest Access

#### 5. Use Free Cognito Identity Pools (Federated Identities) for Guest Access
- **What:** Utilize **Cognito Identity Pools** (unauthenticated guest credentials) for apps requiring temporary AWS IAM credentials (e.g. downloading public assets from S3 or pushing telemetry to Kinesis).
- **Why It Saves Money:** Cognito Identity Pools are **100% FREE ($0.00)** for credential broker calls. Creating dummy User Pool records for guest visitors inflates MAU counts unnecessarily.
- **Detailed Implementation Steps:**
  1. Configure Cognito Identity Pool with `EnableUnauthenticatedIdentities = true`.
- **Estimated Savings:** 100% of guest visitor MAU billing.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Temporary IAM role policy configuration.

#### 6. Consolidate Enterprise SAML / OIDC Federated User Pools
- **What:** Consolidate corporate SSO federations (Okta, Azure AD, Ping) into a central shared User Pool rather than creating duplicate federated pools across accounts.
- **Why It Saves Money:** Enterprise SAML/OIDC federated MAUs cost **$0.0150 per MAU** (after the first 50 free). Consolidating pools maximizes the 50 free federated MAUs per account and prevents duplicate active user billing.
- **Detailed Implementation Steps:**
  1. Re-route application authentication to central identity pool.
- **Estimated Savings:** 20–40% reduction in federated user billing.
- **Risk Level:** Low.
- **Implementation Scope:** Identity & Access Management Specialist / DevOps
- **Prerequisites:** Corporate SSO architecture alignment.

---

### 5. Access Token & Session Lifetime Tuning

#### 7. Extend Refresh Token Validity Window (Reduce Refresh Invocations)
- **What:** Configure Refresh Token expiration settings to 30 days (default) or longer for trusted mobile applications.
- **Why It Saves Money:** Reduces the frequency of user re-authentications that trigger active MAU billing cycles.
- **Detailed Implementation Steps:**
  1. Set refresh token expiration via CLI:
     ```bash
     aws cognito-idp update-user-pool-client \
       --user-pool-id us-east-1_123456789 \
       --client-id app-client-id-123 \
       --refresh-token-validity 30 --token-validity-units RefreshToken=days
     ```
- **Estimated Savings:** Smooths out active MAU spikes.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Security token policy alignment.

---

### 6. Security, Anti-Spam & Governance

#### 8. Implement WAF Rate Limiting on Cognito Auth Endpoints
- **What:** Attach AWS WAF to Cognito User Pool domain endpoints (`/oauth2/token`, `/signup`).
- **Why It Saves Money:** Blocks automated credential stuffing and bot registration floods before they create fake active accounts that inflate MAU bills.
- **Detailed Implementation Steps:**
  1. Associate WAF Web ACL with Cognito User Pool ARN.
- **Estimated Savings:** Protects against $1,000s in bot registration MAU spikes.
- **Risk Level:** Low.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** WAF Web ACL configuration.

#### 9. Enforce CAPTCHA on Public Sign-Up Pages
- **What:** Add Google reCAPTCHA or AWS WAF CAPTCHA on user registration forms.
- **Why It Saves Money:** Prevents automated bot scripts from creating thousands of spam accounts.
- **Detailed Implementation Steps:**
  1. Integrate reCAPTCHA validation in backend sign-up API.
- **Estimated Savings:** Prevents spam MAU inflation.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Web Developer
- **Prerequisites:** reCAPTCHA API keys.

#### 10. Audit & Delete Unused App Clients
- **What:** Delete orphaned App Clients (`aws cognito-idp delete-user-pool-client`) in legacy user pools.
- **Why It Saves Money:** Administrative security hygiene.
- **Detailed Implementation Steps:**
  1. Delete unused app clients via CLI.
- **Estimated Savings:** Administrative cleanliness.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 11. Enforce CloudWatch Alarms for Daily Active User Spikes
- **What:** Set CloudWatch alarm on `SignInSuccesses` and `TokenRefreshSuccesses` metrics.
- **Why It Saves Money:** Instant alert on abnormal authentication volume.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm targeting `AWS/Cognito`.
- **Estimated Savings:** Proactive billing risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 12. Utilize SES Free Tier for Custom Email Verifications
- **What:** Ensure custom email verification messages use Amazon SES free tier allocations.
- **Why It Saves Money:** Keeps outbound email verification cost at **$0.00**.
- **Detailed Implementation Steps:**
  1. Configure Cognito custom email sender to use SES.
- **Estimated Savings:** Free email verification.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SES domain verification.

#### 13. Optimize Custom Lambda Triggers (Pre-Sign-up / Post-Confirmation)
- **What:** Minimize execution duration and memory allocation on custom Cognito Lambda triggers.
- **Why It Saves Money:** Cuts auxiliary Lambda compute billing on high-volume authentication events.
- **Detailed Implementation Steps:**
  1. Optimize Python/Node.js code in pre-signup Lambda triggers.
- **Estimated Savings:** 20–40% Lambda trigger compute savings.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Lambda trigger code review.

#### 14. Standardize User Directory Retention Policies
- **What:** Establish corporate retention guidelines for deleting inactive user accounts (> 1 year inactive).
- **Why It Saves Money:** Keeps user pool storage lean.
- **Detailed Implementation Steps:**
  1. Schedule annual user directory purge script.
- **Estimated Savings:** Long-term MAU cost containment.
- **Risk Level:** Low.
- **Implementation Scope:** Security / FinOps Team
- **Prerequisites:** Compliance approval.

#### 15. Utilize Account Free Tier Allowances Across Dev Accounts
- **What:** Structure dev accounts to utilize the **10,000 free MAU allowance** per account (Lite & Essentials).
- **Why It Saves Money:** Delivers $0.00 Cognito bills for small project directories.
- **Detailed Implementation Steps:**
  1. Monitor free tier usage in AWS Billing Console.
- **Estimated Savings:** Baseline free testing.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Account setup.

#### 16. Implement Domain Prefix Cleanup on Deleted User Pools
- **What:** Delete custom Cognito domain prefixes attached to deleted user pools.
- **Why It Saves Money:** Reclaims domain namespace availability.
- **Detailed Implementation Steps:**
  1. Delete domain: `aws cognito-idp delete-user-pool-domain`.
- **Estimated Savings:** Administrative hygiene.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

---

## Cross-Service Synergies

```
[ Application Client ] 
        │
        ├──(M2M Token Caching)─> [ Redis / Local Cache ] (Saves 99.9% on M2M token request fees)
        │
        ├──(MFA Migration)──────> [ Email OTP / TOTP Apps ] (100% FREE - Avoids SMS $0.02-0.15/SMS tax)
        │
        └──(Feature Plan)───────> [ Lite Plan (Dev/Test) ] (10,000 Free MAUs; 63% cheaper MAU rates)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `CognitoLiteMAU`, `CognitoEssentialsMAU`, `CognitoPlusMAU`, `CognitoM2MTokenRequest`, `SNS:DirectPublishSMS`.
- `line_item_resource_id`: Cognito User Pool ID (`us-east-1_xxxx`).

### B. CloudWatch Metrics
- `AWS/Cognito` Namespace: `SignInSuccesses`, `SignUpSuccesses`, `TokenRefreshSuccesses`, `AdminSearchUser`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "COG-SMS-001",
  "service": "Cognito",
  "category": "SMS Telecom Hazard Avoidance",
  "resource_id": "arn:aws:cognito-idp:us-east-1:123456789012:userpool/us-east-1_123456789",
  "resource_name": "global-customer-user-pool",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "mfa_type": "SMS_OTP",
    "monthly_international_sms_count": 85000,
    "avg_sms_rate_usd": 0.05,
    "monthly_cost_usd": 4250.00
  },
  "recommended_config": {
    "mfa_type": "TOTP_SOFTWARE_TOKEN_OR_EMAIL",
    "projected_monthly_cost_usd": 0.00
  },
  "financial_impact": {
    "monthly_savings_usd": 4250.00,
    "annual_savings_usd": 51000.00,
    "savings_percentage": 100.0
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "Migrating to TOTP Authenticator Apps improves security posture while completely eliminating SMS carrier charges."
  },
  "implementation": {
    "scope": "Software Engineer / Product Manager",
    "effort_estimate": "1-2 days",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **SMS MFA -> TOTP/Email Migration**| 6 | $12,500.00 | $12,500.00 | 100.0% | Low |
| **Local M2M Token Caching** | 10 | $18,200.00 | $18,181.80 | 99.9% | Zero |
| **Dev Pool Downgrade to Lite** | 8 | $3,400.00 | $2,142.00 | 63.0% | Zero |
| **Unconfirmed User Auto-Cleanup** | 12 | $2,100.00 | $525.00 | 25.0% | Low |
| **Total** | **36** | **$36,200.00** | **$33,348.80** | **92.1%** | -- |
