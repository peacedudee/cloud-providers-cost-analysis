# AWS Service Cost Research: Amazon Connect

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Connect is a fully managed, cloud-based omnichannel contact center service that enables businesses to set up customer service contact centers (supporting voice calls, web chat, SMS, and task management) in minutes. Amazon Connect includes interactive voice response (IVR) flow routing, agent workspace management, real-time metrics, and AI-powered conversational analytics (Contact Lens). Amazon Connect is 100% pay-as-you-go, billing per call minute, phone number daily rental, chat message, and AI analytics minute.

---

## 2. Billing Mechanics
Amazon Connect billing consists of four primary components:
1.  **Voice Usage (Inbound / Outbound Minutes):**
    *   *Inbound Voice Usage:* Billed per minute ($0.018 per minute).
    *   *Outbound Voice Usage (US):* Billed per minute ($0.0048 per minute + carrier rates).
    *   *Phone Number Rental:* Billed per day per claimed number ($0.03/day = ~$0.90/mo for US DID; $0.12/day = ~$3.60/mo for Toll-Free).
2.  **Chat Messaging:** Billed per message sent/received ($0.004 per message).
3.  **Task Management:** Billed per task created ($0.04 per task).
4.  **Contact Lens (AI Conversational Analytics):** Billed per speech analytics minute ($0.015 per minute).

---

## 3. Key Cost Dimensions

### A. Voice Telephony Rates (us-east-1)
*   **Inbound Voice Minute:** **$0.018 per minute**.
*   **Outbound Voice Minute (US):** **$0.0048 per minute**.
*   **US DID Phone Number:** **$0.03 per day** (~$0.90 per month).
*   **US Toll-Free Phone Number:** **$0.12 per day** (~$3.60 per month).

### B. Chat & AI Analytics Rates
*   **Web Chat Message:** **$0.004 per message**.
*   **Contact Lens Conversational Analytics:** **$0.015 per minute**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Feature / Protocol | Billed Unit | Rate (us-east-1) | Price for 10,000 Minutes / Messages |
|--------------------|-------------|------------------|-------------------------------------|
| **Inbound Call** | Per minute | **$0.018** | **$180.00** |
| **Outbound Call (US)**| Per minute | **$0.0048** | **$48.00** |
| **Contact Lens AI**| Per minute | **$0.015** | **$150.00** |
| **Chat Messages** | Per message | **$0.004** | **$40.00** |

---

## 5. AWS Free Tier Coverage
*   **Amazon Connect Free Tier:** Includes **90 minutes of voice usage per month**, 1 claimed DID phone number, 500 chat messages, and 100 tasks per month for 12 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Running Contact Lens AI Analytics on 100% of Inbound Calls:**
    *   Enabling Contact Lens speech-to-text and sentiment analysis on all routine customer service calls.
    *   *Math:* 100,000 call minutes/month × $0.015/min = **$1,500.00/month in AI analytics fees alone!**
*   **Accumulating Unused Toll-Free Numbers:** Claiming dozens of toll-free numbers ($3.60/mo each + higher per-minute rates) across staging accounts.

---

## 7. Actionable Cost Optimization Strategies
1.  **Sample Contact Lens AI Analytics (10–20% Rate):**
    *   In Amazon Connect contact flow settings, configure Contact Lens to analyze a **10% to 20% random sample** of call minutes rather than 100%.
    *   *Why:* Provides statistical quality assurance without paying for 100% of call transcriptions.
    *   **The Savings:** Slashes Contact Lens AI analytics billing ($0.015/min) by **80–90%**.
2.  **Release Unused DID and Toll-Free Numbers:** Audit claimed phone numbers monthly and release inactive numbers to stop daily rental charges ($0.03–$0.12/day).
3.  **Deflect Routine Inbound Calls to Self-Service Lex Chatbots:** Integrate Amazon Lex chatbots into IVR flows to handle routine questions (account status, operating hours) automatically, reducing agent voice duration minutes ($0.018/min).
