# AWS Service Cost Research: Amazon EC2 (Elastic Compute Cloud)

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon EC2 provides resizable virtual servers (instances) in the cloud. It is the bedrock of most AWS architectures and typically the single largest contributor to an enterprise's compute spend.

---

## 2. Billing Mechanics
EC2 uses several consumption models with varying pricing structures:
*   **On-Demand:** Pay for compute capacity by the second (minimum of 60 seconds) or by the hour, depending on the OS (Linux, Windows, macOS). There are no long-term commitments.
*   **Savings Plans:** Commit to a consistent amount of usage (measured in $/hour) for a 1- or 3-year term. Provides up to 72% discount on compute usage (EC2, Fargate, Lambda).
    *   *Compute Savings Plans:* Most flexible, applies automatically across any region, instance family, OS, or tenancy.
    *   *EC2 Instance Savings Plans:* Less flexible, requires committing to a specific instance family in a region (e.g., `m6i` in `us-east-1`), but yields higher discounts.
*   **Reserved Instances (RIs):** Similar to Savings Plans, providing up to 72% discount for a 1- or 3-year commitment, but scoped to specific instance configurations.
*   **Spot Instances:** Spare AWS compute capacity available at discounts of up to 90% off On-Demand rates. AWS can reclaim these instances with a 2-minute interruption notice. Recommended only for fault-tolerant, stateless, or batch workloads.

---

## 3. Key Cost Dimensions

### A. Compute Instances
Billed per-second (with a 60-second minimum) based on:
*   **Instance Family & Size:** CPU/Memory ratio. The latest **AWS Graviton4-based (m8g, c8g, r8g)** instance families offer up to **30% better price-performance** and are priced **~20% cheaper** than their x86 counterparts.
*   **Operating System & Licensing:** Linux/UNIX is the cheapest (no license fee). Windows and enterprise operating systems add licensing fees.
    *   *RHEL Pricing Model:* In mid-2024, Red Hat Enterprise Linux (RHEL) transitioned to a **per-vCPU-hour** pricing model (e.g., **$0.03 per vCPU-hour** for standard instances), replacing the old flat tier-based instance pricing.
*   **Tenancy:** Shared tenancy is standard.
    *   *Dedicated Instances:* Run on single-tenant hardware. Incurs a flat regional fee of **$2.00 per hour** plus instance charges.
    *   *Dedicated Hosts:* You rent a physical server dedicated entirely to your use. Billed per hour for the physical host regardless of how many instances you run on it. Helpful for Bring-Your-Own-License (BYOL) compliance.

### B. Storage (Amazon EBS)
*   **Storage Volume Capacity:** Billed per GB-month provisioned.
*   **EBS Volume Type:** `gp3` (General Purpose SSD) is recommended ($0.08/GB-month in `us-east-1` with 3,000 IOPS and 125 MB/s throughput included free). Legacy `gp2` volumes cost 25% more ($0.10/GB-month) and couple performance with capacity.

### C. Public IPv4 & Elastic IP Addresses
*   **Standard Hourly Charge:** AWS charges **$0.005 per hour** (~$3.60/month) for **every** public IPv4 address assigned to a running EC2 instance, network interface, NAT Gateway, or Load Balancer.
*   **Unattached / Stopped Instance IPs:** Elastic IPs allocated to your account but either unattached or associated with a stopped EC2 instance are charged at the same **$0.005 per hour** rate.
*   **Secondary Elastic IPs:** Any additional Elastic IP attached to a single running EC2 instance is billed at **$0.005 per hour**.

### D. Data Transfer (Networking Egress)
*   **Inbound Data:** Free (data ingress from the internet).
*   **Intra-Region / Inter-AZ (Same Region):**
    *   Traffic between instances in the *same* Availability Zone using private IPs is **free ($0.00/GB)**.
    *   Traffic between instances in *different* AZs in the same VPC/Region is billed at **$0.01 per GB in each direction** (sender pays $0.01, receiver pays $0.01 = $0.02/GB total).
*   **Inter-Region:** Traffic leaving an EC2 instance in one region to another AWS region is billed at **$0.01–$0.02 per GB**.
*   **Internet Egress:** Data transferred from EC2 out to the public internet is billed on a tiered scale:
    *   First 100 GB/month: **Free** (global account-wide allowance).
    *   Up to 10 TB/month: **$0.09 per GB**.
    *   Next 40 TB/month: **$0.085 per GB**.

---

## 4. Detailed Pricing Rates (us-east-1)
*Below are standard On-Demand rates for Linux (shared tenancy) in N. Virginia:*

| Instance Type | vCPU | Memory (GiB) | On-Demand Rate (/hr) | Monthly Cost (Est.) |
|---------------|------|--------------|----------------------|---------------------|
| **t3.micro** (Burstable) | 2 | 1 | $0.0104 | $7.59 |
| **t3.medium** (Burstable) | 2 | 4 | $0.0416 | $30.37 |
| **m6i.large** (General) | 2 | 8 | $0.0960 | $70.08 |
| **c6i.large** (Compute) | 2 | 4 | $0.0850 | $62.05 |
| **r6i.large** (Memory) | 2 | 16 | $0.1260 | $91.98 |
| **t4g.medium** (Graviton2) | 2 | 4 | $0.0336 | $24.53 |
| **c8g.large** (Graviton4) | 2 | 4 | $0.0680 | $49.64 |

*Note: The Graviton4 `c8g.large` instance ($0.0680/hr) is ~20% cheaper than the x86 `c6i.large` instance ($0.0850/hr) while offering superior price-performance.*

---

## 5. AWS Free Tier Coverage
*   **Compute:** 750 hours/month of `t2.micro` (or `t3.micro` in regions where `t2.micro` is unavailable) for the first 12 months.
*   **Storage:** 30 GB of total EBS storage (any combination of gp2/gp3 SSD or magnetic), plus 2 million I/Os and 1 GB of snapshot storage.
*   **Public IP:** 750 hours/month of public IPv4 address usage.
*   **Egress:** 100 GB/month of internet egress (always free).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Unused Elastic IP Addresses:** Keeping allocated Elastic IP addresses unattached or attached to stopped instances.
*   **Using Public IPs for Internal Traffic:** Resolving DNS to a public IP instead of a private IP within the same VPC. This routes traffic through the internet border, triggering inter-AZ or egress fees.
*   **Zombie EBS Volumes:** Terminating an EC2 instance without deleting its attached volumes, generating GB-month storage charges indefinitely.

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate to AWS Graviton4 Instances:**
    *   Transition x86 workloads (e.g., `c6i.large` at $0.062/mo compute rate equivalent) to Graviton4 instances (e.g., `c8g.large`).
    *   **The Savings:** Slashes compute bills by **20%** directly while improving performance.
2.  **Right-Size RHEL Instances Post-2024 Update:**
    *   Since RHEL licenses are now billed **per-vCPU-hour**, review instance sizes. Scale down instances with over-provisioned vCPUs to minimize license surcharges.
3.  **Upgrade EBS volumes from gp2 to gp3:**
    *   Convert all attached volumes to gp3.
    *   **The Savings:** Instantly saves **20%** on storage capacity charges.
4.  **Use VPC Endpoints for S3/DynamoDB:**
    *   Create Gateway VPC Endpoints (which are free) to route S3 and DynamoDB traffic directly, bypassing NAT Gateways and avoiding both NAT hourly and per-GB charges.
5.  **Release Idle Elastic IPs:**
    *   Set up an automated cleanup script to find Elastic IPs that are unattached or attached to stopped instances and release them to stop the $0.005/hour fee.
