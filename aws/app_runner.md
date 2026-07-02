# AWS Service Cost Research: AWS App Runner

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS App Runner is a fully managed service that enables developers to quickly deploy containerized web applications and APIs at scale without needing prior infrastructure experience. 

> **⚠️ IMPORTANT SERVICE LIFECYCLE NOTICE:**  
> AWS App Runner is in **maintenance mode** and is not accepting new customers. Existing workloads continue to run smoothly, but no new major features are planned. For new containerized web applications, AWS recommends deploying on **AWS Fargate (with Amazon ECS)** or **AWS Amplify Gen 2**.

---

## 2. Billing Mechanics
App Runner bills on a pay-as-you-go model combining compute duration (differentiated by active vs. provisioned/idle status) and deployments:

* **vCPU-hours (Active only):** Billed only when container instances are actively processing requests.
* **Memory-hours (Active & Provisioned):** Billed continuously for allocated memory to keep container instances warm and prevent cold starts.
* **Build Fees (Code-based deployments):** Billed when App Runner builds container images from source repositories (e.g., GitHub).
* **Data Transfer (Egress):** Standard AWS outbound networking fees apply ($0.09/GB baseline).

---

## 3. Key Cost Dimensions

### A. Compute: Active vs. Provisioned Instances
To handle incoming traffic, App Runner scales instances dynamically. Charges are split as follows:
* **Provisioned Container Instances (Memory only):** To eliminate cold starts, App Runner keeps at least one instance warm. While idle, you pay **only for memory** ($0.007 per GB-hour). vCPU is not billed during idle periods.
* **Active Container Instances (vCPU & Memory):** When processing requests, the instance is marked active. You are charged for both **vCPU ($0.064/vCPU-hour)** and **memory ($0.007/GB-hour)**. Active status persists for approximately 80 seconds after the last request completes before reverting to provisioned (idle) status.

### B. Deployment & Build Charges
* **Code-Based Deployments (GitHub / Source Code):** App Runner provisions a build environment to compile code and build container images. Billed at **$0.005 per build minute**.
* **Image-Based Deployments (Amazon ECR):** Deploying pre-built container images from Amazon ECR bypasses App Runner build fees entirely.

### C. Auto Scaling & Concurrency Settings
App Runner autoscales based on concurrent requests per instance (default limit is 100 concurrent requests). Setting concurrency targets too low causes App Runner to spin up additional container instances prematurely, increasing active vCPU and provisioned memory costs.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Dimension | Unit Billing Rate | Monthly Baseline (Est. per 24/7 Unit) |
|----------------|-------------------|---------------------------------------|
| **Active vCPU** | $0.064 / vCPU-hour | $46.72 (if 100% active) |
| **Active / Provisioned Memory** | $0.007 / GB-hour | $5.11 (always charged 24/7) |
| **Build Fee (Code-based)** | $0.005 / build-minute | $0.30 per hour of build time |

### Mathematical Cost Calculation Example
*Scenario: 1 container instance configured with 1 vCPU and 2 GB RAM, active 15% of the time (processing requests) and idle/provisioned 85% of the time.*

1. **Memory Charge (100% of time):**
   $$\text{Memory Cost} = 1\text{ instance} \times 2\text{ GB} \times 730\text{ hours} \times \$0.007 = \$10.22\text{ / month}$$
2. **vCPU Charge (15% active time):**
   $$\text{vCPU Cost} = 1\text{ instance} \times 1\text{ vCPU} \times (730\text{ hours} \times 0.15) \times \$0.064 = \$7.01\text{ / month}$$
3. **Total Monthly Cost:** **$17.23 / month**

---

## 5. AWS Free Tier Coverage
* **Compute:** No free tier for App Runner compute duration.
* **Builds:** No free tier for build minutes.

---

## 6. Common Cost Hotspots & Pitfalls
* **Over-Allocated Memory on Low-Traffic Apps:** Assigning high memory specs (e.g., 4 GB) to low-traffic services. Even when vCPU scales to zero during idle periods, memory is billed 24/7 ($20.44/month per idle 4 GB instance).
* **High Build Minute Consumption on Frequent Commits:** Enabling automatic builds on all repository commits or branch pushes.
* **Low Concurrency Settings:** Setting `MaxConcurrency` too low forces App Runner to scale out to multiple parallel instances under light traffic loads.

---

## 7. Actionable Cost Optimization Strategies
1. **Plan Migration to AWS Fargate / ECS:** Evaluate migrating existing App Runner applications to **AWS Fargate (Amazon ECS)** or **AWS Amplify Gen 2** to align with current AWS container architecture patterns.
2. **Right-Size Provisioned Memory:** Review memory utilization. Reducing instance memory configuration from 2 GB to 1 GB cuts the 24/7 baseline memory charge in half (from $10.22/mo to $5.11/mo).
3. **Deploy Pre-Built Images from ECR:** Build Docker images in CI/CD pipelines (e.g., GitHub Actions) and push to **Amazon ECR**. Configure App Runner to deploy from ECR to eliminate $0.005/min build charges.
4. **Optimize Concurrency Auto Scaling:** Raise `MaxConcurrency` settings for asynchronous or fast I/O workloads so more requests share warm active instances before triggering scale-out events.
