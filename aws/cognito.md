# AWS Service Cost Research: Amazon Cognito

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Cognito delivers customer identity management, authentication, user directories, and single sign-on (SSO) for web and mobile applications. It handles user sign-up, sign-in, multi-factor authentication (MFA), and federated identity integration (Google, Facebook, Apple, SAML 2.0, OIDC). Cognito is serverless, billing primarily based on Monthly Active Users (MAUs), enterprise federation, threat protection features, and verification message charges.

---

## 2. Billing Mechanics
Cognito billing is calculated monthly across four main dimensions:
1. **Cognito User Pools (MAU Billing):** Billed based on the count of unique users who sign in, sign up, or refresh tokens within a calendar month. Inactive users are **100% Free ($0.00)**.
2. **SAML / OIDC Enterprise Federation:** Billed per federated MAU authenticated via enterprise identity providers (e.g., Okta, Azure AD).
3. **Advanced Security Features (Threat Protection):** Billed per MAU for adaptive authentication and compromised credential checks.
4. **SMS / Email Verification Surcharges:** Outbound SMS verification messages are billed separately via Amazon SNS telecom rates.

---

## 3. Key Cost Dimensions

### A. User Pools MAU Volume Tiers (us-east-1 Rates)
* **First 10,000 MAUs / month:** **100% Free ($0.00)**.
* **Next 40,000 MAUs (10,001 to 50,000):** **$0.0055 per MAU**.
* **Next 50,000 MAUs (50,001 to 100,000):** **$0.0046 per MAU**.
* **Over 100,000 MAUs:** Drops to **$0.0025 per MAU**.

### B. Enterprise SAML / OIDC Federation
* **First 50 Federated MAUs / month:** **Free ($0.00)**.
* **Additional Federated MAUs:** **$0.0150 per MAU** ($15.00 per 1,000 users).

### C. Advanced Security Features (Threat Protection)
* Adds an extra **$0.0200 per MAU** across all active users (no free tier for Advanced Security).

### D. Outbound SMS Verification Surcharges (Telecom Hazard)
* Sending 2FA/MFA OTP codes via SMS uses Amazon SNS.
* *US SMS Rate:* ~$0.0065 per SMS. International SMS rates range from **$0.02 to $0.15 per SMS**.
* *Math Impact:* Sending 100,000 international SMS verification codes costs **$2,000.00 to $15,000.00/month** in telecom fees!

---

## 4. Detailed Pricing Rates (us-east-1 User Pools)

| Usage Level (MAUs) | Rate per MAU (Standard) | Cost for Tier Block | Cumulative Monthly Cost |
|--------------------|-------------------------|---------------------|-------------------------|
| **0 to 10,000** | **Free ($0.00)** | $0.00 | **$0.00** |
| **10,001 to 50,000** | **$0.0055** | $220.00 | **$220.00** (50K users) |
| **50,001 to 100,000** | **$0.0046** | $230.00 | **$450.00** (100K users) |
| **100,000+** | **$0.0025** | $2.50 per 1,000 MAUs | Scale-based |

---

## 5. AWS Free Tier Coverage
* **Cognito User Pools:** **10,000 MAUs** always free per month for all AWS accounts indefinitely.
* **Federated Identity Pools:** 100% Free for credential broker calls.

---

## 6. Common Cost Hotspots & Pitfalls
* **Relying on SMS for 2FA / Account Verification:** Using SMS for routine sign-in OTPs. Telecom carrier surcharges dominate Cognito bills.
* **Enabling Advanced Security Features ($0.02/MAU) Globally:** Turning on Advanced Security in dev/test user pools or for low-risk public user directories, quadrupling base MAU charges.
* **Accumulating Unverified Spam Users:** Allowing bot registrations to linger in User Pools without automated cleanup, driving up active user counts.

---

## 7. Actionable Cost Optimization Strategies
1. **Migrate from SMS OTP to Email OTP or TOTP Authenticator Apps:**
   * Configure User Pool MFA settings to use **Email Verification ($0.00 via SES free tier)** or **TOTP Authenticator Apps** (Google Authenticator, Authy, 1Password).
   * **The Savings:** Eliminates 100% of SMS telecom fees, saving thousands of dollars per month on high-volume user apps.
2. **Purge Unconfirmed Users Automatically:**
   * Implement an automated Lambda function or cron job using `AdminDeleteUser`.
   * Delete user accounts that remain in `UNCONFIRMED` status for more than 48 hours.
   * **The Savings:** Prevents spam/bot signups from converting into billed MAUs.
3. **Use Standard Tiers in Non-Production:** Turn off Advanced Security Features ($0.02/MAU) in development and staging environments.
4. **Use Cognito Identity Pools for Unauthenticated Guest Access:** For mobile apps requiring temporary AWS credentials (e.g., uploading photos directly to S3), use **Cognito Identity Pools (Federated Identities)** which are **100% Free**.
