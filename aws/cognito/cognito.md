# AWS Service Cost Research: Amazon Cognito

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Cognito delivers customer identity management, authentication, user directories, and single sign-on (SSO) for web and mobile applications. It handles user sign-up, sign-in, multi-factor authentication (MFA), and federated identity integration (Google, Facebook, Apple, SAML 2.0, OIDC). Cognito is serverless, billing based on Monthly Active Users (MAUs) under a selected feature plan, enterprise federation, machine-to-machine (M2M) token requests, and outbound verification message charges.

---

## 2. Billing Mechanics
Cognito billing is calculated monthly across four main dimensions:
1. **User Pool Feature Plans (MAU Billing):** Billed based on the count of unique users who sign in, sign up, or refresh tokens within a calendar month. The cost per MAU depends on whether the user pool is set to the **Lite**, **Essentials**, or **Plus** feature plan. Inactive users are **100% Free ($0.00)**.
2. **SAML / OIDC Enterprise Federation:** Billed per federated MAU authenticated via enterprise identity providers (e.g., Okta, Azure AD).
3. **Machine-to-Machine (M2M) Token Requests:** Billed per successful token request made using the OAuth 2.0 client credentials grant flow.
4. **SMS / Email Verification Surcharges:** Outbound SMS verification messages are billed separately via Amazon SNS telecom rates, whereas email verification is free under Amazon SES free tier allocations.

---

## 3. Key Cost Dimensions

### A. User Pool Feature Plans (Standard MAU Rates - us-east-1)
Each user pool is assigned a feature plan that determines its capabilities and standard MAU pricing. New user pools default to the **Essentials** plan.
*   **Lite Plan:** Designed for basic authentication workloads.
    *   **First 10,000 MAUs / month:** **100% Free ($0.00)**.
    *   **Next 40,000 MAUs (10,001 to 50,000):** **$0.0055 per MAU**.
    *   **Next 50,000 MAUs (50,001 to 100,000):** **$0.0046 per MAU**.
    *   **Over 100,000 MAUs:** **$0.0025 per MAU**.
*   **Essentials Plan:** Adds Managed Login, passkeys, passwordless authentication (email/SMS OTP), and customizable access tokens.
    *   **First 10,000 MAUs / month:** **100% Free ($0.00)**.
    *   **Usage above 10,000 MAUs:** Flat **$0.0150 per MAU** ($15.00 per 1,000 MAUs).
*   **Plus Plan:** Adds advanced security features (compromised credentials checks, adaptive risk-based authentication) and exportable user activity/risk logs.
    *   **Standard MAUs:** Flat **$0.0200 per MAU** ($20.00 per 1,000 MAUs). **No free tier** is offered for standard MAUs in the Plus plan.

### B. Enterprise SAML / OIDC Federation
*   **First 50 Federated MAUs / month:** **Free ($0.00)** across all feature plans.
*   **Additional Federated MAUs:** Flat **$0.0150 per MAU** ($15.00 per 1,000 users) across all feature plans.

### C. Machine-to-Machine (M2M) Token Requests
Charged per successful OAuth 2.0 token request using the client credentials grant:
*   **First 250,000 requests / month:** **$2.25 per 1,000 requests**.
*   **Next 4,750,000 requests / month (250,001 to 5M):** **$1.50 per 1,000 requests**.
*   **Over 5,000,000 requests / month:** **$1.125 per 1,000 requests**.
*   *Note: As of late 2025, AWS has eliminated the flat monthly fee that was previously charged for each M2M app client. You pay only for successful token requests.*

### D. Outbound SMS Verification Surcharges (Telecom Hazard)
*   Sending MFA or account verification codes via SMS utilizes Amazon SNS.
*   *US SMS Rate:* ~$0.0065 per SMS. International SMS rates range from **$0.02 to $0.15 per SMS**.
*   *Math Impact:* Sending 100,000 international SMS verification codes costs **$2,000.00 to $15,000.00/month** in carrier surcharges.

---

## 4. Detailed Pricing Rates (us-east-1)

### Standard MAUs Billed per Plan Tier
| Feature Plan | First 10,000 MAUs | Next 40,000 MAUs (10K–50K) | Next 50,000 MAUs (50K–100K) | 100,000+ MAUs |
|--------------|-------------------|-----------------------------|------------------------------|---------------|
| **Lite** | **Free ($0.00)** | $0.0055 / MAU | $0.0046 / MAU | $0.0025 / MAU |
| **Essentials**| **Free ($0.00)** | $0.0150 / MAU | $0.0150 / MAU | $0.0150 / MAU |
| **Plus** | $0.0200 / MAU | $0.0200 / MAU | $0.0200 / MAU | $0.0200 / MAU |

### Example Monthly Cost Calculation
*Workload: A standard user directory has 25,000 standard active users (MAUs), 500 federated SAML users, and performs 300,000 successful M2M token requests.*

1.  **Lite Plan Cost:**
    *   **Standard MAU Cost:**
        $$\text{Standard MAUs Billed} = 25,000 - 10,000\text{ (Free)} = 15,000\text{ MAUs}$$
        $$\text{Standard MAU Fee} = 15,000 \times \$0.0055 = \$82.50$$
    *   **Federated MAU Cost:**
        $$\text{Federated MAUs Billed} = 500 - 50\text{ (Free)} = 450\text{ MAUs}$$
        $$\text{Federated MAU Fee} = 450 \times \$0.0150 = \$6.75$$
    *   **M2M Token Cost:**
        $$\text{M2M Fee} = 250 \times \$2.25 + 50 \times \$1.50 = \$562.50 + \$75.00 = \$637.50$$
    *   **Total Monthly Lite Cost:** **$726.75/month**

2.  **Essentials Plan Cost:**
    *   **Standard MAU Cost:**
        $$\text{Standard MAU Fee} = (25,000 - 10,000) \times \$0.0150 = \$225.00$$
    *   **Federated MAU Cost:** **$6.75** (same across all plans)
    *   **M2M Token Cost:** **$637.50** (same across all plans)
    *   **Total Monthly Essentials Cost:** **$869.25/month**

3.  **Plus Plan Cost:**
    *   **Standard MAU Cost:**
        $$\text{Standard MAU Fee} = 25,000 \times \$0.0200 = \$500.00\text{ (No Free Tier)}$$
    *   **Federated MAU Cost:** **$6.75**
    *   **M2M Token Cost:** **$637.50**
    *   **Total Monthly Plus Cost:** **$1,144.25/month**

---

## 5. AWS Free Tier Coverage
*   **Lite & Essentials Plans:** **10,000 MAUs** always free per month per account (indefinite, does not expire after 12 months).
*   **Federated Identity Pools:** 100% Free for credential broker calls.
*   **SAML/OIDC Federation:** **50 Federated MAUs** always free per month per account.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Defaulting to Essentials/Plus in Dev/Test:** Allowing development, staging, or sandbox user pools to sit on the Essentials ($0.015/MAU) or Plus ($0.020/MAU) tiers when basic authentication (Lite plan: $0.00 to $0.0055/MAU) is sufficient for testing.
*   **Relying on SMS for 2FA / Account Verification:** Using SMS for routine sign-in OTPs. Carrier surcharges dominate Cognito bills, especially for international users.
*   **Spam/Bot User Registration:** Allowing fake bot signups to inflate standard MAU counts without automated cleanup.
*   **Aggressive/Un-cached M2M Token Requests:** Microservices requesting a new M2M client credentials token for every individual service-to-service API call instead of caching tokens, multiplying Cognito M2M request costs by thousands of times.

---

## 7. Actionable Cost Optimization Strategies
1.  **Implement M2M Token Caching:**
    *   Store M2M access tokens in a local cache (in-memory or Redis) and reuse them until they expire (default is 1 hour).
    *   **The Savings:** Reduces the frequency of successful token request calls to Cognito by up to **99.9%**, translating directly to massive savings on high-volume service-to-service communication.
2.  **Use the Lite Plan for Dev/Test User Pools:**
    *   Downgrade non-production user pools to the **Lite** feature plan unless specific Essentials/Plus features (like Managed Login styling or risk-based analytics) are actively being verified.
    *   **The Savings:** Maximizes the 10,000 free MAU threshold and saves up to **63%** on billable active testing accounts.
3.  **Migrate from SMS OTP to Email OTP or TOTP Authenticator Apps:**
    *   Configure User Pool MFA settings to use Email Verification (free via Amazon SES free tier) or standard TOTP Authenticator Apps (Google Authenticator, Authy).
    *   **The Savings:** Completely avoids SMS telecom fees, eliminating a major cost hazard.
4.  **Automate Unconfirmed User Deletion:**
    *   Create a scheduled Lambda function to query and delete users in `UNCONFIRMED` status who have not completed verification within 48 hours.
    *   **The Savings:** Prevents bot signups from rolling over into active, billed MAUs.
5.  **Leverage Free Cognito Identity Pools for Guests:**
    *   For apps requiring temporary AWS credentials for guest users (such as downloading assets from S3), use **Cognito Identity Pools (Federated Identities)** which are **100% Free** instead of creating dummy User Pool records.
