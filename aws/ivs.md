# AWS Service Cost Research: Amazon Interactive Video Service (IVS)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Interactive Video Service (IVS) is a managed live video streaming solution designed to build interactive, low-latency (<3 seconds) or real-time WebRTC (<300ms) video streaming experiences for live commerce, gaming, e-learning, and social applications (built on the core technology that powers Twitch). IVS manages ingestion, ABR transcoding, CDN distribution, and interactive chat APIs. IVS is billed per channel input hour and per viewer output hour.

---

## 2. Billing Mechanics
Amazon IVS billing consists of two core components:
1.  **Live Video Input Duration:** Billed per hour that live video is ingested into an IVS channel ($0.035 to $0.15 per channel-hour).
2.  **Live Video Output Duration (Viewer Hours):** Billed per hour of video delivered to viewers, tiered by resolution (HD vs. SD) and viewer region.
3.  **Real-Time WebRTC Stage Sessions:** Billed per participant-minute connected to interactive WebRTC stages ($0.0035 to $0.0050 per minute).

---

## 3. Key Cost Dimensions

### A. Channel Input Rates (us-east-1)
*   **Basic Channel Input (up to 720p 30fps):** **$0.035 per channel-hour**.
*   **Standard Channel Input (up to 1080p 60fps):** **$0.150 per channel-hour**.

### B. Viewer Output Delivery Rates (US / EU Viewers)
*   **HD Output (1080p):** **$0.075 per viewer-hour**.
*   **SD Output (480p):** **$0.015 per viewer-hour** (**80% cheaper!**).
*   **Mobile / Low-Res Output (360p / 160p):** **$0.0075 per viewer-hour** (**90% cheaper!**).

---

## 4. Detailed Pricing Rates (us-east-1)

| IVS Stream Component | Output Resolution | Rate per Hour | Cost for 1,000 Viewer-Hours |
|----------------------|-------------------|---------------|-----------------------------|
| **Input Channel** | 1080p Standard | **$0.150 / hr** | **$0.15** (1 channel-hr) |
| **Viewer Output (HD)** | 1080p HD | **$0.075 / viewer-hr**| **$75.00** |
| **Viewer Output (SD)** | 480p SD | **$0.015 / viewer-hr**| **$15.00** |
| **Viewer Output (Low)**| 360p Mobile | **$0.0075 / viewer-hr**| **$7.50** |

---

## 5. AWS Free Tier Coverage
*   **Amazon IVS Free Tier:** Includes **5 hours of live video input** and **100 hours of SD video output** per month for 12 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Streaming 1080p HD to All Mobile Viewers ($0.075/hr):**
    *   Defaulting all mobile app video players to 1080p HD stream renditions.
    *   *Math:* 10,000 mobile viewers watching a 2-hour live stream at 1080p HD:
        $$20,000 \text{ viewer-hours} \times \$0.075 = \mathbf{\$1,500.00 \text{ in delivery fees!}}$$

---

## 7. Actionable Cost Optimization Strategies
1.  **Default Mobile App Players to SD (480p) Renditions:**
    *   Configure mobile player SDKs to default to 480p SD or 360p renditions on cellular connections.
    *   *Why:* On small smartphone screens, 480p SD provides identical visual quality while costing **$0.015/hr vs $0.075/hr**.
    *   **The Savings:** Slashes live video delivery costs by **80%** ($300.00 vs $1,500.00 for 20,000 viewer-hours).
2.  **Use Basic Input Channels for SD-Only Streams:** Select Basic Channel type ($0.035/hr) for sub-720p streams instead of Standard ($0.15/hr).
3.  **Auto-Stop Idle Stream Channels:** Set maximum stream duration limits in your app backend to terminate abandoned live streams.
