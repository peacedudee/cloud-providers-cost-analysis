# Bare Metal Solution (BMS) Cost Optimization & Research

Google Cloud Bare Metal Solution (BMS) provides dedicated, bare-metal servers located in Google-adjacent colocation facilities with sub-millisecond latency to GCP resources. BMS is primarily used for running legacy, licensing-heavy enterprise applications like Oracle Databases, SAP, or specialized Windows workloads that cannot run on hypervisors. Because BMS is a subscription-based, hardware-hosting model, cost management differs significantly from standard serverless cloud components.

---

## 1. BMS Billing Mechanics

BMS pricing is built around long-term subscriptions rather than pay-as-you-go usage:
1. **Server Subscriptions:** Monthly fee based on the selected server hardware configuration (CPU, RAM, architecture, e.g., o2-standard-16, o2-medium-32). Requires a **1-year or 3-year commitment**. There is no scale-to-zero or hour-by-hour billing.
2. **Dedicated Storage (SAN):** Billed per GB/month for SAN-attached SSD or HDD storage provisioned for the bare-metal servers.
3. **Networking Egress/Bandwidth:** Standard data transfer rates apply when transferring data out of the BMS environment. Connection to GCP is routed through Partner Interconnects.

---

## 2. Core Cost-Optimization Levers

### A. Rigorous Capacity Planning & Multi-Tenancy
Since BMS servers are billed on a fixed monthly subscription basis for the entire host chassis:
* **The Waste:** Provisioning a large bare metal server for a single application that only uses 10% of the host's capacity.
* **The Solution (Virtualization & Consolidation):**
  * Consolidate multiple database instances (e.g., Oracle databases, SQL Server) onto a single physical BMS server.
  * Use hypervisors/virtualization layers compatible with your licensing (e.g., Oracle VM (OVM), VMware OVM, or physical partitioning) to run multiple virtual machines on the same physical bare metal node.
  * Pack workloads to maintain host CPU and memory utilization above 70-80%.

### B. Storage Optimization & Snapshot Policies
BMS SAN storage is provisioned and billed per GB:
* **Tactic:** Implement strict storage volume controls. Do not over-provision storage sizes initially; SAN volumes can be expanded as needed.
* **Snapshot Lifecycle:** Use local database-level snapshots or SAN snapshot features for recovery points, but enforce strict snapshot retention rules (e.g., delete snapshots older than 7 days) to prevent SAN storage billing bloat.
* **Archive to GCS:** Migrate historical database backups, logs, and exports off the expensive BMS SAN storage into **Cloud Storage (GCS) Archive** class. Data transfer between BMS and GCS is local and fast, and GCS Archive storage is a fraction of the SAN cost.

### C. Partner Interconnect and Egress Reduction
* BMS connects to VPC networks via dedicated Partner Interconnect links.
* **Tactic:** Compress data backups and batches before sending them from BMS to external environments or other cloud regions to minimize egress and bandwidth consumption charges.

---

## 3. BMS Audit Checklist

1. [ ] **Hardware Consolidation Review:** Audit CPU/Memory utilization across all BMS hosts. Identify underutilized servers that can be consolidated using virtualization layers.
2. [ ] **SAN Storage Utilization:** Verify that provisioned SAN storage is not sitting empty. Downsize future allocations.
3. [ ] **Snapshot Retention Policy:** Ensure SAN snapshot retention scripts are active and aging out old backups.
4. [ ] **Archive to GCS:** Move database exports and historical cold backups from SAN storage to GCS Archive storage.
5. [ ] **Subscription Renewal Auditing:** Review subscription end-dates. Plan workload migrations well in advance of the 1-year or 3-year renewal windows to avoid locked-in hardware costs for decommissioned apps.
