# AWS Service Cost Research: AWS Identity and Access Management (IAM)

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Identity and Access Management (IAM) is a foundational security service that enables you to manage access to AWS services and resources securely. You use IAM to create and manage AWS users, groups, and roles, and use permissions to allow or deny access to AWS resources. IAM is a core control plane service and is provided at **no additional charge**. However, associated governance tools and network architectures can introduce indirect and direct usage costs.

---

## 2. Billing Mechanics
1.  **Core IAM Service:** **100% Free ($0.00)**. There are no hourly charges, setup fees, or per-user subscription fees for creating users, roles, policies, or performing authorization checks.
2.  **IAM Access Analyzer (External Access and Policy Validation):** Generating findings for resources shared externally (public or cross-account) and standard policy validation are **free**.
3.  **IAM Access Analyzer (Advanced Analysis - Paid):**
    *   *Unused Access Analysis:* Billed monthly based on the number of IAM users and roles monitored.
    *   *Custom Policy Checks:* Billed per API check performed via the IAM Access Analyzer APIs.
4.  **Security Token Service (STS):** Global STS endpoints are free. Using regional private endpoints or cross-region calls can incur standard data transfer or VPC endpoint charges.

---

## 3. Key Cost Dimensions

### A. Free Core Components
*   **IAM Users, Groups, Roles, & Policies:** Billed at **$0.00**.
*   **MFA (Multi-Factor Authentication):** Virtual MFA applications (like Google Authenticator or AWS Virtual MFA) are free.
*   **External Access Analysis & Console Policy Validation:** Billed at **$0.00**.

### B. Paid Access Analyzer Features (us-east-1)
*   **Unused Access Analyzer:** Identifies unused roles, access keys, and overly permissive policies.
    *   *Rate:* **$0.20 per IAM user or role** analyzed per month.
*   **Custom Policy Checks:** Validates developer policies against custom security standards.
    *   *Rate:* **$0.0020 per API check** (e.g. calling `CheckNoNewAccess` or `CheckAccessNotGranted`).
*   **Internal Access Analyzer:** Monitors internal access paths to critical resources.
    *   *Rate:* **$9.00 per AWS resource** monitored per region per month.

### C. STS Endpoints & Network Egress

| Feature / Component | Billed Metric | Rate (us-east-1) | Cost Impact |
|---------------------|---------------|------------------|-------------|
| **Core IAM Operations** | Per request / user | **Free ($0.00)** | $0 base charge |
| **Access Analyzer External**| Policy validation | **Free ($0.00)** | $0.00 |
| **Unused Access Analyzer** | Per IAM role/user / month| **$0.20** | Paid monthly |
| **Custom Policy Checks** | Per API check call | **$0.0020** | Paid per API check |
| **STS Regional Endpoints**| Cross-region egress | **$0.01 - $0.02 / GB** | Inter-region data transfer |
| **STS VPC Endpoint** | Per ENI hour | **$0.01 / hour** | ~$7.20/mo per AZ + $0.01/GB |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Identity Management:** $0.00 per user / group / role.
*   **Unused Access Rate:** $0.20 per identity monitored.
*   **Custom Policy Check Rate:** $0.0020 per API call.
*   **Internal Access Rate:** $9.00 per resource-month.

### Example Monthly Cost Calculation (Unused Access and Custom Checks)
*Workload: A company monitors unused permissions across 1,000 IAM roles and 200 IAM users in their production account. In addition, their CI/CD pipeline executes 50,000 Custom Policy Checks per month to block overly permissive policies.*

*   **Unused Access Analyzer Cost:**
    $$\text{Identities Billed} = 1,000\text{ roles} + 200\text{ users} = 1,200\text{ identities}$$
    $$\text{Unused Access Cost} = 1,200 \times \$0.20/\text{month} = \$240.00$$
*   **Custom Policy Checks Cost:**
    $$\text{Custom Checks Cost} = 50,000\text{ checks} \times \$0.0020 = \$100.00$$
*   **Total Monthly Direct IAM Cost:** **$340.00/month**

---

## 5. AWS Free Tier Coverage
*   **AWS IAM:** Core capabilities and external access analysis are always 100% free with no time limits.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Monolith Monitoring of Unused Access:** Enabling Unused Access Analyzer on large organization structures containing thousands of legacy, inactive, or temporary developer roles that do not require continuous monthly analysis ($0.20/identity-month).
*   **STS VPC Endpoint Proliferation:** Deploying dedicated STS VPC endpoints ($7.20/mo per AZ) across hundreds of small staging VPCs, resulting in high idle VPC endpoint charges when internet gateways or NAT gateways are already provisioned.

---

## 7. Actionable Cost Optimization Strategies
1.  **Configure Exclusions for Unused Access Analyzer:**
    *   Tag development, sandbox, or highly dynamic temporary roles to exclude them from the Unused Access Analyzer scan loop.
    *   **The Savings:** Reduces the billable IAM user/role count by **40–60%** in large developer accounts.
2.  **Use Regional STS Endpoints:**
    *   Configure SDKs and CLI to use regional endpoints (e.g. `sts.us-east-1.amazonaws.com`) by setting `AWS_STS_REGIONAL_ENDPOINTS=regional`.
    *   **The Savings:** Eliminates cross-region network egress charges for authentication requests.
3.  **Consolidate STS VPC Endpoints:**
    *   Route STS traffic from private subnets through a shared Central VPC or Transit Gateway endpoint.
    *   **The Savings:** Saves **$21.60/month** per VPC by eliminating redundant endpoint ENIs.
4.  **Optimize Custom Policy Check Invocations:**
    *   Trigger Custom Policy Checks in CI/CD pipelines only when pull requests modify the actual IAM policy files, rather than running scans on every single commit or testing build.
