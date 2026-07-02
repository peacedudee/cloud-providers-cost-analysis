# AWS Service Cost Research: AWS CodeCommit

> **Status:** ⚠️ Deprecated for New Customers (Existing Workloads Only)  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS CodeCommit is a managed source control service that hosts private Git repositories, integrating natively with IAM, CodeBuild, CodePipeline, and CodeDeploy.
* **Critical Status Notice:** AWS closed CodeCommit to **new signups in mid-2024 / 2025**. Existing AWS accounts with active CodeCommit usage can continue operating the service.

---

## 2. Billing Mechanics
AWS CodeCommit operates on an active user monthly model for existing accounts:
1. **Active User Monthly Fee:** Billed per unique user who accesses a CodeCommit repository (via Git pull/push, CLI, or console) during a calendar month.
2. **First 5 Active Users:** **100% Free ($0.00)** per account per month (includes 50 GB storage and 10,000 Git requests).
3. **Additional Active Users:** $1.00 per active user per month (adds 10 GB storage and 2,000 Git requests to the account pool).
4. **Overage Storage & Requests:** Billed per GB of storage ($0.06/GB-mo) and per Git request ($0.001/req) above pool quotas.

---

## 3. Key Cost Dimensions

| CodeCommit Component | Allowance / Unit | Rate (us-east-1) | Price for 10 Active Users |
|----------------------|------------------|------------------|---------------------------|
| **First 5 Active Users** | 50 GB / 10k Git reqs | **Free ($0.00)** | **$0.00** |
| **Additional Active Users** | +10 GB / +2k Git reqs | **$1.00 / user-mo** | **$5.00 / month** (5 extra) |
| **Extra Storage Overage** | Per GB-month | **$0.06** | $0.06 per extra GB |
| **Extra Git Requests** | Per Git request | **$0.001** | $1.00 per 1,000 reqs |

---

## 4. Detailed Pricing Rates (us-east-1)

* **Active User Rate:** $1.00 per user-month beyond the 5 free users.
* **Storage Overage Rate:** $0.06 per GB-month.

---

## 5. AWS Free Tier Coverage
* **AWS CodeCommit Free Tier:** **5 active users** free per month indefinitely for all existing AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
* **Dedicated IAM Users for CI/CD Build Agents:** Using dedicated IAM users for automated build scripts that clone repositories, pushing user counts past the 5 free user threshold ($1.00/user-mo).

---

## 7. Actionable Cost Optimization Strategies
1. **Authenticate CI/CD Agents Using IAM Roles (Not IAM Users):**
   * Configure CodeBuild or EC2 build runners to authenticate via **IAM Roles** rather than IAM User credentials when cloning CodeCommit repositories.
   * *Benefit:* IAM Roles do not count as billable "active workforce users".
   * **The Savings:** Keeps active workforce user counts below 5 ($0.00 total bill).
2. **Plan Migration to GitHub or GitLab:**
   * Plan a migration of legacy CodeCommit repositories to GitHub or GitLab for long-term source control.
   * **The Savings:** Prepares workloads for eventual service sunsetting while eliminating overage storage fees.
