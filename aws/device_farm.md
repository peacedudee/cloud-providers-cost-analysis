# AWS Service Cost Research: AWS Device Farm

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Device Farm is an application testing service that lets mobile app and web developers test and interact with Android, iOS, and web applications on real, physical devices and desktop browsers hosted in the AWS Cloud. Device Farm allows you to run automated test suites (Appium, Espresso, XCTest) concurrently across hundreds of real devices or perform manual interactive testing in a browser. Device Farm offers two pricing models: **Pay-As-You-Go (Metered)** and **Unmetered Monthly Slots**.

---

## 2. Billing Mechanics
1.  **Pay-As-You-Go (Metered Rate):** Billed per device-minute of test execution or manual remote access ($0.17 per device-minute).
2.  **Unmetered Monthly Slots:** Flat monthly subscription per device slot ($250.00 per slot per month) for unlimited test runs or remote access.
3.  **Free Trial:** Includes a one-time free trial of **1,000 device-minutes** for new accounts.

---

## 3. Key Cost Dimensions

| Device Farm Plan | Pricing Metric | Rate (us-west-2 / us-east-1) | Price for 2,000 Device-Minutes |
|------------------|----------------|------------------------------|--------------------------------|
| **Pay-As-You-Go** | Per device-minute | **$0.17 / minute** | **$340.00** |
| **Unmetered Slot**| Per slot-month | **$250.00 / month** | **$250.00** (Unlimited runs) |

---

## 4. Detailed Pricing Rates (us-east-1 / us-west-2)

*   **Metered Rate:** $0.17 per device-minute ($10.20 per device-hour).
*   **Unmetered Automated Testing Slot:** $250.00 per month per slot.
*   **Unmetered Remote Access Slot:** $250.00 per month per slot.

---

## 5. AWS Free Tier Coverage
*   **AWS Device Farm Free Tier:** Includes a one-time trial of **1,000 free device-minutes** for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Running Parallel Regressions on Metered Rates ($0.17/min):**
    *   Triggering an automated test suite across 20 physical devices on every git commit.
    *   *Math:* 20 devices × 30 min test = 600 device-minutes per build = **$102.00 per build trigger!**

---

## 7. Actionable Cost Optimization Strategies
1.  **Switch to Unmetered Slots ($250/mo) for CI/CD Automation:**
    *   If your team runs more than 1,470 device-minutes (~24.5 hours) of automated testing per month, purchase an **Unmetered Slot ($250.00/month)**.
    *   *Why:* Unmetered slots allow unlimited test executions sequentially on one device at a time for a flat fee.
    *   **The Savings:** Slashes testing bills from $1,000+/mo down to a flat $250/mo.
2.  **Filter Tests on Local Emulators Prior to Device Farm Runs:**
    *   Run unit and smoke tests locally on Android emulators / iOS simulators before triggering Device Farm runs.
    *   **The Savings:** Prevents wasting $0.17/min on failing builds due to simple syntax errors.
