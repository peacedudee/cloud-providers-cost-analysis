# AWS Service Cost Research: AWS Systems Manager (SSM)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Systems Manager (SSM) is an operational management cockpit for AWS and hybrid cloud environments. It provides visibility and control over EC2 instances, on-premises virtual machines, edge devices, and serverless infrastructure. Systems Manager includes Fleet Manager, Session Manager, Run Command, Patch Manager, State Manager, Parameter Store, Automation, Change Manager, and Incident Manager. Core management features for EC2 are **100% free**, while advanced management features bill on a consumption basis.

---

## 2. Billing Mechanics
1.  **Core EC2 Node Management & Session Manager:** **100% Free ($0.00)**. No charges for managing EC2 instances, executing SSH/shell sessions via Session Manager, or running patch updates.
2.  **Advanced On-Premises Instances:** Billed per instance-hour for on-premises servers or multi-cloud VMs registered via SSM Agent.
3.  **SSM Parameter Store:** Standard parameters are free; advanced parameters and high-throughput API calls incur small fees.
4.  **SSM Automation:** First 5,000 step-minutes free per month; excess step-minutes are billed per minute.

---

## 3. Key Cost Dimensions

### A. Core vs. Advanced Management (us-east-1 Rates)
*   **Standard EC2 Management (Session Manager, Run Command, Patch Manager):**
    *   *Cost:* **100% Free ($0.00)**.
*   **Advanced On-Premises Managed Instances:**
    *   *Rate:* **$0.0069 per instance-hour** (~$5.00 per instance-month).

### B. Parameter Store Pricing
*   **Standard Parameters (Size <= 4 KB, up to 10,000 per account):**
    *   *Storage Cost:* **100% Free ($0.00)**.
    *   *Standard API Throughput (up to 40 req/sec):* **100% Free ($0.00)**.
*   **Advanced Parameters (Size > 4 KB up to 8 KB, expiration policies):**
    *   *Storage Cost:* **$0.05 per parameter per month**.
*   **High Throughput API (`GetParameter` > 40 req/sec):**
    *   *API Cost:* **$0.05 per 10,000 requests** ($5.00 per Million).

### C. Automation & Operations Management
*   **SSM Automation Step-Minutes:**
    *   *First 5,000 step-minutes / month:* **Free ($0.00)**.
    *   *Additional step-minutes:* **$0.002 per step-minute**.
*   **Change Manager:** **$0.29 per change request**.
*   **Incident Manager:** **$7.00 per response plan per month** (includes 100 free SMS/voice notifications).

---

## 4. Detailed Pricing Rates (us-east-1)

| Systems Manager Component | Usage Level | Rate (us-east-1) | Monthly Cost Impact |
|---------------------------|-------------|------------------|---------------------|
| **Session Manager (EC2)** | Per session | **Free ($0.00)** | $0.00 |
| **Standard Parameter Store**| <= 4 KB, 40 req/sec | **Free ($0.00)** | **$0.00** |
| **Advanced Parameter Store**| Per parameter | **$0.05 / month**| $0.05 per parameter |
| **Parameter High Throughput**| Per 10k API calls | **$0.05** | $5.00 per 1M calls |
| **On-Premises Instance** | Per instance-hour | **$0.0069** | ~$5.00 / instance/mo |

---

## 5. AWS Free Tier Coverage
*   **AWS Systems Manager:** Core EC2 management and Standard Parameter Store are always 100% free with no time limits.
*   **Automation:** 5,000 free step-minutes per month.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Enabling High Throughput on Parameter Store Unnecessarily:** Enabling High Throughput ($0.05/10K requests) across accounts where application-level memory caching could easily handle parameter reads for free.
*   **Using AWS Secrets Manager for Non-Sensitive Config Parameters:** Using AWS Secrets Manager ($0.40/secret-mo + $0.05/10K calls) for non-sensitive application settings instead of **Standard Parameter Store ($0.00)**.

---

## 7. Actionable Cost Optimization Strategies
1.  **Use Standard Parameter Store Over Secrets Manager for Non-Sensitive Data:**
    *   Store environment variables, feature flags, and non-sensitive API endpoints in **Standard Parameter Store ($0.00)** instead of Secrets Manager ($0.40/secret-mo).
    *   **The Savings:** Slashes configuration storage bills to $0.00.
2.  **Cache Parameter Store Values in Application Memory:**
    *   Cache parameter values in application memory (TTL 15 mins) inside your Lambda functions or microservices.
    *   *Why:* Keeps request volume below 40 req/sec, avoiding the need to enable High Throughput API fees ($5.00/M requests).
3.  **Terminate SSH Bastion Host EC2 Instances:**
    *   Replace EC2 SSH bastion host instances (which cost $15.00 to $50.00/mo in compute and public IP fees) with **SSM Session Manager**.
    *   Session Manager provides secure, auditable browser and CLI shell access for **$0.00** without requiring public IP addresses.
    *   **The Savings:** Eliminates 100% of bastion instance compute and IP charges.
