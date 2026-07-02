# Google Cloud VMware Engine (GCVE) Cost Optimization

Google Cloud VMware Engine (GCVE) allows organizations to migrate VMware workloads to Google Cloud as-is, running on dedicated bare-metal hypervisor nodes. Because GCVE clusters require a **minimum of 3 nodes** to establish a VMware vSAN quorum, the entry-level baseline cost is high (often thousands of dollars per month). Optimizing VM packing, storage provisioning, and pricing commitments is critical to control spend.

---

## 1. GCVE Billing Mechanics

GCVE is billed based on physical node hours:
1. **Node Instances:** Billed hourly per node (e.g. ve1-standard-72 nodes). You pay for the entire physical node capacity (CPU, RAM, local NVMe storage).
2. **Cluster Minimums:** Production VMware clusters require a minimum of **3 nodes** (stretched clusters require more).
3. **Commitment Discounts:** GCVE is highly suited for **1-year and 3-year Committed Use Discounts (CUDs)**, which offer discounts up to 50% compared to on-demand hourly rates.

---

## 2. Core Cost-Optimization Levers

### A. Apply Compute Oversubscription
Because GCVE nodes are dedicated physical servers, you control the hypervisor CPU and memory allocation ratios.
* **The Action:** Oversubscribe compute resources on your ESXi hosts.
* **Ratios:** A vCPU-to-physical-core oversubscription ratio of **4:1** is standard for general enterprise workloads. For development and testing environments, this can be increased up to **8:1** safely.
* **The Benefit:** Allows you to pack more VMs onto your existing GCVE hosts, preventing the need to spin up additional expensive bare-metal nodes.

### B. Enforce Thin Provisioning on VMware vSAN
* **Thick Provisioning:** Allocates and reserves the entire disk space for a VM immediately upon creation (e.g., creating a 200 GB VM disk consumes 200 GB of vSAN capacity immediately).
* **Thin Provisioning:** Consumes storage dynamically as files are written.
* **Action:** Enforce a policy requiring **Thin Provisioning** for all VM storage policies in vCenter.
* **The Benefit:** Saves massive amounts of vSAN storage space. Because GCVE storage is bundled with nodes, running out of vSAN space forces you to add entire new nodes (compute + storage) to the cluster, which is highly expensive. Keep vSAN usage lean to avoid storage-forced node scaling.

### C. Leverage 1-Year or 3-Year CUDs
* Due to the high entry-level cost and long-term nature of VMware migrations, running GCVE at on-demand hourly rates should be strictly temporary (e.g., only during the migration testing phase).
* **Action:** Commit to 1-year or 3-year GCVE CUDs as soon as baseline migration volumes are validated.

### D. Delete Zombie VMs and Orphaned Backups
* In VMware environments, virtual machines are easily orphaned or abandoned by teams.
* **Action:** Run regular audits using VMware vRealize Operations (vROps) or native vCenter reporting to find idle, powered-off, or zombie VMs. Delete them to reclaim vSAN storage capacity.

---

## 3. GCVE Audit Checklist

1. [ ] **Oversubscription Check:** Review ESXi host CPU/Memory oversubscription ratios. Increase ratios for non-production clusters.
2. [ ] **Storage Policy Verification:** Confirm that all VM storage policies in vCenter use "Thin Provisioning."
3. [ ] **Zombie VM Cleanup:** Audit vCenter for powered-off or idle VMs and decommission them to free up vSAN space.
4. [ ] **Commitment Coverage:** Verify that 1-year or 3-year GCVE CUDs are active and cover the baseline node count.
5. [ ] **Log Retention in GCVE:** Ensure VMware log forwarding (vRealize Log Insight) to Cloud Logging has active filters to prevent duplicate ingestion charges.
