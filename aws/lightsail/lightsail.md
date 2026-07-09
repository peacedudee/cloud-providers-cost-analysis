# AWS Service Cost Research: Amazon Lightsail

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Lightsail is designed to be the easiest way to launch and manage virtual private servers, databases, and networking on AWS. It provides a simple, predictable pricing structure based on bundled flat-rate plans, making it ideal for small businesses, developers, and simple web applications (like WordPress blogs, static sites, or microservices).

---

## 2. Billing Mechanics
Lightsail uses a simple monthly subscription-bundle model:
* **Hourly Billing up to a Cap:** You are billed an hourly rate for your running resources up to the maximum monthly plan price. If an instance is active for only half a month, you pay only half the monthly bundle cost.
* **Predictable Packages:** Every virtual private server (instance) bundle includes a set allocation of:
  1. vCPU compute.
  2. RAM (memory).
  3. SSD storage (block storage).
  4. A generous data transfer (egress) allowance.
* **Overage Billing:** If you exceed your monthly data transfer allowance, you pay standard overage fees (typically **$0.09 per GB** in `us-east-1`).

---

## 3. Key Cost Dimensions

### A. Instance Bundles (Linux vs. Windows)
* **Linux/Unix:** Baseline bundle rates start at **$3.50/month** (Nano tier with public IPv4 included).
* **Windows:** Bundles incur licensing overhead and are roughly double the cost of Linux bundles (starting at **$8.00/month**).

### B. Static IPv4 Addresses
* **Attached Static IPs:** Free as long as they are attached to an active, running Lightsail instance.
* **Unattached or Stopped Instance Static IPs:** If you detach a Static IP or the instance it is attached to is stopped, AWS charges **$0.005 per hour** (~$3.60/month) for the idle IP address to prevent resource hoarding.

### C. Lightsail Databases & Containers
* **Database Bundles:** Start at $15/month (standard) or $30/month (high availability with a secondary replica).
* **Container Services:** Start at $7/month per node (Micro tier) and scale based on active node count.

### D. Snapshots & Backups
* Manual snapshots and automatic backups are charged separately from the instance bundle at a flat rate of **$0.05 per GB-month** in `us-east-1`.

---

## 4. Detailed Pricing Rates (us-east-1)
*Below are the standard monthly pricing tiers for Linux instances (which include public IPv4):*

| Instance Bundle | Monthly Cost | RAM | vCPU | SSD Storage | Data Transfer Egress |
|-----------------|--------------|-----|------|-------------|----------------------|
| **Nano** | $3.50 | 512 MB | 1 | 20 GB | 1 TB |
| **Micro** | $5.00 | 1 GB | 1 | 40 GB | 2 TB |
| **Small** | $10.00 | 2 GB | 1 | 60 GB | 3 TB |
| **Medium** | $20.00 | 4 GB | 2 | 80 GB | 4 TB |
| **Large** | $40.00 | 8 GB | 2 | 160 GB | 5 TB |
| **X-Large** | $80.00 | 16 GB | 4 | 320 GB | 6 TB |

*Note: Ingress (data transfer in) is completely free and does not count towards your monthly transfer allowance.*

---

## 5. AWS Free Tier Coverage
* **Linux Bundles:** 750 hours/month (one full month) of the **$3.50 Linux Nano**, **$5.00 Linux Micro**, or **$10.00 Linux Small** bundle free for the first 3 months.
* **Windows Bundles:** 750 hours/month of the Windows Nano ($8.00) or Micro ($12.00) bundle free for the first 3 months.
* **CDN & Object Storage:** 12 months free of 50 GB CDN distribution and 5 GB object storage bundle.

---

## 6. Common Cost Hotspots & Pitfalls
* **Double-Billing on Stopped Instances:** Stopping a Lightsail instance **does not stop billing**. You continue to be billed for the instance bundle (SSD storage, CPU reservation) as well as $0.005/hr for the now-idle static IP. The only way to stop billing is to **delete** the instance.
* **Detached Static IPs:** Keeping unattached Static IPs in your account, accumulating $0.005/hr ($3.60/mo per IP) idle fees.
* **Accumulated Snapshot Storage:** Taking manual daily snapshots of a database or server and never cleaning them up. Multiple large snapshots will quickly exceed the cost of the running instance itself.
* **Exceeding Data Transfer Allowance:** Exceeding your generous egress allowance results in $0.09/GB overage fees.

---

## 7. Actionable Cost Optimization Strategies
1. **Delete Unneeded Stopped Instances:** If you have dev, test, or prototype instances that are stopped and no longer needed, **delete them**. If you need to preserve the data, create a snapshot (billed at $0.05/GB-month) and delete the instance.
2. **Release Unused Static IPs:** Audit your Lightsail account regularly. Delete any Static IPs that are not attached to active, running instances to avoid the $0.005/hour idle fee.
3. **Implement Snapshot Lifecycle Cleanup:** Set up automated snapshot limits. Delete old manual snapshots that are no longer relevant to avoid accumulating TBs of backup storage fees.
4. **Use Lightsail as a Reverse Proxy for EC2 Egress:** Because EC2 charges $0.09/GB for all internet egress (above 100 GB), you can run a $5/month Lightsail instance (which includes 2 TB egress) as a proxy/cache. 2 TB of egress on EC2 costs **$180.00**, representing a **97% savings**!
