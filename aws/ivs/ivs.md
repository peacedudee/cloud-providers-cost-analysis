# AWS Service Cost Research: Amazon Interactive Video Service (IVS)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Interactive Video Service (IVS) is a managed live video streaming solution designed to build interactive, low-latency (<3 seconds) or real-time WebRTC (<300ms) video streaming experiences for live commerce, gaming, e-learning, and social applications (built on the core technology that powers Twitch). IVS manages ingestion, ABR transcoding, CDN distribution, and interactive chat APIs. IVS is billed based on channel input hours, viewer output hours, stage participant hours, and chat message volumes.

---

## 2. Billing Mechanics
Amazon IVS billing consists of three core models depending on the feature used:
1. **Low-Latency Streaming (Channels):**
   * **Live Video Input Duration:** Billed per hour that live video is ingested into an IVS channel, based on channel type ($0.035 per basic channel hour up to 720p; $0.150 per standard channel hour up to 1080p).
   * **Live Video Output Duration (Viewer Hours):** Billed per hour of video delivered to viewers, tiered by resolution (HD vs. SD vs. Low) and viewer region.
2. **Real-Time Streaming (WebRTC Stages):**
   * **Participant Hours:** Billed per hour that a host or viewer is connected to an interactive stage. Standard video rate is **$0.0720 per participant-hour** in `us-east-1` (evaluated per minute).
   * **Audio-Only Rate:** Billed at **1/10th of the video rate ($0.0072 per participant-hour)** if a user only publishes/subscribes to audio.
3. **Amazon IVS Chat:**
   * **Messages Sent:** Billed at **$0.560 per 1,000 messages** sent (SendMessage API calls).
   * **Messages Delivered:** Billed at **$0.008 per 1,000 messages** delivered to connected clients in a chat room.
   * *Chat Integration Credits:* Every 1 hour of low-latency video input grants **2,700 free sent messages** and **270,000 free delivered messages** per month.

---

## 3. Key Cost Dimensions

### A. Channel Input Rates (us-east-1)
* **Basic Channel Input (up to 720p 30fps):** **$0.035 per channel-hour**.
* **Standard Channel Input (up to 1080p 60fps):** **$0.150 per channel-hour**.

### B. Viewer Output Delivery Rates (US / EU Viewers)
* **HD Output (1080p):** **$0.075 per viewer-hour**.
* **SD Output (480p):** **$0.015 per viewer-hour** (**80% cheaper!**).
* **Mobile / Low-Res Output (360p / 160p):** **$0.0075 per viewer-hour** (**90% cheaper!**).

### C. Chat and Stage Rates (us-east-1)
* **Real-Time Video Participant Rate:** **$0.0720 per participant-hour**.
* **Real-Time Audio Participant Rate:** **$0.0072 per participant-hour**.
* **Chat Messages Sent:** **$0.560 per 1,000 messages**.
* **Chat Messages Delivered:** **$0.008 per 1,000 messages**.

---

## 4. Detailed Pricing Rates (us-east-1)

| IVS Stream Component | Output Resolution / Detail | Rate (us-east-1) | Cost for 10,000 Units |
|----------------------|-------------------|---------------|-----------------------------|
| **Input Channel** | 1080p Standard | **$0.150 / hr** | **$1.50** (10 channel-hrs) |
| **Input Channel** | 720p Basic | **$0.035 / hr** | **$0.35** (10 channel-hrs) |
| **Viewer Output (HD)** | 1080p HD | **$0.075 / viewer-hr**| **$750.00** (10k viewer-hrs) |
| **Viewer Output (SD)** | 480p SD | **$0.015 / viewer-hr**| **$150.00** (10k viewer-hrs) |
| **Viewer Output (Low)**| 360p Mobile | **$0.0075 / viewer-hr**| **$75.00** (10k viewer-hrs) |
| **Stage Participant**| Video | **$0.072 / participant-hr**| **$720.00** (10k participant-hrs)|
| **Stage Participant**| Audio-Only | **$0.0072 / participant-hr**| **$72.00** (10k participant-hrs)|
| **Chat Messages Sent**| Sent Message | **$0.560 / 1,000 msgs**| **$5.60** (10k msgs) |
| **Chat Messages Deliv.**| Delivered Message | **$0.008 / 1,000 msgs**| **$0.08** (10k msgs) |

---

## 5. AWS Free Tier Coverage
* **Amazon IVS Free Tier:** New AWS accounts receive the following monthly allowances for the first 12 months:
  * **Low-Latency Ingestion:** **5 hours of live video input** (basic channel) and **100 hours of SD video output** per month.
  * **Real-Time Stages:** **20 participant-hours** of video or 200 participant-hours of audio-only per month.
  * **Chat:** **13,500 messages sent** and **270,000 messages delivered** per month.

---

## 6. Common Cost Hotspots & Pitfalls
* **Streaming 1080p HD to All Mobile Viewers ($0.075/hr):**
  * Defaulting all mobile app video players to 1080p HD stream renditions.
  * *Math:* 10,000 mobile viewers watching a 2-hour live stream at 1080p HD:
    $$20,000 \text{ viewer-hours} \times \$0.075 = \mathbf{\$1,500.00 \text{ in delivery fees!}}$$
* **Massive Chat Delivery Multiplication:**
  * Deluging chat rooms with frequent bot messages or animations. If a chat room has 5,000 users, sending 100 messages results in 500,000 delivered messages:
    $$\text{Sent:} \ 100 \text{ msgs} \times \$0.56 / 1,000 = \$0.056$$
    $$\text{Delivered:} \ 500,000 \text{ msgs} \times \$0.008 / 1,000 = \mathbf{\$4.00 \text{ (Multiplication factor!)}}$$

---

## 7. Actionable Cost Optimization Strategies
1. **Default Mobile App Players to SD (480p) Renditions:**
   * Configure mobile player SDKs to default to 480p SD or 360p renditions on cellular connections.
   * *Why:* On small smartphone screens, 480p SD provides identical visual quality while costing **$0.015/hr vs $0.075/hr**.
   * **The Savings:** Slashes live video delivery costs by **80%** ($300.00 vs $1,500.00 for 20,000 viewer-hours).
2. **Use Basic Input Channels for SD-Only Streams:**
   * Select Basic Channel type ($0.035/hr) for sub-720p streams instead of Standard ($0.15/hr).
3. **Optimize Chat Frequency (Throttling / Batching):**
   * Implement client-side chat message throttling or batch updates to reduce the number of messages sent and delivered.
4. **Auto-Stop Stage Sessions & Idle Stream Channels:**
   * Set maximum stream duration limits in your app backend to terminate abandoned stage sessions or live streams when no publishers are present.
