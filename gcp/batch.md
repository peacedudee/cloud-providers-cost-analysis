# Google Cloud Batch Cost Optimization & Research

Google Cloud Batch is a fully managed, serverless batch scheduler that allows you to run scale-out batch jobs (e.g., simulations, rendering, data processing, bioinformatics) on Google Cloud infrastructure. Because **there is no additional management fee for the Batch service itself**, you only pay for the underlying Compute Engine resources (VMs, storage, network) that are spun up to execute your tasks.

---

## 1. Batch Billing Mechanics

Google Cloud Batch charges are entirely driven by the infrastructure provisioned for your jobs:
1. **Compute Engine Resources:** Billed for the vCPUs, RAM, GPUs, and Persistent Disks allocated to the batch worker VMs during task execution. Billed on a per-second basis.
2. **Network Egress:** Standard network transfer charges apply if the batch job moves data across regions or to the internet.
3. **No Scheduler Fee:** The control plane, queuing mechanism, and job scheduling service are **100% free**.

---

## 2. Core Cost-Optimization Levers

### A. Natively Leverage Spot VMs
Since batch workloads are by definition batch-oriented and design-to-completion, they are the single best candidate for Spot VMs.
* **Action:** Configure the job definition to use Spot VMs (`"preemptible": true` or `"provisioningModel": "SPOT"` in the API).
* **The Benefit:** Instantly cuts compute costs by **up to 91%** compared to on-demand VM pricing.
* **Resiliency:** Google Cloud Batch handles VM preemption automatically. If a Spot VM is reclaimed mid-job, Batch automatically resubmits the task to the queue and spins up a new worker VM to resume, requiring zero manual developer intervention.

### B. Avoid Over-Provisioning Worker VM Shapes
* **The Waste:** Specifying standard machine types (e.g. `n2-standard-8` with 8 vCPUs / 32 GB RAM) for a job that is single-threaded or requires very little memory.
* **Action:** In the Batch job JSON template, define custom resource requirements matching the actual application needs (e.g. requesting exactly 1 vCPU and 2 GB memory per task). Batch will group tasks onto the most efficient VM shapes possible.

### C. Right-Size Job Storage (Local SSD Scratch Space)
* Batch jobs often download large datasets, process them, and upload results. Using standard Persistent Disks (PD) for high-I/O scratch space can slow down job execution, resulting in longer VM runtimes and higher bills.
* **Action:** For jobs requiring fast read/write scratch space, attach a **Local SSD** to the worker VM template. Local SSDs provide massive IOPS, speeding up task execution, reducing the overall run duration, and lowering total VM billing.

### D. Regional Co-Location (Data Egress)
* **The Waste:** Running batch VMs in `us-central1` while reading input datasets from a GCS bucket in `europe-west1` and writing outputs back. You pay intercontinental network egress fees for every gigabyte read.
* **Action:** Always specify the `allowedLocations` parameter in the Batch job configuration to force worker VMs to run in the **exact same region** as your input/output Cloud Storage buckets.

---

## 3. Cloud Batch Audit Checklist

1. [ ] **Spot VM Enablement:** Verify that `"preemptible": true` or `"provisioningModel": "SPOT"` is active in all non-critical batch job configurations.
2. [ ] **Resource Sizing Check:** Profile batch runtimes to verify that requested CPU/Memory limits align with actual utilization (aim for > 80% peak utilization on workers).
3. [ ] **Location Alignment:** Confirm that batch job VM execution regions match the region of the associated GCS source/destination buckets.
4. [ ] **Storage Type Optimization:** Review disk configurations. Migrate I/O-heavy workloads from standard PDs to Local SSDs to speed up execution.
5. [ ] **Job Timeout Limits:** Enforce maximum run limits (`maxRunDuration`) on jobs to prevent stalled or hung batch tasks from billing indefinitely.
