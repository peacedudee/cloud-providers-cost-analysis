# Cloud Dataproc Cost Optimization & Research

Google Cloud Dataproc is a managed Apache Spark and Apache Hadoop service that lets you take advantage of open source data tools for batch processing, querying, and machine learning. Dataproc clusters run on top of Google Compute Engine (GCE) VMs, incurring both VM compute costs and Dataproc premium management fees. This file outlines key strategies for optimizing Dataproc spend.

---

## 1. Dataproc Billing Models

Dataproc charges run along two deployment paths:

### A. Managed Dataproc Clusters (GCE-based)
You pay for:
1. **Underlying Infrastructure:** GCE VM charges (vCPU, RAM, Persistent Disks) for all master, primary worker, and secondary worker nodes.
2. **Dataproc Premium Fee:** A flat fee of **$0.01 per vCPU per hour** added on top of standard GCE billing for all nodes in the cluster.

### B. Dataproc Serverless (Spark Batches)
* **How it works:** You submit your Spark job directly to GCP. Dataproc provisions, runs, and tears down the execution environment automatically.
* **Billing Unit:** Billed per second of execution using **Dataproc Compute Units (DCUs)** (covering vCPU, Memory, and Ephemeral Storage).
* **The Cost Benefit:** There is no master node, no VM cluster management, and **zero idle runtime cost**. You pay only for the exact milliseconds the job executes.

---

## 2. Core Cost-Reduction Tactics

### A. Decouple Compute and Storage (GCS over HDFS)
The traditional Hadoop architecture stores data locally on worker nodes using the Hadoop Distributed File System (HDFS).
* **The Waste:** To keep your data, the Dataproc cluster must run 24/7. Shutting down the cluster destroys the HDFS data.
* **The Solution:** Decouple compute and storage by storing all data in **Google Cloud Storage (GCS)** (using `gs://` paths) and leveraging the GCS connector.
* **The Benefit:** GCS storage is incredibly cheap (~$0.020/GB/month) compared to running VMs. You can spin up a Dataproc cluster, run a job, write the results to GCS, and immediately delete the cluster.

### B. Enforce Cluster Auto-Deletion Policies
Leaving static Dataproc clusters running idle after a job has finished is a common source of cloud waste.
* **Action:** Always create clusters with **Auto-delete policies** enabled in your Terraform or CLI configs.
  * **`--max-idle-time`:** Automatically delete the cluster if it has been idle (no active jobs running) for a specified duration (e.g., `15m`).
  * **`--max-age`:** Automatically delete the cluster after a hard time limit (e.g., `8h`) to prevent runaway clusters.
  * **`--auto-delete-ttl`:** Set a relative time-to-live after creation.

### C. Leverage Spot VMs for Secondary Workers
Dataproc clusters are divided into Primary Workers (which handle compute and HDFS data storage) and Secondary Workers (which only handle compute and do not store HDFS data).
* **Action:** Configure secondary workers to run entirely on **Spot VMs** (saving up to 91% on the underlying Compute Engine VM costs, though the $0.01/vCPU Dataproc management fee still applies).
* **Mitigation:** Enable **Enhanced Flexibility Mode (EFM)** on the cluster. EFM prevents job failures when Spot VMs are pre-empted by safely routing shuffle data away from secondary nodes.

### D. Implement Autoscaling Policies
* **Tactic:** Define a Dataproc Autoscaling Policy.
* **Action:** Set the primary worker group to a small, fixed baseline (e.g. 2 nodes to maintain HDFS quorum) and configure the secondary worker group to autoscale dynamically from 0 to 100 Spot nodes based on YARN pending memory and pending core metrics.

### E. Dataproc Serverless for Ad-Hoc/Cron Jobs
* **Action:** If you run scheduled batch jobs once a day or once a week, do not spin up a cluster. Submit the job using **Dataproc Serverless**.
* **The Benefit:** Dataproc Serverless automatically right-sizes the resources and scales execution up and down, preventing you from over-specifying node shapes and paying for cluster boot/teardown overhead.

---

## 3. Cloud Dataproc Audit Checklist

1. [ ] **Auto-delete Policy Check:** Verify that every Dataproc cluster has active `--max-idle-time` or `--max-age` rules.
2. [ ] **HDFS Elimination:** Confirm that all jobs read and write data to GCS (`gs://`) rather than local `hdfs://` storage.
3. [ ] **Autoscaling Configuration:** Ensure all clusters with variable workloads use Autoscaling Policies.
4. [ ] **Spot Node Utilization:** Verify that secondary workers are configured to use Spot VMs.
5. [ ] **Serverless Transition:** Audit scheduled cron jobs running on static clusters and migrate them to Dataproc Serverless.
6. [ ] **Image Modernization:** Ensure clusters are updated to the latest Dataproc image versions to take advantage of Spark Adaptive Query Execution (AQE) optimization.
