# Cloud VPN Cost Optimization & Research

Google Cloud VPN securely connects your peer network (on-premises or other clouds) to your Google Cloud Virtual Private Cloud (VPC) network through an IPsec VPN connection. While VPN is a cost-effective alternative to Cloud Interconnect, idle tunnels and unmonitored egress routing can lead to unnecessary spending.

---

## 1. Cloud VPN Billing Mechanics

Cloud VPN pricing is composed of two main items:
1. **Tunnel Hours:** Billed hourly per active tunnel (approx. $0.05 per tunnel hour, or ~$36.50/month per tunnel).
   * **HA (High Availability) VPN:** Google's SLA requires provisioning **two active tunnels** for HA routing, doubling the base rate to **~$73.00/month** per HA connection.
2. **Data Transfer (Network Egress):** Billed per GB of data sent outbound through the VPN tunnel. Data transfer into GCP (ingress) is free. Egress rates match standard GCP internet egress pricing ($0.08–$0.12/GB).

---

## 2. Core Cost-Optimization Levers

### A. Decommission Idle/Stale Tunnels
* **The Waste:** Teams create VPN tunnels to migrate datasets or run short-term integration tests with on-premises databases, and then leave the tunnels active for months after the test completes.
* **Action:** Audit active VPN tunnels in the GCP Console. Delete any tunnel that shows zero packet traffic over the last 14 days.
  ```bash
  # List all active VPN tunnels in a project
  gcloud compute vpn-tunnels list
  ```

### B. Use Single-Tunnel Classic VPN for Non-Prod
* For production environments, High Availability (HA) VPN (requiring 2 tunnels) is mandatory to receive Google's 99.99% availability SLA.
* **Action:** In development, sandbox, or test projects where high availability is not required, use **Classic VPN** with a **single tunnel** instead of HA VPN.
* **The Benefit:** Cuts the baseline tunnel hosting charges by **50%**.

### C. Prevent Hairpinning Egress
* **The Cost Trap (Hairpinning):** If routing is configured poorly, private VMs calling Google Cloud services (like downloading backups from Cloud Storage or pulling images from Artifact Registry) might route their traffic through the VPN to your on-premises gateway, which then routes it back to Google. This is called "hairpinning."
* **The Cost:** You pay for egress over the VPN, egress from your on-premises router, and NAT/internet costs, multiplying fees.
* **Action:** Always enable **Private Google Access** on your VPC subnets. This forces all calls to Google APIs to stay entirely within Google's internal network, completely bypassing the VPN tunnel.

---

## 3. Cloud VPN Audit Checklist

1. [ ] **Idle Tunnel Purge:** Locate and delete VPN tunnels showing zero traffic over the last 2 weeks.
2. [ ] **Classic VPN for Dev:** Confirm that dev/test connectivity uses single-tunnel Classic VPNs where SLA compliance is not required.
3. [ ] **Hairpinning Audit:** Check routing tables to ensure Google API traffic (`*.googleapis.com`) does not route through the VPN gateway.
4. [ ] **Interconnect Migration Evaluation:** If your VPN egress data volume consistently exceeds 10 TB/month, calculate whether migrating to **Cloud Interconnect** (which has higher base port fees but significantly discounted egress rates) would yield a lower total cost.
