# AWS Service Cost Research: AWS Identity and Access Management (IAM)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Identity and Access Management (IAM) is a foundational security service that enables you to manage access to AWS services and resources securely. You use IAM to create and manage AWS users, groups, and roles, and use permissions to allow or deny access to AWS resources. IAM is a core control plane service and is provided at **no additional charge**. However, associated governance tools and network architectures can introduce indirect costs.

---

## 2. Billing Mechanics
1.  **Core IAM Service:** **100% Free ($0.00)**. There are no hourly charges, setup fees, or per-user subscription fees for creating users, roles, policies, or performing authorization checks.
2.  **IAM Access Analyzer:** Standard unused access analysis and policy generation are free. Custom policy validation checks incur minor usage fees.
3.  **Security Token Service (STS):** Global STS endpoints are free. Using regional private endpoints or cross-region calls can incur standard data transfer or VPC endpoint charges.

---

## 3. Key Cost Dimensions

### A. Free Core Components
*   **IAM Users, Groups, Roles, & Policies:** Billed at **$0.00**.
*   **MFA (Multi-Factor Authentication):** Virtual MFA applications (like Google Authenticator or AWS Virtual MFA) are free. (Physical YubiKeys/hardware tokens are purchased separately from hardware vendors).
*   **IAM Identity Center (formerly AWS SSO):** Free of charge for managing single sign-on access across AWS accounts and cloud applications.

### B. Indirect & Associated Costs

| Feature / Component | Billed Metric | Rate (us-east-1) | Cost Impact |
|---------------------|---------------|------------------|-------------|
| **Core IAM Operations** | Per request / user | **Free ($0.00)** | $0 base charge |
| **IAM Access Analyzer** | Policy generation | **Free ($0.00)** | Standard policy checks |
| **STS Regional Endpoints**| Cross-region egress | **$0.01 - $0.02 / GB** | Inter-region data transfer |
| **STS VPC Endpoint** | Per ENI hour | **$0.01 / hour** | ~$7.20/mo per AZ + $0.01/GB |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Identity Management:** $0.00 per user / group / role.
*   **Policy Evaluation:** $0.00 per API authorization check.
*   **Access Analyzer Standard:** $0.00 per finding / policy generated.

---

## 5. AWS Free Tier Coverage
*   **AWS IAM:** Always 100% free for all accounts with no time or volume expiration limits.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Cross-Region STS Token Calls:** Applications running in `eu-west-1` requesting temporary credentials from the global STS endpoint in `us-east-1` instead of using the regional STS endpoint. This incurs unnecessary cross-region network egress charges ($0.02/GB) and introduces latency.
*   **Unnecessary STS VPC Endpoints in Low-Traffic Subnets:** Deploying private AWS PrivateLink VPC Endpoints for STS in every single VPC and Availability Zone without auditing traffic volume ($7.20/mo per AZ = $21.60/mo per VPC).

---

## 7. Actionable Cost Optimization Strategies
1.  **Use Regional STS Endpoints:**
    *   Configure your AWS SDKs and CLI to use **Regional STS Endpoints** (e.g. `sts.us-east-1.amazonaws.com` or `sts.eu-west-1.amazonaws.com`) by setting the environment variable:
        `AWS_STS_REGIONAL_ENDPOINTS=regional`
    *   **The Savings:** Eliminates cross-region network egress charges for authentication requests and reduces request latency.
2.  **Use IAM Roles Over Long-Lived IAM User Access Keys:**
    *   Do not create IAM users with permanent access keys for EC2, Lambda, or ECS workloads.
    *   Use IAM Roles for EC2/ECS/Lambda execution.
    *   **The Benefit:** Completely free, eliminates key rotation maintenance overhead, and prevents credential leak risks that lead to run-away resource abuse bills.
3.  **Audit Unused Roles with IAM Access Analyzer (Free):**
    *   Enable **IAM Access Analyzer** to automatically scan for unused roles, access keys, and overly permissive policies.
    *   Prune stale roles to maintain security posture and reduce organizational complexity at $0 cost.
4.  **Consolidate STS VPC Endpoints:** For private subnets, evaluate whether traffic to STS can route through centralized transit gateways or shared endpoints rather than provisioning dedicated PrivateLink endpoints in every single isolated VPC.
