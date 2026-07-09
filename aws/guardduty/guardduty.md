# AWS Service Cost Research: AWS GuardDuty

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS GuardDuty is an intelligent threat detection service that continuously monitors your AWS accounts, workloads, and data stored in S3 for malicious activity. It analyzes billions of events from multiple data sources—such as AWS CloudTrail management events, VPC Flow Logs, DNS query logs, S3 data events, EKS/ECS audit logs, and RDS login attempts. GuardDuty uses machine learning and threat intelligence feeds. It operates on a serverless pay-as-you-go consumption model based on log volume analyzed.

---

## 2. Billing Mechanics
GuardDuty charges are split across foundational log sources and optional protection modules:
1.  **CloudTrail Management Events:** Billed per 1 Million management events analyzed.
2.  **VPC Flow Logs & DNS Query Logs:** Billed per GB of log data processed.
3.  **S3 Protection:** Billed per 1 Million S3 data events analyzed.
4.  **Runtime Monitoring (EC2, ECS, EKS):** Billed per vCPU-month for monitored workloads.
5.  **RDS Protection:** Billed per 1 Million RDS login attempts.
6.  **Lambda Protection:** Billed per 1 Million Lambda executions analyzed.
7.  **Malware Protection:** Billed per GB of EBS storage or S3 object scanned upon threat detection.

---

## 3. Key Cost Dimensions

### A. Foundational Log Analysis (us-east-1 Tiers)
*   **CloudTrail Management Events:**
    *   *First 10 Million events/month:* **$4.00 per 1 Million events**.
    *   *Next 90 Million events/month:* **$2.00 per 1 Million events**.
    *   *Above 100 Million events/month:* **$1.00 per 1 Million events**.
*   **VPC Flow Logs & DNS Query Logs (Combined volume):**
    *   *First 500 GB/month:* **$1.00 per GB**.
    *   *Next 2,000 GB/month:* **$0.50 per GB**.
    *   *Above 2,500 GB/month:* **$0.25 per GB**.
    *   *Note:* You do **not** need to enable VPC Flow Logs in CloudWatch to use GuardDuty. GuardDuty reads VPC flow logs directly from the network hypervisor abstraction layer.

### B. Runtime Monitoring (EC2, ECS, EKS)
Billed based on the number of vCPUs monitored per month (calculated by aggregating monitored hours):
*   **Tier 1 (First 500 vCPUs/month):** **$1.50 per vCPU-month**.
*   **Tier 2 (Next 4,500 vCPUs/month):** **$0.75 per vCPU-month**.
*   **Tier 3 (Above 5,000 vCPUs/month):** Further volume discounts apply.
*   **The VPC Flow Log Waiver:** When GuardDuty Runtime Monitoring is active on an EC2 instance or ECS container, GuardDuty **waives all charges for VPC Flow Logs analysis** for that specific instance. This prevents double-charging for network security logs.

### C. Other Protection Modules
*   **S3 Data Event Protection:** Billed at **$0.50 per 1 Million S3 data events** (`GetObject`, `PutObject`).
*   **RDS Protection:** Billed at **$0.80 per 1 Million RDS login attempts**.
*   **Lambda Protection:** Billed at **$0.60 per 1 Million Lambda invocations**.
*   **Malware Protection:** Billed at **$0.09 per GB** of EBS volume or S3 object scanned.

---

## 4. Detailed Pricing Rates (us-east-1)

| GuardDuty Module | Billing Unit | Rate (us-east-1) | Price per 1,000 Units / GB |
|------------------|--------------|------------------|----------------------------|
| **CloudTrail Events** | Per 1M events (Tier 1)| **$4.00** | $0.004 per 1,000 events |
| **VPC & DNS Logs** | Per GB (Tier 1) | **$1.00** | **$1.00 per GB** |
| **S3 Data Events** | Per 1M events | **$0.50** | $0.0005 per 1,000 events |
| **Runtime Monitoring**| Per vCPU-month (Tier 1)| **$1.50** | $1.50 / node vCPU-month |
| **Malware Scan** | Per GB scanned | **$0.09** | $0.09 per GB |

### Example Monthly Cost Calculation (VPC Flow Log Waiver Impact)
*Workload: A production web cluster runs on 100 EC2 instances. Each instance is a `c6i.xlarge` (4 vCPUs), representing 400 vCPUs total. Collectively, these instances generate 600 GB of VPC Flow Logs and DNS logs per month. We evaluate enabling Runtime Monitoring vs. standard log scanning.*

*   **Scenario A: Foundational Scanning Only (Logs billed standard):**
    *   CloudTrail Events (assume 5 million events):
        $$5\text{ Million events} \times \$4.00 = \$20.00$$
    *   VPC & DNS Logs (600 GB):
        $$\text{Logs Cost} = (500\text{ GB} \times \$1.00) + (100\text{ GB} \times \$0.50) = \$550.00$$
    *   Total Cost (Scenario A): **$570.00/month**
*   **Scenario B: Runtime Monitoring Enabled (Waives VPC Flow Logs charges):**
    *   CloudTrail Events (5 million events):
        $$5\text{ Million events} \times \$4.00 = \$20.00$$
    *   VPC & DNS Logs (assume DNS logs make up 100 GB; 500 GB VPC logs are waived):
        $$\text{Remaining Logs Cost} = 100\text{ GB} \times \$1.00 = \$100.00$$
    *   Runtime Monitoring Cost (400 vCPUs in Tier 1):
        $$\text{Runtime Cost} = 400\text{ vCPUs} \times \$1.50/\text{vCPU} = \$600.00$$
    *   Total Cost (Scenario B): **$720.00/month** (The net cost of adding agent-based runtime security is only $150.00/month after the $450.00 VPC logs waiver).

---

## 5. AWS Free Tier Coverage
*   **GuardDuty 30-Day Free Trial:** Includes a 30-day free trial for all new accounts.
*   *Cost Estimate Feature:* During the 30-day trial, the GuardDuty console provides an exact dollar-for-dollar breakdown of what GuardDuty will cost across each log category once the trial ends.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Terabyte-Scale DNS Query Logs in High-Throughput EKS/ECS Clusters:** High-density Kubernetes clusters generating massive internal DNS lookup traffic.
*   **Enabling S3 Data Protection on High-Frequency Data Pipelines:** Activating S3 Protection on buckets receiving millions of micro-writes from Kinesis or Glue.
*   **Enabling Runtime Protection on Non-Production Clusters:** Running $1.50/vCPU-month runtime monitoring on ephemeral testing or staging clusters.

---

## 7. Actionable Cost Optimization Strategies
1.  **Use the 30-Day Free Trial Console Breakdown:**
    *   Enable GuardDuty and review the **Usage** tab in the GuardDuty console before day 30 to identify if **DNS Logs**, **VPC Flow Logs**, or **S3 Protection** represents the majority of spend.
2.  **Selective Feature Enablement (Disable Modules in Non-Prod):**
    *   Keep **CloudTrail Management Event Analysis** enabled across all accounts. Disable optional modules like **S3 Protection**, **Runtime Monitoring**, and **Malware Protection** in development and sandbox environments.
    *   **The Savings:** Cuts non-production GuardDuty bills by **70–80%**.
3.  **Leverage the VPC Flow Log Waiver:**
    *   If you choose to enable **Runtime Monitoring** on EC2/ECS/EKS workloads, review your VPC Flow Log charges. Confirm that the VPC Flow Logs analysis fee is waived for these active instances.
4.  **Reduce Internal Cluster DNS Noise:**
    *   Configure CoreDNS caching in Kubernetes clusters to cache local service lookups.
    *   **The Savings:** Reduces external DNS resolution queries, cutting GuardDuty DNS log volumes ($1.00/GB).
5.  **Consolidate Centralized Management via AWS Organizations:** Designate a **GuardDuty Delegated Administrator** account. Use Organization auto-enablement policies to manage GuardDuty configurations across multi-account environments consistently.
