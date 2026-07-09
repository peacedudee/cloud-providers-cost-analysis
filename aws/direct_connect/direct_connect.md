# AWS Service Cost Research: AWS Direct Connect

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Direct Connect is a cloud service solution that makes it easy to establish a dedicated network connection from your on-premises datacenter or office directly to AWS. By using industry-standard VLANs, this dedicated connection can partition into multiple virtual interfaces. Direct Connect bypasses the public internet, providing lower latency, higher bandwidth, and critically, **highly discounted data egress rates**, making it a primary cost-saving mechanism for high-volume hybrid architectures.

---

## 2. Billing Mechanics
Direct Connect is billed on a monthly cycle based on three primary dimensions:
1.  **Port Hours (Capacity):** Hourly charges based on the physical connection speed and whether you use a Dedicated or Hosted Connection.
2.  **Data Transfer Out (Egress):** Billed per GB of data transferred from AWS to your on-premises datacenter over the Direct Connect link.
3.  **SiteLink Fees (Optional):** Extra charges if you use Direct Connect to route traffic directly between your on-premises offices over the AWS backbone network.
4.  **No-Charge Features:**
    *   **MACsec Encryption:** Available on dedicated physical connections at **no additional charge ($0.00)**.
    *   **Direct Connect Gateway (DXGW):** Establishing a global DXGW to route traffic to multiple VPCs/regions is **100% free ($0.00)**.

---

## 3. Key Cost Dimensions

### A. Port Hours (Dedicated Connections - us-east-1)
Dedicated Connections are physical ports allocated directly to your account.
*   **1 Gbps Connection:** **$0.30 per hour** (~$219.00/month).
*   **10 Gbps Connection:** **$2.25 per hour** (~$1,642.50/month).
*   **100 Gbps Connection:** **$22.50 per hour** (~$16,425.00/month).
*   *Note: Billed continuously from the moment the port is provisioned, regardless of whether you send any traffic over the link.*

### B. Hosted Connections (Partner Ports)
Hosted Connections are provisioned through an AWS Direct Connect Partner. Billed at slightly lower hourly AWS rates, but the partner will charge their own separate monthly port fees:
*   *50 Mbps Hosted Port:* **$0.03 per hour** (~$21.90/month).
*   *500 Mbps Hosted Port:* **$0.08 per hour** (~$58.40/month).
*   *1 Gbps Hosted Port:* **$0.21 per hour** (~$153.30/month).
*   *10 Gbps Hosted Port:* **$1.58 per hour** (~$1,153.40/month).

### C. Data Transfer Out (DTO) Discount
This is the primary financial incentive for deploying Direct Connect:
*   **Internet Egress Rate:** Standard AWS internet egress is **$0.09 per GB**.
*   **Direct Connect Egress Rate (US/Europe):** Billed at a flat rate of **$0.02 per GB** (a **78% direct discount**).
*   *Data Ingress:* All data transfer *into* AWS over Direct Connect is **free ($0.00/GB)**.
*   *Break-Even Math:* If your enterprise downloads 50 TB (50,000 GB) of data per month from AWS:
    *   *Via Internet:*
        $$50,000\text{ GB} \times \$0.09/\text{GB} = \$4,500.00\text{ / month}$$
    *   *Via 1 Gbps Direct Connect:*
        $$\text{Port Cost} = \$0.30 \times 730\text{ hours} = \$219.00$$
        $$\text{Egress Cost} = 50,000\text{ GB} \times \$0.02/\text{GB} = \$1,000.00$$
        $$\text{Total DX Cost} = \$219.00 + \$1,000.00 = \$1,219.00\text{ / month}$$
    *   *Net Savings:* Saves **73%** ($3,281.00/month saved) compared to internet routing.

### D. Direct Connect SiteLink
SiteLink allows you to connect your on-premises offices together over the AWS network.
*   **Hourly Fee:** **$0.50 per hour** per active Virtual Interface (VIF) where SiteLink is enabled.
*   **Data Processing Fee:** Charged per GB of data sent between locations (e.g. **$0.06 per GB** in the US).

---

## 4. Detailed Pricing Rates (us-east-1 Dedicated Ports)

| Connection Speed | Hourly Port Rate | Monthly Port Cost (Base) | Direct Connect Egress (/GB) | Standard Internet Egress (/GB) |
|------------------|------------------|---------------------------|-----------------------------|--------------------------------|
| **1 Gbps** | **$0.30 / hour** | ~$219.00 | **$0.0200** | $0.0900 |
| **10 Gbps** | **$2.25 / hour** | ~$1,642.50 | **$0.0200** | $0.0900 |
| **100 Gbps** | **$22.50 / hour**| ~$16,425.00 | **$0.0200** | $0.0900 |

---

## 5. AWS Free Tier Coverage
*   **AWS Direct Connect:** No free tier is available. Port fees accumulate from the moment the connection is established.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Provisioning 10 Gbps Dedicated Ports for Low-Traffic Environments:** Ordering 10 Gbps ports for failover or backup links that only process 1-2 TB/month. The flat port-hour fee (**$1,642.50/month**) far outweighs the egress discount.
*   **Leaving Ordered Connections Active but Unconnected:** AWS bills you for the port starting **90 days after ordering** (or when the port goes active), even if no fiber cable is physically plugged in yet.
*   **Leaving SiteLink Enabled on Backup Links:** Keeping SiteLink active on primary and secondary virtual interfaces continuously, accumulating $0.50/hour per interface.

---

## 7. Actionable Cost Optimization Strategies
1.  **Perform Break-Even Analysis (Hosted vs. Dedicated):**
    *   If your monthly data egress is under **10 TB**, use a **Hosted Connection** (e.g., 200 Mbps or 500 Mbps) through an AWS partner to avoid the high base rate of dedicated physical ports.
    *   If monthly egress exceeds **20 TB**, deploy a **1 Gbps Dedicated Connection** to maximize the egress discount.
2.  **Cancel Unconnected Direct Connect Ports:**
    *   If your datacenter migration is delayed, cancel the Direct Connect port request before the 90-day automatic billing window triggers, and re-order once physical fiber cabling schedules are locked.
3.  **Disable SiteLink Outside Active Transfers:**
    *   Write a script to disable virtual interfaces or disable SiteLink configuration during idle weekdays to save the $0.50/hour connection charge.
4.  **Consolidate VPC Connections with Direct Connect Gateway (DXGW) and Transit Gateway:**
    *   Connect your Direct Connect link to a **Direct Connect Gateway (DXGW)** and link it to Transit Gateway.
    *   **The Savings:** Allows up to 10 VPCs to share a single Direct Connect virtual interface, avoiding paying for multiple port hours.
