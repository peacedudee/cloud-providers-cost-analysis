# AWS Service Cost Research: Amazon Inspector

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Inspector is an automated vulnerability management service that continually scans AWS workloads for software vulnerabilities (CVEs) and unintended network exposure. Inspector automatically discovers and scans Amazon EC2 instances, container images in Amazon ECR, and AWS Lambda functions. Inspector operates on a serverless consumption model based on the number of active resources monitored and scanned.

> [!WARNING]
> **Amazon Inspector Classic** officially reached its end-of-support date on **May 20, 2026**. All users must migrate to the current version of Amazon Inspector, which provides continuous, automated scanning.

---

## 2. Billing Mechanics
Inspector billing is calculated monthly based on resource coverage hours and scan events:
1.  **EC2 Instance Scanning:** Billed monthly per active EC2 instance covered by continuous monitoring.
2.  **ECR Image Scanning:** Billed per initial container image scan on push, plus a small fee for automated continuous rescans.
3.  **Lambda Function Scanning:**
    *   *Lambda Standard Scanning:* Billed monthly per function for package dependencies.
    *   *Lambda Code Scanning:* Billed monthly per function for custom code analysis (requires Standard Scanning).
4.  **CIS Benchmark Assessments:** Billed per assessment run per EC2 instance.

---

## 3. Key Cost Dimensions

### A. Resource Scanning Pricing (us-east-1)
*   **EC2 Instance Scanning (Continuous Monitoring):**
    *   *Agent-based (via SSM Agent):* **$1.25 per instance-month** (prorated hourly at ~$0.0017/hour).
    *   *Agentless Scanning:* **$1.75 per instance-month** (prorated hourly).
    *   *CIS Benchmark Assessment:* **$0.03 per assessment** per instance.
*   **ECR Container Image Scanning:**
    *   *Initial Scan on Push:* **$0.09 per image** pushed to ECR.
    *   *Automated Rescans (Continuous CVE Updates):* **$0.01 per rescan**.
*   **AWS Lambda Function Scanning:**
    *   *Lambda Standard Scanning (Dependencies):* **$0.30 per function-month** (prorated hourly at ~$0.00041/hour).
    *   *Lambda Code Scanning (Custom Application Code):* **$0.60 per function-month** (prorated hourly at ~$0.00082/hour).
    *   *Combined Scan Rate:* **$0.90 per function-month** when both are enabled.

---

## 4. Detailed Pricing Rates (us-east-1)

| Monitored Resource Type | Billing Unit | Rate (us-east-1) | Price for 100 Units / Month |
|-------------------------|--------------|------------------|-----------------------------|
| **EC2 Instance (Agent-based)**| Per instance-month | **$1.25** | **$125.00** |
| **EC2 Instance (Agentless)** | Per instance-month | **$1.75** | **$175.00** |
| **ECR Image (Initial Push)** | Per image scan | **$0.09** | **$9.00** |
| **ECR Image (Rescan)** | Per rescan | **$0.01** | $1.00 |
| **Lambda Standard Scan** | Per function-month | **$0.30** | **$30.00** |
| **Lambda Code Scan** | Per function-month | **$0.60** | **$60.00** |
| **CIS Assessment** | Per run per instance| **$0.03** | $3.00 |

### Example Monthly Cost Calculation (ECR Push Trap vs Lambda Scan)
*Workload: A developer team triggers continuous integration builds that push 2,000 temporary container images to ECR each month. They also run vulnerability scanning on 50 Lambda functions (both Standard and Code scans enabled).*

*   **ECR Scanning Cost (Initial pushes):**
    $$\text{ECR Scan Cost} = 2,000\text{ images} \times \$0.09 = \$180.00$$
*   **Lambda Scanning Cost (Combined Standard + Code):**
    $$\text{Lambda Scan Cost} = 50\text{ functions} \times (\$0.30 + \$0.60)/\text{function-mo} = \$45.00$$
*   **Total Monthly Cost:** **$225.00/month**

---

## 5. AWS Free Tier Coverage
*   **Amazon Inspector Free Trial:** 15-day free trial for all new accounts, allowing full vulnerability scanning across EC2, ECR, and Lambda resources.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Scanning Ephemeral Build Images in CI/CD (The ECR Trap):** Pushing dynamic git-commit-SHA container image tags to Amazon ECR with Inspector ECR Scanning enabled for *all* repositories, resulting in high push-scan charges ($0.09/image).
*   **Agentless EC2 Surcharge:** Running agentless EC2 scanning on thousands of instances without installing the SSM Agent, incurring a **40% cost premium** ($1.75/mo vs $1.25/mo).
*   **Monitoring Non-Production Lambda Functions:** Enabling continuous Lambda code scanning ($0.60/function-mo) on thousands of experimental developer or testing Lambda functions.

---

## 7. Actionable Cost Optimization Strategies
1.  **Filter ECR Repository Scanning in Inspector Settings:**
    *   Go to Inspector settings -> **ECR Repository Filtering**. Configure scan rules to scan **ONLY production repository patterns** (e.g. `prod/*`) or release tags (e.g. `v*`). Exclude sandbox, test, and feature-branch repositories.
    *   **The Savings:** Slashes ECR image scanning bills by **80–95%** by bypassing temporary CI/CD build images.
2.  **Enforce ECR Lifecycle Rules:**
    *   Configure ECR lifecycle policies to delete untagged or temporary feature-branch container images after 3 days. Deleting old images stops automated rescan fees ($0.01/rescan).
3.  **Disable Lambda Scanning in Non-Production Accounts:**
    *   Disable Inspector Lambda scanning (especially Code Scanning) in developer sandbox accounts. Keep Lambda scanning active only in staging and production accounts.
4.  **Use Agent-Based EC2 Scanning Over Agentless:**
    *   Ensure the **AWS Systems Manager (SSM) Agent** is installed and running on all EC2 instances. Agent-based scanning ($1.25/mo) is **28% cheaper** than agentless scanning ($1.75/mo).
