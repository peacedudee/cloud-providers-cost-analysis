# AWS Service Cost Research: Amazon WorkSpaces Core

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon WorkSpaces Core is an API-driven virtual desktop infrastructure (VDI) service designed for enterprise customers who want to use third-party VDI management solutions (such as Citrix Virtual Apps and Desktops, VMware Horizon, or Workspot) to manage cloud desktop instances running on AWS. WorkSpaces Core provides the underlying cloud compute, storage, and networking infrastructure while allowing enterprises to retain their existing VDI software licenses and management control planes.

---

## 2. Billing Mechanics
1.  **Fixed Monthly Core Rate (AlwaysOn):** Billed flat monthly per virtual desktop instance ($10.00 to $22.00 per instance-month).
2.  **Hourly Core Rate (AutoStop):** Billed a lower base monthly fee ($3.25 to $7.00 per month) plus an hourly instance rate ($0.03 to $0.15 per hour).
3.  **VDI Partner License Fees:** Customer pays third-party VDI management software licenses (Citrix / VMware) directly to the vendor.

---

## 3. Key Cost Dimensions

| WorkSpaces Core Instance Type | AlwaysOn Flat Rate | AutoStop Base + Hourly Rate | Target Workload |
|-------------------------------|--------------------|-----------------------------|-----------------|
| **`Core Standard` (2 vCPU, 4 GB)** | **$11.00 / month** | **$3.25/mo + $0.05/hr** | Task Workers |
| **`Core Performance` (2 vCPU, 8 GB)**| **$16.00 / month** | **$4.75/mo + $0.08/hr** | Knowledge Workers |
| **`Core Power` (4 vCPU, 16 GB)** | **$22.00 / month** | **$7.00/mo + $0.15/hr** | Power Users |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **`Core Standard` Rate:** $11.00 per instance-month (AlwaysOn) or $3.25/mo + $0.05/hr (AutoStop).
*   **Storage Rate:** $0.08 per GB-month.

---

## 5. AWS Free Tier Coverage
*   **Amazon WorkSpaces Core:** No standalone free tier.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Double-Paying for OS Licenses:** Licensing Windows Server via AWS when Microsoft Enterprise Agreements already include Azure Hybrid / Bring Your Own License (BYOL) rights for cloud VDI.

---

## 7. Actionable Cost Optimization Strategies
1.  **Leverage Existing Citrix / VMware Enterprise Agreements:**
    *   Deploy WorkSpaces Core instead of full Amazon WorkSpaces if your enterprise has existing long-term Citrix or VMware Horizon licensing contracts.
    *   *Why:* WorkSpaces Core strips out the AWS management console layer ($11.00/mo vs $33.00/mo), allowing you to reuse existing software licenses.
    *   **The Savings:** Slashes infrastructure charges per virtual desktop by **66%**.
2.  **Apply AutoStop Hourly Rates for Part-Time Desktop Pools:** Configure AutoStop mode ($3.25/mo base + $0.05/hr) for non-persistent VDI pools that operate during business hours only.
