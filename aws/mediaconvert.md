# AWS Service Cost Research: AWS Elemental MediaConvert

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Elemental MediaConvert is a file-based video transcoding service that transforms video content (VOD archives, user-generated video uploads, movies) into adaptive bitrate (ABR) formats (HLS, DASH) or MP4 files for high-quality playback on smartphones, tablets, PCs, and smart TVs. MediaConvert handles complex video processing tasks including frame rate conversion, graphic overlays, multi-channel audio, captioning, and DRM encryption. MediaConvert is billed per minute of output video created.

---

## 2. Billing Mechanics
1.  **Output Video Duration:** Billed per minute of output video generated (in 1-second increments).
2.  **Service Tier:**
    *   *Basic Tier:* Simple single-file MP4 outputs ($0.0075 per minute for HD).
    *   *Professional Tier:* Advanced ABR packaging, multi-language audio, DRM, graphic overlays ($0.015 per minute for HD AVC).
3.  **Output Resolution & Codec:** Rates depend on output resolution (SD, HD, 4K/UHD) and codec (AVC/H.264, HEVC/H.265, AV1).
4.  **Volume Tier Discounts:** Automatically discounts rates as monthly "normalized minutes" increase.

---

## 3. Key Cost Dimensions

### A. Professional Tier Rates (us-east-1 AVC Codec)
*   **SD Output (up to 480p):** **$0.0075 per minute** ($0.45 per hour).
*   **HD Output (up to 1080p):** **$0.0150 per minute** ($0.90 per hour).
*   **UHD / 4K Output:** **$0.0300 per minute** ($1.80 per hour).

### B. Basic Tier Rates (us-east-1)
*   **HD Output (up to 1080p):** **$0.0075 per minute** (**50% cheaper** than Professional).

---

## 4. Detailed Pricing Rates (us-east-1)

| Feature / Output Format | Service Tier | Rate per Output Minute | Price for 1,000 Hours Transcoded |
|-------------------------|--------------|------------------------|----------------------------------|
| **HD Output (1080p)** | Basic Tier | **$0.0075** | **$450.00** |
| **HD Output (1080p)** | Professional | **$0.0150** | **$900.00** |
| **4K Output (UHD)** | Professional | **$0.0300** | **$1,800.00** |

---

## 5. AWS Free Tier Coverage
*   **AWS Elemental MediaConvert Free Tier:** Includes **20 free minutes per month** of Basic or Professional tier video processing indefinitely.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Using Professional Tier for Simple Single-File Conversions:** Selecting Professional Tier ($0.015/min) for simple MP4 transcode jobs that do not require multi-bitrate ABR packaging or DRM ($0.0075/min in Basic Tier).

---

## 7. Actionable Cost Optimization Strategies
1.  **Use Basic Tier for Single-File MP4 Outputs:**
    *   Select **Basic Tier ($0.0075/min)** for simple video transcoding jobs (e.g. user avatar uploads, preview clips, internal web MP4 files).
    *   **The Savings:** Cuts transcoding costs by **50%**.
2.  **Enable Automated ABR Stack Generation:**
    *   Enable MediaConvert **Automated ABR**.
    *   *Why:* Automated ABR analyzes video complexity dynamically and generates ONLY the renditions needed for optimal quality, preventing redundant high-bitrate outputs.
    *   **The Savings:** Reduces output file volume and total billed output minutes by **20–30%**.
3.  **Accelerated Transcoding for High-Priority VOD:** Enable Accelerated Transcoding for time-sensitive jobs (compresses processing time up to 25x with minimal cost impact).
