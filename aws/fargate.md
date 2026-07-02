# AWS Service Cost Research: AWS Fargate

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Fargate is a serverless, pay-as-you-go compute engine for containers. It works with both Amazon ECS and Amazon EKS. Fargate removes the operational overhead of provisioning, scaling, and patching virtual machine fleets, but requires careful configuration since you are billed directly for the resources you *request*, regardless of whether your container utilizes them.

---

## 2. Billing Mechanics
Fargate uses a granular, per-second billing model (with a 1-minute minimum). Charges are based on:
1.  **vCPU:** The number of virtual CPUs allocated to the task.
2.  **Memory:** The amount of RAM (in GB) allocated to the task.
3.  **Ephemeral Storage:** Any disk storage configured above the default 20 GB.

Billing begins when your task pulls the container image and runs until the task terminates.

---

## 3. Key Cost Dimensions

### A. vCPU and Memory Allocation & Sizing Constraints
*   **Requested vs. Used:** You pay for the resources defined in your ECS task definition or Kubernetes pod spec. If you configure a task with 2 vCPUs and 8 GB memory, but the app only consumes 5% CPU, you are still billed for the full 2 vCPUs and 8 GB.
*   **Resource Sizing Constraints:** Fargate only supports discrete combinations of vCPU and memory. CPU configurations (0.25, 0.5, 1, 2, 4, 8, 16 vCPUs) have specific valid memory ranges (e.g., 0.25 vCPU must have between 0.5 GB and 2 GB memory; 1 vCPU must have between 2 GB and 8 GB). If you configure an off-tier size, AWS will round it up to the next valid configuration, and you will be billed for the higher rounded-up tier.
*   **Architectures:**
    *   **x86 Architecture:** Standard container runtime.
    *   **ARM Architecture (AWS Graviton):** Runs on ARM-based Graviton processors, billed at a **20% discount** compared to x86.

### B. Fargate Spot
*   **The Discount:** Allows you to run interruption-tolerant tasks at up to a **70% discount** off standard Fargate rates.
*   **Mechanics:** Similar to EC2 Spot, AWS can reclaim Fargate Spot tasks when compute capacity is needed elsewhere. You receive a 2-minute warning before a task is terminated.

### C. Ephemeral Storage
*   Every Fargate task includes **20 GB of free ephemeral storage**.
*   You can provision up to 200 GB of ephemeral storage per task.
*   Storage allocated above the 20 GB baseline is billed at a rate of **$0.000111 per GB-hour** (approximately **$0.08 per GB-month** in `us-east-1`).

### D. Operating System Premiums
*   Linux is the baseline. Running **Windows Containers on Fargate** incurs an additional license fee per vCPU-hour.

### E. Public IPv4 Address Charges on Fargate
*   **The Charge:** Running Fargate tasks in a public subnet with a public IP assigned (which is a common default for ECS tasks to pull images or download patches) incurs the standard **$0.005 per hour** (~$3.60/month) public IPv4 fee for every active task.
*   **Scale Multiplier:** If you run a service scaled to 50 Fargate tasks with public IPs, this adds **$180.00/month** in pure IPv4 fees on top of compute and egress costs.

---

## 4. Detailed Pricing Rates (us-east-1)
*Below are the On-Demand rates for Linux tasks in N. Virginia:*

### A. Standard On-Demand Rates (Linux)
*   **x86 Architecture:**
    *   **vCPU:** $0.04048 per vCPU-hour ($0.0000112444 per vCPU-second)
    *   **Memory:** $0.004445 per GB-hour ($0.0000012347 per GB-second)
*   **ARM Architecture (Graviton):**
    *   **vCPU:** $0.03238 per vCPU-hour ($0.0000089944 per vCPU-second)
    *   **Memory:** $0.003560 per GB-hour ($0.0000009889 per GB-second)

### B. Fargate Spot Rates (Linux, x86)
*   **vCPU:** ~$0.012144 per vCPU-hour ($0.0000033733 per vCPU-second) — *70% discount*
*   **Memory:** ~$0.0013335 per GB-hour ($0.0000003704 per GB-second) — *70% discount*

### C. Example Monthly Cost Calculation
*Configuration: 2 tasks running 24/7 (730 hours/month) on Linux x86. Each task is configured with 1 vCPU and 2 GB memory.*

*   **vCPU Cost:**
    $$2\text{ tasks} \times 1\text{ vCPU} \times 730\text{ hours} \times \$0.04048 = \$59.10$$
*   **Memory Cost:**
    $$2\text{ tasks} \times 2\text{ GB} \times 730\text{ hours} \times \$0.004445 = \$12.98$$
*   **Total Cost:** **$72.08/month**

*If migrated to Graviton (ARM):*
*   **vCPU Cost:**
    $$2 \times 1 \times 730 \times \$0.03238 = \$47.27$$
*   **Memory Cost:**
    $$2 \times 2 \times 730 \times \$0.00356 = \$10.40$$
*   **Total Cost (Graviton):** **$57.67/month** (Save 20%)

---

## 5. AWS Free Tier Coverage
*   **Fargate:** There is **no free tier** for AWS Fargate compute. Any running Fargate task generates active billing from the first second.

---

## 6. Common Cost Hotspots & Pitfalls
*   **The "Copy-Paste" Configuration Waste:** Developers copy task configurations across services. A basic database connector running on 2 vCPUs and 8 GB RAM when it only needs 0.25 vCPU and 512 MB RAM wastes over 85% of its billed cost.
*   **NAT Gateway Data Processing Fees:** Fargate tasks running in private subnets that pull large container images from public registries or talk to external APIs. Each gigabyte pulled through a NAT Gateway incurs a $0.045/GB fee.
*   **Stuck Tasks Restarting in Infinite Loops:** A task that crashes immediately upon startup due to a config error, restarts, and crashes again. In EKS or ECS, the orchestrator will constantly restart it. While Fargate has a 1-minute minimum charge, continuous rapid restarts can still build up significant duration costs.
*   **Ignoring Ephemeral Storage Cleanup:** Configuring tasks with large ephemeral storage disks (e.g., 100 GB) for temporary batch files and leaving the tasks running 24/7. Storage is billed for every hour the task is active.

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate to Graviton (ARM):** Update the CPU architecture in your ECS task definition or EKS Pod configuration to `ARM64`. Ensure your container images are compiled for ARM. This provides a **20% direct price reduction** with equal or better performance.
2.  **Employ Fargate Spot:** Configure ECS Capacity Providers to route non-essential services, development environments, and batch queues to **Fargate Spot**. You can set a base percentage of tasks to run on On-Demand (for stability) and the remaining to scale on Spot.
3.  **Apply Compute Savings Plans:** AWS Fargate is covered by **Compute Savings Plans**. Committing to a baseline hourly spend over 1 or 3 years can save you up to **52%** on Fargate costs.
4.  **Right-Size Container Task CPU/Memory Limits:** Analyze CloudWatch metrics (CPUUtilization and MemoryUtilization) for your ECS services over a 2-week period. Scale down task sizes to match actual peaks plus a 20-30% buffer.
5.  **Use Private ECR and VPC Endpoints:** Ensure your container images are stored in a private ECR registry and set up ECR/S3 VPC interface endpoints. This routes image pull traffic internally rather than through NAT Gateways, avoiding high NAT transfer costs.
