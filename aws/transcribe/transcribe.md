# AWS Service Cost Research: Amazon Transcribe

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Transcribe is a fully managed, automatic speech recognition (ASR) service that makes it easy to add speech-to-text capabilities to your applications. It uses advanced machine learning models to convert audio and video files or real-time media streams into written text. Transcribe is serverless and billed purely on a pay-as-you-go model based on the duration of the audio processed.

---

## 2. Billing Mechanics
Amazon Transcribe bills monthly based on the duration of speech processed:
1.  **Audio Duration:** Billed per second of audio transcribed, calculated at one-second resolution.
2.  **Minimum Request Charge:** Each transcription request has a **15-second minimum** charge.
3.  **Tiers and Add-ons:** Pricing varies based on the domain (Standard vs. Medical), interaction model (Batch vs. Call Analytics), and additional features (redaction, summarization).

---

## 3. Key Cost Dimensions

### A. Core Transcription Rates (us-east-1)
*   **Standard Transcription (Batch or Real-time Streaming):**
    *   *Rate:* **$0.024 per minute** ($0.0004 per second).
    *   *Price per Hour:* **$1.44 per hour** of audio processed.
*   **Call Analytics (Includes Transcription + Sentiment + Category Tags + Call Summarization):**
    *   *Rate:* **$0.036 per minute** ($0.00060 per second).
    *   *Price per Hour:* **$2.16 per hour** of audio processed.
*   **Medical Transcription (For clinical speech):**
    *   *Rate:* **$0.078 per minute** ($0.00130 per second).
    *   *Price per Hour:* **$4.68 per hour** of audio processed.

### B. Standard Volume Discounts
Standard transcription has volume-based tier discounts for high-volume accounts in a calendar month:
*   *First 250,000 minutes:* **$0.0240 per minute** ($1.44/hour).
*   *Next 750,000 minutes:* **$0.0150 per minute** ($0.90/hour).
*   *Beyond 1 Million minutes:* **$0.0102 per minute** ($0.612/hour).

### C. Feature Add-on Surcharges
*   **PII Redaction:** Adding automatic redaction of personally identifiable information (credit cards, SSNs) adds **$0.0024 per minute** ($0.144/hour) to standard rates.
*   **Generative Call Summarization:** Adding LLM-generated summaries to Call Analytics adds **$0.0024 per minute** of processed audio.

---

## 4. Detailed Pricing Rates (us-east-1)

| Transcribe Tier | Feature Level | Rate per Second | Rate per Minute | Price per Hour |
|-----------------|---------------|-----------------|-----------------|----------------|
| **Standard** | Core Speech-to-Text | **$0.00040** | **$0.0240** | **$1.44** |
| **Standard + Redaction**| OCR/Speech + PII filter| **$0.00044** | **$0.0264** | **$1.58** |
| **Call Analytics**| Conversation insight | **$0.00060** | **$0.0360** | **$2.16** |
| **Medical** | Medical Vocabulary | **$0.00130** | **$0.0780** | **$4.68** |

---

## 5. AWS Free Tier Coverage
*   **Standard Transcription:** **60 minutes** of free audio transcription per month for the first 12 months.

---

## 6. Common Cost Hotspots & Pitfalls
*   **The 15-Second Minimum Penalty for Short Audio Clips:** Processing thousands of tiny 2-second audio clips (e.g. voice commands). Since each is billed at the **15-second minimum**, you pay for 15,000 seconds of compute for only 2,000 seconds of actual audio (a **7.5x payload inflation bill**).
*   **Ingesting Silent Audio files:** Leaving call recording streams active when customers are put on hold or call lines are dead, paying $1.44/hour to transcribe silence.
*   **Enabling Call Analytics on Non-Analytics Sprints:** Using the $2.16/hour Call Analytics API for developer testing or basic archival transcription when the $1.44/hour Standard API would suffice.

---

## 7. Actionable Cost Optimization Strategies
1.  **Concatenate Short Audio Files Before Ingesting:**
    *   If you are transcribing short audio snippets (e.g. 2-second voice triggers), do not send them to the API individually.
    *   Use a lightweight tool (like FFMpeg in a Lambda function) to stitch multiple audio clips into a single larger file (e.g. 5 minutes) before invoking Amazon Transcribe.
    *   **The Savings:** Bypasses the 15-second minimum request surcharge, cutting small audio bills by up to **80%**.
2.  **Filter Out Silences at the Edge:**
    *   Use a Voice Activity Detection (VAD) library locally on your recording client or media gateway.
    *   Strip out silence and hold intervals before uploading the media file to S3 for transcription.
3.  **Audit Feature Surcharges:** Review your Transcribe templates. Disable PII Redaction or Generative Summarization in development, staging, or internal QA environments.
4.  **Use Custom Vocabularies over LLM correction:** Instead of running downstream LLM prompts to correct misspelled domain terms (like company or product names), upload a free **Custom Vocabulary** file directly to Amazon Transcribe. This improves accuracy at the source for $0.00.
