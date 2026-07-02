# Network Connectivity Center (NCC) Cost Optimization

Google Cloud Network Connectivity Center (NCC) is a network connectivity management service that provides a single, unified hub-and-spoke model to connect VPCs, on-premises networks (via Cloud Interconnect, Cloud VPN, or SD-WAN Router Appliance), and remote offices. Because NCC introduces per-spoke hourly fees and data processing fees for VPC-to-VPC routing, keeping the network architecture lean is critical.

---

## 1. Network Connectivity Center Billing Mechanics

NCC charges are determined by three items:
1. **Hub Control Plane:** The NCC Hub itself is free of charge.
2. **Spoke Hourly Fee:** Billed per spoke connected to the hub per hour:
   * Spokes include VPCs, VLAN attachments, VPN tunnels, and Router Appliances.
   * Rate: Approx. $0.075 per spoke hour (approx. $54.00 per month per active spoke). The first 5 spokes are free in some tiers, but standard enterprise usage quickly exceeds this.
3. **Data Processing Fee:** Billed per GB of data transferred between spokes through the hub.

---

## 2. Core Cost-Reduction Levers

### A. Consolidate Spoke Connections
* **The Waste:** Creating separate NCC spokes for every subnet or staging VPC, resulting in many separate active spokes billing hourly.
* **Action:** Consolidate your VPC architecture. Instead of linking 20 separate development VPC spokes to the hub, combine them into a single Shared VPC containing multiple subnets, and attach that single Shared VPC as a single spoke.
* **The Benefit:** Saves hundreds of dollars monthly in spoke fees.

### B. Prune Stale router spokes & VPN connections
* When projects are archived, VPN tunnels or Router Appliances might be left registered as active spokes.
* **Action:** Run regular audits to verify that all spokes connected to the NCC Hub are actively routing network traffic. Remove any spokes associated with decommissioned offices, VPCs, or third-party integrations.

---

## 3. NCC Audit Checklist

1. [ ] **Spoke Optimization Sweep:** List active NCC spokes and identify consolidation candidates (e.g. migrating separate VPCs to a Shared VPC structure).
2. [ ] **Idle Spoke Deletion:** Identify and detach spokes with zero traffic throughput over the last 30 days.
3. [ ] **Data Transfer Analysis:** Audit data processing fees in GCP Billing to ensure traffic is routing through the most cost-effective paths.
