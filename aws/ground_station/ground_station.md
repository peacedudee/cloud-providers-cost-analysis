# AWS Service Cost Research: AWS Ground Station

> **Status:** ✅ Research Complete & Ground Truth Verified  
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

### A. Bandwidth Tiers & Contact Pricing
*   **Narrowband (<=2 MHz):** Optimized for command, telemetry, and tracking (TT&C) operations.
    *   *On-Demand:* **$9.90 per contact minute**.
    *   *Reserved (1-Yr):* **$4.95 per contact minute** (50% savings).
*   **Wideband (>2 MHz):** Optimized for high-rate earth observation, remote sensing, and payload data downlinks.
    *   *On-Demand:* **$21.78 per contact minute**.
    *   *Reserved (1-Yr):* **$10.89 per contact minute** (50% savings).

### B. Cancellation Window
*   **No Charge Window:** Cancellations of scheduled satellite contacts are free of charge if made more than 15 minutes before the scheduled contact start time.
*   **Late Cancellation Penalty:** Cancellations made **within 15 minutes** of the contact start time are billed at **100% of the scheduled duration rate**, even if no satellite connection was established.

---

## 4. Detailed Pricing Rates (us-east-1)

| Ground Station Bandwidth | Pricing Model | Rate per Contact Minute | Price for 100 Contact Minutes |
|--------------------------|---------------|-------------------------|-------------------------------|
| **Narrowband (<=2 MHz)** | On-Demand | **$9.90 / minute** | **$990.00** |
| **Narrowband (<=2 MHz)** | Reserved (1-Yr) | **$4.95 / minute** | **$495.00** (50% off) |
| **Wideband (>2 MHz)** | On-Demand | **$21.78 / minute** | **$2,178.00** |
| **Wideband (>2 MHz)** | Reserved (1-Yr) | **$10.89 / minute** | **$1,089.00** (50% off) |

### Example Monthly Cost Calculation (On-Demand vs Reserved)
*Workload: A weather satellite downlinks data twice a day using a Wideband (>2 MHz) receiver. Each contact window is exactly 12 minutes. The team schedules 60 passes per month.*

*   **Option A: On-Demand Wideband Routing:**
    *   Cost per pass:
        $$12\text{ minutes} \times \$21.78/\text{min} = \$261.36$$
    *   Monthly Cost (60 passes):
        $$\text{Monthly Cost} = \$261.36 \times 60 = \$15,681.60\text{ / month}$$
*   **Option B: Reserved Wideband Routing (50% Discount):**
    *   Cost per pass:
        $$12\text{ minutes} \times \$10.89/\text{min} = \$130.68$$
    *   Monthly Cost (60 passes):
        $$\text{Monthly Cost} = \$130.68 \times 60 = \$7,840.80\text{ / month}$$
*   **Monthly Savings:** **$7,840.80/month** (A significant cost reduction for predictable pass orbits).

---

## 5. AWS Free Tier Coverage
*   **AWS Ground Station:** No free tier is available. Any scheduled antenna contact window incurs active billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **No-Show Contact Fees for Unused Scheduled Antenna Passes:** Scheduling satellite pass windows and failing to cancel them at least 15 minutes before execution, incurring 100% of the antenna contact fee for zero data downlinked.

---

## 7. Actionable Cost Optimization Strategies
1.  **Cancel Unneeded Passes >15 Minutes in Advance:**
    *   Automate Ground Station contact scheduling scripts to evaluate weather/cloud coverage and cancel unneeded downlink passes at least 20 minutes before the pass window starts.
    *   **The Savings:** Avoids the 100% late cancellation penalty.
2.  **Purchase Reserved Contact Minutes for Predictable Orbits:**
    *   For satellites in fixed sun-synchronous orbits with predictable daily passes, purchase a 1-Year Ground Station Reserved plan.
    *   **The Savings:** Slashes contact minute rates by **50%**.
3.  **Optimize Pass Sizing:** Schedule contacts only for the actual orbital duration when the satellite is at an elevation angle that guarantees high signal quality. Avoid scheduling the flat horizons when signal quality is low but antenna minutes are billed at the full rate.
