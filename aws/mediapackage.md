# AWS Service Cost Research: AWS Elemental MediaPackage

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Elemental MediaPackage is a video packaging and origination service that prepares live and VOD video streams for secure Internet delivery. MediaPackage ingests single-bitrate or adaptive-bitrate video streams from encoders (like MediaLive), packages them on-the-fly into multiple streaming formats (HLS, DASH, CMAF, Smooth Streaming), and applies Digital Rights Management (DRM) encryption. MediaPackage also provides time-shifted TV features (start-over, catch-up TV, pause live). MediaPackage is billed per GB of data ingested and packaged/egressed.

---

## 2. Billing Mechanics
1.  **Video Ingest:** Billed per GB of video content received into MediaPackage from encoders ($0.05 per GB ingested).
2.  **Video Packaging & Origination (Egress):** Billed per GB of video content packaged and delivered to downstream destinations ($0.04 per GB packaged).
3.  **No Per-Channel Hourly Fee:** MediaPackage has no fixed hourly instance or channel fees; billing is 100% data volume driven.

---

## 3. Key Cost Dimensions

| MediaPackage Component | Billed Metric | Rate (us-east-1) | Price for 1 TB Data |
|------------------------|---------------|------------------|---------------------|
| **Video Ingest** | Per GB ingested | **$0.05 / GB** | **$50.00** |
| **Video Packaging & Egress**| Per GB packaged| **$0.04 / GB** | **$40.00** |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Ingest Rate:** $0.05 per GB.
*   **Packaging / Egress Rate:** $0.04 per GB.

---

## 5. AWS Free Tier Coverage
*   **AWS Elemental MediaPackage Free Tier:** Includes **50 GB of video ingest** and **50 GB of video packaging** per month for 2 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Bypassing CDN Caching (Direct-to-Viewer Packaging):**
    *   Routing viewer HTTP playback requests directly to MediaPackage endpoints without an edge CDN like CloudFront.
    *   *Math:* If 10,000 viewers watch a 2 GB live stream directly from MediaPackage:
        $$10,000 \text{ viewers} \times 2 \text{ GB} \times \$0.04 / \text{GB} = \mathbf{\$800.00 \text{ in MediaPackage packaging fees!}}$$

---

## 7. Actionable Cost Optimization Strategies
1.  **Always Front MediaPackage with Amazon CloudFront (CDN):**
    *   Route 100% of viewer playback requests through **Amazon CloudFront**.
    *   *How it works:* CloudFront edge locations cache video segment chunks. MediaPackage packages each video segment **only ONCE per origin request** ($0.04/GB) regardless of whether 1 viewer or 1,000,000 viewers watch the live stream.
    *   **The Savings:** Slashes MediaPackage packaging egress fees by **99.9%** ($0.08 origin cost vs $800.00 direct cost).
2.  **Set Short Time-Shifted Window Buffers:** Configure live window DVR buffers (start-over window) to 1–2 hours rather than 24 hours to reduce origin storage overhead.
