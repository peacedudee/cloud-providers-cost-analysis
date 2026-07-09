# AWS Service Cost Research: AWS IAM Identity Center (formerly AWS SSO)

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS IAM Identity Center (successor to AWS Single Sign-On) is the recommended service for managing workforce identities and controlling access to AWS accounts and cloud applications. It provides a centralized web portal where workforce users can log in once using single sign-on (SSO) and access assigned AWS accounts, CLI sessions, and third-party SAML 2.0 cloud applications (like Microsoft 365, Salesforce, or Jira). IAM Identity Center is a core AWS management feature provided at **no additional charge**.

---

## 2. Billing Mechanics
1.  **Core Service Pricing:** **100% Free ($0.00)**. There are no setup fees, monthly active user (MAU) charges, per-user subscription fees, or application connection fees.
2.  **Built-in Identity Store:** Free of charge for managing users and groups directly within IAM Identity Center.
3.  **External Identity Providers (IdPs):** Connecting external SAML 2.0 or SCIM providers (such as Okta, Azure AD / Microsoft Entra ID, or Google Workspace) is **free** from the AWS side (third-party IdP licensing fees apply independently).

---

## 3. Key Cost Dimensions

| Service Feature | Billed Metric | Rate (us-east-1) | Cost Impact |
|-----------------|---------------|------------------|-------------|
| **User & Group Management** | Per user / month | **Free ($0.00)** | $0 base charge |
| **AWS Account Portal Access**| Per login / session | **Free ($0.00)** | Unlimited logins |
| **SAML 2.0 Application Sync**| Per app connection | **Free ($0.00)** | Unlimited app integration |
| **SCIM User Provisioning** | Per SCIM sync | **Free ($0.00)** | Automatic directory sync |

---

## 4. Detailed Pricing Rates (us-east-1)
*   **Identity Center Base Fee:** $0.00 per month.
*   **Active User Fee:** $0.00 per active workforce user.
*   **CLI Temporary Token Issuance:** $0.00 per STS token exchange.

---

## 5. AWS Free Tier Coverage
*   **IAM Identity Center:** Always 100% free for all AWS accounts with no user or time limitations.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Paying for AWS Directory Service When IAM Identity Center Suffices:** Deploying AWS Managed Microsoft AD ($175.20 to $584.00/month) strictly to provide corporate users with SSO logins to the AWS Management Console when IAM Identity Center provides single sign-on out of the box for $0.00.
*   **Third-Party IdP User Over-Licensing:** Purchasing extra user seats on third-party identity providers for users who only need access to AWS accounts, when they could be managed directly within IAM Identity Center's free native directory.

---

## 7. Actionable Cost Optimization Strategies
1.  **Replace AWS Managed AD for Portal SSO:**
    *   If your organization launched AWS Managed Microsoft AD solely to authenticate developers logging into the AWS Console or CLI, migrate them to **IAM Identity Center**. Connect IAM Identity Center to your existing free/standard corporate IdP (such as Google Workspace or Microsoft Entra ID Free) via SAML 2.0.
    *   **The Savings:** Deletes the Managed AD cluster, saving **$175.20 to $584.00/month**.
2.  **Centralize CLI Credential Management (`aws sso login`):**
    *   Enforce `aws sso login` across all engineering teams instead of issuing static IAM access keys.
    *   **The Benefit:** Completely free, eliminates manual access key rotation workflows, improves security posture, and enforces short-lived 1-hour to 12-hour temporary credentials.
3.  **Consolidate Multi-Account Permission Sets:** Use IAM Identity Center **Permission Sets** to manage IAM policies centrally across all child accounts in AWS Organizations. This prevents policy drift and eliminates custom identity broker infrastructure.
