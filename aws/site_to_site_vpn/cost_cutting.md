# Cost-Cutting Playbook: AWS Site-to-Site VPN
> **Companion File:** [site_to_site_vpn.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/site_to_site_vpn/site_to_site_vpn.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Site-to-Site VPN provides a secure, IPsec-encrypted connection between your AWS VPC and on-premises networks. While the service provides essential hybrid cloud connectivity, costs can easily spiral due to abandoned connections, unnecessary use of Accelerated VPN features, and high data egress volumes. This playbook details 20 actionable strategies to optimize your Site-to-Site VPN architecture, minimize redundant links, and drastically reduce data transfer charges.

## Strategy Categories

### 1. Waste Elimination
1. **S2SVPN-WE-001: Delete Inactive VPN Connections:** Audit the VPC Dashboard for VPN tunnels that have been continually in a `DOWN` state for more than 30 days. Deleting these saves the flat ~$36.50/month per connection.
2. **S2SVPN-WE-002: Terminate Orphaned Customer Gateways:** Remove Customer Gateways (CGWs) that are no longer attached to any active VPN connection to clean up infrastructure state.
3. **S2SVPN-WE-003: Remove Duplicate Tunnels for Test/Dev:** Identify non-production environments that have multiple redundant VPN links provisioned and scale them down to a single link or software alternative.
4. **S2SVPN-WE-004: Audit Post-Migration Infrastructure:** Ensure VPN connections used temporarily for data center migrations or bulk data loads are decommissioned once the migration is complete.

### 2. Rightsizing
5. **S2SVPN-RS-001: Disable Acceleration for Local/Regional Links:** Accelerated VPN adds a $0.025/hr base and $0.015/GB premium surcharge. Disable this feature if the on-premises gateway and the AWS region are in the same geographical area.
6. **S2SVPN-RS-002: Use Software-Based VPNs for Non-Critical Dev Links:** For non-production branch offices, evaluate running a software-based VPN appliance (e.g., OpenVPN on a `t3.micro` EC2 instance) instead of managed AWS VPN to lower base hourly costs.

### 3. Commitment Discounts
7. **S2SVPN-CD-001: Apply Savings Plans to EC2 Software VPNs:** If migrating to a software-based VPN on EC2, ensure the underlying compute instances are covered by Compute Savings Plans.
8. **S2SVPN-CD-002: Leverage EDP for Data Transfer:** Site-to-Site VPN egress contributes to total AWS Data Transfer Out. Ensure your total data transfer is factored into Enterprise Discount Program (EDP) negotiations to secure tiered volume discounts.

### 4. Architecture Changes
9. **S2SVPN-AC-001: Evaluate Direct Connect for High-Volume Egress:** If egress over VPN exceeds 20 TB/month ($1,800/mo), migrate to AWS Direct Connect where egress drops from $0.09/GB to $0.02/GB, generating substantial savings.
10. **S2SVPN-AC-002: Consolidate Connections via Transit Gateway:** Avoid creating a separate VPN connection for each VPC. Deploy a single Site-to-Site VPN to a Transit Gateway and route traffic efficiently to all attached VPCs.
11. **S2SVPN-AC-003: Replace VPN with VPC Peering for Inter-Region Links:** If using Site-to-Site VPN to connect two AWS environments across regions, replace it with VPC Peering or Transit Gateway Inter-Region Peering for lower latency and reduced costs.
12. **S2SVPN-AC-004: Implement SD-WAN Appliances:** For complex, multi-site routing architectures, evaluate third-party SD-WAN virtual appliances (e.g., Cisco, Fortinet) which can sometimes offer better data compression and routing efficiency than native AWS VPN.

### 5. Scheduling & Auto-Scaling
13. **S2SVPN-SA-001: Automate Provisioning via IaC:** Use Infrastructure as Code (Terraform/CloudFormation) to dynamically spin up Site-to-Site VPN connections only when required for specific temporary workflows, tearing them down immediately after.
14. **S2SVPN-SA-002: Schedule Software VPN Uptime:** If using EC2-based software VPNs for development environments, schedule the EC2 instances to shut down outside of business hours (e.g., nights and weekends) to save compute costs.

### 6. Pricing Model Optimization
15. **S2SVPN-PM-001: Monitor DT-Premium ROI:** Continuously monitor the Data Transfer-Premium (DT-Premium) costs associated with Accelerated VPNs. If the performance gains do not justify the cost markup, revert to Standard VPN.
16. **S2SVPN-PM-002: Optimize Multi-Cloud Connectivity:** When connecting AWS to other cloud providers (e.g., Azure, GCP), evaluate third-party interconnect fabrics (like Megaport or Equinix Fabric) rather than relying solely on high-cost internet-based VPN tunnels.

### 7. Network & Data Transfer Optimization
17. **S2SVPN-ND-001: Implement Data Compression:** Enable payload compression on your on-premises Customer Gateway router to reduce the raw volume of data egressing from AWS.
18. **S2SVPN-ND-002: Prevent Internet Hair-Pinning:** Ensure on-premises internet-bound traffic routes locally out of the on-premises ISP rather than traveling over the VPN to exit through an AWS NAT Gateway.
19. **S2SVPN-ND-003: Filter Broadcast/Multicast Traffic:** Configure strict ACLs on the Customer Gateway to ensure chatter, broadcast, or unnecessary multicast traffic is not routed over the VPN link.
20. **S2SVPN-ND-004: Local Data Caching:** Cache frequently accessed AWS resources (like static S3 assets or database read replicas) on-premises to minimize repetitive, costly data egress over the VPN.

---
## Cross-Service Synergies
* **AWS Direct Connect:** High VPN costs are the primary trigger for migrating to Direct Connect for reduced per-GB egress rates.
* **AWS Transit Gateway:** Consolidates multiple VPN connections into a single hub, significantly reducing connection hourly fees.
* **AWS Global Accelerator:** Powers the Accelerated VPN feature; optimizing VPNs reduces Global Accelerator surcharges.
* **Amazon EC2 / VPC:** Migrating to software-based VPNs increases EC2 footprint but lowers managed VPN hourly costs.

---
## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
* **LineItems/ProductCode:** `AmazonVPC`
* **LineItems/UsageType:** Look for `VPN-Usage-Hours`, `Global-Accelerator-Hours`, and `DataTransfer-Out-Bytes`.
* **LineItems/Operation:** Filter for `CreateVpnConnection`, `AcceleratedVPN`.

### B. CloudWatch Metrics
* `TunnelState`: To identify tunnels that have been continuously down (0).
* `TunnelDataOut`: To identify connections with zero or negligible egress data.
* `TunnelDataIn`: To identify connections with zero or negligible ingress data.

### C. AWS Config / Trusted Advisor
* **Trusted Advisor:** "Idle Virtual Private Network (VPN) Connections" check.
* **AWS Config:** Monitor changes to `AWS::EC2::VPNConnection` to track the creation of new links.

### D. Company Policies
* Network redundancy requirements (whether Dev/Test needs dual-tunnel high availability).
* Approved hybrid connectivity patterns (Direct Connect vs. VPN vs. SD-WAN).

### E. IaC (Optional)
* Terraform state files (`aws_vpn_connection`) to identify hardcoded connections that should be modularized.

---
## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "S2SVPN-WE-001",
  "resource_id": "vpn-0abcdef1234567890",
  "account_id": "123456789012",
  "region": "us-east-1",
  "strategy_category": "Waste Elimination",
  "status": "Inactive for 30+ days",
  "current_monthly_cost": 36.50,
  "projected_monthly_cost": 0.00,
  "estimated_savings_monthly": 36.50,
  "recommendation": "Delete the idle VPN connection.",
  "effort_level": "Low"
}
```

### Summary Report Table

| Finding ID | Strategy Category | Target Resource | Action | Est. Monthly Savings | Effort |
|------------|-------------------|-----------------|--------|----------------------|--------|
| S2SVPN-WE-001 | Waste Elimination | `vpn-0abc...` | Delete inactive VPN link | $36.50 | Low |
| S2SVPN-RS-001 | Rightsizing | `vpn-0def...` | Disable Acceleration | $85.00 | Low |
| S2SVPN-AC-001 | Architecture Changes | All VPNs | Migrate >20TB/mo to Direct Connect | $1,300.00 | High |
| S2SVPN-AC-002 | Architecture Changes | Multiple VPNs | Consolidate behind Transit Gateway | $146.00 | Medium |
| S2SVPN-ND-002 | Network Optimization | On-Prem Router | Stop internet traffic hair-pinning | $250.00 | Medium |
