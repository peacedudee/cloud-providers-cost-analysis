# AWS Service Cost Research: Amazon S3 Glacier

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon S3 Glacier is a secure, durable, and extremely low-cost storage service class family designed for data archiving and long-term backup. While the baseline storage cost is the cheapest in AWS (down to $0.00099/GB-month), S3 Glacier has highly complex access, retrieval, and early deletion fee structures that can easily lead to massive, unexpected bills if not carefully managed.

---

## 2. Billing Mechanics
Glacier can be consumed via two models, each with distinct pricing:
1.  **S3 Glacier Storage Classes (Recommended):** Integrated directly into standard S3 buckets. Managed via S3 lifecycle rules and API calls.
2.  **S3 Glacier Vaults (Direct Archive API):** Legacy direct API model. Involves creating vaults, archives, and jobs directly via Glacier client APIs.

Your total Glacier bill is driven by:
*   **Storage Volume:** Gigabytes stored per month.
*   **Retrieval Volume:** Gigabytes of data read out of the archive.
*   **Retrieval Requests:** The number of jobs or API calls executed to initiate retrievals.
*   **Early Deletion Fees:** Pro-rated charges if data is deleted before a class-specific minimum retention period.
*   **Transition Request Fees:** API charges for moving objects from standard S3 to Glacier.

---

## 3. Key Cost Dimensions

### A. Glacier Storage Classes (us-east-1)
*   **S3 Glacier Instant Retrieval:**
    *   *Storage:* **$0.004 per GB-month**.
    *   *Retrieval:* **$0.03 per GB** processed.
    *   *Access Speed:* Milliseconds (behaves like standard S3).
    *   *Minimum Retention:* **90 days**.
    *   *Minimum Object Size:* **128 KB**.
*   **S3 Glacier Flexible Retrieval (formerly S3 Glacier):**
    *   *Storage:* **$0.0036 per GB-month**.
    *   *Retrieval:* **$0.03 per GB** processed (standard/expedited). Bulk retrievals are **free**.
    *   *Access Speed:* 1–5 minutes (Expedited), 3–5 hours (Standard), 5–12 hours (Bulk).
    *   *Minimum Retention:* **90 days**.
    *   *Minimum Object Size:* **40 KB**.
*   **S3 Glacier Deep Archive:**
    *   *Storage:* **$0.00099 per GB-month** (~$1.01/TB).
    *   *Retrieval:* **$0.09 per GB** processed (Standard). Bulk retrievals are **$0.0025 per GB**.
    *   *Access Speed:* 12 hours (Standard), 48 hours (Bulk).
    *   *Minimum Retention:* **180 days**.
    *   *Minimum Object Size:* **40 KB**.

### B. Retrieval Options & Speeds (Glacier Flexible/Deep Archive)
*   **Expedited Retrieval (Flexible only):** Billed at a high premium (**$0.03 per GB** + **$0.01 per request**). Used for urgent restorations. You can purchase **Provisioned Capacity** for **$100.00/month** to guarantee expedited retrieval throughput during traffic spikes.
*   **Standard Retrieval:** Billed at standard rates (**$0.03/GB** for Flexible; **$0.09/GB** for Deep Archive).
*   **Bulk Retrieval:** The cheapest option. **Free** for Glacier Flexible, and only **$0.0025/GB** for Glacier Deep Archive.

### C. Transition API Request Charges
Moving objects to Glacier incurs write charges (Class A S3 requests):
*   Standard to Glacier Flexible transition: **$0.03 per 1,000 requests**
*   Standard to Glacier Deep Archive transition: **$0.05 per 1,000 requests**
*   *Note: If you transition 1 million files individually, you pay $50.00 in transition fees, regardless of file size.*

### D. Early Deletion Penalties
If an object is deleted or transitioned to a colder tier before the minimum storage duration is met, you are charged for the remaining days:
*   Glacier Instant / Flexible Retrieval: **90 days**
*   Glacier Deep Archive: **180 days**
*   *Example:* If you upload 1 TB to Deep Archive and delete it after 10 days, you will be billed for the remaining 170 days of storage (170/180 × $0.99 = $0.93 early deletion fee).

---

## 4. Detailed Pricing Rates (us-east-1)

| Storage Tier | Storage Rate (/GB-mo) | Standard Retrieval (/GB) | Bulk Retrieval (/GB) | Transition Fee (/1K) | Min Duration |
|--------------|-----------------------|--------------------------|----------------------|----------------------|--------------|
| **Instant** | $0.0040 | $0.0300 | N/A | $0.0200 | 90 Days |
| **Flexible** | $0.0036 | $0.0300 | **Free** | $0.0300 | 90 Days |
| **Deep Archive**| $0.00099 | $0.0900 | $0.0025 | $0.0500 | 180 Days |

---

## 5. AWS Free Tier Coverage
*   **Retrieval:** Glacier Flexible direct API allows **10 GB/month** of free Standard retrievals.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Glacier Vault Lock Inescapable Billing:** The Glacier Vault Lock allows you to enforce compliance policies (WORM - Write Once Read Many) on your archive. If a policy is locked and prevents deletion, **you cannot delete the vault or the data** until the compliance period expires (which can be years). This results in completely unavoidable storage fees.
*   **Transitioning Tiny Objects (<128 KB/40 KB):** S3 Glacier adds **32 KB of billing overhead** (for metadata index) and **8 KB of S3 metadata** per object. Transitioning a 1 KB file to Glacier Deep Archive results in being billed for **40 KB** of Deep Archive storage plus **8 KB** of S3 Standard storage. This completely destroys any cost benefit.
*   **Expedited Retrieval Spikes:** Initiating large restorations using the Expedited tier without provisioned capacity or executing thousands of expedited requests.
*   **Frequent Data Deletions (Backup Rotations):** Storing temporary backups in Glacier Deep Archive and rotating/deleting them every 30 days. You pay early deletion penalties for 150 days of unused storage on every file deleted.

---

## 7. Actionable Cost Optimization Strategies
1.  **Always Use Bulk Retrievals:** When restoring data from Glacier Flexible or Deep Archive, configure your restore jobs to use **Bulk** mode. It is completely **free** for Glacier Flexible and represents a **97% discount** for Deep Archive compared to Standard retrievals.
2.  **Consolidate (Tar/Zip) Small Files Before Upload:** Never transition individual small logs or files to Glacier. Bundle them into larger archives (minimum 100 MB, ideally 1 GB+) before transitioning. This:
    *   Bypasses the 32 KB + 8 KB metadata billing overhead per object.
    *   Reduces transition API request charges (Class A) by 99%+.
    *   Makes future restores much faster and cheaper.
3.  **Align Backup Retention with Min Storage Terms:** Set your backup retention policies in AWS Backup or S3 Lifecycle to match Glacier minimum storage rules:
    *   Keep Glacier Flexible files for at least **90 days**.
    *   Keep Glacier Deep Archive files for at least **180 days**.
4.  **Audit Vault Lock Policies Before Enforcement:** Carefully test Vault Lock policies in staging or test environments. Never apply a strict compliance lock to a production vault without validating that the retention periods are exact.
5.  **Use S3 Storage Lens:** Monitor the "Active/Glacier Metadata Storage Ratio" metric in Storage Lens to find buckets where metadata overhead is dominating Glacier storage costs due to small file transitions.
