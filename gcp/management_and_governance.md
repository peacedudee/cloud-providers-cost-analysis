# Management & Governance Cost Research (Free Core Tools)

Google Cloud provides several core management, configuration, and governance services that are **100% free of charge**. While these tools do not generate direct billing, they are the essential pillars for building FinOps rules, setting budget alerts, restricting resources via quotas, and managing infrastructure policy across your Google Cloud organization.

---

## 1. Directory of Free Management Services ($0 Billing)

### A. Cloud Billing & Budgets
* **Billing Model:** **100% Free**.
* **Role in Cost Control:** Provides billing exports to BigQuery, cost anomaly detection, and automated budget notifications.
* **Best Practice:** Always set up multi-threshold budgets (60%, 80%, 100%) and link them to email alerts and Pub/Sub notifications to automate resource shutdown if thresholds are exceeded.

### B. Cloud Quotas
* **Billing Model:** **100% Free**.
* **Role in Cost Control:** Restricts the maximum number of resources (like GPUs, CPUs, or IPs) that can be spun up in a project.
* **Best Practice:** Keep quotas in sandbox and developer projects set to a tight minimum (e.g. max 4 vCPUs, max 0 GPUs) to prevent developers from accidentally spinning up expensive high-compute nodes.

### C. Resource Manager
* **Billing Model:** **100% Free**.
* **Role in Cost Control:** Defines the folder and project hierarchy. Allows you to apply Org Policies (e.g., restricting which regions resources can be deployed in, or blocking the creation of external IP addresses).

### D. Cloud Shell
* **Billing Model:** **100% Free** within weekly usage limits.
* **Role in Cost Control:** Provides a free browser-based terminal with 5 GB of persistent home directory storage. Home directory storage is retained for free as long as you log in at least once every 120 days.

### E. Config Connector
* **Billing Model:** **100% Free**.
* **Role in Cost Control:** An open-source Kubernetes addon that allows you to manage GCP resources (like Cloud SQL or GCS buckets) using Kubernetes manifests, enabling GitOps-driven deployment.

### F. Deployment Manager
* **Billing Model:** **100% Free** for the orchestrator (you pay only for the GCE/SQL/GCS resources you deploy).
* **Status:** Legacy. For modern infrastructure-as-code, migrate to Terraform.

### G. Service Catalog & Cloud Console
* **Billing Model:** **100% Free**.
* **Role in Cost Control:** Service Catalog allows admins to create a curated list of approved, cost-effective templates that developers can deploy, preventing the use of unapproved, high-cost resources.

---

## 2. FinOps Policy Checklist for Governance

1. [ ] **Org-Level Region Restrictions:** Use Resource Manager Organization Policies to restrict resource deployments to specific, cost-effective regions (e.g., restricting deployments to `us-central1` and blocking expensive European/Asian regions unless legally required).
2. [ ] **Budget Alert Coverage:** Confirm that every active billing sub-account or project has at least one active Cloud Billing budget alert configured.
3. [ ] **Developer Quota Caps:** Downscale GPU and high-CPU quotas in sandbox projects to prevent accidental high-compute deployments.
