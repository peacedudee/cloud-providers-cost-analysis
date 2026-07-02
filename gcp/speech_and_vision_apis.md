# Pre-trained AI Services Cost Optimization & Research

This research file covers Google Cloud's pre-trained cognitive APIs: **Speech-to-Text (STT)**, **Text-to-Speech (TTS)**, **Vision AI**, **Natural Language API**, and **Translation API**. These serverless APIs operate on a pay-as-you-go model. Because they charge per unit processed (seconds of audio, millions of characters, or features per image), optimization relies on payload sizing, caching, and model selection.

---

## 1. Billing Models & Pricing

* **Speech-to-Text (STT):** Billed per second of audio processed. Premium models (e.g. Chirp, Telephony) carry higher rates than standard models.
* **Text-to-Speech (TTS):** Billed per million characters. WaveNet and Neural2 (high-fidelity voices) cost up to 4x more than Standard voices.
* **Vision AI:** Billed per image feature requested (e.g., Label Detection, Object Detection, Text Detection are billed separately; if you request all 3 on a single image, you are billed for 3 features).
* **Natural Language API:** Billed per 1,000 characters analyzed (units of 1 "document").
* **Translation API:** Billed per million characters translated.

---

## 2. Core Cost-Optimization Levers

### A. Speech-to-Text: Trim Silence and Compress Audio
* **The Waste:** Sending a 2-hour audio recording containing 45 minutes of silence or music to the STT API. You are billed for the full 7,200 seconds of audio.
* **Action:**
  1. Use a local open-source tool (like FFmpeg or a Python script) to perform **Silence Detection** and strip silence segments before sending the audio file to GCP.
  2. Compress audio using a lightweight codec (e.g. FLAC or AMR) to reduce upload data egress.
  3. Use standard models rather than premium models (like Chirp) for clear, single-speaker recordings.

### B. Text-to-Speech: Cache Static Audio Generation
* **The Cost Trap:** An interactive voice system (IVR) that plays a welcome message (e.g. "Thank you for calling. Please listen to the options.") generating the speech dynamically via the TTS API on every user call.
* **Action:** Generate static prompts once, save the resulting `.mp3` or `.wav` file to a **Cloud Storage bucket**, and stream it from GCS. Only call the TTS API dynamically for variable data (e.g., reciting a user's account balance).
* **The Savings:** Eliminates 90% of TTS API query volume for typical application bots.
* **Tactic:** Use Standard voices instead of WaveNet/Neural2 in development and testing.

### C. Vision AI: Request Only the Necessary Features
* **The Waste:** Using the default configuration which requests all features (Landmark, Logo, Safe Search, Object, Text) on every image, multiplying the per-image charge.
* **Action:** Programmatically limit the `features` list in the API request JSON:
  ```json
  // GOOD (Only request text detection)
  {
    "requests": [
      {
        "image": { "source": { "imageUri": "gs://..." } },
        "features": [ { "type": "TEXT_DETECTION" } ]
      }
    ]
  }
  ```

### D. Translation & Natural Language: Truncate Text and Cache Results
* **Action (Translation):** Keep a local database table mapping common UI terms and translations (e.g., "Submit" = "Enviar"). Do not call the Translation API repeatedly for static website labels.
* **Action (Natural Language):** Strip HTML tags, markdown syntax, and redundant header/footer metadata before submitting text documents to the Natural Language API. You are billed for every character submitted, including whitespace and formatting characters.

---

## 3. Pre-trained AI Audit Checklist

1. [ ] **STT Silence Removal:** Verify that transcription pipelines filter out silence blocks prior to API ingestion.
2. [ ] **TTS Audio Caching:** Ensure static IVR or bot messages are cached in GCS as audio files rather than generated on-the-fly.
3. [ ] **Vision AI Feature Limits:** Check Vision API call templates and confirm that only required features (e.g. `TEXT_DETECTION` only) are requested.
4. [ ] **Translation Lookup Cache:** Implement a database lookup table for translated key-value pairs to prevent redundant API queries.
5. [ ] **Payload Sanitization:** Trim whitespace, HTML, and boilerplate text before sending documents to Translation and Natural Language APIs.
