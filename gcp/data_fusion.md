# Cloud Data Fusion Cost Optimization & Research

Google Cloud Data Fusion is a fully managed, cloud-native data integration service built on the CDAP open-source framework. It allows users to build visual ETL/ELT pipelines. Because Data Fusion has high flat-rate instance costs (up to $3,200/month for Enterprise) and spins up separate execution clusters, managing instances and execution profiles is crucial to control costs.

---

## 1. Cloud Data Fusion Billing Components

Data Fusion billing is divided into two primary areas:
1. **Instance Fee:** An hourly rate charged for hosting the Data Fusion instance (management plane), billed continuously 24/7:
   * **Developer Edition:** Approx. $0.35 per hour (~$250/month). No SLA, restricted capacity.
   * **Basic Edition:** Approx. $1.85 per hour (~$1,350/month). Includes SLA.
   * **Enterprise Edition:** Approx. $4.50 per hour (~$3,285/month). Includes advanced security, VPC-SC, and HA.
2. **Execution Resources:** When pipelines run, Data Fusion provisions execution clusters to process the data. You pay standard rates for the **Dataproc** clusters or **Dataflow** workers that execute these pipelines.

---

## 2. Core Cost-Optimization Levers

### A. Avoid Idle 24/7 Instances
The biggest cost leak in Data Fusion is keeping Basic or Enterprise instances running 24/7 when pipelines only execute for a few hours a day.
* **Tactic 1 (Automated Provisioning):** For batch pipelines that run on wide intervals (e.g., once daily or weekly), write a workflow (e.g., using Cloud Composer or Cloud Workflows) to spin up the Data Fusion instance via Terraform/CLI, trigger the pipeline execution, verify completion, and immediately delete the Data Fusion instance.
* **Tactic 2 (Migration):** If pipelines are simple, migrate them to Cloud Composer (Airflow) + BigQuery SQL, which runs entirely serverless and avoids Data Fusion's high flat-rate hosting fees.

### B. Use Ephemeral Dataproc Execution Profiles
* **The Waste:** Running Data Fusion jobs on a persistent, 24/7 Dataproc cluster. You pay for the VMs during all hours of idle time.
* **The Solution:** Configure your execution profiles to use **Ephemeral Dataproc Clusters**.
* **Action:** In the pipeline execution configuration, select the default ephemeral profile. Data Fusion will automatically spin up the Dataproc nodes, execute the transformation, and tear them down immediately upon completion.

### C. Restrict Enterprise Edition to Production
* **Action:** Never use the Enterprise Edition for development, staging, or sandbox environments. Use the **Developer Edition** for all building, testing, and pipeline design, which cuts your environment costs by **92%** compared to Enterprise.

---

## 3. Data Fusion Audit Checklist

1. [ ] **Developer Edition Enforcement:** Confirm that all non-production projects run Data Fusion on the Developer Edition.
2. [ ] **Ephemeral Profiling:** Verify that all active pipelines run on ephemeral execution profiles rather than permanent clusters.
3. [ ] **Instance Idle Review:** Review execution schedules. If instances run < 6 hours of work per day, evaluate migrating to an automated build-and-destroy model.
4. [ ] **Dataproc VM Sizing:** Review the VM sizes configured in the ephemeral profile templates. Ensure worker nodes use cost-effective machine types (like `e2-standard-4`) and utilize Spot VMs where appropriate.
