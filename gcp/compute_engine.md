# Compute Engine (GCE) Cost Optimization & Research

Compute Engine (GCE) is the foundation of infrastructure-as-a-service (IaaS) on Google Cloud and is typically the largest contributor to a cloud bill. This file contains a comprehensive breakdown of GCE billing models, cost-reduction tactics, hidden trap costs, and auditing checklists.

---

## 1. Billing Mechanics & Pricing Models

Understanding how Google bills for GCE is the key to identifying cost-saving opportunities.

### Second-by-Second Billing
* GCE VMs are billed on a **per-second basis** with a **1-minute minimum**. If a VM runs for 20 seconds, it is billed for 60 seconds. Beyond 1 minute, billing is fractional to the second.
* **Stopped VMs:** When a VM is in the `TERMINATED` state, you stop paying for vCPU, RAM, and GPUs. However, you **continue paying** for the attached Persistent Disk (and any premium OS license fees if applicable, though generally OS fees are also suspended).

### Machine Families
GCE offers several machine families optimized for different workloads:
1. **General Purpose (E2, N2, N2D, N4, Tau T2D):** Best for web servers, databases, and general workloads.
   * **E2:** Extremely cost-effective. Uses shared physical resources (no SUDs, but lower baseline price).
   * **N4:** The latest general-purpose family offering excellent price-performance ratios.
   * **Tau T2D:** Scale-out optimized AMD VMs with excellent price-performance.
   * **Axion (C4A):** Google's custom Arm-based CPU offering up to 20-40% better price-performance than comparable x86 VMs.
2. **Compute Optimized (C2, C2D, C3, C4):** High-frequency processors. Best for high-performance computing (HPC) and gaming.
3. **Memory Optimized (M1, M2, M3):** Huge RAM-to-CPU ratios (up to 12TB RAM). Billed at a high premium.
4. **Accelerator Optimized (A2, A3, G2):** VMs with attached NVIDIA GPUs (A100, H100, L4).

### Discount Structures
* **Sustained Use Discounts (SUDs):**
  * Automatic discounts of up to 30% that apply when you run a VM for more than 25% of a billing month.
  * **Caution:** SUDs do *not* apply to newer VM families like E2, N4, C3, C4, or Spot VMs.
* **Committed Use Discounts (CUDs):**
  * You commit to a contract (1-year or 3-year) in exchange for significant discounts (up to 70% off on compute, up to 65% off on GPUs).
  * **Resource-based CUDs:** You commit to a specific amount of vCPU/RAM in a specific region and machine family. Inflexible but offers the highest discount.
  * **Flexible/Spend-based CUDs:** You commit to a minimum hourly spend (e.g., $10/hour) across multiple VM families and regions. Highly recommended for evolving architectures.
* **Spot VMs (formerly Preemptible VMs):**
  * Excess Compute Engine capacity offered at discounts up to 91%.
  * **Risk:** Google can terminate Spot VMs at any time with a 30-second warning if it needs the capacity back. Billed per second.

---

## 2. Core Cost-Reduction Tactics

### A. Continuous Right-Sizing
Most workloads run on VMs that are severely over-provisioned.
* **Target Metric:** Average CPU utilization under 10-15% and RAM utilization under 30% are prime candidates for rightsizing.
* **Use Custom Machine Types:** If a standard VM (e.g., `n2-standard-4` with 4 vCPUs / 16 GB RAM) is constrained only by memory but CPU is idle, migrate to a Custom Machine Type (e.g., `n2-custom-2-12` with 2 vCPUs / 12 GB RAM) to save on unused vCPU charges.
* **Upgrade to New-Gen Families:**
  * Upgrading `N1` to `N2` or `E2` typically yields immediate savings.
  * Migrating x86 workloads to **Axion (Arm-based)** instances yields substantial savings per core.

### B. Automated Off-Hours Scheduling
Development, testing, and staging VMs do not need to run 24/7.
* **The Math:** A VM running 24/7 = 730 hours/month. A VM running 9-to-5 on weekdays = 176 hours/month (a **76% cost reduction**).
* **Implementation:**
  * Use GCE **Instance Schedule resource policies** to automatically start and stop VMs at specific times (e.g., start at 8:00 AM, stop at 6:00 PM, skip weekends).
  * Alternatively, write lightweight Cloud Functions triggered by Pub/Sub (Cloud Scheduler) to stop/start VMs across projects.

### C. Spot VM Integration
Identify workloads that are stateless, batch-oriented, or fault-tolerant.
* **Ideal Candidates:** CI/CD runners, rendering queues, stateless microservices behind a load balancer, big data processing nodes (Dataproc/Spark).
* **Implementation:** Convert non-critical workloads to Spot VMs to immediately slash compute costs by up to 91%.

### D. Committed Use Discount (CUD) Planning
Never pay on-demand prices for steady-state workloads.
1. Analyze your baseline minimum GCE usage over the last 3 months.
2. Buy **Flexible CUDs** to cover 70-80% of this baseline (leaving 20-30% buffer for scaling and Spot usage).
3. Review and purchase monthly to adjust to changing requirements.

---

## 3. Storage & Disk Cost Optimization

Persistent Disks (PD) can silently bloat your bill, even if the VM hosting them is stopped.

### A. Orphaned (Zombie) Disks
When deleting a GCE VM, there is a checkbox option to "Delete persistent disk when instance is deleted." If unchecked, the disk remains as an active billing resource.
* **Tactic:** Run script/query to find disks where `status` is `READY` but `users` is empty. Delete or snapshot-and-delete these disks.

### B. Choose the Right Disk Tier
* **pd-standard (HDD):** $0.040/GB/month. Best for cold storage, logging directories, or archive databases.
* **pd-balanced (SSD-lite):** $0.100/GB/month. Standard boot disk choice. 100% adequate for most production workloads at a 40% savings compared to `pd-ssd`.
* **pd-ssd:** $0.170/GB/month. Best for high-throughput databases.
* **Tactic:** Audit boot disks and swap `pd-ssd` to `pd-balanced` where high IOPS is not required.

### C. Shrink Over-Provisioned Volumes
Disks are often provisioned to handle peak storage requirements but remain 90% empty.
* **Warning:** Google Cloud allows you to expand a disk online, but **does not support shrinking a disk**.
* **Workaround:**
  1. Create a new, smaller disk.
  2. Mount it to the VM.
  3. Rsync/copy data from the old disk to the new one.
  4. Swap mount paths, unmount the old disk, and delete it.

### D. Snapshot Retention Policies
Old snapshots accumulate and generate high storage costs.
* **Tactic:** Implement **Snapshot Schedules** with strict retention limits (e.g., delete snapshots older than 14 days).

---

## 4. Hidden & Trap Costs

Keep an eye out for these subtle charges that accumulate:

* **Unused Static External IP Addresses:**
  * Google charges a premium for static external IP addresses that are reserved but **not** attached to an active VM. Billed to prevent IP hoarding.
  * **Tactic:** Always release unused static IPs.
* **Inter-Zone Data Egress:**
  * Sending data between two VMs in different zones of the *same* region costs $0.01 per GB for egress (outbound). Ingress is free.
  * **Tactic:** Place high-chatter microservices/nodes in the same zone or use regional load balancing with zone-affinity.
* **Premium OS Licenses:**
  * Windows Server, SQL Server, and Enterprise Linux (RHEL/SLES) licenses are billed per vCPU-hour.
  * **Tactic:** Migrate databases and applications to Linux (Rocky/Debian) or utilize containerization (GKE/Cloud Run running Linux containers) to eliminate OS licensing costs.

---

## 5. FinOps Audit & Automation Checklist

Use this checklist during cost audits:

1. [ ] **Active Assist Recommender:** Query Google Cloud Recommender daily/weekly for "Underutilized VM" and "Idle VM" recommendations.
2. [ ] **Unattached Disks:** Search for and delete unattached Persistent Disks.
3. [ ] **Unused Static IPs:** Query and delete reserved external IPs that are unattached.
4. [ ] **Dev/Test Schedules:** Ensure all non-production instances have active Instance Schedules.
5. [ ] **Disk Tiering:** Migrate Boot Disks from `pd-ssd` to `pd-balanced`.
6. [ ] **Axion Arm Migration:** Identify instances running open-source stacks (Python, Node, Java, Go) that can be easily compiled for Arm and run on Axion instances.
7. [ ] **Labeling Enforcement:** Ensure all VMs have tags for `owner`, `environment` (prod/dev/stage), and `cost-center`.
