# AWS Service Cost Research: AWS GuardDuty

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS GuardDuty is an intelligent threat detection service that continuously monitors your AWS accounts, workloads, and data stored in S3 for malicious activity. It analyzes billions of events from multiple data sources—such as AWS CloudTrail management events, VPC Flow Logs, DNS query logs, S3 data events, EKS audit logs, and RDS login attempts. GuardDuty uses machine learning and threat intelligence feeds. It operates on a serverless pay-as-you-go consumption model based on log volume analyzed.

---

## 2. Billing Mechanics
GuardDuty charges are split across foundational log sources and optional protection modules:
1.  **CloudTrail Management Events:** Billed per 1 Million management events analyzed.
2.  **VPC Flow Logs & DNS Query Logs:** Billed per GB of log data processed.
3.  **S3 Protection:** Billed per 1 Million S3 data events analyzed.
4.  **EKS & Lambda Protection:** Billed per vCPU-month or per 1 Million executions/audit logs.
5.  **Malware Protection:** Billed per GB of EBS storage or S3 object scanned upon threat detection.

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
    *   *Note:* You do **not** need to enable VPC Flow Logs in CloudWatch to use GuardDuty. GuardDuty reads VPC flow logs directly from the network hypervisor abstraction layer without writing to CloudWatch.

### B. Optional Protection Modules
*   **S3 Data Event Protection:** Billed at **$0.50 per 1 Million S3 data events** (`GetObject`, `PutObject`).
*   **EKS Protection:**
    *   *EKS Audit Logs:* **$0.0015 per 1,000 audit logs**.
    *   *EKS Runtime Monitoring:* **$1.50 per vCPU-month** for active container nodes.
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
| **EKS Runtime** | Per vCPU-month | **$1.50** | $1.50 / node vCPU-month |
| **Malware Scan** | Per GB scanned | **$0.09** | $0.09 per GB |

---

## 5. AWS Free Tier Coverage
*   **GuardDuty 30-Day Free Trial:** Includes a 30-day free trial for all new accounts.
*   *Cost Estimate Feature:* During the 30-day trial, the GuardDuty console provides an exact dollar-for-dollar breakdown of what GuardDuty will cost across each log category once the trial ends.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Terabyte-Scale DNS Query Logs in High-Throughput EKS/ECS Clusters:** High-density Kubernetes clusters generating massive internal DNS lookup traffic (e.g. 5 TB of DNS logs/month). At $1.00/GB, DNS query log analysis alone bills **$3,250.00/month**.
*   **Enabling S3 Data Protection on High-Frequency Data Pipelines:** Activating S3 Protection on buckets receiving millions of micro-writes from Kinesis or Glue.
*   **Enabling EKS Runtime Protection on Non-Production Clusters:** Running $1.50/vCPU-month EKS runtime monitoring on ephemeral testing or staging clusters.

---

## 7. Actionable Cost Optimization Strategies
1.  **Use the 30-Day Free Trial Console Breakdown:**
    *   Enable GuardDuty and review the **Usage** tab in the GuardDuty console before day 30.
    *   Analyze the cost breakdown: identify if **DNS Logs**, **VPC Flow Logs**, or **S3 Protection** represents the majority of spend.
    *   **The Savings:** Gives exact visibility to target optimizations before any billing occurs.
2.  **Selective Feature Enablement (Disable Modules in Non-Prod):**
    *   Keep **CloudTrail Management Event Analysis** enabled across all accounts (cheap, critical security coverage).
    *   Disable optional modules like **S3 Protection**, **EKS Runtime Monitoring**, and **Malware Protection** in development and sandbox environments.
    *   **The Savings:** Cuts non-production GuardDuty bills by **70–80%**.
3.  **Reduce Internal Cluster DNS Noise:** Configure CoreDNS caching in Kubernetes clusters to cache local service lookups. This reduces external DNS resolution queries, cutting GuardDuty DNS log volumes ($1.00/GB).
4.  **Consolidate Centralized Management via AWS Organizations:** Designate a **GuardDuty Delegated Administrator** account. Use Organization auto-enablement policies to manage GuardDuty configurations across multi-account environments consistently.
