# AWS Service Cost Research: AWS Snowcone

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Snowcone is the smallest, most portable member of the AWS Snow Family of physical edge computing and data transfer devices. Weighing only 4.5 lbs and USB-port sized, it offers 8 TB of usable storage (or 14 TB in Snowcone SSD models) along with 2 vCPUs and 4 GB of memory. It is designed for migrating smaller datasets in rugged settings or running lightweight local compute. 

> [!IMPORTANT]
> **Deprecation Notice:** As of late 2025 / early 2026, AWS has restricted the ordering of new Snowcone (and Snowball Edge) devices to existing customers only. New customer migrations should evaluate online migration tools (like AWS DataSync) or contact AWS support for custom appliance ordering.

---

## 2. Billing Mechanics
Snowcone billing is structured similarly to Snowball Edge, but scaled down in cost:
1.  **Service Fee (One-Time per Job):** A flat fee per device order.
2.  **Onsite Daily Fees:** Every job includes **5 free onsite days**. A daily extension fee is charged if kept longer.
3.  **Data Transfer:** Data uploads to AWS are free. Exporting data out of AWS onto Snowcone is billed at S3 egress rates.
4.  **Shipping:** Standard shipping carrier rates apply round-trip.

---

## 3. Key Cost Dimensions

### A. One-Time Job Fees (us-east-1)
*   **AWS Snowcone (8 TB HDD / 2 vCPUs):** **$60.00 per job**.
*   **AWS Snowcone SSD (14 TB SSD / 2 vCPUs):** **$120.00 per job**.

### B. Daily Onsite Fees (Extension Charges)
*   **5 Free Onsite Days:** The one-time job fee includes 5 days of onsite usage (shipping transit days are excluded).
*   **Daily Extension Rates:** If you keep the device beyond the 5-day allowance, you are billed:
    *   *Snowcone (8 TB HDD):* **$6.00 per day**.
    *   *Snowcone SSD (14 TB SSD):* **$15.00 per day**.

### C. Data Transfer Rates
*   **Data Ingress (Upload to S3):** **Free** (no ingestion charges for writing data from returned device to S3).
*   **Data Egress (Download from S3):** Exporting data out of S3 to Snowcone is billed at **$0.03 per GB**.

### D. Shipping and Power Options
*   **Shipping:** Standard shipping carrier rates apply based on distance and speed.
*   **Power Supplies:** Snowcone does not ship with a power supply by default. You must purchase your own standard USB-C PD (Power Delivery) power adapter (minimum 45W) or a local battery pack, which adds to your local project capital expenditure.

---

## 4. Detailed Pricing Rates (us-east-1)

| Device Type | Usable Storage | Job Service Fee | Included Onsite Days | Daily Extension Fee | Egress (Data Out) | Ingress (Data In) |
|-------------|----------------|-----------------|----------------------|---------------------|-------------------|-------------------|
| **Snowcone (HDD)** | 8 TB HDD | $60.00 | 5 Days | **$6.00 / day** | $0.030 / GB | **Free** |
| **Snowcone SSD** | 14 TB SSD | $120.00 | 5 Days | **$15.00 / day** | $0.030 / GB | **Free** |

---

## 5. AWS Free Tier Coverage
*   **AWS Snowcone:** No free tier available. All orders generate active billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Project Delays Accumulating Daily Fees:** Forgetting to return the device on time, or letting it sit idle during on-premises network changes. While $6.00/day sounds small, keeping an HDD Snowcone for 90 days adds **$510.00** in extension fees.
*   **Missing Shipping E-Ink Label Updates:** The device uses an e-ink label that automatically updates with return shipping barcodes. If the label fails to update and the return carrier cannot scan it, return transit is delayed, potentially inflating onsite daily fees if not manually resolved with AWS support.
*   **Data Export Egress Fees:** Exporting a full 14 TB from S3 on a Snowcone SSD adds **$420.00** in S3 data egress fees ($0.03/GB).

---

## 7. Actionable Cost Optimization Strategies
1.  **Prep Network and Power Before Device Arrival:** Purchase a compatible 45W+ USB-C PD power adapter beforehand and prepare local network folders so data can be copied onto the Snowcone within the **5 free onsite days**.
2.  **Compare Against AWS DataSync First:** If you are migrating under 10 TB and have a stable internet connection, use **AWS DataSync**. DataSync at $0.0125/GB has no job fees, no shipping fees, and requires no physical device logistics.
3.  **Ship Back Immediately After Transfer:** Generate the return label in the AWS console, verify the e-ink screen displays the UPS return address, and drop it off at the carrier immediately after copy confirmation to stop the daily rental timer.
