# AWS Service Cost Research: AWS CodeBuild

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS CodeBuild is a fully managed, serverless build service that compiles source code, runs unit tests, and produces deployable software artifacts (such as Docker container images or zip files). CodeBuild eliminates the need to provision, manage, and scale build server clusters (like Jenkins). CodeBuild scales dynamically, billing per build-minute based on compute architecture and container instance size requested.

---

## 2. Billing Mechanics
1. **Build-Minutes Duration:** Billed per minute (or second) that build container instances execute.
2. **Compute Instance Architecture & Size:** Rates vary based on CPU architecture (Linux x86 vs Linux ARM Graviton), memory capacity, Windows OS support, and GPU hardware acceleration.
3. **Free Tier:** First 100 build-minutes per month on `build.general1.small` are **100% Free ($0.00)**.

---

## 3. Key Cost Dimensions

### A. Linux x86 Compute Tiers (us-east-1 Rates)
* **`build.general1.small` (3 vCPU, 4 GB RAM):** **$0.005 per build-minute**.
* **`build.general1.medium` (4 vCPU, 7 GB RAM):** **$0.010 per build-minute**.
* **`build.general1.large` (8 vCPU, 15 GB RAM):** **$0.020 per build-minute**.

### B. Linux ARM Graviton Compute Tiers (32% Cost Reduction)
* **`build.arm1.small` (2 vCPU, 4 GB RAM):** **$0.0034 per build-minute** (**32% cheaper** than x86 small).
* **`build.arm1.large` (8 vCPU, 16 GB RAM):** **$0.0136 per build-minute** (**32% cheaper** than x86 large).

### C. Windows & GPU Compute Tiers
* **Windows Container (`build.general1.small`):** **$0.016 per build-minute**.
* **GPU Compute (`build.gpu.small`):** **$0.050 to $0.200 per build-minute**.

---

## 4. Detailed Pricing Rates (us-east-1)

| CodeBuild Compute Type | CPU / RAM | Rate per Build-Minute | Price for 1,000 Build-Minutes |
|------------------------|-----------|-----------------------|--------------------------------|
| **`build.arm1.small` (Graviton)** | 2 vCPU, 4 GB | **$0.0034** | **$3.40** |
| **`build.general1.small`** | 3 vCPU, 4 GB | **$0.0050** | **$5.00** |
| **`build.general1.medium`** | 4 vCPU, 7 GB | **$0.0100** | **$10.00** |
| **`build.general1.large`** | 8 vCPU, 15 GB | **$0.0200** | **$20.00** |

---

## 5. AWS Free Tier Coverage
* **AWS CodeBuild Free Tier:** Includes **100 free build-minutes per month** on `build.general1.small` per account indefinitely.

---

## 6. Common Cost Hotspots & Pitfalls
* **Un-Cached Docker Image Builds:** Running Docker builds (`docker build`) where base image layers, `npm install`, or Maven dependencies are downloaded from scratch on every single build commit, quadrupling build execution minutes ($0.01–$0.02/min).
* **Over-Provisioning Compute Sizes:** Launching `build.general1.large` ($0.02/min) for simple shell scripts or microservice tests that execute comfortably on `build.arm1.small` ($0.0034/min).

---

## 7. Actionable Cost Optimization Strategies
1. **Migrate Linux Builds to ARM Graviton (`build.arm1`):**
   * Update CodeBuild project settings to use **ARM Graviton compute** (`build.arm1.small` at **$0.0034/min** vs $0.005/min).
   * *Benefit:* ARM Graviton builds run faster and cost **32% less** per minute.
   * **The Savings:** Instantly slashes CI/CD build bills by 32%.
2. **Enable Local & S3 Docker Layer Caching:**
   * In CodeBuild settings, enable **Local Caching** or **S3 Caching** for Docker layers (`--cache-from`) and dependency directories (`node_modules`, `.m2`).
   * **The Savings:** Cuts build runtimes from 10 minutes to 2 minutes (**80% reduction in billed build-minutes**).
3. **Set Enforced Build Timeout Limits:** Configure `timeoutInMinutes: 15` in CodeBuild project settings to prevent hung test scripts from burning build-minutes indefinitely.
