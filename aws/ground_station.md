# AWS Service Cost Research: AWS Ground Station

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Ground Station is a fully managed service that enables satellite operators to control satellite communications, downlink and process satellite data (earth observation, weather imaging, telemetry), and scale satellite operations without building or managing physical ground station infrastructure. Ground Station provides global parabolic dish antenna arrays, streaming raw satellite data directly into AWS services (S3, EC2, Kinesis) via direct VPC fiber connections. Ground Station is billed per antenna contact minute.

---

## 2. Billing Mechanics
1.  **Contact Minutes:** Billed per minute of scheduled antenna contact time with a satellite.
2.  **Bandwidth Tier:**
    *   *Narrowband (<=2 MHz):* Low-bandwidth telemetry downlinks ($9.90 per minute On-Demand).
    *   *Wideband (>2 MHz):* High-bandwidth payload imagery downlinks ($21.78 per minute On-Demand).
3.  **Pricing Models:**
    *   *On-Demand:* Pay-as-you-go with no long-term commitment.
    *   *Reserved Contact Minutes:* 1-Year monthly minute commitments offer **50% savings**.
4.  **Cancellation Policy:** Cancellations made within 15 minutes of scheduled contact time incur full 100% contact charges.

---

## 3. Key Cost Dimensions

| Ground Station Bandwidth | Pricing Model | Rate per Contact Minute | Price for 100 Contact Minutes |
|--------------------------|---------------|-------------------------|-------------------------------|
| **Narrowband (<=2 MHz)** | On-Demand | **$9.90 / minute** | **$990.00** |
| **Narrowband (<=2 MHz)** | Reserved (1-Yr) | **$4.95 / minute** | **$495.00** (50% off) |
| **Wideband (>2 MHz)** | On-Demand | **$21.78 / minute** | **$2,178.00** |
| **Wideband (>2 MHz)** | Reserved (1-Yr) | **$10.89 / minute** | **$1,089.00** (50% off) |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Narrowband On-Demand Rate:** $9.90 per contact minute.
*   **Wideband On-Demand Rate:** $21.78 per contact minute.
*   **Reserved Rate:** 50% discount off On-Demand rates.

---

## 5. AWS Free Tier Coverage
*   **AWS Ground Station Free Tier:** None. Antenna contact time incurs per-minute charges.

---

## 6. Common Cost Hotspots & Pitfalls
*   **No-Show Contact Fees for Unused Scheduled Antenna Passes:**
    *   Scheduling satellite pass windows and failing to cancel them at least 15 minutes before execution, incurring 100% of the antenna contact fee ($217.80 per 10-minute pass) for zero data downlinked.

---

## 7. Actionable Cost Optimization Strategies
1.  **Cancel Unneeded Passes >15 Minutes in Advance:**
    *   Automate Ground Station contact scheduling scripts to evaluate weather/cloud coverage and cancel unneeded downlink passes at least 20 minutes before the pass window starts.
    *   **The Savings:** Avoids 100% no-show penalty fees.
2.  **Purchase Reserved Contact Minutes for Predictable Orbits:**
    *   For satellites in fixed sun-synchronous orbits with predictable daily passes, purchase a 1-Year Ground Station Reserved plan.
    *   **The Savings:** Slashes contact minute rates by **50% ($4.95/min vs $9.90/min)**.
