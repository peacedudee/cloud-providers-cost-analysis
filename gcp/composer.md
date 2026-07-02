# Cloud Composer Cost Optimization & Research

Google Cloud Composer is a fully managed workflow orchestration service built on Apache Airflow. Composer 2 utilizes GKE Autopilot under the hood to run the Airflow control plane (Web Server, Metadata Database, Schedulers) and worker nodes. Because Composer combines multiple GKE containers and database storage, running idle environments or over-provisioned worker pools can drive up significant monthly costs.

---

## 1. Cloud Composer Billing Components

Composer 2 billing is calculated based on the CPU, RAM, and Storage resources consumed by the environment:
1. **Composer Environment Fee:** A base hourly fee for running the service.
2. **Airflow Control Plane (Web Server & Schedulers):** Charged for the vCPU and RAM allocated to these system containers.
3. **Airflow Workers:** Charged per vCPU, RAM, and Storage hour consumed by the active workers.
4. **Metadata Database Storage:** Billed per GB/month for the SQL database storing Airflow state.

---

## 2. Core Cost-Optimization Levers

### A. Tune Autoscaling Worker Limits
Airflow workloads are bursty (e.g., all DAGs run at midnight, then workers sit idle).
* **The Waste:** Running a fixed count of 5 or 10 worker nodes 24/7.
* **The Solution:** Configure **Autoscaling**.
* **Action:** Set the minimum worker count to **1** and the maximum worker count to a conservative peak (e.g. 5). GKE will scale workers down to 1 during the idle hours of the day.

### B. Use the "Small" Presets in Non-Production
* **The Waste:** Provisioning standard or large Composer environments for development testing.
* **Action:** Select the **Small Environment Preset** when creating dev/test environments. Manually tune the web server and scheduler resources down to the lowest allowable configurations (e.g. 0.5 vCPU for the web server).
* **The Benefit:** Lowers the baseline idle environment cost by **60% to 75%** compared to medium presets.

### C. Offload Heavy Workloads to Serverless Engines
* **The Cost Trap:** Running memory-intensive Python pandas data-cleaning scripts or AI model training directly inside Airflow tasks on the Composer workers. This forces you to request large worker VM shapes, which bills you for compute continuously.
* **The Solution:** Use Airflow strictly as an **orchestrator**, not a processing engine.
* **Action:** Use operators that offload execution to serverless services (e.g. `BigQueryInsertJobOperator` for SQL transformations, or `DataflowTemplatedJobStartOperator` for ETL). The Airflow worker then only consumes minimal resources while waiting for the serverless job to finish.

### D. Enforce Log and Database Pruning
* Composer stores run histories and task logs. Over time, the metadata database and Cloud Storage buckets accumulate gigabytes of historical logs, driving up storage fees.
* **Action:** Set GCS lifecycle rules to delete or archive execution logs older than 30 days. Deploy a weekly maintenance DAG that runs `airflow db clean` to purge task history logs from the metadata database.

---

## 3. Cloud Composer Audit Checklist

1. [ ] **Worker Autoscale Limits:** Confirm worker minimum is set to 1 and maximum has a hard ceiling.
2. [ ] **Developer Preset Check:** Ensure dev/test projects use the "Small" preset with minimal CPU/RAM.
3. [ ] **Workload Offloading Audit:** Scan DAG code for resource-intensive Python processing. Migrate heavy processing to BigQuery or Dataflow.
4. [ ] **GCS Log Retention:** Apply GCS lifecycle rules to the Composer environment bucket to purge or archive logs older than 30 days.
5. [ ] **Database Pruning DAG:** Verify a database maintenance DAG is active to clean the Airflow metadata database.
