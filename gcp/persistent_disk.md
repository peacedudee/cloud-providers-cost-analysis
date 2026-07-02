# Persistent Disk (PD) Cost Optimization & Research

Persistent Disks (PD) are durable network storage devices attached to Google Compute Engine (GCE) VMs or Google Kubernetes Engine (GKE) nodes. Because PD is billed based on **provisioned capacity** rather than actual data stored, and because disks are easily orphaned during VM deletions, storage can become a primary source of hidden waste. This file details cost optimization strategies for Persistent Disks.

---

## 1. Persistent Disk Billing Mechanics

PD costs are calculated using three main vectors:
1. **Provisioned Capacity (GB/month):** You are billed for the size of the disk you request (e.g., a 100 GB disk costs the same monthly rate whether it contains 1 GB of data or 99 GB).
2. **Replication Model:**
   * **Zonal PD:** Replicated within a single zone.
   * **Regional PD:** Synchronously replicated across two zones in the same region. Regional PDs cost **exactly 2x** the price of Zonal PDs for the same capacity.
3. **Provisioned Performance (IOPS & Throughput):** Billed separately for high-performance disks like `pd-extreme` or Google's latest **Hyperdisk** series.

---

## 2. Disk Type Selection (Pricing & Performance)

Choosing the correct disk class is the most direct lever to reduce storage spend:

| Disk Type | Price Factor | Target Workload | Cost Strategy |
| :--- | :--- | :--- | :--- |
| **pd-standard** (HDD) | Lowest (approx. $0.040/GB) | Large sequential reads/writes, logging, warm archives. | Use for backup/logging directories. Avoid for boot disks. |
| **pd-balanced** (SSD) | Mid-low (approx. $0.100/GB) | General-purpose VM boot disks, web servers, dev/test databases. | **Default choice.** Migrating boot disks from `pd-ssd` to `pd-balanced` saves 40% immediately. |
| **pd-ssd** (Premium SSD) | Mid-high (approx. $0.170/GB) | High-performance databases, latency-critical applications. | Limit usage to core transactional databases. |
| **Hyperdisk Extreme / Balanced** | Performance-based pricing | Custom high-IOPS enterprise databases, SAP HANA. | Scale capacity and IOPS independently to avoid buying excess storage just for speed. |

---

## 3. Core Cost-Reduction Tactics

### A. Identify and Eliminate Zombie (Unattached) Disks
When VMs are deleted, attached persistent disks are frequently left intact (unless "Delete boot disk when instance is deleted" was checked).
* **The Waste:** These unattached disks continue to run up charges month after month.
* **Tactic:** Run a gcloud command or check Google Cloud Recommender daily for idle persistent disks.
  ```bash
  # List all unattached persistent disks in a project
  gcloud compute disks list --filter="users:-"
  ```
* **Action:** Automate a script to take a final snapshot of any disk unattached for > 14 days, then delete the disk.

### B. Downgrade from Regional to Zonal PDs
* Regional PDs provide active replication for disaster recovery. However, running regional PDs in development, staging, or testing environments is completely unnecessary.
* **Action:** Audit all disks in non-production projects. Recreate any `regional-pd` as a standard `zonal-pd` to cut the cost of that disk by **50%**.

### C. Right-Size Provisioned IOPS on Hyperdisks
* In older generations (`pd-ssd`), the only way to get more IOPS was to provision a larger disk size (e.g., you had to provision a 500 GB disk to get 15,000 IOPS, even if you only needed 50 GB of storage).
* **Hyperdisk Solution:** In modern GCP VMs, use **Hyperdisk**. You can provision a small storage capacity (e.g., 50 GB) and independently scale up the IOPS and throughput slider to get high performance without paying for blank storage space.

### D. Snapshot Retention Policies
* While snapshots are incremental and compressed, keeping a long history of daily snapshots of a database disk can grow to exceed the cost of the active disk itself.
* **Action:**
  * Enforce snapshot retention limits using **Snapshot Schedules** (e.g., retain daily snapshots for a maximum of 14 days).
  * Consolidate snapshot policies. Do not capture snapshots of temp directories or cache drives.

---

## 4. Persistent Disk Audit Checklist

1. [ ] **Unattached Disk Sweep:** Find and delete all persistent disks that are not attached to any VM.
2. [ ] **Regional Disk Audit:** Identify all regional PDs. Migrate non-prod regional disks to zonal disks.
3. [ ] **Boot Disk Optimization:** Migrate VM boot disks from `pd-ssd` to `pd-balanced` where applicable.
4. [ ] **Hyperdisk Tuning:** For VMs using Hyperdisks, review actual IOPS and throughput metrics in Cloud Monitoring and downscale provisioned performance to match actual peaks.
5. [ ] **Snapshot Schedule & Retention:** Audit all snapshots and verify that every snapshot has a strict lifecycle retention rule.
6. [ ] **Recommender Review:** Action all disk optimization recommendations in the Active Assist Console.
