# Cloud Storage (GCS) Cost Optimization & Research

Google Cloud Storage (GCS) is a RESTful service for storing and accessing your data on Google Cloud's infrastructure. While storage seems cheap per gigabyte, GCS bills across multiple dimensions (storage capacity, operation class, retrieval, replication, and egress). Without strict lifecycle policies, version control, and location planning, GCS costs can grow exponentially. This file details optimization tactics for GCS.

---

## 1. GCS Billing Dimensions

GCS bills on five major axes:
1. **Data Storage Capacity (GB/month):** Based on the amount of data and its storage class.
2. **Operations (per 10,000 operations):** Billed based on API requests:
   * **Class A (Writing & Listing):** `insert`, `copy`, `compose`, `list` (highest cost).
   * **Class B (Reading):** `get` (cheaper).
   * **Free:** `delete`.
3. **Data Retrieval (per GB):** Billed when reading data stored in Nearline, Coldline, or Archive tiers.
4. **Early Deletion Fees:** Pro-rated charges if data is deleted or overwritten before the minimum duration threshold of its class.
5. **Network Egress (per GB):** Reading data from GCS and sending it to the internet, another GCP region, or a different zone.

### Storage Class Comparison (2026 update)

| Storage Class | Minimum Duration | Retrieval Fee | Use Case |
| :--- | :--- | :--- | :--- |
| **Rapid Storage** | None | None | Optimized for AI/ML training, checkpointing, HPC. |
| **Standard** | None | None | Frequently accessed data, active application assets, web hosting. |
| **Nearline** | 30 Days | $0.01 / GB | Data accessed < once a month. Backups, old reports. |
| **Coldline** | 90 Days | $0.02 / GB | Data accessed < once a quarter. Disaster recovery. |
| **Archive** | 365 Days | $0.05 / GB | Data accessed < once a year. Long-term compliance archives. |

---

## 2. Core Cost-Reduction Tactics

### A. Implement Lifecycle Management (Lifecycle Rules)
Setting manual policies to move data to colder tiers as it ages is the easiest way to cut GCS costs.
* **The Rule Strategy:**
  * Transition files in the `logs/` directory to **Archive** storage after 30 days, and delete them after 90 days.
  * Move raw transaction backups from Standard to **Nearline** after 7 days, to **Coldline** after 30 days, and delete them after 365 days.
* **Caution:** Ensure files transitioned to Nearline/Coldline/Archive are *not* accessed by active applications, as retrieval fees can easily wipe out storage savings.

### B. Use Autoclass for Unpredictable Access
If you don't know the access patterns of your data, do not guess. Enabling **Autoclass** allows Google to manage transitions automatically:
* **How it works:** Data starts in Standard storage. If an object is not accessed for 30 days, it is automatically transitioned to Nearline. If it remains unaccessed, it cascades down to Coldline and Archive.
* **The Catch:** If a cold file is accessed, it is immediately transitioned back to Standard storage at **no retrieval fee** and **no early deletion fee**. However, Autoclass charges a management fee (approx. $0.0025 per 1,000 objects per month).
* **Action:** Enable Autoclass on buckets containing user-generated content or shared databases where access patterns are highly variable.

### C. Audit and Manage Object Versioning
Object Versioning protects against accidental deletes by keeping non-current versions of files when they are overwritten or deleted.
* **The Cost Trap:** If Object Versioning is enabled without a lifecycle policy, every overwrite creates a new hidden version of the file. A 10 GB file overwritten daily for a year will consume **3.65 TB** of storage billing!
* **The Solution:** Always set a lifecycle rule specifically targeting non-current versions:
  * Example: Delete non-current versions after 7 days, or keep only the 3 most recent non-current versions.

### D. Avoid Dual-Region & Multi-Region Unless Required
* **Standard Zonal/Regional storage** is the cheapest.
* **Dual-Region / Multi-Region storage** replicates data across geolocations for high availability, costing **2x to 3x** more per GB.
* **Action:** Use single-region buckets for application temporary files, scratch disks, and regional compute environments. Reserve multi-region buckets only for global asset delivery or critical disaster recovery plans.

### E. Egress Cost Mitigation
* **Data Locality:** Ensure GCS buckets are located in the *same region* as the Compute Engine VMs, GKE clusters, or Cloud Run services that read them. Reading data across regions incurs inter-region egress charges ($0.01 to $0.02 per GB).
* **Requester Pays:** For public datasets or sharing data with external clients, enable **Requester Pays** on the bucket. This shifts the network egress billing from your account to the GCP account of the entity downloading the data.
* **CDN for Public Assets:** If serving static images/assets to the public, put **Cloud CDN** in front of GCS. Serving cache-hit data from CDN edges is cheaper than egressing data directly from GCS.

### F. Leverage Anywhere Cache (New)
* If your GCS buckets are used for AI/ML model training or HPC, enable **Anywhere Cache** (SSD-backed read cache).
* This provides sub-millisecond local reads, reducing data transfer costs and decreasing VM runtime since GPU/CPU nodes don't spend idle time waiting for GCS I/O.

---

## 3. GCS Audit Checklist

1. [ ] **Find Giant Buckets:** Run a billing report grouped by bucket name to identify the top 5 cost-driving buckets.
2. [ ] **Lifecycle Verification:** Ensure all top buckets have either a **Lifecycle Rule** or **Autoclass** enabled.
3. [ ] **Object Versioning Cleanup:** Audit buckets with Object Versioning enabled and ensure they have a rule to delete non-current versions after `N` days.
4. [ ] **Region Alignment:** Verify that compute instances and their corresponding storage buckets reside in the exact same region (e.g. both in `us-central1`).
5. [ ] **Incomplete Multipart Uploads:** If using multipart API uploads, write a lifecycle rule to abort and clean up incomplete multipart uploads after 7 days (otherwise, you pay for the uploaded fragments indefinitely).
6. [ ] **Storage Intelligence Audit:** Check GCS Storage Intelligence dashboards for anomaly detection and rightsizing recommendations.
