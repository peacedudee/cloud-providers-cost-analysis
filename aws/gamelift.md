# AWS Service Cost Research: Amazon GameLift

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon GameLift is a managed dedicated game server hosting and matchmaking service designed to deploy, operate, and auto-scale low-latency multiplayer game servers for games built on Unreal Engine, Unity, Custom C++, or Godot. GameLift manages game session placement, fleet scaling based on player demand, and player matchmaking (GameLift FlexMatch). It offers **GameLift Managed EC2 Fleets** and **GameLift Anywhere** (for hybrid/on-premise server hosting).

---

## 2. Billing Mechanics
1.  **GameLift Managed Instance Hours:** Billed per instance-hour based on EC2 instance class (e.g. `c5.large`, `c6i.large`) and pricing model (On-Demand vs Spot).
2.  **GameLift Anywhere (Hybrid / On-Premise Servers):** Billed per active game process connection hour ($0.000833 per process-hour = ~$0.60 per month per process).
3.  **GameLift FlexMatch:** Matchmaking engine is **100% FREE ($0.00)**.
4.  **Outbound Data Egress (Gen 6+ Free Bandwidth):**
    *   *Gen 6/7 Instances (`c6i`, `c7g`):* Outbound network data egress is **100% FREE ($0.00)** as of June 2026!
    *   *Legacy Gen 5 Instances (`c5`):* Standard AWS data egress rates apply ($0.09/GB).

---

## 3. Key Cost Dimensions

| GameLift Fleet Configuration | Billing Model | Hourly Rate (us-east-1) | Price for 100 Instance-Hours |
|------------------------------|---------------|-------------------------|------------------------------|
| **`c5.large` Managed Fleet** | On-Demand | **$0.107 / hour** | **$10.70** |
| **`c5.large` Managed Fleet** | Spot Fleet (FleetIQ)| **$0.032 / hour** | **$3.20** (70% off) |
| **`c6i.large` Managed Fleet**| On-Demand | **$0.102 / hour** | **$10.20** (Includes **FREE Egress**) |
| **GameLift Anywhere Process**| Custom Hardware| **$0.000833 / process-hr**| **$0.08** |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **`c6i.large` On-Demand Rate:** $0.102 per hour ($0.00 egress).
*   **Spot Instance Rate:** ~70% discount off On-Demand rates.
*   **GameLift Anywhere Rate:** $0.000833 per process-hour.
*   **FlexMatch Rate:** $0.00.

---

## 5. AWS Free Tier Coverage
*   **Amazon GameLift Free Tier:** Includes **125 free hours of `c5.large` or `c4.large` On-Demand instance usage** per month for 12 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Running Legacy Gen 5 Fleets (`c5`) and Paying Data Egress Fees:**
    *   Hosting high-bandwidth multiplayer games on legacy `c5` instances, paying $0.09/GB for multi-terabyte game traffic egress when Gen 6 `c6i` instances provide 100% FREE bandwidth egress.

---

## 7. Actionable Cost Optimization Strategies
1.  **Upgrade Fleets to Gen 6/7 Instances (`c6i`, `c7g`) for Free Egress:**
    *   Migrate GameLift fleets to Generation 6 or 7 instances (`c6i.large`, `c7g.large`).
    *   *Why:* AWS provides **100% FREE outbound bandwidth egress ($0.00)** on Gen 6+ GameLift instances.
    *   **The Savings:** Saves thousands of dollars per month in network egress charges for bandwidth-heavy multiplayer games.
2.  **Use GameLift FleetIQ with Spot Instance Queues:**
    *   Configure GameLift queues using **Spot Instances (70% discount)** backed by On-Demand capacity fallbacks.
    *   **The Savings:** Slashes game server compute costs by **70%** ($0.032/hr vs $0.107/hr per instance).
