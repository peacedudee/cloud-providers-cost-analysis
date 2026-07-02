# Sole-Tenant Nodes Cost Optimization & Research

Google Cloud Sole-Tenant Nodes provide dedicated physical Compute Engine servers solely for your project's VMs. This service is primarily used to meet strict security and compliance requirements (physical isolation) or to optimize software licensing costs (BYOL - Bring Your Own License for Windows Server, SQL Server, etc.). While dedicated hardware incurs a premium, proper node management and CPU overcommit can make sole tenancy highly cost-effective.

---

## 1. Sole-Tenant Node Billing Mechanics

Sole-Tenant Nodes are billed based on the physical host configuration:
1. **Host hourly rate:** You pay for the entire physical node (e.g. `n2-node-80-640` with 80 vCPUs and 640 GB RAM) plus a **10% sole-tenancy premium surcharge**.
2. **Commitment Options:** Both the host compute resources and the 10% tenancy premium are eligible for **1-year and 3-year Committed Use Discounts (CUDs)**, reducing rates by up to 50-60%.
3. **VM Licensing:** Since you control the physical host, you can choose:
   * **Pay-as-you-go licenses:** Google bills OS licenses (like Windows) per VM vCPU.
   * **BYOL (Bring Your Own License):** You use your existing licenses (e.g., Microsoft Software Assurance), paying $0 to Google for OS licensing.

---

## 2. Core Cost-Optimization Levers

### A. Leverage CPU Overcommit (Pack Nodes)
By default, you can only allocate as many VM vCPUs as there are physical cores on the host. However, Sole-Tenant nodes allow you to **overcommit CPUs**.
* **How it works:** You can configure a sole-tenant node group to allow CPU overcommit up to **2.0x** (allocating up to 160 vCPUs of VMs on an 80 vCPU physical node).
* **The Benefit:** Since VMs rarely run at 100% CPU simultaneously, overcommitting allows you to pack double the number of VMs onto the same host, effectively cutting your hosting and hardware-tenancy costs in half.
* **Best Practice:** Use overcommit for development, staging, or environments with variable, non-synchronized CPU spikes (e.g. CI/CD runners, dev workstations). Avoid overcommitting RAM (memory cannot be overcommitted).

### B. Bring Your Own License (BYOL) for Windows and SQL Server
Using Google's Windows Server license templates bills you per vCPU-hour, which is extremely expensive for large-scale enterprise clusters.
* **Action:** Export and upload your on-premises Windows Server or SQL Server VM images and configure them to run under **BYOL node affinity**.
* **The Benefit:** Eliminates the per-hour licensing fee, utilizing your existing corporate licensing investments.

### C. Maximize Node Packing (Minimize Idle Hosts)
You pay for the physical node whether it contains one tiny VM or is 100% full.
* **Tactic:** Configure **Node Affinity Policies** to force VMs to pack onto as few physical hosts as possible. Use auto-restart and auto-migrate policies.
* **Action:** Consolidate VMs with similar compliance requirements onto the same node group instead of spreading them across multiple separate node groups.

---

## 3. Sole-Tenant Node Audit Checklist

1. [ ] **CPU Overcommit Sweep:** Check node groups for CPU overcommit configuration. Enable overcommit (e.g., 1.5x to 2.0x) on all non-prod node groups.
2. [ ] **BYOL Verification:** Audit Windows and SQL Server VMs running on sole-tenant nodes. Confirm they use BYOL licenses rather than Google pay-as-you-go licenses.
3. [ ] **Node Packing Audit:** Review VM distribution across physical hosts. Consolidate sparse nodes to downscale the overall physical host count.
4. [ ] **CUD Alignment:** Verify that 1-year or 3-year Sole-Tenant CUDs cover your baseline node group requirements (including the 10% tenancy premium).
5. [ ] **Taint and Affinity Review:** Ensure node affinity rules are not overly restrictive, which can prevent VMs from sharing nodes and force new hosts to spin up.
