# Cost-Cutting Playbook: AWS Direct Connect

> **Companion File:** [direct_connect.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/direct_connect/direct_connect.md)  
> **Last Updated:** July 2026

---

## Executive Summary

AWS Direct Connect establishes a dedicated physical network connection between on-premises datacenters and AWS. Billing includes **Port Hours** (1 Gbps Dedicated at $0.30/hr = $219/mo; 10 Gbps Dedicated at $2.25/hr = $1,642.50/mo; 100 Gbps Dedicated at $22.50/hr = $16,425/mo), **Data Transfer Out (DTO)** ($0.02/GB flat vs $0.09/GB public internet — **78% direct discount**), and **SiteLink Fees** ($0.50/hr per VIF).

Direct Connect serves as the ultimate data transfer cost-cutter when bandwidth is high, but creates severe cost traps when misconfigured:
1. **Provisioning 10 Gbps Dedicated Ports for Low-Traffic Workloads:** Paying $1,642.50/month for a 10 Gbps port when monthly egress is < 10 TB.
2. **Unconnected Port Billing:** Billing automatically starts **90 days after ordering** even if fiber optic cables have not been plugged in yet.
3. **SiteLink Left Enabled 24/7 on Backup VIFs:** Accumulating $0.50/hr per active VIF continuously.

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **30–78% reduction in network egress and connectivity spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Perform Hosted vs Dedicated Port Break-Even Analysis:** Use Hosted Partner Ports (50 Mbps – 500 Mbps) for egress < 10 TB/mo to avoid high dedicated port fees ($1,642.50/mo).
2. **Consolidate VPC Access with Direct Connect Gateway (DXGW) & Transit Gateway:** Shares a single Direct Connect link across up to 10 VPCs globally for **$0.00 extra port fee**.
3. **Disable SiteLink on Idle Backup Virtual Interfaces (VIFs):** Saves $0.50/hr per VIF ($365/mo per interface).

---

## Strategy Categories

### 1. Port Capacity & Connection Type Selection

#### 1. Perform Break-Even Analysis (Hosted vs Dedicated Connections)
- **What:** Evaluate monthly data egress volume to choose between **Hosted Partner Connections** (50 Mbps to 500 Mbps) and **Dedicated Connections** (1 Gbps, 10 Gbps, 100 Gbps).
- **Why It Saves Money:**
  - **10 Gbps Dedicated Port:** Costs **$1,642.50/month** in base port fees.
  - **500 Mbps Hosted Port:** Costs **~$58.40/month** in base AWS port fees.
  - If monthly egress is only 5 TB (5,000 GB), deploying a 500 Mbps Hosted Connection saves **$1,584.10/month ($19,009.20/year)**!
  - **Break-Even Point:** Deploy 1 Gbps Dedicated when egress exceeds 20 TB/mo; deploy 10 Gbps Dedicated only when egress exceeds 100 TB/mo.
- **Detailed Implementation Steps:**
  1. Audit monthly egress volume from Cost Explorer (`line_item_usage_type = DirectConnect-Out-Bytes`).
  2. Order Hosted Connection from an AWS Direct Connect Partner (Equinix, Megaport) if egress < 20 TB/mo.
- **Estimated Savings:** **70–95% reduction** in port capacity fees for low-to-medium throughput links.
- **Risk Level:** Zero risk (partner ports scale dynamically).
- **Implementation Scope:** Network Architect / Procurement
- **Prerequisites:** 30-day network egress audit.

---

### 2. Egress Traffic Routing Optimization

#### 2. Maximize the 78% Data Transfer Out (DTO) Discount vs Internet Egress
- **What:** Route all corporate on-premises data downloads, database replication, and backup traffic over Direct Connect instead of public internet gateways.
- **Why It Saves Money:**
  - **Public Internet Egress:** **$0.0900 per GB** ($90.00/TB).
  - **Direct Connect Egress (US/EU):** **$0.0200 per GB** ($20.00/TB).
  - Downloading 100 TB/month over Direct Connect costs **$2,000.00** vs **$9,000.00** over the internet — saving **$7,000.00/month (a 78% direct discount)**!
- **Detailed Implementation Steps:**
  1. Configure VPC route tables to direct corporate IP ranges (`10.0.0.0/8`, `172.16.0.0/12`) to the Virtual Private Gateway (VGW) or Transit Gateway (TGW) linked to Direct Connect.
- **Estimated Savings:** **78% reduction** in data egress fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Direct Connect VIF operational.

---

### 3. Architecture Consolidation & Gateway Sharing

#### 3. Consolidate Multi-VPC Access via Direct Connect Gateway (DXGW) & Transit Gateway
- **What:** Attach Direct Connect virtual interfaces (Transit VIF or Private VIF) to a **Direct Connect Gateway (DXGW)** and link it to AWS Transit Gateway or multiple VPCs.
- **Why It Saves Money:** Allows up to 10 VPCs across multiple AWS regions to share a single physical Direct Connect link. Direct Connect Gateway is **100% FREE ($0.00)**, avoiding purchasing multiple physical ports per region.
- **Detailed Implementation Steps:**
  1. Create Direct Connect Gateway in Terraform:
     ```hcl
     resource "aws_dx_gateway" "central_dxgw" {
       name            = "central-dxgw"
       amazon_side_asn = "64512"
     }

     resource "aws_dx_gateway_association" "tgw_assoc" {
       dx_gateway_id         = aws_dx_gateway.central_dxgw.id
       associated_gateway_id = aws_ec2_transit_gateway.main.id
     }
     ```
- **Estimated Savings:** **100% savings** on additional regional port fees.
- **Risk Level:** Zero risk (standard enterprise networking topology).
- **Implementation Scope:** Network Architect
- **Prerequisites:** DXGW and Transit Gateway deployment.

---

### 4. SiteLink & Optional Feature Control

#### 4. Disable Direct Connect SiteLink on Secondary / Backup VIFs
- **What:** Turn off **Direct Connect SiteLink** on backup or idle Virtual Interfaces when office-to-office routing over the AWS backbone is not actively required.
- **Why It Saves Money:**
  - SiteLink bills **$0.50 per hour per active VIF** ($365.00/month) plus data processing fees ($0.06/GB).
  - Disabling SiteLink on 4 idle backup VIFs saves **$1,460.00/month ($17,520.00/year)**!
- **Detailed Implementation Steps:**
  1. Disable SiteLink attribute on VIF via AWS CLI:
     ```bash
     aws directconnect update-private-virtual-interface \
       --virtual-interface-id dxvif-12345678 \
       --no-enable-site-link
     ```
- **Estimated Savings:** **$365.00 per month saved** per disabled VIF.
- **Risk Level:** Zero risk (if site-to-site communication is handled via SD-WAN).
- **Implementation Scope:** Network Engineer
- **Prerequisites:** SiteLink usage audit.

---

### 5. Order & Lifecycle Governance

#### 5. Cancel Unconnected Port Orders Before 90-Day Auto-Billing Triggers
- **What:** Audit pending Direct Connect port orders (`aws directconnect describe-connections`). Cancel port requests if datacenter fiber installation is delayed.
- **Why It Saves Money:** AWS automatically starts billing for Direct Connect port hours **90 days after ordering** (or when port goes active), even if no fiber cable is physically connected!
- **Detailed Implementation Steps:**
  1. Cancel pending connection: `aws directconnect delete-connection --connection-id dxcon-12345678`.
- **Estimated Savings:** Prevents $1,642.50/mo billing for un-connected ports.
- **Risk Level:** Low.
- **Implementation Scope:** Procurement / Network Team
- **Prerequisites:** Connection status audit.

---

### 6. Observability & Governance

#### 6. Utilize Free MACsec Encryption on Dedicated Ports
- **What:** Enable **MACsec Encryption** (IEEE 802.1AE) on 10 Gbps and 100 Gbps Dedicated Connections.
- **Why It Saves Money:** MACsec encryption operates at Layer 2 at the hardware port level for **100% FREE ($0.00)**. Replaces expensive software VPN encryption appliances.
- **Detailed Implementation Steps:**
  1. Enable MACsec on dedicated port in Direct Connect console.
- **Estimated Savings:** $0.00 encryption cost.
- **Risk Level:** Zero.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** MACsec-compatible router equipment.

#### 7. Audit Dual-Port Active/Passive Failover Configurations
- **What:** Configure active/passive dual-port failover using BGP Local Preference so secondary ports operate in standby mode.
- **Why It Saves Money:** Ensures failover links are ready without incurring active SiteLink data processing fees.
- **Detailed Implementation Steps:**
  1. Set BGP community tags for Local Preference on standby router.
- **Estimated Savings:** Data processing fee optimization.
- **Risk Level:** Zero.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** BGP router configuration.

#### 8. Enforce CloudWatch Alarms for `ConnectionError` and `ConnectionState`
- **What:** Put CloudWatch alarm on `ConnectionState` metric.
- **Why It Saves Money:** Instant alert if physical port drops to offline state while billing continues.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm targeting `AWS/DX`.
- **Estimated Savings:** Operational risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 9. Monitor Interface Bandwidth Utilization (`VirtualInterfaceBpsEgress`)
- **What:** Monitor `VirtualInterfaceBpsEgress` to downsize 10 Gbps ports to 1 Gbps if peak utilization is < 800 Mbps.
- **Why It Saves Money:** Downsizing from 10 Gbps ($1,642/mo) to 1 Gbps ($219/mo) saves **$1,423.50/month**.
- **Detailed Implementation Steps:**
  1. Request port speed downsize or re-order 1 Gbps port.
- **Estimated Savings:** 86% port fee reduction.
- **Risk Level:** Medium.
- **Implementation Scope:** Network Architect
- **Prerequisites:** Peak bandwidth audit.

#### 10. Optimize Public VIF Access for AWS Public Endpoints
- **What:** Use **Public VIFs** to route traffic to S3, DynamoDB, and Glacier over Direct Connect.
- **Why It Saves Money:** Applies the discounted $0.02/GB egress rate to public S3 downloads.
- **Detailed Implementation Steps:**
  1. Create Public VIF and advertise public IP prefixes over BGP.
- **Estimated Savings:** 78% egress discount on public service downloads.
- **Risk Level:** Medium.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Public IP prefix ownership.

#### 11. Right-Size Transit Gateway Peering Connections
- **What:** Peer Direct Connect Gateways with Transit Gateways in target regions rather than deploying separate Direct Connect ports per region.
- **Why It Saves Money:** Consolidates global network topology.
- **Detailed Implementation Steps:**
  1. Create Transit Gateway Peering attachments.
- **Estimated Savings:** Multi-region port fee elimination.
- **Risk Level:** Zero.
- **Implementation Scope:** Network Architect
- **Prerequisites:** TGW cross-region peering setup.

#### 12. Standardize Partner Cross-Connect Pricing Contracts
- **What:** Negotiate flat-rate datacenter cross-connect contracts with colocation providers (Equinix, Digital Realty).
- **Why It Saves Money:** Controls third-party physical cabling recurring fees.
- **Detailed Implementation Steps:**
  1. Review colocation provider cross-connect invoice rates.
- **Estimated Savings:** 20–40% third-party colocation savings.
- **Risk Level:** Zero.
- **Implementation Scope:** Procurement Team
- **Prerequisites:** Vendor contract negotiation.

#### 13. Audit Link Aggregation Group (LAG) Configurations
- **What:** Consolidate multiple 1 Gbps ports into a **Link Aggregation Group (LAG)** only when bandwidth exceeds 800 Mbps.
- **Why It Saves Money:** Prevents paying multiple $219/mo port fees for under-utilized parallel ports.
- **Detailed Implementation Steps:**
  1. Disband unneeded LAG member ports via CLI.
- **Estimated Savings:** $219.00/mo saved per removed LAG port.
- **Risk Level:** Low.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** LAG bandwidth audit.

#### 14. Enable Jumbo Frames (9000 MTU) on Virtual Interfaces
- **What:** Set MTU = 9000 (Jumbo Frames) on Private VIFs and Transit VIFs.
- **Why It Saves Money:** Reduces network packet header overhead and router CPU utilization, accelerating bulk data transfers.
- **Detailed Implementation Steps:**
  1. Set `mtu = 9000` on VIF in Terraform.
- **Estimated Savings:** Transfer throughput efficiency.
- **Risk Level:** Zero.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Router MTU 9000 support.

#### 15. Enforce IAM Network Administrator Access Control
- **What:** Restrict `directconnect:Allocate*` and `directconnect:Create*` permissions via IAM.
- **Why It Saves Money:** Security governance best practice.
- **Detailed Implementation Steps:**
  1. Restrict IAM policies for DX administration.
- **Estimated Savings:** Security risk mitigation.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** IAM policy review.

#### 16. Audit BGP Session Stability to Prevent Flapping Egress Fallback
- **What:** Configure BGP keepalive timers (10s) and hold timers (30s) to prevent BGP route flapping.
- **Why It Saves Money:** Prevents network traffic from falling back to expensive public internet egress ($0.09/GB) during BGP drops.
- **Detailed Implementation Steps:**
  1. Tune BGP timers on customer gateway router.
- **Estimated Savings:** Prevents internet egress price penalty spikes.
- **Risk Level:** Zero.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** BGP router access.

---

## Cross-Service Synergies

```
[ Datacenter Traffic ] 
        │
        ├──(Egress Routing)────> [ Direct Connect ($0.02/GB) ] (78% discount vs Public Internet $0.09/GB)
        │
        ├──(Connection Sizing)─> [ Hosted Partner Port (500 Mbps) ] (Saves $1,584/mo vs 10 Gbps Dedicated)
        │
        └──(Multi-VPC Sharing)─> [ Direct Connect Gateway + TGW ] (100% FREE - Shares 1 port across 10 VPCs)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `DirectConnect-PortHour:10G`, `DirectConnect-PortHour:1G`, `DirectConnect-Out-Bytes`, `DX-SiteLink-Hours`.
- `line_item_resource_id`: Direct Connect Connection ID / VIF ID (`dxcon-12345678`, `dxvif-12345678`).

### B. CloudWatch Metrics
- `AWS/DX` Namespace: `ConnectionState`, `ConnectionErrorCount`, `VirtualInterfaceBpsEgress`, `VirtualInterfaceBpsIngress`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "DX-PRT-001",
  "service": "Direct Connect",
  "category": "Port Capacity & Connection Type Selection",
  "resource_id": "dxcon-12345678",
  "resource_name": "primary-datacenter-10g-port",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "connection_type": "DEDICATED",
    "port_speed": "10 Gbps",
    "monthly_egress_tb": 4.5,
    "monthly_port_cost_usd": 1642.50
  },
  "recommended_config": {
    "connection_type": "HOSTED_PARTNER",
    "port_speed": "500 Mbps",
    "projected_monthly_port_cost_usd": 58.40
  },
  "financial_impact": {
    "monthly_savings_usd": 1584.10,
    "annual_savings_usd": 19009.20,
    "savings_percentage": 96.4
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Monthly egress is 4.5 TB; 500 Mbps hosted port easily handles bandwidth while saving $1,584/mo in port fees."
  },
  "implementation": {
    "scope": "Network Architect / Procurement",
    "effort_estimate": "1-2 weeks (partner port order)",
    "automation_eligible": false
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Hosted vs Dedicated Port Downsizing**| 4 | $6,570.00 | $6,336.40 | 96.4% | Zero |
| **Direct Connect Egress Routing (78%)**| 10 | $45,000.00 | $35,100.00 | 78.0% | Zero |
| **SiteLink Disabling on Backup VIFs**  | 6 | $2,190.00 | $1,460.00 | 66.6% | Zero |
| **DXGW Multi-VPC Port Consolidation** | 5 | $3,285.00 | $3,285.00 | 100.0% | Zero |
| **Total** | **25** | **$57,045.00** | **$46,181.40** | **81.0%** | -- |
