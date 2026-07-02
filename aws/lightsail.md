# AWS Service Cost Research: Amazon Lightsail

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Lightsail is designed to be the easiest way to launch and manage virtual private servers, databases, and networking on AWS. It provides a simple, predictable pricing structure based on bundled flat-rate plans, making it ideal for small businesses, developers, and simple web applications (like WordPress blogs or static sites).

---

## 2. Billing Mechanics
Lightsail uses a simple monthly subscription-bundle model:
*   **Hourly Billing up to a Cap:** You are billed an hourly rate for your running resources up to the maximum monthly plan price. If an instance is active for only half a month, you pay only half the monthly bundle cost.
*   **Predictable Packages:** Every virtual private server (instance) bundle includes a set allocation of:
    1.  vCPU compute.
    2.  RAM (memory).
    3.  SSD storage (block storage).
    4.  A generous data transfer (egress) allowance.
*   **Overage Billing:** If you exceed your monthly data transfer allowance, you pay standard overage fees (typically **$0.09 per GB** in `us-east-1`).

---

## 3. Key Cost Dimensions

### A. Instance Bundles (Linux vs. Windows)
*   **Linux/Unix:** Baseline bundle rates are very cheap, starting at **$3.50/month**.
*   **Windows:** Bundles incur licensing overhead and are roughly double the cost of Linux bundles (starting at **$8.00/month**).

### B. Static IP Addresses (Elastic IPs equivalent)
*   Static IPs are **free** as long as they are attached to a running Lightsail instance.
*   If you detach a Static IP or the instance it is attached to is stopped, AWS charges **$0.005 per hour** (~$3.60/month) for the idle IP address to prevent resource waste.

### C. Lightsail Databases & Containers
*   Lightsail offers managed databases and container services with independent flat-rate pricing tiers.
*   *Database Bundles:* Start at $15/month (standard) or $30/month (high availability with a secondary replica).
*   *Container Services:* Start at $7/month per node (Micro tier) and scale based on active nodes.

### D. Snapshots & Backups
*   Manual snapshots and automatic backups are charged separately from the instance bundle at a flat rate of **$0.05 per GB-month** in `us-east-1`.

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
*   **Nano Bundle:** 750 hours/month (one full month) of the **$3.50 Linux Nano bundle** free for the first 3 months.
*   **Micro Bundle:** 750 hours/month of the **$5.00 Linux Micro bundle** free for the first 3 months.
*   **Windows Nano/Micro:** 750 hours/month of the Windows Nano ($8.00) or Micro ($12.00) bundle free for the first 3 months.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Detached / Stopped Instance Static IPs:** Shutting down a Lightsail instance for the weekend but keeping its Static IP allocated. The idle IP will generate $0.005/hr charges.
*   **Accumulated Snapshot Storage:** Taking manual daily snapshots of a database or server and never cleaning them up. Multiple large snapshots will quickly exceed the cost of the running instance itself.
*   **Exceeding Data Transfer Allowance:** While Lightsail's data transfer limits are massive (e.g., 3 TB egress on a $10 plan), if you run video streaming or file-sharing workloads, overage fees ($0.09/GB) can generate high bills.
*   **Double-Billing on Stopped Instances:** Stopping a Lightsail instance does *not* pause its monthly bundle charge. You are still billed for the SSD storage, IP allocation, and CPU reservation. The only way to stop billing is to **delete** the instance.

---

## 7. Actionable Cost Optimization Strategies
1.  **Delete Stopped Instances:** If you have dev, test, or prototype instances that are stopped and no longer needed, **delete them**. If you need to preserve the data, create a snapshot (billed at $0.05/GB-month) and delete the instance (which is billed at the full bundle price).
2.  **Release Unused Static IPs:** Audit your Lightsail account regularly. Delete any Static IPs that are not attached to active, running instances to avoid the $0.005/hour idle fee.
3.  **Implement Snapshot Lifecycle Cleanup:** Set up automated snapshot limits. Delete old manual snapshots that are no longer relevant to avoid accumulating TBs of backup storage fees.
4.  **Consolidate Multi-Instance Deployments:** If you are running three or four $3.50/month Nano instances for minor microservices, consider consolidating them onto a single $10/month Small instance. This yields better overall resource utilization and saves money.
5.  **Use Lightsail for High-Egress Workloads:** Because EC2 charges $0.09/GB for *all* egress (above 100 GB), you can use Lightsail instances as reverse proxies or media caches. A $5/month Lightsail instance includes 2 TB of free egress, which would cost **$180.00** in standard EC2 data transfer fees!
