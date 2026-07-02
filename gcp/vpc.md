# Virtual Private Cloud (VPC) Cost Optimization & Research

Google Cloud Virtual Private Cloud (VPC) provides managed networking for resources. While the VPC network itself is completely free, associated networking features—such as firewall rules, VPC flow logs, external IP addresses, and data transfer paths—are major drivers of network spend. Managing these auxiliary costs is the foundation of network FinOps.

---

## 1. VPC Billing Components

VPC usage itself does not carry a base fee, but you are billed for:
1. **External IP Addresses:** Hourly fees for static and ephemeral external IP addresses. Billed higher if reserved but unattached (idle) to prevent hoarding.
2. **VPC Flow Logs:** Billed per GB of network packet metadata generated and routed to Cloud Logging.
3. **Cloud Firewall Rules:**
   * **Standard Firewall Rules:** Free.
   * **Advanced Cloud Firewall (Standard/Plus Tiers):** Billed per GB of data inspected or per rule-month (for advanced features like Threat Intelligence or FQDN filtering).
4. **Data Transfer (Network Egress):** Billed when traffic crosses availability zones ($0.01/GB), regions ($0.01–$0.02/GB), or leaves Google's network to the internet ($0.08–$0.23/GB).

---

## 2. Core Cost-Optimization Levers

### A. Strict External IP Address Reclamation
* **The Waste:** Teams reserve static external IP addresses for testing, delete the VM, and leave the IP address active. Google bills for idle reserved external IPs to encourage reclamation.
* **Action:** Run a regular sweep to locate and delete unattached static IP addresses.
  ```bash
  # List all reserved but unused external IP addresses
  gcloud compute addresses list --filter="status=RESERVED AND users:-"
  ```

### B. Optimize VPC Flow Logs (Sampling and Filtering)
VPC Flow Logs record network traffic flows between VMs. For high-traffic networks, logging every network packet flow generates terabytes of logs.
* **The Cost Trap:** Flow logs are ingested directly into Cloud Logging at **$0.25/GB** (the reduced vended network log rate, updated late 2024). Even at this reduced rate, massive scale can cause cost spikes.
* **Action:**
  1. **Disable in Non-Production:** Turn off VPC Flow Logs in all dev, staging, and sandbox environments.
  2. **Enable Log Sampling:** In production, do not set sampling to 100%. Set the sampling rate parameter to **0.01** (1%) or **0.05** (5%).
  3. **Filter to Metadata Only:** Exclude raw packet logging and capture only metadata (source/destination IP, port, protocol).
  4. **Filter on Rejections:** Configure flow logs to only record `REJECT` traffic (to log firewall blockages for security audits) rather than all successful `ACCEPT` traffic.

### C. Utilize Standard Firewall Rules over Advanced Tiers
* **The Trap:** Upgrading the VPC to Cloud Firewall Standard or Plus tiers globally, which charges for advanced rules or per-GB data inspection.
* **Action:** Use GCP's native, free **Standard Firewall Rules** for standard layer-3 and layer-4 IP/port blocking. Restrict the usage of advanced FQDN or Threat Intelligence rules to specific public-facing ingress subnets rather than applying them VPC-wide.

### D. Multi-Zone Resource Co-location (Internal Egress)
* **The Waste:** High-volume databases and compute clients running in different zones of the same region (e.g. database in `us-central1-a` and VM in `us-central1-f`). Data transfer between them costs $0.01 per GB in each direction.
* **Action:** Use zone affinity to deploy chatty microservice architectures in the same zone. Reserve multi-zone deployments strictly for high-availability production workloads.

---

## 3. VPC Audit Checklist

1. [ ] **Unattached IP Sweep:** Confirm all reserved static IPs are attached; delete idle ones.
2. [ ] **Flow Logs Configuration:** Review VPC Flow Logs settings. Set sampling rate to <= 5% and verify that non-prod networks have flow logs disabled.
3. [ ] **Firewall Tier Check:** Confirm that you are not paying for advanced Firewall tiers on VPCs that only require basic IP/port filtering.
4. [ ] **Private Google Access:** Ensure Private Google Access is enabled on all subnets to route API traffic internally.
