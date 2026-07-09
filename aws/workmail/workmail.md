# AWS Service Cost Research: Amazon WorkMail

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon WorkMail is a managed, secure business email and calendar service that supports existing desktop and mobile email clients (Microsoft Outlook, iOS, Android, and macOS Mail) as well as web browsers. WorkMail integrates natively with AWS Directory Service / Active Directory for user authentication and uses AWS KMS for email encryption at rest. WorkMail is billed on a per-user monthly subscription basis.

---

## 2. Billing Mechanics
1.  **Monthly User License:** Billed monthly per active mailbox ($4.00 per user per month).
2.  **Included Storage:** Each user license includes **50 GB of mailbox storage** at no extra charge.
3.  **Additional Storage:** Billed per GB per month ($0.20 per GB-month) for mailboxes exceeding 50 GB.
4.  **Free Trial:** Includes a 30-day free trial for up to 25 users.

---

## 3. Key Cost Dimensions

| WorkMail Service Tier | Mailbox Storage Included | Monthly Rate per User | Price for 100 Users / Month |
|-----------------------|--------------------------|-----------------------|-----------------------------|
| **Standard User** | **50 GB / user** | **$4.00 / user-mo** | **$400.00 / month** |
| **Additional Storage** | Per GB over 50 GB | **$0.20 / GB-month** | $0.20 per extra GB |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **User Rate:** $4.00 per user per month.
*   **Mailbox Storage Rate:** $0.00 for first 50 GB; $0.20 per GB-month thereafter.

---

## 5. AWS Free Tier Coverage
*   **Amazon WorkMail Free Tier:** First **30 days free** for up to 25 users.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Leaving Active Mailboxes Open for Terminated Staff ($4.00/mo):** Wasting $4.00 per user-month on active mailboxes for former employees or seasonal workers.

---

## 7. Actionable Cost Optimization Strategies
1.  **Export & Archive Departing Mailboxes to S3 Glacier:**
    *   When an employee leaves, export their 50 GB mailbox to an EML or PST archive file and store it in **Amazon S3 Glacier Instant Retrieval ($0.004/GB-mo = $0.20/mo)**.
    *   Delete the active WorkMail mailbox.
    *   **The Savings:** Slashes monthly storage costs for departing staff from **$4.00 down to $0.20 per month (95% savings)**.
2.  **Automate User Provisioning via Active Directory:** Sync WorkMail with AWS Directory Service so mailboxes are automatically disabled upon employee offboarding in AD.
