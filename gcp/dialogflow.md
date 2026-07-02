# Dialogflow Cost Optimization & Research

Google Cloud Dialogflow is a natural language understanding platform that design-builds conversational user interfaces (chatbots, voicebots) for websites, mobile apps, and telephony systems. Dialogflow is available in two main editions: **Dialogflow ES** (Essentials) and **Dialogflow CX** (Customer Experience). Because Dialogflow CX shifts pricing to a **per-chat-session** model, session timeout handling and intent routing are major cost levers.

---

## 1. Dialogflow Billing Editions

1. **Dialogflow ES (Essentials):**
   * Billed per request (approx. $0.002 per text request, and $0.0065 per audio request).
2. **Dialogflow CX (Customer Experience - Enterprise):**
   * Billed per chat session (approx. $0.07 per session, which can include up to 40 turns of text).
   * Billed per voice session (approx. $0.20 per session, which includes speech recognition and synthesis).

---

## 2. Core Cost-Optimization Levers

### A. Active Session Closure (Dialogflow CX)
* **The Cost Trap:** Dialogflow CX sessions stay active for 30 minutes of idle time. If a user starts a chat, asks one question, and leaves the page, the session remains active. If they return 35 minutes later, it spawns a second session, charging you another $0.07.
* **Action:**
  1. Add an explicit "Exit" button in the chatbot UI. If clicked, send a final event to the backend to programmatically close the Dialogflow session immediately.
  2. For web clients, configure a shorter client-side inactivity timeout (e.g. 5 minutes). If the user is inactive, destroy the session ID.

### B. Pre-Filter FAQ Queries at the Application Layer
* **The Waste:** Sending 100% of user input to Dialogflow, including simple, repetitive questions (e.g., "What is your website?", "Is the office open today?").
* **Action:** Implement a lightweight local keyword filter or a simple dictionary lookup in your frontend gateway. If a user types "hours" or "address", intercept the query and return a static response immediately without calling Dialogflow.
* **The Benefit:** Saves on both ES request charges and CX session usage.

### C. Separate Telephony Voice Transcriptions
* **The Waste:** Sending raw audio files to Dialogflow CX voice sessions ($0.20/session).
* **Action:** If you use an external telephony provider (like Twilio or Genesys) that already performs Speech-to-Text (STT) locally or has a cheaper STT rate, translate the audio to text first. Send the text string to Dialogflow CX ($0.07/session), saving more than **60%** on the session fee.

---

## 3. Dialogflow Audit Checklist

1. [ ] **CX Session Pruning:** Confirm that chat applications trigger explicit session termination when users close chat windows.
2. [ ] **FAQ Interceptor:** Implement client-side or gateway filters for static, high-frequency questions to avoid API calls.
3. [ ] **Speech-to-Text Decoupling:** Audit voice bot architectures to determine if sending text streams is cheaper than raw audio.
4. [ ] **Test Project Isolation:** Ensure developers are using the free sandbox tier or isolated test agents during bot training and design phases.
