# AWS Service Cost Research: AWS Site-to-Site VPN

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Site-to-Site VPN establishes a secure, IPsec-encrypted tunnel between your Virtual Private Cloud (VPC) and your on-premises offices or datacenters. By default, AWS provisions two tunnels per connection for high availability (failover redundancy). Unlike Client VPN, Site-to-Site VPN has no connection charges for data volume, instead billing a flat hourly rate per connection. However, enabling performance features (like Acceleration) can significantly impact the bill.

---

## 2. Billing Mechanics
AWS Site-to-Site VPN is billed on a monthly cycle based on three dimensions:
1.  **VPN Connection Hourly Fee:** A flat fee charged per hour or partial hour that a VPN connection is provisioned.
2.  **AWS Global Accelerator Surcharges (Accelerated VPN only):** Hourly base fees and per-GB Data Transfer-Premium (DT-Premium) charges for routing traffic over the AWS global network.
3.  **Data Transfer (Egress):** Standard AWS egress charges apply to traffic exiting AWS to on-premises over the VPN.

---

## 3. Key Cost Dimensions

### A. VPN Connection Fee (us-east-1 Flat Fee)
*   **The Rate:** **$0.05 per hour** (~$36.50/month) per connection.
*   **Tunnel Redundancy:** Although each connection automatically provisions **2 tunnels** for high availability, you are billed only for the single connection (you do not pay double).
*   *Scale Impact:* Running 10 Site-to-Site VPN links from AWS to various branch offices creates a flat baseline fee of:
    $$10\text{ links} \times 730\text{ hours} \times \$0.05 = \$365.00\text{ / month flat}$$

### B. Accelerated Site-to-Site VPN
Accelerated VPN utilizes **AWS Global Accelerator** to route traffic to the nearest AWS edge location, improving connection reliability over long distances.
*   **Hourly Connection Fee:** **$0.05 per hour** (~$36.50/month).
*   **Global Accelerator Fixed Fee:** **$0.025 per hour** (~$18.25/month).
*   **Data Transfer-Premium (DT-Premium) Fee:** Billed per GB of data transferred through the accelerated tunnels (dominant direction rules apply).
    *   *US / Canada / Europe:* **$0.015 per GB** (in addition to standard egress).
    *   *Asia Pacific:* **$0.035 per GB**.
*   *Baseline Cost:* Running an Accelerated VPN link costs a baseline of **$54.75/month** (excluding data fees).

### C. Data Transfer Out
*   Data transfer *into* AWS over the VPN is **free ($0.00/GB)**.
*   Data transfer *out* of AWS to on-premises is billed at standard internet egress rates:
    *   First 100 GB/month: **Free** (global allowance).
    *   Up to 10 TB/month: **$0.09 per GB**.

---

## 4. Detailed Pricing Rates (us-east-1)

| VPN Configuration | Hourly Connection Fee | Accelerator Base Rate | Egress Data Premium (/GB) | Monthly Base Cost (Est.) |
|-------------------|-----------------------|-----------------------|---------------------------|--------------------------|
| **Standard VPN** | **$0.0500** | N/A | N/A | ~$36.50 |
| **Accelerated VPN**| **$0.0500** | **$0.0250** | **$0.0150** | **$54.75** (plus DT-Prem) |

---

## 5. AWS Free Tier Coverage
*   **AWS Site-to-Site VPN:** No free tier is available. Connection hourly charges begin immediately upon provisioning.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Leaving Temporary VPN Connections Active:** Deploying Site-to-Site VPN connections for data center migration tasks or developer testing and leaving the links active in the VPC dashboard after the migration is complete.
*   **Selecting Accelerated VPN by Default for Short Ranges:** Deploying Accelerated VPN when connecting an office to an AWS region in the same geographical area (e.g. N. Virginia office to `us-east-1`). Over short physical distances, Global Accelerator yields no performance benefit but adds the $0.025/hr base and $0.015/GB premium surcharges.
*   **Routing Cloud Backups over Standard VPN instead of Direct Connect:** Syncing terabytes of database logs or backups to on-premises over a standard VPN. VPN egress bills at standard internet rates (**$0.09/GB**), whereas **Direct Connect** egress is highly discounted at **$0.02/GB**.

---

## 7. Actionable Cost Optimization Strategies
1.  **Delete Inactive VPN Connections:** Audit the VPC Dashboard under VPN Connections. Identify tunnels that show as `DOWN` continuously for more than 30 days and delete them to save $36.50/month per link.
2.  **Bypass Acceleration for Local/Regional Links:** Verify the geographical location of your on-premises customer gateway. If the gateway is located in the same country/region as the AWS datacenter, disable the **Acceleration** flag on the VPN connection.
3.  **Evaluate Direct Connect for High-Volume Egress:**
    *   If your monthly egress from AWS to on-premises over VPN exceeds **20 TB**, standard VPN egress costs:
        $$20,000\text{ GB} \times \$0.09 = \$1,800.00\text{ / month}$$
    *   Migrating this traffic to a **Hosted 500 Mbps Direct Connect link** costs ~$58.40/month port base + $0.02/GB egress ($400) = **$458.40/month** (saving 74%!).
4.  **Consolidate VPN Connections using Transit Gateway:** If connecting on-premises to multiple VPCs, do not create a separate VPN connection for each VPC. Deploy a single **Site-to-Site VPN connection** to a **Transit Gateway**, and route traffic to your VPC attachments.
5.  **Use Software-Based VPNs for Non-Critical Dev Links:** For non-production branch offices, evaluate running a software-based VPN appliance (like OpenVPN Access Server or pfSense) on a t3.micro EC2 instance instead of a managed AWS VPN.
