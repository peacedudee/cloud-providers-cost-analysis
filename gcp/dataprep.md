# Dataprep (by Trifacta) Cost Optimization & Research

Google Cloud Dataprep is an intelligent, visual data preparation service for exploring, cleaning, and preparing structured and unstructured data for analysis. Dataprep runs on top of Google Cloud Dataflow. Understanding the dual-billing architecture (Dataprep licensing + Dataflow runner resources) is key to managing its costs.

---

## 1. Dataprep Billing Components

Dataprep billing is split into two parts:
1. **Dataprep Service Units:** Charged based on the amount of data processed by your flows (or billed as Dataprep Units per hour of execution).
2. **Underlying Dataflow Worker Cost:** When you execute a Dataprep job, it translates your visual recipes into an Apache Beam pipeline and runs it on **Google Cloud Dataflow**. You pay standard Dataflow rates for the worker VMs (vCPUs, RAM, Disks) during execution.

---

## 2. Core Cost-Optimization Levers

### A. Filter Early in the Visual Flow
* **The Waste:** Reading a 10 TB dataset, performing complex text formatting on all rows, and then filtering down to only 1 GB of data at the very end of the flow. You pay Dataprep and Dataflow for processing the entire 10 TB throughout the pipeline.
* **Action:** Place your **Filter Rows** and **Keep Columns** steps as the absolute first steps in your Dataprep recipe. Drop unneeded metadata and rows before applying transformations.
* **The Benefit:** Drastically reduces the volume of data processed, directly lowering Dataprep Service Units and shortening Dataflow worker execution time.

### B. Run Profiles Only on Samples
* **Action:** Dataprep allows you to generate visual data profiles (histograms, quality metrics). Running full profiles on massive production tables during every execution is highly compute-intensive.
* **Tactic:** Limit profiling to development and exploration phases using a representative data sample. Turn off automatic profile generation in production scheduler settings.

### C. Right-Size Dataflow Worker Limits
* When running Dataprep jobs, you can configure the runtime execution settings.
* **Action:** Set a cap on the maximum number of Dataflow workers (`maxNumWorkers`) to prevent Dataflow from autoscaling to massive cluster sizes for non-critical pipelines, which can lead to high transient billing spikes.

---

## 3. Dataprep Audit Checklist

1. [ ] **Early Filter Check:** Verify that data cleansing recipes have filter/select steps positioned at the beginning of the flow.
2. [ ] **Production Profiling Audit:** Disable "Data Profiling" on scheduled, automated production Dataprep runs.
3. [ ] **Dataflow Worker Capping:** Set `maxNumWorkers` limits on Dataprep execution profiles to restrict autoscaling budget boundaries.
