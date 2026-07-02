# AWS Service Cost Research: Amazon EC2 (Elastic Compute Cloud)

> **Status:** ✅ Research Complete
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
    *   *EC2 Instance Savings Plans:* Less flexible, requires committing to a specific instance family in a specific region (e.g., `m6i` in `us-east-1`), but yields higher discounts.
*   **Reserved Instances (RIs):** Similar to Savings Plans, providing up to 72% discount for a 1- or 3-year commitment, but scoped to specific instance configurations (Standard or Convertible).
*   **Spot Instances:** Spare AWS compute capacity available at discounts of up to 90% off On-Demand rates. AWS can reclaim these instances with a 2-minute interruption notice. Recommended only for fault-tolerant, stateless, or batch workloads.

---

## 3. Key Cost Dimensions
Your total EC2 bill is comprised of several distinct billing dimensions:

### A. Compute Instances
Billed per-second (with a 60-second minimum) based on:
*   **Instance Family & Size:** CPU/Memory ratio (e.g., `t3` burstable vs. `c6i` compute-optimized).
*   **Operating System:** Linux/UNIX is the cheapest (no license fee). Windows and enterprise operating systems (RHEL, SLES) add premium license costs per hour.
*   **Tenancy:** Shared tenancy is standard. 
    *   *Dedicated Instances:* Run on single-tenant hardware at the hypervisor level. Incurs a flat regional fee of **$2.00 per hour** plus instance charges.
    *   *Dedicated Hosts:* You rent a physical server dedicated entirely to your use. Billed per hour for the physical host regardless of how many instances you run on it. Helpful for Bring-Your-Own-License (BYOL) compliance, but highly expensive if not densely populated.

### B. Storage (Amazon EBS - Elastic Block Store)
Every EC2 instance requires a root volume and optional data volumes. EBS storage is billed independently:
*   **Storage Volume Capacity:** Billed per GB-month provisioned (regardless of whether you write data to it).
*   **EBS Volume Type:**
    *   `gp3` (General Purpose SSD - Recommended): Billed at a baseline storage rate ($0.08/GB-month in `us-east-1`). Includes 3,000 IOPS and 125 MB/s throughput for free. Additional IOPS ($0.005/IOPS-month) and throughput ($0.04/MB/s-month) are billed.
    *   `gp2` (Older General Purpose SSD): Billed at $0.10/GB-month. Performance scales with volume size (3 IOPS per GB), which forces over-provisioning storage just to get performance.
    *   `io2` / `io2 Block Express` (Provisioned IOPS SSD): High performance. Billed for storage capacity plus a high rate per provisioned IOPS.
    *   `st1` (Throughput Optimized HDD) & `sc1` (Cold HDD): Cheap spinning disk options for large sequential data.
*   **EBS Snapshots:** Billed per GB-month for incremental data stored in S3 ($0.05/GB-month).

### C. Public IPv4 & Elastic IP Addresses
*   **The Charge:** AWS charges **$0.005 per hour** (~$3.60/month) for **every** public IPv4 address.
*   **Scope:** This charge applies to public IPv4 addresses assigned to running EC2 instances, elastic network interfaces (ENIs), NAT Gateways, and Load Balancers.
*   **Secondary Elastic IPs:** If you attach more than one Elastic IP to a single running EC2 instance, each additional (secondary) Elastic IP is billed at **$0.005 per hour** even while the instance is running.
*   **Unattached / Stopped Instance IPs:** Elastic IPs that are allocated to your account but either unattached or associated with a stopped EC2 instance are charged at the same **$0.005 per hour** rate.

### D. Data Transfer (Networking Egress)
*   **Inbound Data:** Free (data ingress from the internet).
*   **Intra-Region / Inter-AZ (Same Region):**
    *   Traffic between instances in the *same* Availability Zone using private IPs is **free**.
    *   Traffic between instances in *different* AZs in the same VPC/Region is billed at **$0.01 per GB in each direction** (sender pays $0.01, receiver pays $0.01 = $0.02/GB total).
*   **Inter-Region:** Traffic leaving an EC2 instance in one region to another AWS region is billed at **$0.01–$0.02 per GB** (varies by source/destination).
*   **Internet Egress:** Data transferred from EC2 out to the public internet is billed on a tiered scale:
    *   First 100 GB/month: **Free** (global account-wide allowance).
    *   Up to 10 TB/month: **$0.09 per GB**.
    *   Next 40 TB/month: **$0.085 per GB**.
    *   Next 100 TB/month: **$0.07 per GB**.

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
| **t4g.medium** (Graviton ARM) | 2 | 4 | $0.0336 | $24.53 |

*Note: The `t4g` instance (AWS Graviton2 ARM) is ~20% cheaper than the equivalent x86 `t3.medium` while offering equal or better performance.*

---

## 5. AWS Free Tier Coverage
*   **Compute:** 750 hours/month of `t2.micro` (or `t3.micro` in regions where `t2.micro` is unavailable) for the first 12 months.
*   **Storage:** 30 GB of total EBS storage (any combination of gp2/gp3 SSD or magnetic), plus 2 million I/Os and 1 GB of snapshot storage.
*   **Public IP:** 750 hours/month of public IPv4 address usage.
*   **Egress:** 100 GB/month of internet egress (always free).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Orphaned EBS Volumes:** Terminating an EC2 instance does *not* automatically delete its attached EBS volumes unless "Delete on Termination" was enabled. Unattached EBS volumes continue to generate GB-month storage charges indefinitely.
*   **Unused Elastic IP Addresses:** Keeping allocated Elastic IP addresses unattached or attached to stopped instances.
*   **NAT Gateways and Data Processing:** Routing all EC2 egress traffic (e.g., S3 access, API updates) through a NAT Gateway. NAT Gateways charge $0.045/hour plus a high processing fee of $0.045/GB.
*   **Using Public IPs for Internal Traffic:** Resolving DNS to a public IP instead of a private IP within the same VPC. This routes traffic through the internet border, triggering inter-AZ or egress fees.

---

## 7. Actionable Cost Optimization Strategies
1.  **Upgrade `gp2` to `gp3`:** Convert all EBS volumes to `gp3` immediately. This provides an instant **20% savings** per GB and lets you scale IOPS/throughput without buying extra storage.
2.  **Commitment Coverage:** Apply **Savings Plans** to your baseline, 24/7 workloads. Keep On-Demand usage for seasonal scaling spikes.
3.  **Leverage Spot Instances:** Transition stateless API tiers, worker queues, and CI/CD runners to Spot instances. Use ECS Capacity Providers or Auto Scaling groups to handle Spot terminations gracefully.
4.  **Adopt AWS Graviton (ARM):** Migrate x86 instances (e.g., `m6i`) to Graviton-based instances (e.g., `m7g` or `t4g`) for a **20% direct price reduction** and better performance per watt.
5.  **Use VPC Endpoints for S3/DynamoDB:** Create Gateway VPC Endpoints (which are free) to route S3 and DynamoDB traffic directly, bypassing NAT Gateways and avoiding both NAT hourly and per-GB charges.
6.  **Clean Up Storage:** Set up automation (e.g., AWS Backup or custom scripts) to find and delete unattached EBS volumes and aged snapshots.
