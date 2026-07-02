# AWS Service Cost Research: Amazon Inspector

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Inspector is an automated vulnerability management service that continually scans AWS workloads for software vulnerabilities (CVEs) and unintended network exposure. Inspector automatically discovers and scans Amazon EC2 instances, container images in Amazon ECR, and AWS Lambda functions. Inspector operates on a serverless consumption model based on the number of active resources monitored and scanned.

---

## 2. Billing Mechanics
Inspector billing is calculated monthly based on resource coverage hours and scan events:
1.  **EC2 Instance Scanning:** Billed monthly per active EC2 instance covered by continuous monitoring.
2.  **ECR Image Scanning:** Billed per initial container image scan on push, plus a small fee for automated continuous rescans.
3.  **Lambda Function Scanning:** Billed monthly per active Lambda function assessed.
4.  **CIS Benchmark Assessments:** Billed per assessment run per EC2 instance.

---

## 3. Key Cost Dimensions

### A. Resource Scanning Pricing (us-east-1)
*   **EC2 Instance Scanning (Continuous Monitoring):**
    *   *Agent-based (via SSM Agent):* **$1.25 per instance-month** (prorated hourly at ~$0.0017/hour).
    *   *Agentless Scanning:* **$1.75 per instance-month**.
    *   *CIS Benchmark Assessment:* **$0.03 per assessment** per instance.
*   **ECR Container Image Scanning:**
    *   *Initial Scan on Push:* **$0.09 per image** pushed to ECR.
    *   *Automated Rescans (Continuous CVE Updates):* **$0.01 per rescan**.
*   **AWS Lambda Function Scanning:**
    *   *Standard Code & Dependency Scan:* **$0.30 per function-month** (prorated hourly at ~$0.00041/hour).
    *   *Standard Code-Only Scan:* **$0.09 per function-month**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Monitored Resource Type | Billing Unit | Rate (us-east-1) | Price for 100 Units / Month |
|-------------------------|--------------|------------------|-----------------------------|
| **EC2 Instance (Agent-based)**| Per instance-month | **$1.25** | **$125.00** |
| **ECR Image (Initial Push)** | Per image scan | **$0.09** | **$9.00** |
| **ECR Image (Rescan)** | Per rescan | **$0.01** | $1.00 |
| **Lambda Function** | Per function-month | **$0.30** | **$30.00** |
| **CIS Assessment** | Per run per instance| **$0.03** | $3.00 |

---

## 5. AWS Free Tier Coverage
*   **Amazon Inspector Free Trial:** 15-day free trial for all new accounts, allowing full vulnerability scanning across EC2, ECR, and Lambda resources.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Scanning Ephemeral Build Images in CI/CD (The ECR Trap):**
    *   Pushing dynamic git-commit-SHA container image tags (e.g., 50 build pushes/day) to Amazon ECR with Inspector ECR Scanning enabled for *all* repositories.
    *   *Math:* Pushing 10,000 temporary build images per month:
        $$10,000 \times \$0.09 = \$900.00\text{ / month in Inspector scan fees}$$
*   **Scanning Short-Lived Auto-Scaling EC2 Instances:** Running Inspector continuous scanning on high-churn Spot instances or auto-scaling nodes that launch and terminate in minutes.
*   **Monitoring Non-Production Lambda Functions:** Enabling continuous Lambda code scanning ($0.30/function-mo) on thousands of experimental developer or testing Lambda functions.

---

## 7. Actionable Cost Optimization Strategies
1.  **Filter ECR Repository Scanning in Inspector Settings:**
    *   Go to Inspector settings -> **ECR Repository Filtering**.
    *   Configure scan rules to scan **ONLY production repository patterns** (e.g. `prod/*`) or release tags (e.g. `v*`).
    *   Exclude sandbox, test, and feature-branch repositories.
    *   **The Savings:** Slashes ECR image scanning bills by **80–95%** by bypassing temporary CI/CD build images.
2.  **Enforce ECR Lifecycle Rules:**
    *   Configure S3/ECR lifecycle policies to delete untagged or temporary feature-branch container images after 3 days.
    *   Deleting old images stops automated rescan fees ($0.01/rescan).
3.  **Disable Lambda Scanning in Non-Production Accounts:**
    *   Disable Inspector Lambda scanning in developer sandbox accounts.
    *   Keep Lambda scanning active only in staging and production accounts.
4.  **Use Agent-Based EC2 Scanning Over Agentless:** Ensure the **AWS Systems Manager (SSM) Agent** is installed and running on all EC2 instances. Agent-based scanning ($1.25/mo) is **28% cheaper** than agentless scanning ($1.75/mo).
