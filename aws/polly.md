# AWS Service Cost Research: Amazon Polly

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Polly is a cloud service that converts text into lifelike speech, allowing you to create applications that talk and build entirely new categories of speech-enabled products. Polly uses advanced deep learning technologies to synthesize natural-sounding human speech. Polly is billed on a serverless, pay-as-you-go model based on the volume of characters converted to speech, with pricing variations based on the chosen voice tier (Standard, Neural, Generative, or Long-Form).

---

## 2. Billing Mechanics
Polly billing is calculated monthly based on a single primary metric:
1.  **Characters Processed:** Billed per character converted into audio files.
2.  **SSML Character Tax:** Speech Synthesis Markup Language (SSML) tags used to control speech inflection (e.g. `<break time="1s"/>`) are **counted and billed as standard characters**.

---

## 3. Key Cost Dimensions

### A. Voice Quality Tiers & Character Pricing (us-east-1)
*   **Standard Voices (Traditional synthesized speech):**
    *   *Rate:* **$4.00 per 1 Million characters** ($0.000004 per character).
    *   *Ideal for:* Basic notifications or systems where natural human-like cadence is not critical.
*   **Neural Voices (Highly realistic, human-like cadence):**
    *   *Rate:* **$16.00 per 1 Million characters** ($0.000016 per character).
    *   *Ideal for:* Virtual assistants, audiobooks, and interactive voice response (IVR) systems.
*   **Generative Voices (Highest quality, state-of-the-art expressive speech):**
    *   *Rate:* **$30.00 per 1 Million characters**.
    *   *Ideal for:* Highly engaging conversational interfaces, podcasting, and media narration.
*   **Long-Form Voices (Optimized for long articles and books):**
    *   *Rate:* **$100.00 per 1 Million characters**.

### B. SSML Billing Rule
*   When using SSML tags to add pauses, emphasis, or pronunciation guides:
    *   AWS counts every letter, bracket, and symbol in the SSML tag as a processed character.
    *   *Example:* Processing `<say-as interpret-as="cardinal">12345</say-as>` is billed as **43 characters**, even though only 5 digits ("12345") are spoken.

---

## 4. Detailed Pricing Rates (us-east-1)

| Voice Tier | Price per 1 Million Characters | Price per 10,000 Characters | Standard Book (100K Words / 500K Chars)|
|------------|--------------------------------|------------------------------|---|
| **Standard** | **$4.00** | $0.040 | $2.00 |
| **Neural** | **$16.00** | $0.160 | $8.00 |
| **Generative** | **$30.00** | $0.300 | $15.00 |
| **Long-Form** | **$100.00**| $1.000 | $50.00 |

---

## 5. AWS Free Tier Coverage
Polly features a **12-month free tier** for new accounts:
*   **Standard Voices:** **5 Million characters** per month.
*   **Neural Voices:** **1 Million characters** per month.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Synthesizing the Same Text Repeatedly:** Calling the Polly API on every user request for static text (e.g. generating "Welcome to our portal" every time a user logs in).
*   **Bloated SSML Markups:** Writing long, verbose SSML tags with excessive spacing, which inflates character counts without altering speech outcomes.
*   **Selecting Generative/Long-Form Tiers by Default:** Using $100/Million Long-Form voices for simple, short IVR menu options (e.g. "Press 1 for Sales") which would sound perfectly adequate on Standard or Neural tiers.

---

## 7. Actionable Cost Optimization Strategies
1.  **Cache synthesized Audio Files (S3 Caching):**
    *   Never call Polly for static text.
    *   When text is synthesized, save the output audio file (e.g., MP3) to **Amazon S3**.
    *   Have your application check S3 first. Serve the cached audio file for identical text requests, bypassing Polly billing entirely.
    *   **The Savings:** Drops API execution fees to **$0** for recurring static scripts.
2.  **Optimize SSML Tags (Remove Whitespace):**
    *   Strip out unnecessary spaces and shorten tag properties in your SSML templates before sending them to the API.
    *   Use custom lexicons instead of verbose `<phoneme>` tags for repeating terms.
3.  **Evaluate Quality Tier Requirements:** Use cheap **Standard ($4/M)** or **Neural ($16/M)** voices for short transactional alerts and notifications. Reserve **Generative ($30/M)** voices strictly for customer-facing marketing content.
4.  **Use Polly Batch Synthesis (`SynthesizeSpeech` vs `StartSpeechSynthesisTask`):** For large text blocks (e.g., articles), use asynchronous batch tasks which write output directly to S3. This handles large-scale operations reliably without request timeouts.
