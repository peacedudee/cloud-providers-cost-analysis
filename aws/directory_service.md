# AWS Service Cost Research: AWS Directory Service

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Directory Service provides managed Directory Service solutions for workloads requiring Microsoft Active Directory (AD). It allows your AWS resources (such as EC2 instances, WorkSpaces, QuickSight, and RDS) to join an AD domain, execute LDAP queries, and enforce Group Policy Objects (GPOs). AWS Directory Service is offered in three main modes: **AWS Managed Microsoft AD** (actual AD domain controllers), **AD Connector** (proxy gateway to on-premises AD), and **Simple AD** (Samba-based AD).

---

## 2. Billing Mechanics
AWS Directory Service is billed on an hourly provisioned model:
1.  **Domain Controller Runtimes:** Billed hourly per active Domain Controller (DC) or proxy instance.
2.  **Minimum Multi-AZ Requirement:** Every managed directory requires a minimum of **2 Domain Controllers** across 2 Availability Zones for high availability.
3.  **Directory Type and Tier:** Pricing varies based on the deployment type (Managed AD, AD Connector, Simple AD) and capacity size (Standard vs. Enterprise).

---

## 3. Key Cost Dimensions

### A. AWS Managed Microsoft AD (us-east-1 Rates)
*   **Standard Edition (Supports up to 5,000 users or 30,000 directory objects):**
    *   *Rate:* **$0.12 per hour per Domain Controller**.
    *   *Minimum 2 DCs:* $0.24 per hour = **$175.20 per month base**.
    *   *Includes:* 1 GB of directory database storage.
*   **Enterprise Edition (Supports up to 50,000 users or 500,000 directory objects):**
    *   *Rate:* **$0.40 per hour per Domain Controller**.
    *   *Minimum 2 DCs:* $0.80 per hour = **$584.00 per month base**.
    *   *Includes:* 17 GB of directory database storage.

### B. AD Connector (Proxy Gateway)
Proxies authentication requests directly to your existing on-premises or EC2 Active Directory.
*   **Small AD Connector (Under 10,000 users):**
    *   *Rate:* **$0.05 per hour per connector**.
    *   *Minimum 2 Connectors:* $0.10 per hour = **$73.00 per month base**.
*   **Large AD Connector (Over 10,000 users):**
    *   *Rate:* **$0.15 per hour per connector**.
    *   *Minimum 2 Connectors:* $0.30 per hour = **$219.00 per month base**.

### C. Simple AD (Samba 4 Powered)
*   *Notice:* Simple AD is **no longer open to new customers as of July 30, 2026**.
*   **Small Directory:** $0.05 per hour per DC ($0.10/hr = **$73.00 per month** for 2 DCs).
*   **Large Directory:** $0.15 per hour per DC ($0.30/hr = **$219.00 per month** for 2 DCs).

---

## 4. Detailed Pricing Rates (us-east-1 Base Costs)

| Directory Type | Size / Tier | Hourly Rate (per DC) | Minimum DCs | Base Monthly Cost (24/7) |
|----------------|-------------|----------------------|-------------|--------------------------|
| **AD Connector** | Small | **$0.05** | 2 | **$73.00** |
| **AD Connector** | Large | **$0.15** | 2 | **$219.00** |
| **Managed Microsoft AD** | Standard | **$0.12** | 2 | **$175.20** |
| **Managed Microsoft AD** | Enterprise | **$0.40** | 2 | **$584.00** |

---

## 5. AWS Free Tier Coverage
*   **AWS Directory Service:** 30-day free trial providing up to **1,500 free hours** across Managed Microsoft AD or AD Connector for new directory accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Provisioning Enterprise Edition for Small Deployments:** Launching Managed Microsoft AD Enterprise Edition ($584.00/mo) for a small application stack with under 1,000 users where Standard Edition ($175.20/mo) or AD Connector ($73.00/mo) is sufficient.
*   **Adding Unnecessary Additional Domain Controllers:** Scaling out Domain Controller counts (e.g., adding 4 extra DCs across secondary regions for minor read queries), scaling Enterprise Edition costs to **$1,168.00/month**.
*   **Running Duplicate AWS Directories in Dev/Test Accounts:** Creating separate AWS Managed AD directories in every AWS sandbox account instead of sharing a centralized directory via VPC Peering or AWS Transit Gateway.

---

## 7. Actionable Cost Optimization Strategies
1.  **Use AD Connector If You Have On-Premises Active Directory:**
    *   Do not deploy a new Managed AD ($175–$584/mo) if you already operate an on-premises or EC2-hosted Active Directory.
    *   Deploy **AD Connector Small ($73.00/month)**.
    *   **The Savings:** Slashes baseline directory costs by **58–87%** while avoiding identity duplication.
2.  **Select Managed AD Standard Edition Over Enterprise:** Choose **Standard Edition ($175.20/month)** for all workloads with fewer than 5,000 users or 30,000 objects. Upgrade to Enterprise only when storage or schema constraints require it.
3.  **Share Managed Directories Across Accounts (Seamless Domain Join):**
    *   Use the **AWS Directory Service Seamless Domain Join / Directory Sharing** feature.
    *   Share a single central Managed AD directory from your Management/Security account with multiple child AWS accounts.
    *   **The Savings:** Eliminates $175.20/mo directory fees in child accounts.
4.  **Bypass AD for SSO (Use IAM Identity Center):** If your goal is single sign-on access to AWS accounts and applications (and not joining EC2 Windows instances to a domain), use **IAM Identity Center (100% Free)** instead of AWS Directory Service.
