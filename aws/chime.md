# AWS Service Cost Research: Amazon Chime / Amazon Chime SDK

> **Status:** ⚠️ Client App Terminated (Developer SDK Active)  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Chime was originally a unified communications service for online meetings, video conferencing, and business calling.
* **Critical Service Status Notice:** Support for the standalone **Amazon Chime end-user client application was terminated on February 20, 2026**.
* **Active Developer SDK:** The **Amazon Chime SDK** remains fully active and supported. It enables developers to build real-time audio, video, screen sharing, and WebRTC media processing directly into custom web and mobile applications. Chime SDK is serverless, billing strictly per attendee-minute.

---

## 2. Billing Mechanics
1. **Audio Attendee Minutes:** Billed per minute per participant connected to audio ($0.0017 per audio attendee minute).
2. **Video Attendee Minutes:** Billed per minute per participant streaming or receiving video ($0.0040 per video attendee minute).
3. **Media Pipelines (Recording & Composition):** Billed per minute of active WebRTC media capture or recording pipeline ($0.005 to $0.020 per minute).

---

## 3. Key Cost Dimensions

| Chime SDK Feature | Billed Unit | Rate (us-east-1) | Price for 1,000 Attendee-Minutes |
|-------------------|-------------|------------------|----------------------------------|
| **Audio Attendee** | Per participant-min | **$0.0017** | **$1.70** |
| **Video Attendee** | Per participant-min | **$0.0040** | **$4.00** |
| **Media Recording** | Per pipeline-min | **$0.0125** | **$12.50** |

---

## 4. Detailed Pricing Rates (us-east-1)

* **Audio Attendee Rate:** $0.0017 per minute ($1.70 per 1,000 audio minutes).
* **Video Attendee Rate:** $0.0040 per minute ($4.00 per 1,000 video minutes).
* **Media Pipeline Rate:** $0.0125 per composite recording minute.

---

## 5. AWS Free Tier Coverage
* **Amazon Chime SDK Free Tier:** Includes **250 free audio attendee minutes** per month for 12 months for new AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
* **Leaving Background Video Streams Active When Minimized:** Allowing web or mobile clients to stream HD video ($0.0040/min) when the application is minimized or hidden in a background tab.

---

## 7. Actionable Cost Optimization Strategies
1. **Switch Inactive/Minimized Clients to Audio-Only Mode:**
   * Detect when a user minimizes their browser tab or mobile app, automatically pausing incoming video streams.
   * *Benefit:* Pausing video drops the attendee rate from **$0.0040/min down to $0.0017/min**.
   * **The Savings:** Cuts media session billing by **57%**.
2. **Auto-Disconnect Inactive Meetings After 15 Minutes:** Implement WebSockets timers to automatically terminate meeting sessions when all participants have been muted or idle for 15 consecutive minutes.
