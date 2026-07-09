# AWS Service Cost Research: Amazon Connect

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Connect is a fully managed, cloud-based omnichannel contact center service that enables businesses to set up customer service contact centers (supporting voice calls, web chat, SMS, and task management) in minutes. It includes interactive voice response (IVR) flow routing, agent workspace management, real-time metrics, and AI-powered conversational analytics (Contact Lens). Amazon Connect is serverless and bills on a pure pay-as-you-go model with two billing options: **Individual Feature (à la carte)** and **Unlimited AI (bundled)**.

---

## 2. Billing Mechanics
Amazon Connect offers two primary billing models for voice and chat channels:
1.  **Individual Feature (à la carte) Pricing:**
    *   **Base Voice Usage:** Billed per minute ($0.018 per minute).
    *   **Contact Lens AI (Optional):** Billed separately per minute ($0.015 per minute for speech analytics).
    *   **Amazon Lex / Amazon Q (Optional):** Charged separately per request or agent session.
2.  **Unlimited AI (bundled) Pricing:**
    *   **Bundled Voice Usage:** Flat **$0.045 per minute**.
    *   **Included Features:** Voice usage, Contact Lens (analytics, evaluations, screen recording), Amazon Q in Connect, forecasting, capacity planning, agent scheduling, and Amazon Lex (voice/chat bot) requests.
3.  **Other Charges (Apply to both models):**
    *   **Telephony Rates:** Outbound calling carrier rates and daily phone number rentals.
    *   **Omnichannel Messaging:** Chat ($0.004/msg), SMS (per-message carrier fees), and Email ($0.004/email).
    *   **Task Management:** $0.04 per task managed.

---

## 3. Key Cost Dimensions

### A. Voice Telephony Rates (us-east-1)
*   **Inbound Voice Minute:** **$0.018 per minute** (base voice rate under à la carte model).
*   **Outbound Voice Minute (US):** **$0.0048 per minute** (plus destination-specific carrier rates).
*   **US DID Phone Number:** **$0.03 per day** (~$0.90 per month).
*   **US Toll-Free Phone Number:** **$0.12 per day** (~$3.60 per month).

### B. Chat & AI Analytics Rates
*   **Web Chat Message:** **$0.004 per message** (à la carte).
*   **Contact Lens Conversational Analytics:** **$0.015 per minute** (à la carte; includes speech-to-text transcription, sentiment analysis, and redaction).

### C. Unlimited AI Bundle Rate
*   **Flat rate:** **$0.045 per minute** for all voice channels (includes voice, Contact Lens, Amazon Q, Lex, scheduling, and forecasting).

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Model | Service/Feature | Billed Unit | Rate (us-east-1) | Price for 10,000 Minutes / Messages |
|---------------|-----------------|-------------|------------------|-------------------------------------|
| **À la carte**| **Base Inbound Call** | Per minute | **$0.018** | **$180.00** |
| **À la carte**| **Base Outbound Call**| Per minute | **$0.0048** | **$48.00** |
| **À la carte**| **Contact Lens AI** | Per minute | **$0.015** | **$150.00** |
| **À la carte**| **Chat Messages** | Per message | **$0.004** | **$40.00** |
| **Bundled** | **Unlimited AI Voice**| Per minute | **$0.045** | **$450.00** |

### Example Monthly Cost Calculation
*Workload: A contact center processes 50,000 inbound call minutes (US) using Contact Lens AI on 100% of calls, and uses 10 claimed DID phone numbers ($0.03/day).*

1.  **Individual Feature (À la carte) Cost:**
    *   **Inbound Voice Cost:**
        $$50,000 \times \$0.018 = \$900.00$$
    *   **Contact Lens AI Cost:**
        $$50,000 \times \$0.015 = \$750.00$$
    *   **Phone Number Rental:**
        $$10\text{ numbers} \times 30\text{ days} \times \$0.03 = \$9.00$$
    *   *Total À la carte:* **$1,659.00/month**

2.  **Unlimited AI Bundle Cost:**
    *   **Inbound Voice (Bundled) Cost:**
        $$50,000 \times \$0.045 = \$2,250.00$$
    *   **Phone Number Rental:**
        $$10\text{ numbers} \times 30\text{ days} \times \$0.03 = \$9.00$$
    *   *Total Unlimited AI:* **$2,259.00/month**

---

## 5. AWS Free Tier Coverage
*   **Amazon Connect Free Tier:** Includes **90 minutes of voice usage**, 1 claimed DID phone number, 500 chat messages, and 100 tasks per month for 12 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Running Contact Lens on 100% of Calls Under À La Carte:** Enabling Contact Lens speech-to-text and sentiment analysis on all routine calls when only a subset needs audit evaluations.
    *   *Math:* 100,000 call minutes/month × $0.015/min = **$1,500.00/month in AI analytics fees!**
*   **Accumulating Unused Toll-Free Numbers:** Claiming toll-free numbers ($3.60/mo each + higher per-minute rates) in developer or testing accounts.
*   **Overpaying for Unused Q in Connect/Lex Features:** Subscribing to the Unlimited AI bundle ($0.045/min) when your agent workflows only require standard voice without Q or Lex assistance.

---

## 7. Actionable Cost Optimization Strategies
1.  **Select the Right Pricing Model (Unlimited AI vs. À la carte):**
    *   Choose the **Unlimited AI Bundle ($0.045/min)** if your contact center actively uses Amazon Q, Amazon Lex, agent scheduling, AND Contact Lens on 100% of calls.
    *   Choose the **Individual Feature (À la carte)** model if you do not use Q or Lex, or if you sample analytics.
2.  **Sample Contact Lens AI Analytics (10–20% Rate):**
    *   In the Amazon Connect Contact Flow editor, configure Contact Lens to analyze a **10% to 20% random sample** of call minutes rather than 100%.
    *   **The Savings:** Under the à la carte model, sampling 10% of 100,000 minutes lowers the Contact Lens portion of the bill from $1,500.00 to **$150.00/month** (a **90% savings**).
3.  **Release Unused DID and Toll-Free Numbers:**
    *   Audit claimed phone numbers monthly and release inactive numbers to stop daily rental charges ($0.03–$0.12/day).
4.  **Deflect Inbound Calls to Self-Service Lex Chatbots:**
    *   Integrate Amazon Lex chatbots into IVR flows to handle routine questions (account status, operating hours) automatically, reducing agent voice duration minutes ($0.018/min).
