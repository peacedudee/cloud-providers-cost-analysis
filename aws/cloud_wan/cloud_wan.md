# AWS Service Cost Research: AWS Cloud WAN

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Cloud WAN is a managed wide area network (WAN) service that connects on-premises datacenters, branch offices, and Amazon VPCs across the AWS global backbone network. It allows enterprise network administrators to build, manage, and monitor a global transit network using a central dashboard and declarative network policies. Because Cloud WAN carries significant fixed compute charges (Core Network Edges), its deployment requires explicit architectural justification over single-region transit alternatives.

---

## 2. Billing Mechanics
AWS Cloud WAN bills monthly based on four key cost dimensions:
1. **Core Network Edge (CNE) Hourly Fee:** Billed per hour for each regional core network edge instance provisioned.
2. **Attachment Hourly Fee:** Billed per hour for each network connection (VPC, VPN, Direct Connect) attached to your core network.
3. **Data Processing Fee:** Billed per GB of data processed through the core network backbone ($0.02/GB).
4. **Inter-Region Data Transfer:** Standard AWS inter-region data transfer egress fees apply to traffic routed between regions.

---

## 3. Key Cost Dimensions

### A. Core Network Edge (CNE) Hourly Fee (us-east-1)
A Core Network Edge (CNE) is a regional routing endpoint that manages traffic within an AWS region.
* **The Rate:** **$0.50 per hour** (~$365.00/month) per CNE.
* **Global Scale Multiplier:** Operating a global network across 4 AWS regions (e.g., `us-east-1`, `us-west-2`, `eu-west-1`, `ap-northeast-1`) requires 4 CNEs, creating a baseline flat charge of:
  $$\text{Baseline CNE Cost} = 4\text{ CNEs} \times 730\text{ hours} \times \$0.50 = \$1,460.00\text{ / month flat}$$
  This charge applies continuously 24/7 even if zero data passes through the network.

### B. Network Attachment Fees
Every connection link to the Cloud WAN backbone incurs an hourly fee:
* **Attachment Rate:** **$0.065 per hour** (~$47.45/month) per attachment (VPC, VPN, or Direct Connect).
* **Cost Comparison:** Cloud WAN attachments ($0.065/hr) carry a **30% premium** over standard Transit Gateway attachments ($0.05/hr).

### C. Data Processing & Inter-Region Transfer
* **Processing Rate:** **$0.02 per GB** of data processed across network attachments.
* **Inter-Region Traffic:** Traffic crossing regions over the Cloud WAN backbone pays the $0.02/GB processing fee plus standard AWS **Inter-Region Data Transfer egress fees** ($0.01 to $0.02/GB).

---

## 4. Detailed Pricing Rates (us-east-1 Baseline)

| Cost Component | Hourly Base Rate | Data Processing Rate (/GB) | Monthly Base Cost (per unit) |
|----------------|------------------|----------------------------|------------------------------|
| **Core Network Edge (CNE)** | **$0.5000** | N/A | ~$365.00 |
| **VPC Attachment** | **$0.0650** | **$0.0200** | ~$47.45 |
| **VPN Attachment** | **$0.0650** | **$0.0200** | ~$47.45 |
| **Direct Connect Attachment** | **$0.0650** | **$0.0200** | ~$47.45 |

---

## 5. AWS Free Tier Coverage
* **AWS Cloud WAN:** No free tier available. CNE and attachment billing accumulate immediately upon deployment.

---

## 6. Common Cost Hotspots & Pitfalls
* **Deploying CNEs in Low-Traffic Regions:** Provisioning a Core Network Edge in a region hosting minor workloads. Each regional CNE adds **$365.00/month** to the bill regardless of usage.
* **Using Cloud WAN for Single-Region Workloads:** Deploying Cloud WAN for network architectures contained entirely within a single AWS region instead of using AWS Transit Gateway.
* **Retaining Unused Attachments:** Leaving attachments active to stopped or deprecated developer VPCs ($0.065/hr per link).

---

## 7. Actionable Cost Optimization Strategies
1. **Use AWS Transit Gateway for Single-Region Architectures:**
   * If your application footprint resides within a single AWS region, do not deploy Cloud WAN.
   * Use **AWS Transit Gateway**, which has lower attachment fees ($0.05/hr vs $0.065/hr) and zero CNE charges (saving $365.00/month per region).
   * **The Savings:** Saves $4,380.00/year per region in baseline fixed costs.
2. **Restrict CNE Provisioning to Core Production Hubs:** Deploy CNEs only in major regional hub locations (e.g., `us-east-1` and `eu-west-1`). Connect minor spoke regions via VPC Peering to avoid local CNE charges.
3. **Establish VPC Peering for Heavy Intra-Region Traffic:** For high-volume VPC-to-VPC traffic in the same region, set up direct VPC Peering to bypass the $0.02/GB Cloud WAN processing tax and attachment fees.
4. **Audit and Prune Inactive Network Links:** Automatically disassociate and delete VPC or VPN attachments when target environments are decommissioned.
