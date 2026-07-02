# AWS Service Cost Research: Amazon AppStream 2.0 (WorkSpaces Applications)

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon AppStream 2.0 (also branded as **Amazon WorkSpaces Applications**) is a fully managed application streaming service that streams desktop applications from AWS to a web browser on any user device (HTML5 browsers, Chromebooks, Macs, PCs) without rewriting software. AppStream 2.0 billing is composed of streaming instance hourly fees, stopped instance host fees for On-Demand fleets, and monthly Microsoft Remote Desktop Services user licensing fees.

---

## 2. Billing Mechanics
1. **Streaming Instance Hourly Fee:** Billed hourly per running fleet instance based on instance type (`stream.standard.small`, `stream.standard.medium`, `stream.graphics.g4dn.xlarge`) and fleet type (Always-On vs. On-Demand).
2. **Stopped / Stopped-Host Fees (On-Demand Fleets):** On-Demand instances incur a reduced stopped-instance fee ($0.025/hour) while waiting in a stopped state to be provisioned to a user.
3. **User Monthly Licensing Fee:** Billed per unique user who accesses an AppStream 2.0 Windows stack during a calendar month ($4.19 per user per month for Microsoft Windows Server Remote Desktop Services SAL). Waived for Linux fleets or when utilizing BYOL.

---

## 3. Key Cost Dimensions

### A. Instance Streaming Hourly Rates (us-east-1 On-Demand)
* **`stream.standard.small` (1 vCPU, 2 GB RAM):** **$0.10 per hour** (~$73.00 per month 24/7).
* **`stream.standard.medium` (2 vCPU, 4 GB RAM):** **$0.20 per hour** (~$146.00 per month 24/7).
* **`stream.standard.large` (4 vCPU, 8 GB RAM):** **$0.40 per hour** (~$292.00 per month 24/7).
* **`stream.graphics.g4dn.xlarge` (4 vCPU, 16 GB, GPU):** **$0.75 per hour** (~$547.50 per month 24/7).

### B. User Licensing Fee
* **Microsoft RDS SAL Fee:** **$4.19 per user per month** (billed once per unique user per calendar month regardless of usage frequency). Waived for Linux streaming instances or eligible Microsoft BYOL licensing.

---

## 4. Detailed Pricing Rates (us-east-1)

| Instance Class | CPU / Memory / GPU | Hourly Streaming Rate | Price for 100 Hours Streaming | Monthly User License Fee |
|----------------|--------------------|-----------------------|-------------------------------|--------------------------|
| **`stream.standard.small`** | 1 vCPU, 2 GB | **$0.10** | **$10.00** | **$4.19 / user-mo** |
| **`stream.standard.medium`** | 2 vCPU, 4 GB | **$0.20** | **$20.00** | **$4.19 / user-mo** |
| **`stream.graphics.g4dn`** | 4 vCPU, 16 GB, GPU | **$0.75** | **$75.00** | **$4.19 / user-mo** |

---

## 5. AWS Free Tier Coverage
* **Amazon AppStream 2.0 Free Tier:** Includes **40 free hours per month** of `stream.standard.light` or `stream.standard.small` instance streaming for 2 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
* **Running Always-On Fleets 24/7 with Over-Provisioned Baseline Capacity:** Keeping 20 `stream.standard.medium` instances running continuously during off-peak hours (nights/weekends).
  * *Math:* $20\text{ instances} \times 730\text{ hours} \times \$0.20 = \mathbf{\$2,920.00\text{ / month}}$ in idle compute charges.
* **Paying RDS SAL Fees for Non-Windows Workloads:** Running Windows AppStream 2.0 instances for open-source or web applications that can run natively on Linux, incurring unnecessary $4.19/user-month license charges.

---

## 7. Actionable Cost Optimization Strategies
1. **Configure Dynamic Auto-Scaling & Schedule-Based Capacity Rules:**
   * Implement Target Tracking Scaling policies based on `CapacityUtilization`.
   * Configure scheduled scaling to set **Minimum Capacity = 0** outside business hours.
   * **The Savings:** Cuts idle fleet streaming fees by **up to 65%**.
2. **Use On-Demand Fleets for Intermittent Usage Patterns:** Switch fleet type from Always-On ($0.20/hr continuous) to On-Demand ($0.20/hr active + $0.025/hr stopped) for training labs or intermittent shift workers.
3. **Deploy Linux Streaming Fleets:** Deploy AppStream 2.0 Linux fleets for Linux-compatible software to completely eliminate the **$4.19 per user per month** Microsoft RDS SAL fee.
