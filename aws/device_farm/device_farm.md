# AWS Service Cost Research: AWS Device Farm

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Device Farm is an application testing service that lets mobile app and web developers test and interact with Android, iOS, and web applications on real, physical devices and desktop browsers hosted in the AWS Cloud. It supports automated test runs (Appium, Espresso, XCTest) and manual interactive debugging. Device Farm offers three pricing models: **Pay-As-You-Go (Metered)**, **Unmetered Monthly Slots**, and **Private Devices (Dedicated Hosting)**.

---

## 2. Billing Mechanics
Device Farm billing is structured across three primary access plans:
1.  **Pay-As-You-Go (Metered):** Billed per device-minute of active automated testing or manual remote access ($0.17 per device-minute).
2.  **Unmetered Monthly Slots:** Flat monthly subscription per device slot ($250.00 per slot per month) for unlimited test runs or remote access sequentially on public device pools.
3.  **Private Devices (Dedicated Hosting):** Physical devices hosted exclusively for your account starting at **$200.00 per month per device** (tiered by device model, minimum subscription duration commitments apply).

---

## 3. Key Cost Dimensions

| Device Farm Plan | Pricing Metric | Rate (us-west-2 / us-east-1) | Price for 10 Devices / Month (Continuous Use) |
|------------------|----------------|------------------------------|----------------------------------------------|
| **Pay-As-You-Go** | Per device-minute | **$0.17 / minute** | Billed by minute ($10.20 per device-hour) |
| **Unmetered Slot**| Per slot-month | **$250.00 / month** | **$2,500.00 / month** (Unlimited runs) |
| **Private Device**| Per device-month | **Starts at $200.00 / month**| **$2,000.00+ / month** (Exclusive access) |

---

## 4. Detailed Pricing Rates (us-east-1 / us-west-2)
*   **Metered Rate:** $0.17 per device-minute ($10.20 per device-hour).
*   **Unmetered Automated Testing Slot:** $250.00 per month per slot.
*   **Unmetered Remote Access Slot:** $250.00 per month per slot.
*   **Private Device Subscription:** Starts at $200.00 per month (exact pricing depends on hardware model).

---

## 5. AWS Free Tier Coverage
*   **AWS Device Farm Free Trial:** Includes a one-time trial of **1,000 free device-minutes** for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Running Parallel Regression Testing on Metered Rates:** Triggering automated testing across 30 physical devices on every single code commit.
    *   *Math:* 30 devices × 40 min test = 1,200 device-minutes per build.
    *   *Cost:* $$1,200 \times \$0.17 = \$204.00\text{ per build trigger!}$$
*   **Underutilizing Active Unmetered Slots:** Purchasing multiple unmetered slots ($250.00/mo each) for infrequent tests that could be handled cheaper via metered rates.

---

## 7. Actionable Cost Optimization Strategies
1.  **Switch to Unmetered Slots ($250/mo) for Automated CI/CD:**
    *   If your team runs more than **1,470 device-minutes** (~24.5 hours) of testing per month, purchase an Unmetered Slot.
    *   **The Savings:** Caps the monthly bill to a flat $250.00 instead of accumulating $10.20/hour metered charges.
2.  **Evaluate Private Devices ($200+/mo) for Custom Environments:**
    *   Choose **Private Devices** if your application testing requires custom OS configurations, static IP addresses, custom preloaded data, or strict compliance requirements that public shared device pools cannot support.
3.  **Run Pre-Filter Smoke Tests Locally:**
    *   Configure unit tests and basic UI smoke tests to run locally on Android emulators or iOS simulators prior to launching Device Farm runs.
    *   **The Savings:** Prevents spending $0.17/min on tests that fail immediately due to minor syntax or compile errors.
4.  **Use Desktop Browser Testing for Web Apps:**
    *   Use Selenium/Playwright browser automation grid features which are billed under lower, standard EC2 compute rates rather than the physical mobile device-minute rates.
