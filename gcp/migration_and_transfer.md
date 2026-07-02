# Migration & Transfer Cost Optimization & Research

Google Cloud offers several tools to accelerate data and workload migrations: **Migration Center** (discovery and planning), **Migrate to VMs** (VM migration), **Migrate to Containers** (converting VMs to GKE), **Storage Transfer Service** (online object transfer), and **Transfer Appliance** (physical data transfer devices). While the migration tools themselves are mostly free of charge, managing transient replication infrastructure and source network egress is critical to prevent budget overruns during migration phases.

---

## 1. Billing Models & Pricing

* **Migration Center:** **100% Free**. No charge for discovery, inventory collection, or TCO assessments.
* **Migrate to Virtual Machines:** **100% Free** for the migration orchestrator itself. You pay strictly for the destination Compute Engine resources (VMs, storage, network) and temporary replication disk space.
* **Migrate to Containers:** **100% Free** for the migration service. You only pay for GKE node resources.
* **Storage Transfer Service (STS):**
  * **STS Service Fee:** **Free** for transferring data into Google Cloud Storage (from AWS S3, Azure Blob, HTTP, or GCS-to-GCS).
  * **Associated Costs:** You pay standard destination GCS storage and API class A write fees, and **source provider network egress fees** (e.g. AWS S3 egress charges when pulling data out to Google Cloud).
* **Transfer Appliance:** Billed as a flat rental fee per appliance run (approx. $600 for a 40 TB appliance, or $1,800 for a 300 TB appliance) plus shipping. No ingestion fees apply at the GCS landing zone.

---

## 2. Core Cost-Optimization Levers

### A. The "Transfer Appliance" Breakeven (Physical vs. Online Egress)
* **The Cost Trap:** Transferring 150 TB of data from an on-premises datacenter or another cloud provider over the public internet to GCS. The source internet egress fees can cost over $12,000 (at $0.08/GB).
* **The Solution:** Use **Transfer Appliance**.
* **Action:** For datasets exceeding **100 TB**, rent a physical Google Transfer Appliance. Copy data locally to the appliance, ship it to Google, and let Google ingest it directly to GCS.
* **The Savings:** Flat fee of ~$600-$1,800 plus shipping, bypassing public internet egress fees entirely and saving up to **80%** on data transit.

### B. Configure Incremental Syncs in Storage Transfer Service
* **Action:** When setting up STS jobs to sync buckets, configure the job to only transfer **New and Modified Objects** (default) rather than copying the entire source directory on every run.
* **The Benefit:** Bypasses redundant API calls and prevents paying for repeated data transfer egress on unchanged files.

### C. Decommission Migration Replicators
* **The Issue:** During VM migration using Migrate to VMs, the orchestrator spins up local replicator instances and copies volumes continuously (CDC phase) to GCP persistent disks.
* **Action:** Once application cutover is complete and the new VM is running natively in production on Compute Engine, immediately **finalize and detach** the migration. Verify that all temporary replication disks and migration logs are deleted.

---

## 3. Migration & Transfer Audit Checklist

1. [ ] **Appliance Breakeven Check:** Evaluate all database/file migration plans. For static datasets > 100 TB, mandate physical Transfer Appliances.
2. [ ] **Incremental Sync Verification:** Verify that active STS configurations are set to transfer only modified objects.
3. [ ] **Migration Clean Up:** Audit GCE for active replication VMs or disk copies from legacy Migrate to VMs jobs; delete them post-cutover.
4. [ ] **PGA for Migration Sinks:** Ensure Private Google Access (PGA) is enabled on GCS ingestion landing zones to route migration traffic internally.
