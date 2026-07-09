# AWS Service Cost Research: Amazon Transcribe Medical

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Transcribe Medical is an automatic speech recognition (ASR) service that makes it easy for developers to add medical speech-to-text capabilities to clinical applications. It is HIPAA-eligible and trained to accurately transcribe clinical audio, such as doctor-patient consultations, physician dictation, and medical education. It is serverless and billed purely on a pay-as-you-go model based on the duration of speech processed.

---

## 2. Billing Mechanics
Amazon Transcribe Medical bills monthly based on the duration of audio transcribed:
1.  **Audio Duration:** Billed per second of audio transcribed, calculated at one-second resolution.
2.  **Minimum Request Charge:** Each transcription request has a **15-second minimum** charge.
3.  **Unified Streaming/Batch Pricing:** Unlike some other services, both real-time streaming and asynchronous batch medical transcriptions use the same minute rate.

---

## 3. Key Cost Dimensions

### A. Transcription Rate (us-east-1)
*   **The Flat Rate:** **$0.075 per minute** ($0.00125 per second).
*   **Price per Hour:** **$4.50 per hour** of audio processed.
    *   *Comparison:* Standard Transcribe is $0.024/minute ($1.44/hour). Transcribe Medical is **3.1x more expensive** due to specialized medical vocabulary models and HIPAA compliance hosting.

### B. The 15-Second Minimum Request Rule
*   Each API execution (whether a batch file or a streaming connection) is metered for at least **15 seconds**.
*   *The Penalty:* If a clinician records a series of short 3-second dictations (e.g. "Patient has fever", "Check BP"), each request is billed as 15 seconds. Sending 100 of these 3-second clips results in paying for:
    $$100 \times 15\text{ seconds} = 1,500\text{ seconds (25 minutes)}$$
    instead of the actual 300 seconds (5 minutes) of recorded audio.

### C. Multi-Channel Transcription
*   If your audio files contain multiple channels (e.g., patient on Channel 1, clinician on Channel 2), Transcribe Medical can transcribe them separately.
*   *Billing impact:* AWS bills only for the overall duration of the audio file, not per channel. You are not charged double for multi-channel processing.

---

## 4. Detailed Pricing Rates (us-east-1)

| Transcribe Medical Mode | Rate per Second | Rate per Minute | Price per Hour | Minimum Billed Unit |
|-------------------------|-----------------|-----------------|----------------|----------------------|
| **Batch Transcription** | **$0.00125** | **$0.0750** | **$4.50** | 15 seconds |
| **Streaming Audio** | **$0.00125** | **$0.0750** | **$4.50** | 15 seconds |

---

## 5. AWS Free Tier Coverage
*   **Amazon Transcribe Medical:** **No free tier** is available. Any clinical audio processing generates standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Segmented Recording Ingestions:** Submitting small snippets of speech individually, triggering the **15-second minimum charge** repeatedly and inflating bills.
*   **Telemetry Streaming during Holds or Silence:** Leaving a real-time streaming dictation portal active when the clinician is away or silent, continuous billing at $4.50/hour.
*   **Transcribing Non-Clinical Conversations:** Using the high-cost Medical Transcribe ($4.50/hr) for general corporate meetings or customer support calls instead of the cheaper Standard Transcribe ($1.44/hr).

---

## 7. Actionable Cost Optimization Strategies
1.  **Stitch and Batch Clinician Dictations Locally:**
    *   Do not upload short dictations individually.
    *   Write a local buffer in your clinical app to record multiple notes, joining them into a single larger audio file (e.g., combining 10 separate patient entries into a single 3-minute file) before invoking the batch API.
    *   **The Savings:** Bypasses the 15-second minimum request surcharge, cutting transcription costs by up to **70–80%** for short dictation tasks.
2.  **Toggle Streaming Audio Buffers Dynamically:**
    *   For real-time streaming clinical apps, monitor the audio input line locally.
    *   If no voice activity is detected for 5 consecutive seconds, pause the streaming connection and restart it when the user speaks again to prevent billing for silence.
3.  **Strictly Audit API Selection:**
    *   Verify the source of audio.
    *   Use **Standard Transcribe ($1.44/hour)** for call center reception, appointments scheduling, and non-clinical meetings.
    *   Reserve **Transcribe Medical ($4.50/hour)** strictly for clinical diagnostics, patient histories, and medication dictation.
4.  **Use Single-Channel Files Where Possible:** If recording notes, configure local microphones to record in mono (single channel) to reduce file transfer payload sizes and optimize ingestion processing.
