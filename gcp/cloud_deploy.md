# Cloud Deploy Cost Optimization & Research

Google Cloud Deploy is a managed continuous delivery service that automates the deployment of applications to Google Kubernetes Engine (GKE), Cloud Run, and Anthos. Unlike build systems that charge per minute, Cloud Deploy uses a **flat-rate fee per active delivery pipeline** per month. Pruning unused pipelines is the primary cost-saving lever.

---

## 1. Cloud Deploy Billing Mechanics

Cloud Deploy pricing is structured around active pipeline configurations:
1. **Active Delivery Pipeline:** Billed at a flat **$15.00 per active pipeline per month**.
   * A pipeline is considered "active" if it has successfully or unsuccessfully triggered at least one rollout in your project during the calendar month.
   * If a pipeline has zero rollout executions during a month, the charge is **$0**.
2. **Free Tier:** The first **1 active delivery pipeline per billing account per month is free**.
3. **Target Environment Cost:** You pay standard rates for the underlying destination resources (GKE node VMs, Cloud Run CPU/Memory, Cloud Load Balancer rules) where the rollouts are deployed.

---

## 2. Core Cost-Optimization Levers

### A. Decommission Inactive & Stale Pipelines
* **The Waste:** Creating unique delivery pipelines for temporary feature branches, running a single rollout to test the configuration, and leaving the pipeline configuration in the project. If a developer runs an ad-hoc deploy or an automated scheduler triggers it next month, it incurs the $15.00 charge.
* **Action:** Delete delivery pipelines for feature branches, retired projects, or completed migrations.
  ```bash
  # Delete a delivery pipeline configuration
  gcloud deploy delivery-pipelines delete my-pipeline \
      --region=us-central1 --force
  ```

### B. Consolidate Pipelines for Dev/Test environments
* While production environments benefit from dedicated delivery pipelines for security and separation of concerns, having 30 separate pipelines for 30 minor microservices in a developer playground can add up ($450/month).
* **Action:** For non-critical internal tools or dev environments, consolidate deployments under a single delivery pipeline configured with multiple target stages or deploy multiple services as a single release bundle where architecturally acceptable.

---

## 3. Cloud Deploy Audit Checklist

1. [ ] **Active Pipeline Sweep:** List all delivery pipelines in the console. Identify and delete configurations for projects that are retired or completed.
2. [ ] **Rollout Trigger Audit:** Inspect automated triggers (e.g. Git triggers or webhook integrations) that may cause accidental rollouts on minor commits, activating the billing status of the pipeline.
3. [ ] **Cleanup Scripts:** Add a step in CI/CD branch cleanup pipelines to delete corresponding Cloud Deploy pipelines when branches are merged and deleted.
