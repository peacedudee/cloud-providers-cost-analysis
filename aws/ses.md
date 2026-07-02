# AWS Service Cost Research: Amazon SES (Simple Email Service)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon SES (Simple Email Service) is a cost-effective, high-throughput outbound and inbound email service for developers and marketers. SES enables applications to send transactional emails (password resets, order receipts), marketing newsletters, and automated notification blasts. SES includes deliverability dashboards, SPF/DKIM authentication management, bounce/complaint handling, and dedicated IP options. SES is billed per email sent/received and per GB of attachment payload.

---

## 2. Billing Mechanics
1.  **Outbound Emails Sent:** Billed per 1,000 emails sent ($0.10 per 1,000 emails = $0.0001 per email).
2.  **Inbound Emails Received:** Billed per 1,000 incoming emails processed ($0.10 per 1,000 emails).
3.  **Data Attachment Payload:** Billed per GB of email message headers and attachments ($0.12 per GB).
4.  **Dedicated IP Address (Optional):** Billed monthly per reserved dedicated IP ($24.95 per dedicated IP per month).

---

## 3. Key Cost Dimensions

| SES Service Component | Free Tier Allowance | Rate (us-east-1) | Price for 100,000 Emails / 10 GB |
|-----------------------|---------------------|------------------|----------------------------------|
| **Outbound Emails** | Vary by plan | **$0.10 / 1,000 emails** | **$10.00** |
| **Inbound Emails** | 1,000 emails / month| **$0.10 / 1,000 emails** | **$10.00** |
| **Data Attachments** | None | **$0.12 / GB** | **$1.20** |
| **Dedicated IP Lease**| None | **$24.95 / IP-month** | $24.95 / month |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Outbound Email Rate:** $0.10 per 1,000 emails ($100.00 per 1 Million emails).
*   **Data Payload Rate:** $0.12 per GB of outbound data.
*   **Dedicated IP Rate:** $24.95 per IP per month.

---

## 5. AWS Free Tier Coverage
*   **Amazon SES Free Tier:** Includes **3,000 free messages per month** for the first 12 months for eligible accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Leasing Dedicated IPs for Low-Volume Senders ($24.95/mo):** Leasing dedicated IPs when sending fewer than 300,000 emails/month, paying $24.95/mo in fixed IP leases while struggling to maintain IP warm-up reputation scores.
*   **Sending Large Email Attachments ($0.12/GB):** Embedding 10 MB PDF invoices directly in email payloads rather than sending download links.

---

## 7. Actionable Cost Optimization Strategies
1.  **Use Amazon SES Shared IP Pools ($0.00) for Volume Below 300k/Mo:**
    *   For senders delivering fewer than 300,000 emails per month, use the standard **Amazon SES Shared IP Pool**.
    *   *Why:* AWS actively monitors shared IP reputation for high deliverability at **$0.00 extra cost**.
    *   **The Savings:** Saves $299.40 per year per dedicated IP.
2.  **Host Attachments on S3 / CloudFront Instead of Inline Payloads:**
    *   Replace inline PDF attachments with secure S3 pre-signed links fronted by CloudFront ($0.085/GB vs $0.12/GB in SES).
    *   **The Savings:** Slashes data payload billing by **30%** while improving email delivery speeds.
3.  **Clean Email Lists to Prevent High Bounce Rates:** Use email verification tools to remove invalid addresses, avoiding bounce rates (>5%) that can trigger SES sending pauses.
