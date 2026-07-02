# AWS Service Cost Research: AWS Payment Cryptography

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Payment Cryptography simplifies the implementation of cryptographic operations in payment processing applications in accordance with PCI PIN, PCI P2PE, and PCI DSS standards. It replaces traditional on-premises payment Hardware Security Modules (Payment HSMs) by providing cloud-native APIs for PIN generation, PIN verification, credit card validation (CVV/CVC), point-of-sale (POS) payload decryption, and payment key management. It operates on a pay-as-you-go key storage and API request model.

---

## 2. Billing Mechanics
AWS Payment Cryptography billing is split into two primary dimensions:
1.  **Active Key Monthly Storage:** Billed hourly per active payment key, with volume-tiered pricing.
2.  **API Requests (Cryptographic Operations):** Billed per 10,000 data plane API operations (e.g. `VerifyPinData`, `DecryptData`, `GenerateCardValidationData`).

---

## 3. Key Cost Dimensions

### A. Active Key Storage Pricing (us-east-1 Volume Tiers)
Active keys are prorated hourly based on monthly tier boundaries:
*   **First 500 active keys / month:** **$1.00 per key-month** (prorated hourly at ~$0.00137/hour).
*   **Next 500 active keys (501 to 1,000) / month:** **$0.10 per key-month**.
*   **Next 50,000 active keys (1,001 to 51,000) / month:** **$0.01 per key-month**.
*   **Over 51,000 active keys / month:** **$0.001 per key-month**.

### B. API Request Pricing (Data Plane Cryptography Tiers)
*   **First 20 Million requests / month:** **$2.00 per 10,000 requests** ($0.20 per 1,000 requests).
*   **Next 200 Million requests / month:** **$0.75 per 10,000 requests**.
*   **Next 300 Million requests / month:** **$0.25 per 10,000 requests**.
*   **Over 520 Million requests / month:** **$0.10 per 10,000 requests**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Dimension | Usage Tier | Rate (us-east-1) | Price for 1M Requests / 100 Keys |
|-------------------|------------|------------------|----------------------------------|
| **Active Key Storage**| 0 - 500 keys | **$1.00 / key-month** | **$100.00 / month** |
| **Active Key Storage**| 501 - 1,000 keys| **$0.10 / key-month** | $10.00 / month |
| **API Requests** | 0 - 20 Million | **$2.00 / 10,000 calls** | **$200.00 per 1M calls** |
| **API Requests** | 20M - 220 Million | **$0.75 / 10,000 calls** | $75.00 per 1M calls |

---

## 5. AWS Free Tier Coverage
*   **AWS Payment Cryptography:** **No free tier** is available. Any key creation or API invocation generates standard usage billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Orphaned Test Payment Keys:** Retaining hundreds of temporary test payment keys ($1.00/mo) in development accounts after POS integration testing concludes.
*   **High-Frequency Uncached Validation Polling:** Calling `VerifyCardValidationData` repeatedly on redundant authorization retry loops.

---

## 7. Actionable Cost Optimization Strategies
1.  **Delete Stale Test Payment Keys:**
    *   Audit active payment key inventories using `ListKeys`.
    *   Delete unused test keys or keys associated with expired merchant profiles.
    *   **The Savings:** Saves $1.00/month per key.
2.  **Batch & Consolidate Transaction API Calls:** Group payment validation tasks where possible within payment gateway logic to minimize standalone cryptographic API calls ($2.00/10K calls).
3.  **Utilize Tiered Key Sizing for Large Card Issuers:** For high-volume card issuers managing over 50,000 active keys, key storage fees automatically drop to $0.01 to $0.001/key-month.
