# AWS Service Cost Research: Amazon Rekognition

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Rekognition is a managed computer vision service that uses deep learning to analyze images and videos. It can identify objects, people, text, scenes, activities, and detect inappropriate content. Rekognition is divided into two primary models: **Standard APIs** (pre-trained general models) and **Custom Labels** (custom-trained models for specific objects or logos). Billing is consumption-based per image or video minute, with Custom Labels adding flat hourly hosting charges.

---

## 2. Billing Mechanics
Rekognition charges are billed monthly based on the volume and type of media analyzed:
1.  **Image Analysis (Standard APIs):** Billed per image processed, split into Group 1 and Group 2 APIs.
2.  **Video Analysis (Standard APIs):** Billed per minute of video processed.
3.  **Custom Labels Training:** Billed per hour of model training.
4.  **Custom Labels Inference (Hosting):** Billed per hour that a trained custom model is active and hosted.

---

## 3. Key Cost Dimensions

### A. Image Analysis Pricing (us-east-1 Tiers)
*   **Group 1 APIs (Object Labels, Moderation, Face Detection, Text in Image):**
    *   *First 1 Million images/month:* **$1.00 per 1,000 images** ($0.001 per image).
    *   *Next 9 Million images/month:* **$0.80 per 1,000 images**.
    *   *Beyond 10 Million images/month:* **$0.40 per 1,000 images**.
*   **Group 2 APIs (Face Search, Celebrity Recognition):**
    *   *First 1 Million images/month:* **$1.00 per 1,000 images**.
    *   *Next 9 Million images/month:* **$0.80 per 1,000 images**.

### B. Video Analysis Pricing
*   **Standard Video Detection:** Billed at **$0.10 per minute** of video processed (for labels, faces, moderation, text).
*   **Face Search / Celebrity Video Recognition:** Billed at **$0.12 per minute** of video processed.

### C. Custom Labels (Custom Models)
*   **Model Training:** Billed at a flat rate of **$4.00 per hour** of training time.
*   **Model Hosting (The Cost Trap):** Billed at **$4.00 per hour** per active Custom Label inference unit.
    *   *Warning:* Custom Labels do **not scale to 0** automatically. Once you start hosting the model, you pay the hourly rate continuously until you stop it.
    *   *Base Hosting Fee:*
        $$\$4.00 \times 730\text{ hours} = \$2,920.00\text{ / month per hosted model}$$

---

## 4. Detailed Pricing Rates (us-east-1 Standard)

| Rekognition Component | Billing Basis | Rate (us-east-1) | Price per 1,000 units / hours |
|-----------------------|---------------|------------------|-------------------------------|
| **Group 1 Image** | Per image (1st tier)| **$0.0010** | **$1.00** |
| **Group 2 Image** | Per image (1st tier)| **$0.0010** | **$1.00** |
| **Standard Video** | Per minute | **$0.1000** | $100.00 per 1,000 mins |
| **Custom Label Train**| Per hour | **$4.0000** | $4.00 / hour |
| **Custom Label Host** | Per hour | **$4.0000** | **$2,920.00 / month flat** |

---

## 5. AWS Free Tier Coverage
*   **Standard APIs:** 5,000 free images analyzed per month (first 12 months).
*   **Custom Labels:** 10 free training hours and 4 free hosting hours per month (first 12 months).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Zombie Custom Label Hosting Units:** Leaving custom-trained models (e.g. brand logo recognizers) hosted continuously in dev or staging environments when no tests are running, leaking **$2,920.00/month** per model.
*   **Processing High-Frame-Rate Video Frame-by-Frame via Image API:** Extracting every frame of a 30fps video and sending it to the Image Detection API instead of using the Video API.
    *   *Image Math (1 hr video):* 108,000 frames × $0.001 = **$108.00**.
    *   *Video Math (1 hr video):* 60 minutes × $0.10 = **$6.00** (Save 94%!).
*   **Analyzing Large, Raw Images Repeatedly:** Sending uncompressed high-resolution images (e.g. 10 MB raw files) to Rekognition, increasing network ingress transfer and processing times.

---

## 7. Actionable Cost Optimization Strategies
1.  **Stop Custom Label Model Hosting When Idle:**
    *   Do not leave Custom Labels models hosted 24/7 in non-production.
    *   Write an AWS Lambda script triggered by EventBridge to start the model (`StartProjectVersion`) at 8 AM and stop the model (`StopProjectVersion`) at 6 PM.
    *   **The Savings:** Bypassing overnight and weekend hosting charges reduces fixed fees from $2,920.00/mo to **$880.00/mo** (a **70% direct savings**).
2.  **Filter Video Ingestion Before Processing:**
    *   If using Rekognition Video for security camera feeds, do not stream continuous 24/7 video.
    *   Use local edge motion detection to trigger video recording, and send only the active motion segments to Rekognition.
3.  **Compress and Resize Images Locally:**
    *   Before calling the Rekognition API, resize and compress input images (e.g., down to 1080p JPEG) on the client or in a Lambda task.
    *   This reduces network transfer costs and speeds up API response times.
4.  **Batch Image Query API Requests:** Instead of querying single images, package multi-image analysis tasks into batch execution queues (using SQS/Lambda) to stay within tier volume limits.
