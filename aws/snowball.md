# AWS Service Cost Research: AWS Snowball Edge

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Snowball Edge is a physical data migration and edge computing device provided by the AWS Snow Family. It is designed to transport petabytes of data securely into and out of AWS (data migration) or to run serverless local compute workloads in disconnected or harsh edge environments (edge computing). Snowball Edge pricing is project-based, combining one-time job fees with daily rental extensions, shipping, and cloud data egress rates.

---

## 2. Billing Mechanics
Snowball Edge billing is calculated per job request based on the following dimensions:
1.  **Service Fee (One-Time per Job):** A flat fee based on the device type selected.
2.  **Onsite Daily Fees:** Every job includes **10 free onsite days**. A daily fee is charged for keeping the device onsite beyond this period.
3.  **Data Transfer:** Data ingestion into AWS is free. Data transfer *out* of AWS to the device incurs standard cloud egress charges.
4.  **Shipping:** Standard carrier shipping rates apply to deliver and return the device.
5.  **Lost or Damaged Device Fees:** High physical replacement penalties if the device is lost or structurally damaged.

---

## 3. Key Cost Dimensions

### A. Device Service Fees (One-Time per Job - us-east-1)
*   **Snowball Edge Storage Optimized (80 TB Storage / 40 vCPUs):** **$300.00 per job**. Optimized for high-capacity data transfers.
*   **Snowball Edge Storage Optimized (210 TB Storage / 104 vCPUs):** **$300.00 per job**.
*   **Snowball Edge Compute Optimized (104 vCPUs / 416 GB RAM):** **$300.00 per job**. Optimized for intensive local data processing.

### B. Daily Onsite Fees (Extension Charges)
*   **10 Free Onsite Days:** The one-time job fee includes 10 days of onsite usage. 
*   *Note: Shipping/transit days (the time the device is in transit with UPS/DHL) do not count toward your 10 free days. The clock starts the day after delivery and stops the day before it is scanned by the shipping carrier for return.*
*   **Daily Extension Rates:** If you retain the device beyond 10 days, you are billed a daily rate:
    *   *Storage Optimized (80 TB):* **$30.00 per day**.
    *   *Storage Optimized (210 TB):* **$50.00 per day**.
    *   *Compute Optimized:* **$40.00 per day**.
    *   *Compute Optimized (with GPU):* **$70.00 per day**.

### C. Data Transfer Rates
*   **Data Ingress (Upload to AWS):** **Free**. There are no data ingestion fees when uploading data from the returned Snowball device into S3.
*   **Data Egress (Download from AWS):** If you use Snowball to extract data *out* of AWS, you are charged standard AWS data transfer egress fees:
    *   *S3 Egress:* **$0.03 per GB** (a discounted egress rate compared to the standard $0.09/GB internet egress fee, but still significant for petabyte-scale exports).

### D. Shipping and Carrier Fees
*   You are charged standard carrier shipping fees to ship the device from the AWS warehouse to your on-premises location, and to ship it back. You cannot use your own shipping carrier; shipping must be arranged via AWS.

---

## 4. Detailed Pricing Rates (us-east-1)

| Device Type | One-Time Job Fee | Included Onsite Days | Daily Extension Fee | Egress (Data Out) | Ingress (Data In) |
|-------------|------------------|----------------------|---------------------|-------------------|-------------------|
| **Storage Optimized (80 TB)** | $300.00 | 10 Days | **$30.00 / day** | $0.030 / GB | **Free** |
| **Storage Optimized (210 TB)**| $300.00 | 10 Days | **$50.00 / day** | $0.030 / GB | **Free** |
| **Compute Optimized** | $300.00 | 10 Days | **$40.00 / day** | $0.030 / GB | **Free** |
| **Compute Optimized + GPU** | $300.00 | 10 Days | **$70.00 / day** | $0.030 / GB | **Free** |

---

## 5. AWS Free Tier Coverage
*   **Snowball Edge:** No free tier available. All jobs generate standard billing from the moment the order is processed.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Project Delays inflating Daily Fees:** Keeping the device onsite for months due to local network cabling issues, power supply delays, or host server misconfigurations. Keeping an 80 TB device for 60 days adds **$1,500.00** in daily extension fees ($30 × 50 extra days).
*   **Massive Egress Bills on Data Export:** Exporting 100 TB of files from S3 to a Snowball Edge. The egress fee of **$0.03/GB** will add **$3,000.00** to the job bill.
*   **Unnecessary Compute-Optimized Selection:** Choosing the Compute-Optimized device ($40/day extension) for standard data storage migrations that only require simple network copy.
*   **Lost or Damaged Device Charges:** Failing to pack the device in its ruggedized container or losing the return e-ink shipping label, resulting in physical replacement liabilities (up to $20,000+).

---

## 7. Actionable Cost Optimization Strategies
1.  **Prepare On-Premises Infrastructure Before Ordering:** Never order a Snowball Edge before your local setup is 100% ready. Ensure:
    *   Cabling (10G/25G SFP+ or RJ45) is routed and tested.
    *   Network shares are fully accessible.
    *   Local copy scripts (e.g. AWS CLI or `rsync`) are written and tested.
    Goal: Copy all data onto the device within the **10 free onsite days** and ship it back immediately.
2.  **Use AWS DataSync for Smaller Migrations:** If your data size is under 10–20 TB and you have a decent internet connection (e.g., 100 Mbps+), use **AWS DataSync** instead. DataSync charges $0.0125/GB and requires no physical device logistics, job fees, or shipping costs.
3.  **Opt for Storage-Optimized Devices for Simple Migrations:** If you only need to move files to S3, always order the **Storage Optimized** device. Avoid Compute-Optimized models unless you are executing active local EC2 instances or Lambda scripts at the edge.
4.  **Consolidate Data Transfers into Single Jobs:** Group migrations to fit within the 80 TB or 210 TB boundaries. Running two 40 TB Snowball jobs costs $600 in job fees + double shipping; running a single 80 TB job costs $300 + single shipping.
5.  **Verify Egress Estimates Before Exporting:** If you must copy data out of AWS via Snowball, calculate the $0.03/GB S3 egress fee beforehand to ensure it is more cost-effective than standard internet download rates or AWS Direct Connect.
