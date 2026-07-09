# AWS Service Cost Research: Amazon WorkSpaces

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon WorkSpaces is a fully managed Desktop-as-a-Service (DaaS) solution that enables businesses to provision virtual cloud desktops for Windows, Linux, and Ubuntu users worldwide. WorkSpaces provides secure access to corporate applications, document storage, and enterprise desktop environments from any device (laptop, tablet, web browser). WorkSpaces offers two billing models: **AlwaysOn (Monthly Flat Rate)** for full-time employees and **AutoStop (Hourly Rate)** for part-time workers.

---

## 2. Billing Mechanics
1.  **AlwaysOn Monthly Flat Rate:** Billed a fixed monthly fee per user for unlimited 24/7 desktop access ($21.00 to $78.00+ per user per month).
2.  **AutoStop Hourly Billing:** Billed a lower base monthly fee ($7.25 to $13.00 per month for storage and infrastructure) plus an hourly rate ($0.22 to $0.67 per hour) when the WorkSpace is active.
3.  **OS & Software Licensing:** Bundled Microsoft Windows / Office software licenses or BYOL discounts.
4.  **Note:** WorkSpaces Pools (non-persistent desktops) stopped accepting new customers on July 31, 2026.

---

## 3. Key Cost Dimensions

### A. Windows Desktop Monthly Bundle Rates (us-east-1)
*   **`Value` (1 vCPU, 2 GB RAM, 80 GB Storage):** **$21.00 / user-month** (AlwaysOn) OR **$7.25/mo + $0.22/hr** (AutoStop).
*   **`Standard` (2 vCPU, 4 GB RAM, 80 GB Storage):** **$33.00 / user-month** (AlwaysOn) OR **$9.75/mo + $0.30/hr** (AutoStop).
*   **`Performance` (2 vCPU, 8 GB RAM, 80 GB Storage):** **$44.00 / user-month** (AlwaysOn) OR **$9.75/mo + $0.47/hr** (AutoStop).
*   **`Power` (4 vCPU, 16 GB RAM, 100 GB Storage):** **$78.00 / user-month** (AlwaysOn) OR **$13.00/mo + $0.67/hr** (AutoStop).

---

## 4. Detailed Pricing Rates (us-east-1)

| WorkSpace Bundle | CPU / Memory | AlwaysOn (Flat Rate) | AutoStop Base + Hourly | Break-Even Point |
|------------------|--------------|----------------------|------------------------|------------------|
| **`Value`** | 1 vCPU, 2 GB | **$21.00 / mo** | **$7.25/mo + $0.22/hr**| ~62 hours / month |
| **`Standard`** | 2 vCPU, 4 GB | **$33.00 / mo** | **$9.75/mo + $0.30/hr**| ~77 hours / month |
| **`Performance`**| 2 vCPU, 8 GB | **$44.00 / mo** | **$9.75/mo + $0.47/hr**| ~73 hours / month |
| **`Power`** | 4 vCPU, 16 GB| **$78.00 / mo** | **$13.00/mo + $0.67/hr**| ~97 hours / month |

---

## 5. AWS Free Tier Coverage
*   **Amazon WorkSpaces Free Tier:** Includes **2 `Standard` bundle WorkSpaces** for up to 40 hours per month total for 2 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Assigning AlwaysOn Flat Rates to Part-Time Contractor Accounts:** Paying $33.00/mo AlwaysOn flat rates for contractor WorkSpaces accessed for only 10 hours a month, overpaying by **$20.25/month per contractor**.

---

## 7. Actionable Cost Optimization Strategies
1.  **Use Amazon WorkSpaces Cost Optimizer (Automated Billing Switcher):**
    *   Deploy the official **AWS WorkSpaces Cost Optimizer** solution (an automated AWS Fargate task).
    *   *How it works:* It analyzes individual user logon hours monthly and automatically switches accounts between **AlwaysOn** and **AutoStop** based on break-even thresholds (~77 hours/month).
    *   **The Savings:** Slashes overall WorkSpaces monthly expenditure by **20–40%**.
2.  **Right-Size User Bundles:** Downgrade basic administrative staff from `Performance` (8 GB RAM, $44/mo) to `Standard` (4 GB RAM, $33/mo).
3.  **Leverage Bring Your Own License (BYOL):** Use existing Microsoft Windows Desktop enterprise licenses to save **$4.00 per user per month**.
