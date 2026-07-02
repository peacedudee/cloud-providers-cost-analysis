# Cloud Interconnect Cost Optimization & Research

Google Cloud Interconnect provides direct physical connections between your on-premises network and Google Cloud's network. It is available as **Dedicated Interconnect** (direct fiber connection to a Google colocation facility) or **Partner Interconnect** (connection through a third-party service provider). While Interconnect requires upfront infrastructure spend, it offers heavily discounted egress rates compared to internet or VPN egress.

---

## 1. Interconnect Billing Dimensions

Cloud Interconnect charges are calculated based on three components:
1. **Port Hourly Charge:** Billed for the physical port connection:
   * **Dedicated Interconnect:** Billed per port hour for 10 Gbps or 100 Gbps circuits.
   * **Partner Interconnect:** Billed per connection capacity (ranging from 50 Mbps up to 10 Gbps).
2. **VLAN Attachment Hourly Charge:** Billed per VLAN attachment (interconnect connection between VPC and port) per hour.
3. **Data Egress (Outbound Data):** Billed per GB for data leaving GCP over the Interconnect link. Egress rates over Interconnect are **up to 70% cheaper** than standard internet or VPN egress rates.

---

## 2. Core Cost-Optimization Levers

### A. The VPN to Interconnect Breakeven Analysis
* **The Cost Difference:** Egress over VPN costs standard internet egress rates (~$0.08/GB for US). Egress over Dedicated or Partner Interconnect is deeply discounted (often up to 50-70% cheaper than internet egress). Note that CDN Interconnect and Peering egress rates are increasing to $0.08/GB in North America by May 1, 2026.
* **Tactic:** Perform a monthly data volume analysis. If your company transfers more than **10–15 TB of data per month** from GCP to your on-premises data centers over VPN, the egress savings of Interconnect will completely offset the monthly port fees, yielding overall savings.

### B. Consolidate VLAN Attachments
* **The Waste:** Provisioning separate VLAN attachments for every VPC or project in your organization. You are billed for each attachment hour.
* **The Solution:** Use **Shared VPC** or **Network Connectivity Center (NCC)**.
* **Action:** Terminate the Interconnect VLAN attachments in a central "Hub VPC" and route traffic from other spoke VPCs internally. This minimizes VLAN attachment fees and simplifies IP routing.

### C. Right-Size Partner Interconnect Bandwidth
* Unlike Dedicated ports which are fixed, Partner Interconnect allows you to provision bandwidth on-demand (e.g. 100 Mbps, 500 Mbps, 1 Gbps, 10 Gbps).
* **Action:** Monitor peak network throughput in Cloud Monitoring. Downsize your Partner Interconnect connection speed during periods of low activity (e.g. staging environments or dev projects), as you are billed for the provisioned speed, not consumed data.

---

## 3. Cloud Interconnect Audit Checklist

1. [ ] **Egress Volume Audit:** Audit monthly VPN egress data volume. Flag accounts exceeding 10 TB/month for Interconnect migration.
2. [ ] **VLAN Attachment Consolidation:** Sweep projects and identify duplicate VLAN attachments that can be consolidated under a central Shared VPC.
3. [ ] **Partner Bandwidth Tuning:** Review peak throughput on Partner Interconnect links and downscale provisioned limits to match actual needs.
4. [ ] **Peering Rate Checks:** Verify interconnect egress accounting following the May 1, 2026 peering price adjustments.
