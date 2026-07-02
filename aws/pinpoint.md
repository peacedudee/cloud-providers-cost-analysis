# AWS Service Cost Research: Amazon Pinpoint

> **Status:** ⚠️ Deprecation / Sunset Notice (End of support October 30, 2026)
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Pinpoint is a flexible customer engagement service that enables marketers and developers to send targeted, multi-channel marketing campaigns, user journeys, push notifications, emails, SMS text messages, and voice prompts.
*   **Critical Status Notice:** AWS has announced the end of support for Amazon Pinpoint on **October 30, 2026**.
*   **Migration Pathway:** Transactional push, SMS, and OTP capabilities have migrated to **AWS End User Messaging**. Marketing email delivery is powered by Amazon SES.

---

## 2. Billing Mechanics
1.  **Monthly Targeted Users (MTU):** Billed per unique user endpoint targeted in campaigns ($0.0012 per MTU = $1.20 per 1,000 MTUs). First 5,000 MTUs/month are free.
2.  **Push Notifications:** Billed per push notification delivered ($1.00 per 1 Million push messages = $0.000001 per push).
3.  **Emails Delivered:** Billed at standard Amazon SES rates ($0.10 per 1,000 emails).
4.  **SMS Messages:** Billed at destination carrier SMS rates ($0.0065 to $0.05+ per SMS).

---

## 3. Key Cost Dimensions

| Pinpoint Feature | Free Allowance / Month | Rate (us-east-1) | Price for 100k Users / Messages |
|------------------|------------------------|------------------|---------------------------------|
| **Targeted Users (MTU)**| First 5,000 MTUs | **$0.0012 / MTU** | **$114.00 / month** |
| **Push Notifications** | First 1M push msgs | **$1.00 / 1M msgs**| **$0.10** |
| **Email Deliveries** | First 10k emails | **$0.10 / 1,000** | **$10.00** |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **MTU Rate:** $0.0012 per monthly targeted user endpoint.
*   **Push Message Rate:** $0.000001 per message ($1.00 per Million).
*   **Email Rate:** $0.10 per 1,000 emails ($0.0001 per email).

---

## 5. AWS Free Tier Coverage
*   **Amazon Pinpoint Free Tier:** Includes **5,000 free MTUs**, 1 Million free push notifications, and 10,000 free emails per month indefinitely.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Targeting Duplicate or Unverified Endpoint Tokens:** Targeting stale push notification tokens or unverified email addresses, inflating MTU volume counts ($1.20/1k MTUs).

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate Push & SMS Workloads to AWS End User Messaging:**
    *   Transition your mobile notification journeys to **AWS End User Messaging** prior to October 30, 2026.
    *   *Why:* Ensures continuous push/SMS notification delivery without disruption when Pinpoint APIs shut down.
2.  **Prune Inactive User Endpoints Monthly:** Implement automated scripts to remove inactive device tokens older than 90 days from Pinpoint segment definitions to keep billable MTU counts below active subscriber numbers.
