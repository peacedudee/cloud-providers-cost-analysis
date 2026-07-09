# AWS Service Cost Research: Amazon Lex

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Lex is a fully managed artificial intelligence (AI) service with advanced natural language models, designed to build conversational interfaces (chatbots) into applications using voice and text. Lex uses the same conversational engine that powers Amazon Alexa. Lex operates on a pay-as-you-go billing model based on user interactions (requests or streaming intervals), with additional charges if using its automated chatbot structure design tools.

---

## 2. Billing Mechanics
Amazon Lex charges are billed monthly based on the conversational model and media type selected:
1. **Request/Response Model (Standard):** Billed per individual user text input or speech request processed.
2. **Streaming Conversation Model:** Billed per-request for text or per 15-second block for continuous audio streaming.
3. **Automated Chatbot Designer:** Billed per minute of training time when analyzing transcription logs to build chatbot paths.

---

## 3. Key Cost Dimensions

### A. Request/Response Model (us-east-1 Rates)
Best for standard transactional chat interfaces (such as web chat panels or SMS bots).
* **Text Requests:** **$0.00075 per request** ($0.75 per 1,000 requests).
* **Speech Requests:** **$0.00400 per request** ($4.00 per 1,000 requests).

### B. Streaming Conversation Model
Used for continuous, real-time voice conversations (such as interactive telephony bots or live customer call centers).
* **Text Requests:** **$0.00200 per request** ($2.00 per 1,000 requests).
* **Speech Intervals:** **$0.00650 per 15-second interval** (equivalent to **$0.026 per minute**).
  * *Interval Metering:* Billed per 15-second block (rounded up). If a user speaks for 5 seconds, you are billed for 15 seconds.

### C. Automated Chatbot Designer
Analyzes call transcripts or chat logs to automatically identify intents and entities, mapping out chatbot flows.
* **The Rate:** **$0.50 per minute** of active log analysis and design training ($30.00/hour).

---

## 4. Detailed Pricing Rates (us-east-1)

| Interaction Model | Media Type | Rate (us-east-1) | Price per 1,000 / Minute Equivalent |
|-------------------|------------|-------------------|-------------------|
| **Request/Response**| Text | **$0.00075 / request** | $0.75 per 1,000 |
| **Request/Response**| Speech | **$0.00400 / request** | $4.00 per 1,000 |
| **Streaming** | Text | **$0.00200 / request** | $2.00 per 1,000 |
| **Streaming** | Speech | **$0.00650 / 15-seconds**| **$0.0260 / minute** |
| **Chatbot Designer**| Transcripts| **$0.5000 / minute** | $30.00 / hour of design run |

---

## 5. AWS Free Tier Coverage
* **AWS Free Tier:** Includes **10,000 text requests** and **5,000 speech requests** per month for the first 12 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
* **Telemetry/Health Check Polling:** Configuring client web chat applications to send health check requests or user metadata syncs through Lex intents, accumulating standard text request fees ($0.75/1,000).
* **Telemetry Audio Streaming Silences:** Keeping streaming speech connections open during long periods of silence or call hold times. Telephony audio streams are billed continuously at $0.026/minute.
* **Runaway Chatbot Designer Runs:** Running the Automated Chatbot Designer on huge collections of raw, uncleaned text logs without setting strict training parameters.

---

## 7. Actionable Cost Optimization Strategies
1. **Filter User Text on the Client Side:**
   * Do not send empty messages, single character keystrokes, or basic system telemetry checks (like "is active?") to the Lex API.
   * Validate user inputs locally in client JavaScript before making Lex API calls to prevent invalid or garbage request fees.
2. **Toggle Telephony Streams Dynamically:**
   * When routing voice calls in Amazon Connect or other telephony services, do not stream the entire call audio to Lex.
   * Enable audio streaming only during active interaction prompts, and pause streaming when transferring users or putting them on hold to stop the $0.026/minute interval billing.
3. **Use Text Requests Over Speech (Where Possible):**
   * Voice analysis ($4.00/1,000) is **5.3x more expensive** than text analysis ($0.75/1,000).
   * For mobile apps, perform speech-to-text conversion on the client device (using native iOS/Android speech engines) and send the resolved text string to Lex. This bypasses Lex speech charges.
4. **Prune and Group Intents:** Consolidate redundant intents. A smaller, well-defined intent model reduces matching iterations, improving response latency and accuracy while preventing over-complex routing.
