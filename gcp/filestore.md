# Cloud Filestore Cost Optimization & Research

Google Cloud Filestore is a managed Network Attached Storage (NAS) service offering Network File System (NFS) storage for Google Compute Engine VMs and GKE clusters. Filestore is billed on **provisioned capacity** and has high minimum entry points, which can quickly lead to high cost overheads if not managed carefully. This file covers optimization strategies for Filestore.

---

## 1. Filestore Tiers & Entry-Level Costs

Filestore capacity pricing depends on the selected tier. Every tier has a **minimum storage size requirement**, meaning there is a high "floor price" just to turn the service on.

* **Basic HDD:** Minimum 1 TiB (approx. $0.20/GB/month). Billed at ~$200/month baseline.
* **Basic SSD:** Minimum 1 TiB (approx. $0.30/GB/month). Billed at ~$300/month baseline.
* **High Scale SSD:** Minimum 10 TiB (approx. $0.30/GB/month). Billed at **~$3,000/month baseline**.
* **Enterprise:** Regional, synchronous replication across multiple zones. Minimum 1 TiB (approx. $0.56/GB/month). Billed at **~$573/month baseline**.

### The Cost Risk:
If you spin up an Enterprise Filestore instance and only store 50 GB of data, you still pay for 1,024 GB (1 TiB), resulting in significant wasted spend.

---

## 2. Core Cost-Reduction Tactics

### A. Evaluate Cheaper Alternatives (The 80/20 Rule)
Before optimizing Filestore, check if NFS/POSIX compatibility is strictly necessary:
1. **Cloud Storage FUSE (GCS FUSE):**
   * If your application only needs to read and write files and doesn't require POSIX file locking or sub-millisecond latency, use **Cloud Storage FUSE**.
   * GCS FUSE allows you to mount a Cloud Storage bucket as a local file system.
   * **The Saving:** GCS Standard storage is ~$0.020/GB/month with **no minimum storage size**. A 50 GB dataset costs ~$1/month on GCS FUSE versus $300/month on Filestore Basic SSD.
2. **Read-Write-Many (RWX) alternative with Persistent Disk:**
   * If you are sharing read-only data across multiple VMs or pods, use GCE Zonal/Regional Persistent Disk in **read-only multi-writer mode**, which is far cheaper than Filestore.

### B. Consolidate into Multishare (GKE Only)
* If you have multiple microservices in GKE that each require small NFS file shares, creating a separate Filestore instance for each service is highly wasteful.
* **Action:** Enable **GKE Multishare** on a single Enterprise Filestore instance. This allows you to split a single 1 TiB+ Filestore instance into up to 80 smaller Persistent Volumes (shares) allocated to different pods, utilizing the entire provisioned capacity.

### C. Right-Size Over-Provisioned Instances
Filestore instances can be scaled up dynamically, but they **cannot be scaled down (shrunk)** in place.
* **Action:** If you are using less than 50% of your Filestore capacity:
  1. Spin up a new Filestore instance at the target smaller size (e.g. 1 TiB minimum).
  2. Mount both the old and new instances to a temporary VM.
  3. Run `rsync -avzh /mnt/old/ /mnt/new/` to migrate the files.
  4. Update mount configurations on your active VMs/pods to point to the new Filestore IP.
  5. Delete the old Filestore instance.

### D. Audit Non-Production Environments
* Ensure all development and testing environments use `Basic HDD` instead of `Basic SSD` or `Enterprise`.
* For staging, evaluate whether Filestore can be turned off entirely and spun up only during active testing windows.

### E. Backup & Snapshot Lifecycle
* Filestore backups are billed per GB of backup storage (~$0.08/GB/month for Basic).
* **Action:** Review retention settings on Filestore backups. Automatically delete backups older than 7 or 14 days for non-production environments.

---

## 3. Filestore Audit Checklist

1. [ ] **NFS Necessity Review:** Verify if workloads on Filestore can be migrated to GCS FUSE.
2. [ ] **Capacity Utilization Check:** Run an audit to identify Filestore instances with < 50% storage utilization.
3. [ ] **GKE Multishare Consolidation:** For GKE workloads, consolidate multiple individual Filestore instances into a single Multishare instance.
4. [ ] **Tier Compliance:** Ensure no development projects are using the `Enterprise` or `High Scale` tiers.
5. [ ] **Backup Retention Audit:** Confirm backup schedules have an active retention cap.
