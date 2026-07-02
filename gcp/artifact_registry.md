# Artifact Registry Cost Optimization & Research

Google Cloud Artifact Registry is the next-generation fully managed registry for container images and language packages (Maven, npm, Python, NuGet). Because active development pipelines push container images continuously, storage volume grows exponentially if old builds are not pruned. This file details how to optimize registry storage and network egress costs.

---

## 1. Artifact Registry Billing Components

Artifact Registry costs are driven by two main factors:
1. **Data Storage (per GB / month):** Billed at approx. $0.10 per GB-month.
2. **Data Transfer (Network Egress):** Billed standard rates when container images or packages are pulled outbound to the internet, on-premises, or across GCP regions. Inbound data transfer (upload/push) is free.

---

## 2. Core Cost-Optimization Levers

### A. Implement Native Cleanup Policies (Prune Old Images)
This is the single most critical cost-saving lever for registries. By default, every build pushes a new tagged container image, which accumulates storage fees forever.
* **Action:** Configure **Cleanup Policies** on your repositories. Under your repository settings:
  1. **Delete Policy (Age-based):** Delete untagged images or intermediate build tags older than 14 or 30 days.
  2. **Keep Policy (Count-based):** Keep only the **5 most recent versions** of any container image.
  3. **Release Protection:** Exclude production tags (e.g. matching `v*` or `release-*`) from deletion rules.
* **The Benefit:** Automates storage management, capping repository sizes and saving up to **80%** on registry billing.

### B. Enforce Multi-Stage Docker Builds (Shrink Image Size)
* **The Waste:** Storing 1.5 GB Docker images containing build tools, temporary compilers, and thick OS distributions (like Ubuntu).
* **Action:** Eductate developers to use **Multi-Stage Builds** and deploy on **Distroless** or **Alpine** base images:
  * Stage 1: Build the app using complete SDK layers (e.g., Maven, Go SDK).
  * Stage 2: Copy only the compiled binaries/artifacts into a tiny runner layer.
* **The Benefit:** Shrinks final image sizes from 1 GB down to **50–100 MB** (a **90% storage and egress reduction**).

### C. Migrate Legacy Container Registry (GCR) to Artifact Registry
* **The Risk:** Google Container Registry (GCR) is legacy and does not support native cleanup policies. Storing thousands of old development images in GCR forces you to pay GCS storage fees forever unless you run manual, complex deletion scripts.
* **Action:** Migrate all GCR repositories to Artifact Registry and apply the cleanup policies defined above.

---

## 3. Artifact Registry Audit Checklist

1. [ ] **Cleanup Policies Implementation:** Confirm cleanup policies (keep/delete rules) are active on all active Docker repositories.
2. [ ] **GCR Migration Check:** Verify that legacy GCR buckets are migrated and decommissioned.
3. [ ] **Base Image Optimization:** Audit container definitions for multi-stage configurations to minimize pushed layer sizes.
4. [ ] **Regional Co-Location:** Ensure Artifact Registry and GKE clusters reside in the same region to avoid inter-region network charges during image pulls.
