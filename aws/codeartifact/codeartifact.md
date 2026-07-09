# AWS Service Cost Research: AWS CodeArtifact

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS CodeArtifact is a fully managed artifact repository service that enables organizations to store, publish, and share software package dependencies (npm, yarn, Maven, Gradle, pip, twine, NuGet, and Swift). CodeArtifact integrates natively with AWS IAM, CodeBuild, and external package registries. CodeArtifact is serverless, billing based on package storage volume, API requests, and data transfer.

---

## 2. Billing Mechanics
1. **Artifact Storage Volume:** Billed per GB of packages stored in CodeArtifact repositories ($0.05 per GB-month).
2. **API Data Requests:** Billed per 10,000 requests to upload, publish, or fetch packages ($0.05 per 10,000 calls).
3. **Free Tier:** First 2 GB of storage and 100,000 API requests per month are **100% Free ($0.00)**.
4. **Data Transfer:** In-region data transfer between CodeArtifact and AWS compute resources (EC2, CodeBuild, Lambda) in the same region is **100% Free ($0.00)**.

---

## 3. Key Cost Dimensions

### A. Core Rates (us-east-1)

| CodeArtifact Metric | Allowance / Tier | Rate (us-east-1) | Price for 100 GB / 1M Requests |
|---------------------|------------------|------------------|--------------------------------|
| **Artifact Storage** | First 2 GB Free | **$0.05 / GB-month** | **$4.90 / month** (98 GB paid) |
| **API Requests** | 1st 100k Free | **$0.05 / 10,000 calls** | **$4.50** (900k calls paid) |
| **In-Region Data Transfer** | Same region | **Free ($0.00)** | **$0.00** |

### B. Mathematical Cost Calculation Example
*Scenario: An engineering team stores 150 GB of npm and Maven packages in CodeArtifact and generates 2 Million API package requests per month.*

1. **Storage Charge (beyond 2 GB free):**
   $$\text{Storage Fee} = (150\text{ GB} - 2\text{ GB}) \times \$0.05 = \$7.40\text{ / month}$$
2. **API Request Charge (beyond 100k free):**
   $$\text{API Fee} = \frac{2,000,000 - 100,000}{10,000} \times \$0.05 = \$9.50\text{ / month}$$
3. **Total Monthly Cost:** **$16.90 / month**

---

## 4. Detailed Pricing Rates (us-east-1)

* **Storage Rate:** $0.05 per GB-month beyond 2 GB.
* **API Request Rate:** $0.05 per 10,000 requests ($5.00 per Million calls) beyond 100,000 free requests.

---

## 5. AWS Free Tier Coverage
* **AWS CodeArtifact Free Tier:** Includes **2 GB of storage** and **100,000 API requests per month** indefinitely for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
* **Accumulating Uncompressed CI/CD Build Snapshots:** Publishing temporary build snapshot packages (e.g., `1.0.0-SNAPSHOT-commitSHA`) on every git commit without automated retention policies, accumulating gigabytes of obsolete artifacts ($0.05/GB-mo).
* **Cross-Region Dependency Downloads:** Fetching packages from CodeArtifact repositories located in a different AWS region than build runners, incurring inter-region data transfer fees.

---

## 7. Actionable Cost Optimization Strategies
1. **Configure Package Lifecycle Deletion Rules:**
   * Implement automated scripts to delete non-release snapshot packages older than 14–30 days.
   * **The Savings:** Slashes CodeArtifact storage bills by **60%–80%**.
2. **Use In-Region VPC Endpoints for Build Agents:** Ensure CodeBuild runners and EC2 build agents connect to CodeArtifact via **AWS PrivateLink VPC Endpoints** in the same region to eliminate public internet or cross-region transfer fees.
3. **Cache External Upstream Dependencies:** Connect CodeArtifact repositories to public upstreams (e.g., npmjs, PyPI, Maven Central) so external packages are cached once locally.
