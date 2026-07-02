# AWS Service Cost Research: AWS Network Firewall

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Network Firewall is a stateful, managed network firewall and intrusion detection and prevention service (IDS/IPS) for your Virtual Private Clouds (VPCs). It provides fine-grained network protection—including stateful inspection, protocol detection, domain name filtering, and Snort-compatible IPS rule evaluation. Network Firewall operates on a provisioned endpoint and data processing model. Because firewall endpoints incur high hourly fees per AZ, unoptimized VPC topologies create massive baseline costs.

---

## 2. Billing Mechanics
AWS Network Firewall billing is calculated across two primary dimensions per Availability Zone (AZ):
1.  **Firewall Endpoint Hourly Fee:** Billed hourly per provisioned firewall endpoint in each Availability Zone.
2.  **Data Processing Fee:** Billed per GB of traffic processed through the firewall endpoints.
3.  **Advanced Inspection Surcharges (TLS Inspection):** Hourly surcharges apply when TLS decryption is enabled.

---

## 3. Key Cost Dimensions

### A. Core Pricing (us-east-1 Rates)
*   **Firewall Endpoint Hourly Fee:** **$0.395 per firewall endpoint per hour** in `us-east-1`.
    *   *Single AZ:* $0.395 × 730 hours = **$288.35 per month**.
    *   *Production 3-AZ High Availability (3 Endpoints):*
        $$3\text{ AZs} \times \$288.35 = \$865.05\text{ / month flat baseline fee per VPC}$$
*   **Data Processing Fee:** **$0.065 per GB** of data processed ($65.00 per TB).

### B. The Spoke VPC Endpoint Hazard (Architectural Trap)
*   If an organization deploys dedicated Network Firewall endpoints inside **10 separate VPCs** across 3 AZs:
    $$\text{Endpoints: } 10\text{ VPCs} \times 3\text{ AZs} = 30\text{ endpoints}$$
    $$\text{Baseline Fee: } 30 \times \$288.35 = \$8,650.50\text{ / month in idle endpoint fees alone!}$$

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Component | Rate (us-east-1) | Price for 100 GB / 3-AZ Month | Notes |
|-------------------|------------------|-------------------------------|-------|
| **Firewall Endpoint** | **$0.395 / hour** | **$865.05 / month** (3-AZ) | Flat hourly charge per AZ |
| **Data Processing** | **$0.065 / GB** | **$6.50** ($65.00 per TB) | Volume processed |
| **TLS Inspection** | Extra hourly rate | Applies per endpoint-hr | When TLS decryption enabled |

---

## 5. AWS Free Tier Coverage
*   **AWS Network Firewall:** **No free tier** is available. Any provisioned endpoint generates hourly charges immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Deploying Dedicated Endpoints in Every Spoke VPC:** Creating individual Network Firewall endpoints inside every developer, staging, and production VPC.
*   **Routing High-Volume Intra-VPC Traffic Through Firewall:** Configuring VPC route tables to send internal database replication, S3 backups, or video streaming traffic through Network Firewall at **$0.065/GB**.
*   **Processing Unneeded S3 Endpoint Traffic:** Routing traffic destined for S3 or DynamoDB through Network Firewall instead of using free **S3 VPC Gateway Endpoints**.

---

## 7. Actionable Cost Optimization Strategies
1.  **Adopt a Centralized Inspection VPC Architecture (AWS Transit Gateway):**
    *   Do not deploy Network Firewall endpoints in individual spoke VPCs.
    *   Create a single **Centralized Inspection VPC** containing 1 set of Network Firewall endpoints across 3 AZs ($865.05/month base).
    *   Connect all spoke VPCs to the Centralized Inspection VPC using **AWS Transit Gateway**.
    *   **The Savings:** Replaces dozens of scattered firewall endpoints, saving **$5,000.00 to $20,000.00+ per month** in idle endpoint fees.
2.  **Filter Traffic Routes (Inspect Internet Egress Only):**
    *   Update VPC route tables so that ONLY outbound internet traffic (`0.0.0.0/0`) is routed to the Network Firewall.
    *   Bypass Network Firewall for trusted inter-VPC traffic and internal subnet traffic.
    *   **The Savings:** Saves $0.065/GB ($65.00/TB) on high-volume internal data transfers.
3.  **Use Free S3 & DynamoDB Gateway Endpoints:**
    *   Ensure all VPCs have **S3 & DynamoDB Gateway VPC Endpoints** installed (which are 100% free).
    *   Direct S3 traffic through Gateway Endpoints so it bypasses Network Firewall processing fees ($0.065/GB).
4.  **Tear Down Non-Production Firewalls:** For temporary staging or development environments, use standard Security Groups and AWS WAF instead of running continuous $865/mo Network Firewall endpoints.
