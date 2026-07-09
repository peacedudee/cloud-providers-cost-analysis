# AWS Service Cost Research: AWS Network Firewall

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Network Firewall is a stateful, managed network firewall and intrusion detection and prevention service (IDS/IPS) for your Virtual Private Clouds (VPCs). It provides fine-grained network protection—including stateful inspection, protocol detection, domain name filtering, and Snort-compatible IPS rule evaluation. Network Firewall operates on a provisioned endpoint and data processing model. Because firewall endpoints incur high hourly fees per AZ, unoptimized VPC topologies create massive baseline costs.

---

## 2. Billing Mechanics
AWS Network Firewall billing is calculated across three primary dimensions per Availability Zone (AZ):
1. **Primary Firewall Endpoint Hourly Fee:** Billed hourly per provisioned primary firewall endpoint in each Availability Zone ($0.395 per endpoint-hour).
2. **Secondary Firewall Endpoint Hourly Fee (Multi-VPC Association):** Billed at a reduced hourly rate for secondary endpoints attached to additional VPCs.
3. **Data Processing Fee:** Billed per GB of traffic processed through the primary firewall endpoints ($0.065 per GB). Secondary endpoints incur **$0.00 data processing fees**.
4. **NAT Gateway 1-to-1 Fee Waiver:** When a NAT Gateway is chained in the same service path with a Network Firewall endpoint, standard NAT Gateway hourly base fees and data processing fees are **waived on a 1-to-1 basis**.
5. **TLS Inspection:** Advanced TLS inspection is charged an additional hourly rate per endpoint, but the extra TLS data processing surcharge was **removed by AWS** (only standard $0.065/GB data processing applies).

---

## 3. Key Cost Dimensions

### A. Core Pricing (us-east-1 Rates)
* **Primary Firewall Endpoint Hourly Fee:** **$0.395 per firewall endpoint per hour** in `us-east-1`.
  * *Single AZ:* $0.395 × 730 hours = **$288.35 per month**.
  * *Production 3-AZ High Availability (3 Endpoints):*
    $$3\text{ AZs} \times \$288.35 = \mathbf{\$865.05\text{ / month flat baseline fee per VPC}}$$
* **Data Processing Fee:** **$0.065 per GB** of data processed ($65.00 per TB).
* **Secondary Endpoint Advantage:** Associating a secondary VPC with an existing firewall endpoint uses a reduced hourly rate and charges **$0.00 for data processing**.

### B. The Spoke VPC Endpoint Hazard (Architectural Trap)
* If an organization deploys dedicated primary Network Firewall endpoints inside **10 separate VPCs** across 3 AZs:
  $$\text{Endpoints: } 10\text{ VPCs} \times 3\text{ AZs} = 30\text{ endpoints}$$
  $$\text{Baseline Fee: } 30 \times \$288.35 = \mathbf{\$8,650.50\text{ / month in idle endpoint fees alone!}}$$

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Component | Rate (us-east-1) | Price for 100 GB / 3-AZ Month | Notes |
|-------------------|------------------|-------------------------------|-------|
| **Primary Endpoint** | **$0.395 / hour** | **$865.05 / month** (3-AZ) | Flat hourly charge per AZ |
| **Secondary Endpoint**| Discounted / hour| Reduced base fee | **$0.00 data processing** |
| **Data Processing** | **$0.065 / GB** | **$6.50** ($65.00 per TB) | Primary endpoint volume processed |
| **TLS Inspection** | Extra hourly rate | Applies per endpoint-hr | **No extra per-GB TLS surcharge** |
| **NAT Gateway Waiver**| **-$0.045 / hr & GB**| **1-to-1 fee credit** | Waives NAT GW fees when chained |

---

## 5. AWS Free Tier Coverage
* **AWS Network Firewall:** **No free tier** is available. Any provisioned endpoint generates hourly charges immediately.

---

## 6. Common Cost Hotspots & Pitfalls
* **Deploying Dedicated Primary Endpoints in Every Spoke VPC:** Creating individual primary Network Firewall endpoints inside every developer, staging, and production VPC ($865.05/mo base each).
* **Routing High-Volume Intra-VPC Traffic Through Firewall:** Configuring VPC route tables to send internal database replication, S3 backups, or video streaming traffic through Network Firewall at **$0.065/GB**.
* **Processing Unneeded S3 Endpoint Traffic:** Routing traffic destined for S3 or DynamoDB through Network Firewall instead of using free **S3 VPC Gateway Endpoints**.

---

## 7. Actionable Cost Optimization Strategies
1. **Adopt a Centralized Inspection VPC Architecture (AWS Transit Gateway):**
   * Do not deploy primary Network Firewall endpoints in individual spoke VPCs.
   * Create a single **Centralized Inspection VPC** containing 1 set of Network Firewall endpoints across 3 AZs ($865.05/month base).
   * Connect all spoke VPCs to the Centralized Inspection VPC using **AWS Transit Gateway** or secondary endpoint associations.
   * **The Savings:** Replaces dozens of scattered firewall endpoints, saving **$5,000.00 to $20,000.00+ per month** in idle endpoint fees.
2. **Leverage the NAT Gateway 1-to-1 Fee Waiver:**
   * Chain NAT Gateways directly with Network Firewall endpoints in your egress VPC route tables.
   * **The Savings:** AWS automatically waives the NAT Gateway hourly fee ($0.045/hr) and data processing fee ($0.045/GB) on a 1-to-1 basis against firewall usage, preventing double-billing on internet egress traffic.
3. **Filter Traffic Routes (Inspect Internet Egress Only):**
   * Update VPC route tables so that ONLY outbound internet traffic (`0.0.0.0/0`) is routed to the Network Firewall.
   * Bypass Network Firewall for trusted inter-VPC traffic and internal subnet traffic.
   * **The Savings:** Saves $0.065/GB ($65.00/TB) on high-volume internal data transfers.
4. **Use Free S3 & DynamoDB Gateway Endpoints:**
   * Ensure all VPCs have **S3 & DynamoDB Gateway VPC Endpoints** installed (which are 100% free).
   * Direct S3 traffic through Gateway Endpoints so it bypasses Network Firewall processing fees ($0.065/GB).
5. **Tear Down Non-Production Firewalls:** For temporary staging or development environments, use standard Security Groups and AWS WAF instead of running continuous $865/mo Network Firewall endpoints.
