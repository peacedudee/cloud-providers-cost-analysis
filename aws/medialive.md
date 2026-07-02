# AWS Service Cost Research: AWS Elemental MediaLive

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Elemental MediaLive is a broadcast-grade live video processing service. It ingests live video streams (via RTP, RTMP, HLS, or MediaConnect) and encodes them in real-time into high-quality adaptive bitrate (ABR) streams for broadcast television and internet-connected devices. MediaLive supports advanced video codecs (AVC/H.264, HEVC/H.265, AV1) and multi-channel audio. MediaLive offers **Standard Pipelines** (dual-pipeline redundant high availability) and **Single Pipelines**. It is billed per channel-hour based on output resolution and encoding configuration.

---

## 2. Billing Mechanics
1.  **Running Channel Hours:** Billed per hour (or fraction of an hour) that a MediaLive channel is in the `RUNNING` state.
2.  **Pipeline Redundancy:**
    *   *Standard Pipeline:* Runs 2 independent encoding pipelines in separate Availability Zones for 100% SLA uptime.
    *   *Single Pipeline:* Runs 1 encoding pipeline (50% cheaper, no AZ redundancy).
3.  **Output Resolution & Codec:** Rates vary based on maximum output resolution (SD, HD, UHD/4K) and codec (AVC, HEVC).
4.  **Reserved Pricing:** 1-Year commitments offer up to **75% savings** for continuous 24/7 channels.

---

## 3. Key Cost Dimensions

### A. On-Demand Channel Hourly Rates (us-east-1 AVC Codec)
*   **HD Output (up to 1080p 30fps):**
    *   *Standard (Dual AZ):* **$1.176 per channel-hour** (~$858.48 per month 24/7).
    *   *Single Pipeline:* **$0.588 per channel-hour** (~$429.24 per month 24/7).
*   **SD Output (up to 480p 30fps):**
    *   *Standard (Dual AZ):* **$0.420 per channel-hour**.
    *   *Single Pipeline:* **$0.210 per channel-hour**.
*   **UHD / 4K Output (up to 60fps):**
    *   *Standard (Dual AZ):* **$4.704 per channel-hour**.
    *   *Single Pipeline:* **$2.352 per channel-hour**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Channel Resolution | Pipeline Class | On-Demand Rate / Hr | Monthly 24/7 On-Demand | 1-Year Reserved Rate / Hr |
|--------------------|----------------|---------------------|------------------------|---------------------------|
| **HD (1080p)** | Single Pipeline | **$0.588** | **$429.24** | **$0.147** (75% off) |
| **HD (1080p)** | Standard (Dual) | **$1.176** | **$858.48** | **$0.294** (75% off) |
| **UHD (4K)** | Single Pipeline | **$2.352** | **$1,716.96** | **$0.588** (75% off) |
| **UHD (4K)** | Standard (Dual) | **$4.704** | **$3,433.92** | **$1.176** (75% off) |

---

## 5. AWS Free Tier Coverage
*   **AWS Elemental MediaLive Free Tier:** None. Continuous channel-hour fees apply upon channel creation and execution.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Leaving Channels Running Idle After Live Events ($1.176/hr):** Forgetting to stop MediaLive channels after a live concert or sporting event finishes, leaving channels in `RUNNING` status ($28.22/day per HD channel).

---

## 7. Actionable Cost Optimization Strategies
1.  **Use Single-Pipeline Channels for Non-Critical Streams:**
    *   Select **Single Pipeline** mode ($0.588/hr) instead of Standard ($1.176/hr) for internal streams, secondary camera feeds, or non-mission-critical live broadcasts.
    *   **The Savings:** Instantly slashes live encoding costs by **50%**.
2.  **Purchase 1-Year Reserved Channels for 24/7 Linear Streams:**
    *   For 24/7 continuous broadcast streams, purchase a 1-Year MediaLive Reserved Channel.
    *   **The Savings:** Cuts hourly channel costs from $1.176/hr down to **$0.294/hr (75% savings)**, saving $6,700+ per year per HD channel.
3.  **Automate Channel Lifecycle via EventBridge & Lambda:** Automatically stop MediaLive channels at the scheduled end time of a live event using EventBridge Scheduler.
