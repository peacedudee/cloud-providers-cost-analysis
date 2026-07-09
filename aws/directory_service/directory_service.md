# AWS Service Cost Research: AWS Directory Service

> **Status:** ✅ Research Complete & Ground Truth Verified  
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
4.  **Directory Sharing Surcharge:** When you share an AWS Managed Microsoft AD directory across multiple AWS accounts:
    *   An additional hourly fee is charged for **each additional AWS account** that is granted access.
    *   No sharing fee applies for sharing across multiple VPCs within the **same AWS account**.
    *   No sharing fee applies for the primary host AWS account where the directory is installed.
    *   *Note: Directory sharing is excluded from the 30-day free trial.*

---

## 3. Key Cost Dimensions

### A. AWS Managed Microsoft AD (us-east-1 Rates)
*   **Standard Edition (Supports up to 5,000 users or 30,000 directory objects):**
    *   *Rate:* **$0.12 per hour per Domain Controller**.
    *   *Minimum 2 DCs:* $0.24 per hour = **$175.20 per month base**.
    *   *Directory Sharing:* **$0.075 per hour** per shared account.
*   **Enterprise Edition (Supports up to 50,000 users or 500,000 directory objects):**
    *   *Rate:* **$0.40 per hour per Domain Controller**.
    *   *Minimum 2 DCs:* $0.80 per hour = **$584.00 per month base**.
    *   *Directory Sharing:* **$0.25 per hour** per shared account.

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

| Directory Type | Size / Tier | Hourly Rate (per DC) | Shared Account Hourly Fee | Base Monthly Cost (2 DCs, 24/7) |
|----------------|-------------|----------------------|---------------------------|---------------------------------|
| **AD Connector** | Small | **$0.05** | N/A | **$73.00** |
| **AD Connector** | Large | **$0.15** | N/A | **$219.00** |
| **Managed Microsoft AD** | Standard | **$0.12** | **$0.075** | **$175.20** |
| **Managed Microsoft AD** | Enterprise | **$0.40** | **$0.250** | **$584.00** |

### Example Monthly Cost Calculation
*Workload: A company deploys AWS Managed Microsoft AD Standard Edition. The directory is shared with 5 child AWS accounts for domain joining workloads, and runs continuously.*

*   **Baseline Directory Cost (2 DCs):**
    $$\text{Base Cost} = 2\text{ DCs} \times \$0.12/\text{hour} \times 730\text{ hours} = \$175.20$$
*   **Directory Sharing Cost:**
    $$\text{Sharing Cost} = 5\text{ shared accounts} \times \$0.075/\text{hour} \times 730\text{ hours} = \$273.75$$
*   **Total Monthly Cost:** **$448.95/month**

---

## 5. AWS Free Tier Coverage
*   **AWS Directory Service Free Trial:** A 30-day free trial is available for new accounts, providing up to **1,500 free hours** of Managed Microsoft AD (Standard or Enterprise) or AD Connector. 
    *   *Warning: Directory sharing fees ($0.075 to $0.25 per hour) are excluded from the free trial and will generate standard billing immediately.*

---

## 6. Common Cost Hotspots & Pitfalls
*   **Unmonitored Directory Sharing Proliferation:**
    *   Sharing an AD directory with dozens of separate developer accounts in the organization, believing directory sharing is free.
    *   *Math:* Sharing Managed Microsoft AD Enterprise Edition with 30 child accounts:
        $$\text{Sharing Fee} = 30\text{ accounts} \times \$0.25/\text{hour} \times 730\text{ hours} = \$5,475.00/\text{month!}$$
*   **Provisioning Enterprise Edition for Small Workloads:** Launching Managed Microsoft AD Enterprise Edition ($584.00/mo base) for a small application stack with under 1,000 users where Standard Edition ($175.20/mo base) is sufficient.
*   **Running Duplicate AWS Directories in Dev/Test Accounts:** Creating separate AWS Managed AD directories in every AWS sandbox account instead of sharing a centralized directory.

---

## 7. Actionable Cost Optimization Strategies
1.  **Audit and Restrict Directory Sharing Scopes:**
    *   Only share your directory with AWS accounts that contain workloads (such as EC2 domain joins or FSx for Windows File Server) that strictly require Active Directory.
    *   **The Savings:** Eliminating 10 shared accounts on a Standard directory saves **$547.50/month**.
2.  **Use AD Connector If You Have On-Premises Active Directory:**
    *   Do not deploy a new Managed AD ($175–$584/mo) if you already operate an on-premises or EC2-hosted Active Directory. Deploy **AD Connector Small ($73.00/month)**.
    *   **The Savings:** Slashes baseline directory costs by **58–87%** while avoiding identity duplication.
3.  **Share Managed Directories Across Accounts (Seamless Domain Join):**
    *   Instead of provisioning separate directories in child accounts ($175.20/mo each), share a single central Managed AD directory.
    *   **The Savings:** Replaces a $175.20/mo directory fee with a $0.075/hr ($54.75/mo) sharing fee, saving **$120.45/month per account**.
4.  **Bypass AD for SSO (Use IAM Identity Center):**
    *   If your goal is single sign-on access to AWS accounts and applications (and not joining EC2 Windows instances to a domain), use **IAM Identity Center (100% Free)** instead of AWS Directory Service.
