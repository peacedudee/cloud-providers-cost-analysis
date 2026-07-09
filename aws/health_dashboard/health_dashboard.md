# AWS Service Cost Research: AWS Health Dashboard

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Health Dashboard provides ongoing visibility into the operational status of AWS services and your specific AWS resources. It provides global AWS service availability status (Service Health) and personalized alerts for events impacting your account (Personal Health Dashboard), such as scheduled EC2 maintenance, hardware degradation, security notifications, or API deprecations. The Health Dashboard console is **100% free**.

---

## 2. Billing Mechanics
1.  **Global & Personal Health Console:** **100% Free ($0.00)**. No fees for accessing dashboard alerts, viewing event histories, or receiving email notifications.
2.  **AWS Health API (Programmatic & Organizational Access):** Access to the AWS Health API (`DescribeEvents`, `DescribeEventDetails`) is included with **AWS Business Support** ($100.00+/mo) or **Enterprise Support** plans.

---

## 3. Key Cost Dimensions

| Feature / Access Mode | Support Tier Requirement | Rate (us-east-1) | Cost Impact |
|-----------------------|--------------------------|------------------|-------------|
| **Global Service Health Console**| Basic Support+ | **Free ($0.00)** | **$0.00** |
| **Personal Health Dashboard**| Basic Support+ | **Free ($0.00)** | **$0.00** |
| **AWS Health API Access** | Business / Enterprise Support | Support Included | Included in Support Plan |
| **EventBridge Health Integration**| Basic Support+ | **Free ($0.00)** | $0.00 (Standard EventBridge rates)|

---

## 4. Detailed Pricing Rates (us-east-1)
*   **Health Dashboard Console Fee:** $0.00 per month.
*   **Health Event Alerts (Console / Email):** $0.00.

---

## 5. AWS Free Tier Coverage
*   **AWS Health Dashboard:** Console access is always 100% free for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Ignoring Hardware Retirement / Degradation Alerts:** Failing to monitor Personal Health Dashboard alerts for EC2 instance hardware degradation or scheduled maintenance, leading to unexpected application downtime and emergency recovery costs.

---

## 7. Actionable Cost Optimization Strategies
1.  **Automate Remediation via EventBridge & Lambda (Free Tier):**
    *   Set up an **Amazon EventBridge Rule** to listen for `AWS Health Event` notifications (e.g., `AWS_EC2_PERSISTENT_INSTANCE_RETIREMENT_SCHEDULED`).
    *   Trigger an automated Lambda function to gracefully migrate or stop affected EC2 instances during off-peak hours.
    *   **The Savings:** Prevents unplanned production outages and avoids emergency high-cost recovery workflows for **$0.00 extra**.
2.  **Use Free Personal Health Dashboard Event History:** Check Personal Health Dashboard history before opening paid AWS Support tickets for resource connectivity issues, verifying whether an active regional AWS outage is already being addressed.
