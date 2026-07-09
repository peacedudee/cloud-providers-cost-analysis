# AWS Service Cost Research: AWS Wickr

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Wickr is an end-to-end encrypted (E2EE) messaging and collaboration service designed to help enterprise organizations communicate securely via 1:1 and group messaging, voice/video calling, file sharing, and screen sharing. Wickr incorporates strict administrative controls, administrative compliance data retention, and zero-trust security architecture. Wickr is billed on a per-user monthly subscription model.

---

## 2. Billing Mechanics
1.  **Wickr Standard Plan:** Billed monthly per active user ($5.00 per user per month). Includes 1:1/group messaging, voice/video calls up to 100 participants, and basic admin controls.
2.  **Wickr Premium Plan:** Billed monthly per active user ($15.00 per user per month). Adds advanced compliance archiving, legal hold capabilities, custom branding, and administrative data retention.
3.  **Free Trial:** Includes a 30-day free trial for up to 30 users.

---

## 3. Key Cost Dimensions

| Wickr Plan Tier | Admin Features | Monthly Rate per User | Price for 100 Users / Month |
|-----------------|----------------|-----------------------|-----------------------------|
| **Wickr Standard** | Basic E2EE Messaging | **$5.00 / user-mo** | **$500.00 / month** |
| **Wickr Premium** | Archiving & Compliance | **$15.00 / user-mo** | **$1,500.00 / month** |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Standard Plan Rate:** $5.00 per user per month.
*   **Premium Plan Rate:** $15.00 per user per month.
*   **External / Guest Users:** Free up to guest quota limits.

---

## 5. AWS Free Tier Coverage
*   **AWS Wickr Free Tier:** First **30 days free** for up to 30 users across both Standard and Premium tiers.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Paying for Inactive Employee Accounts:** Leaving unassigned or former employee accounts active in the Wickr Admin Console, incurring $5.00 to $15.00 per user-month indefinitely for idle accounts.

---

## 7. Actionable Cost Optimization Strategies
1.  **Perform Monthly Active User Seat Audits:**
    *   Integrate Wickr user provisioning with IAM Identity Center or Azure AD (SCIM).
    *   Automate de-provisioning of Wickr licenses when employees or contractors leave the company.
    *   **The Savings:** Avoids paying $60.00 to $180.00 per year per orphaned account.
2.  **Assign Wickr Premium Only to Regulatory-Scoped Users:**
    *   Assign **Wickr Premium ($15/user-mo)** strictly to compliance-regulated users (e.g. executive team or financial traders requiring legal archive retention).
    *   Assign **Wickr Standard ($5/user-mo)** to general staff.
    *   **The Savings:** Slashes per-user messaging costs by **66%** ($5/mo vs $15/mo).
