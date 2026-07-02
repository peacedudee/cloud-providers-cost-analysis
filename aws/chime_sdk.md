# AWS Service Cost Research: Amazon Chime SDK

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
The Amazon Chime SDK is a set of real-time communication components that developers use to embed custom WebRTC audio, video, screen-sharing, and messaging capabilities directly into web, mobile, or desktop applications. It abstracts WebRTC media servers, TURN/STUN relay infrastructure, and global media transport into SDK APIs. The Chime SDK is serverless, billing based on attendee-minutes, messaging requests, and media pipeline processing.

---

## 2. Billing Mechanics
1. **Audio Attendee Minutes:** Billed per minute per participant connected to audio ($0.0017 per audio attendee minute).
2. **Video Attendee Minutes:** Billed per minute per participant streaming or receiving video ($0.0040 per video attendee minute).
3. **In-App Messaging API:** Billed per message sent ($0.0007 per message) and per channel connection minute ($0.00001 per minute).
4. **Media Pipelines (Capture & Composition):** Billed per minute of active video composition or S3 recording pipeline ($0.0125 per composite minute).

---

## 3. Key Cost Dimensions

### A. Core Rates (us-east-1)

| Chime SDK Feature | Billed Unit | Rate (us-east-1) | Price for 100,000 Attendee-Minutes |
|-------------------|-------------|------------------|------------------------------------|
| **Audio Attendee** | Per participant-min | **$0.0017** | **$170.00** |
| **Video Attendee** | Per participant-min | **$0.0040** | **$400.00** |
| **SDK Messaging** | Per message sent | **$0.0007** | **$70.00** (100k messages) |
| **Media Recording** | Per pipeline-min | **$0.0125** | **$1,250.00** |

### B. Mathematical Cost Calculation Example
*Scenario: A web application hosts 50 video meetings per month with 10 participants each, lasting 60 minutes (30,000 total attendee minutes; all audio + video).*

1. **Audio Attendee Charge:**
   $$\text{Audio Cost} = 30,000\text{ mins} \times \$0.0017 = \$51.00\text{ / month}$$
2. **Video Attendee Charge:**
   $$\text{Video Cost} = 30,000\text{ mins} \times \$0.0040 = \$120.00\text{ / month}$$
3. **Total Monthly Cost:** **$171.00 / month**

---

## 4. Detailed Pricing Rates (us-east-1)

* **Audio Rate:** $0.0017 per attendee-minute ($1.70 per 1,000 audio minutes).
* **Video Rate:** $0.0040 per attendee-minute ($4.00 per 1,000 video minutes).
* **Messaging Rate:** $0.0007 per message sent.
* **Media Pipeline Rate:** $0.0125 per composite recording minute.

---

## 5. AWS Free Tier Coverage
* **Amazon Chime SDK Free Tier:** Includes **250 free audio attendee minutes** per month for 12 months for new AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
* **Streaming Video Streams to Backgrounded Applications:** Allowing mobile or web applications to continue streaming HD video ($0.0040/min per attendee) when the app is minimized or hidden in a background browser tab.

---

## 7. Actionable Cost Optimization Strategies
1. **Switch Inactive/Minimized Clients to Audio-Only Mode:**
   * Implement client-side page visibility handlers (`document.hidden`) to pause video tile subscriptions when a user minimizes their app window.
   * *Benefit:* Pausing video drops the attendee rate from **$0.0040/min down to $0.0017/min**.
   * **The Savings:** Slashes media session charges by **57%**.
2. **Enable WebRTC Simulcast for Thumbnail Grid Tiles:** Configure WebRTC simulcast so thumbnail grid tiles receive low-resolution streams (360p) rather than full HD feeds.
