# reCAPTCHA Enterprise Cost Optimization & Research

Google Cloud reCAPTCHA Enterprise protects your website from fraudulent activity, spam, and abuse using risk analysis engines. It is billed on a pay-as-you-go model per **assessment** (validation check). Because it scales automatically with user traffic, un-optimized scripts running on public pages or bot storms targeting unprotected sites can run up large API invoices.

---

## 1. reCAPTCHA Enterprise Billing Mechanics

reCAPTCHA Enterprise pricing is volume-based:
1. **Essentials Tier (No Billing):** Free up to 10,000 assessments per month (requests fail after 10k).
2. **Premium Tier (Pay-as-you-go):** 
   * 1 - 10,000 assessments: Free.
   * 10,001 - 100,000 assessments: $8.00 flat fee.
   * > 100,000 assessments: $1.00 per 1,000 assessments.
3. **Enterprise Tier:** Designed for high-volume users, requiring a fixed monthly volume commitment at $1.00 per 1,000 assessments (often with volume discounts negotiated via sales).

---

## 2. Core Cost-Optimization Levers

### A. Restrict Assessments to High-Value Gates (Avoid Page Load Checks)
* **The Cost Trap:** Loading reCAPTCHA and executing assessments on every single page load across your entire e-commerce store (e.g., browsing products, reading blogs). If you have 5 million page views a month, this generates $5,000/month in reCAPTCHA fees.
* **The Solution:** Target sensitive endpoints only.
* **Action:** Trigger reCAPTCHA assessments strictly on transactional or entry-point actions:
  * User Login form submissions.
  * Account Creation (registration) button clicks.
  * Checkout Payment confirmations.
  * Password Reset submissions.
* **The Benefit:** Drastically reduces total assessment count while maintaining identical security protection.

### B. Bypass Assessments for Authenticated User Sessions
* Once a user has successfully logged in, verified their identity, and established an active JWT session:
* **Action:** Store the session validation state in client-side cookies or tokens. Stop executing reCAPTCHA assessments for subsequent actions (like updating profile info or adding items to carts) within that validated session.

### C. Implement Strict Domain Restrictions
* **The Risk:** An external attacker copies your reCAPTCHA public site key from your source code and uses it on their own high-traffic website, or uses bot scripts to query the key directly, billing your GCP account for the assessments.
* **Action:** In the reCAPTCHA Admin console, turn on **Domain Restriction** for each key. Explicitly list only your verified domain names (e.g., `domain.com` and `*.domain.com`). Reject assessments originating from other domains.

---

## 3. reCAPTCHA Enterprise Audit Checklist

1. [ ] **Assessment Scope Audit:** Review client-side JS integrations. Confirm reCAPTCHA only runs on form submissions, not on general page loads.
2. [ ] **Domain Restriction Check:** Ensure all active reCAPTCHA keys have domain verification turned on.
3. [ ] **Session Caching:** Confirm that authenticated users do not run repeated assessments during their active session.
4. [ ] **High-Volume Tier Review:** If monthly assessments consistently exceed 1 million, contact sales to lock in a cheaper Enterprise commitment tier.
