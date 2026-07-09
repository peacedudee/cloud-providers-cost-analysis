# AWS Service Cost Research: AWS Client VPN

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Client VPN is a fully managed, OpenVPN-compatible client-to-site VPN service that enables secure remote access to AWS resources and on-premises networks. Client VPN uses a hybrid billing model combining a flat hourly subnet association fee with a usage-based user connection rate. Because the subnet association fee is charged continuously 24/7 regardless of user activity, Client VPN endpoints are a common source of idle billing leakage.

---

## 2. Billing Mechanics
Client VPN is billed monthly based on two primary dimensions:
1. **Client VPN Endpoint Subnet Association Fee:** A flat hourly fee charged for each subnet associated with your Client VPN endpoint ($0.10/hour per subnet).
2. **Client Connection Hourly Fee:** Billed per hour or partial hour for each active user connection session ($0.05/hour).
3. **Data Transfer (Egress):** Standard AWS outbound internet data transfer rates apply to traffic exiting AWS over the VPN.

---

## 3. Key Cost Dimensions

### A. Subnet Association Fee (us-east-1 Flat Rate)
* **The Rate:** **$0.10 per hour** (~$73.00/month) per associated subnet.
* **High Availability Multiplier:** Associating subnets across separate Availability Zones for redundancy increases fixed costs:
  * *1 Subnet Association (Single AZ):* ~$73.00/month flat.
  * *2 Subnet Associations (HA):* **$146.00/month flat**.
* *Warning:* This fee is billed continuously 24/7 from the moment a subnet is associated, even if zero users connect.

### B. Client Connection Fee (Pay-as-you-use)
* **The Rate:** **$0.05 per hour** per active user session.
* **Metering:** Billed in 1-hour increments (rounded up).
* **Mathematical Cost Calculation Example:**
  * *Scenario: 30 developers connecting to a 2-subnet HA Client VPN endpoint for 140 hours per month each.*
  1. **Subnet Association Fee (2 AZs):**
     $$\text{Association Cost} = 2\text{ subnets} \times 730\text{ hours} \times \$0.10 = \$146.00\text{ / month}$$
  2. **User Connection Fee:**
     $$\text{Connection Cost} = 30\text{ users} \times 140\text{ hours} \times \$0.05 = \$210.00\text{ / month}$$
  3. **Total Monthly Cost:** **$356.00 / month**

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Hourly Base Rate | Monthly Base Cost (Est.) | Billed Duration / Details |
|----------------|------------------|---------------------------|----------------------------|
| **Subnet Association** | **$0.10 / hour** | ~$73.00 per subnet | Billed continuously 24/7 |
| **User Connection** | **$0.05 / hour** | N/A | Billed per active session hour |
| **HA (2 Subnets)** | **$0.20 / hour** | **$146.00 / month flat** | Billed continuously 24/7 |

---

## 5. AWS Free Tier Coverage
* **AWS Client VPN:** No free tier available. Subnet association charges begin immediately upon endpoint creation.

---

## 6. Common Cost Hotspots & Pitfalls
* **Idle Client VPN Endpoints Left Associated 24/7:** Maintaining VPN endpoints in non-production accounts when developers only connect occasionally. An idle HA endpoint costs **$146.00/month** in flat charges.
* **Connections Left Open Over Weekends:** Developers leaving VPN clients connected over weekends (64 hours). A team of 50 developers leaving connections open over a weekend adds **$160.00** in idle connection fees per weekend.
* **Full Tunneling All Internet Traffic:** Routing all public internet traffic (YouTube, video calls, software updates) through the VPN, generating unnecessary AWS data egress charges.

---

## 7. Actionable Cost Optimization Strategies
1. **Automate Subnet Disassociation Outside Work Hours:**
   * Schedule an AWS Lambda function via EventBridge to disassociate subnets from non-production Client VPN endpoints at 7 PM and re-associate at 7 AM.
   * **The Savings:** Cuts fixed subnet association costs by **64%** (saving ~$93.00/month per HA endpoint).
2. **Enforce Connection Timeout Limits:** Configure the **Connection Timeout** setting on Client VPN endpoints (e.g., max 8 or 10 hours) to disconnect idle or forgotten user sessions automatically.
3. **Enable Split Tunneling:**
   * Enable **Split-tunnel** on the Client VPN endpoint so only traffic destined for private VPC subnets (`10.0.0.0/16`) travels over the VPN. Public internet traffic bypasses the VPN, saving egress fees.
4. **Consolidate VPN Endpoints via Centralized Management VPC:**
   * Deploy a **single Client VPN endpoint** in a central Management VPC and connect to spoke VPCs via VPC Peering.
   * **The Savings:** Consolidates all developers under one flat $146/month HA base fee instead of multiplying endpoint charges per VPC.
