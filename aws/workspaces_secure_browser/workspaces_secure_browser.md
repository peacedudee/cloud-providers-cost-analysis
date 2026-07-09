# AWS Service Cost Research: Amazon WorkSpaces Secure Browser

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon WorkSpaces Secure Browser (formerly **WorkSpaces Web**) is a fully managed enterprise virtual browser service that provides secure access to internal corporate websites and SaaS applications from any unmanaged or personal device (BYOD). It isolates web browsing sessions inside containerized browser instances running in AWS, shielding corporate data with granular data protection policies (disabling copy/paste, file downloads, or local printing). WorkSpaces Secure Browser is billed per Monthly Active User (MAU).

---

## 2. Billing Mechanics
1.  **Monthly Active User Fee (MAU):** Billed monthly per unique user who streams a browser session during a calendar month ($7.00 per MAU).
2.  **Included Streaming Hours:** Includes up to **200 hours of browser streaming** per user per month.
3.  **Overage Streaming Rate:** Billed per additional streaming hour ($0.035 per hour) beyond 200 hours in a month.

---

## 3. Key Cost Dimensions

| Secure Browser Metric | Included Monthly Allowance | Rate (us-east-1) | Price for 100 Active Users |
|-----------------------|----------------------------|------------------|-----------------------------|
| **Monthly Active User (MAU)**| 200 hours / user-month | **$7.00 / MAU** | **$700.00 / month** |
| **Overage Streaming** | Per extra hour | **$0.035 / hour**| $3.50 per 100 extra hours |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **MAU Rate:** $7.00 per user per month (up to 200 hours).
*   **Overage Hourly Rate:** $0.035 per hour ($0.84 per 24 hours of streaming).

---

## 5. AWS Free Tier Coverage
*   **WorkSpaces Secure Browser Free Tier:** Includes a 30-day free trial of **$7.00 credit per user** for up to 30 users.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Provisioning Full VDI Desktops ($33/mo) for Web-Only Workers:** Provisioning full Amazon WorkSpaces virtual desktops ($33.00/mo) for contractors or customer support agents who only require access to internal web apps and SaaS tools.

---

## 7. Actionable Cost Optimization Strategies
1.  **Replace Full VDI Desktops with Secure Browser for Web Workers:**
    *   Migrate contractors, remote support agents, and task workers from full Amazon WorkSpaces ($33.00/mo) to **WorkSpaces Secure Browser ($7.00/mo)**.
    *   *Why:* Secure Browser provides isolated Chromium web browser access to internal web portals without managing Windows OS licenses or desktop storage.
    *   **The Savings:** Slashes virtual workspace costs by **78% ($26.00/month saved per user)**.
2.  **Set Session Timeout Policies:** Enforce a 15-minute idle session disconnect policy in the WorkSpaces Secure Browser console to prevent users from consuming overage streaming hours ($0.035/hr) overnight.
