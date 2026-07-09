# AWS Service Cost Research: Amazon Translate

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Translate is a neural machine translation service that delivers fast, high-quality, and affordable language translation. Neural machine translation is a form of language translation automation that uses deep learning models to deliver more natural and accurate translations than traditional statistical and rule-based algorithms. Translate is serverless and billed purely on a pay-as-you-go model based on the number of characters submitted for translation.

---

## 2. Billing Mechanics
Amazon Translate billing is calculated monthly based on the character volume processed:
1.  **UTF-8 Characters:** Billed per character processed, including spaces, numbers, and punctuation.
2.  **Standard vs. Custom:** Pricing varies between standard pre-trained translation and premium Active Custom Translation (which uses customer-provided parallel text datasets to tune outputs).
3.  **Custom Model Upkeep:** Billed for training compute and monthly storage of custom translation models.

---

## 3. Key Cost Dimensions

### A. Standard Translation (us-east-1 Rates)
*   **The Rate:** **$15.00 per 1 Million characters** ($0.000015 per character).
*   **Billing Specifics:** Whitespace characters (like space, tab, newlines) are billed at the same rate.

### B. Active Custom Translation
Active Custom Translation allows you to customize translations by uploading translation memories (Parallel Data) directly.
*   **Translation Rate:** **$60.00 per 1 Million characters** (a **4x premium** over standard).
*   **Model Training Compute:** Billed at **$0.15 per 1,000 characters** trained ($150.00 per Million characters trained).
*   **Model Storage:** Billed at a flat fee of **$10.00 per custom model per month** for storage and maintenance.

---

## 4. Detailed Pricing Rates (us-east-1)

| Translation Type | Translation Price (per Million Characters) | Training Price (per 1,000 characters) | Model Storage Fee (per Month) |
|------------------|--------------------------------------------|---------------------------------------|------------------------------|
| **Standard** | **$15.00** | N/A | Free ($0.00) |
| **Active Custom**| **$60.00** | **$0.15** | **$10.00** |

---

## 5. AWS Free Tier Coverage
*   **Standard Translation:** **2 Million characters** free per month for the first 12 months.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Translating Repeating Static Strings:** Submitting identical UI text (like "Sign In", "Submit", or footer links) to the Translate API on every client page load instead of caching them.
*   **Translating Markup Code alongside Text:** Passing raw HTML, XML, or JSON strings directly into the API. The API parses and bills for every angle bracket, tag name, and attribute character (e.g. `<div class="main-content">` is billed as 25 characters, even though 0 text is translated).
*   **Orphaned Custom Models:** Leaving historical custom translation models stored in the active state at **$10.00/month** each when they are no longer linked to active workflows.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enforce Client-Side or Local Caching (S3/DynamoDB):**
    *   Do not query the API for strings that have already been translated.
    *   Save translations in a database (like DynamoDB) mapped by the source string and target language code.
    *   Query this local cache before calling the Amazon Translate API.
    *   **The Savings:** Saves up to **95%** on translation costs for static websites and recurring portal text.
2.  **Strip Markup Tag Overhead Locally:**
    *   Before invoking the API, extract the plain text strings from your HTML/XML templates.
    *   Send only the plain text to the API, and reconstruct the HTML markup locally using the returned translated strings.
    *   This prevents paying the **$15.00/Million** character tax on code markup.
3.  **Evaluate Custom Terminology vs. Active Custom Translation:**
    *   Instead of deploying a full Custom Model ($60/M characters + $10/mo storage), use the **Custom Terminology** feature.
    *   Custom Terminology allows you to upload a CSV dictionary of brand/domain names to override standard translations (e.g. ensuring "Amazon Web Services" is not translated literally).
    *   **The Savings:** Custom Terminology is **free ($0.00)** and uses standard **$15/M** billing.
4.  **Prune Inactive Custom Models:** Review the custom model catalog in the Translate console. Delete any custom models that have not been called in the last 60 days to stop the recurring $10.00/month storage charge.
