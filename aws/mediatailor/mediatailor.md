# AWS Service Cost Research: AWS Elemental MediaTailor

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Elemental MediaTailor is a channel assembly and server-side ad insertion (SSAI) service that enables video providers to insert targeted, personalized advertising into live or VOD streams without client-side ad blockers detecting them. It also allows content owners to assemble linear OTT channels from existing VOD content assets cost-effectively. MediaTailor stitchers insert targeted ads directly into the ABR manifest on the server side. MediaTailor is billed per ad insertion request, channel assembly hour, and monetization lifecycle hook.

---

## 2. Billing Mechanics
1. **Server-Side Ad Insertion (SSAI):**
   * *Live Ad Insertion:* Billed per 1,000 ad insertion requests processed ($0.50 per 1,000 ad insertions).
   * *VOD Ad Insertion:* Billed per 1,000 ad insertion requests processed ($0.25 per 1,000 ad insertions).
2. **Channel Assembly:** Billed per hour that a linear OTT channel is running ($0.10 per channel-hour). Inactive channels incur $0.00.
3. **Ad Transcoding (Transcode On-Demand):** Includes **10 free unique ad transcodes per 1,000 ad insertions**; extra transcodes are billed at standard MediaConvert rates.
4. **Monetization Lifecycle Functions:** Billed per invocation for custom Ad Decision Server (ADS) request customization hooks.

---

## 3. Key Cost Dimensions

| MediaTailor Feature | Billed Metric | Rate (us-east-1) | Price for 100,000 Ad Insertions |
|---------------------|---------------|------------------|---------------------------------|
| **Live Ad Insertion** | Per 1,000 ad requests | **$0.50 / 1k ads** | **$50.00** |
| **VOD Ad Insertion** | Per 1,000 ad requests | **$0.25 / 1k ads** | **$25.00** |
| **Channel Assembly** | Per channel-hour | **$0.10 / hour** | **$73.00 / month** (24/7 channel) |

*Note: Custom volume discounts apply for accounts exceeding 60 million ad insertions per month.*

---

## 4. Detailed Pricing Rates (us-east-1)

* **Live Ad Rate:** $0.50 per 1,000 ad insertions ($500.00 per Million ads).
* **VOD Ad Rate:** $0.25 per 1,000 ad insertions ($250.00 per Million ads).
* **Channel Assembly Rate:** $0.10 per channel-hour ($2.40 per day).

---

## 5. AWS Free Tier Coverage
* **AWS Elemental MediaTailor Free Tier:** Includes **100,000 free ad insertion requests** during your first month of usage.

---

## 6. Common Cost Hotspots & Pitfalls
* **Un-Cached Ad Transcoding (Ad Creative Sprawl):** Requesting new third-party ad decision server (ADS) ad URLs that trigger dynamic MediaConvert transcodes on every ad break instead of caching transcoded ad creatives.

---

## 7. Actionable Cost Optimization Strategies
1. **Pre-Transcode & Cache Frequent Ad Creatives:**
   * Use MediaTailor **Ad Decision Server (ADS)** response caching and pre-transcode common commercial ad assets into matching ABR profiles in S3.
   * *Why:* Avoids triggering full MediaConvert transcode jobs ($0.015/min) for identical commercial ads.
   * **The Savings:** Eliminates extra ad transcoding fees.
2. **Use Channel Assembly for Linear OTT Channels Instead of 24/7 Live Encoding:**
   * Build 24/7 linear streaming channels from existing VOD assets using **MediaTailor Channel Assembly ($0.10/hr)** instead of running continuous live encoding via MediaLive ($1.176/hr).
   * **The Savings:** Slashes linear OTT channel costs by **91% ($73.00/mo vs $858.48/mo)**.
