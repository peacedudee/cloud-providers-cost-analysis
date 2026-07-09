# AWS Service Cost Research: AWS Security Hub

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Security Hub provides a comprehensive view of your security state in AWS and helps you check your environment against security industry standards and best practices. It aggregates, organizes, and prioritizes security findings from multiple AWS services (such as GuardDuty, Inspector, Macie, IAM Access Analyzer) and partner solutions. While Security Hub check rates seem small ($0.001/check), enabling redundant compliance standards triggers massive hidden **AWS Config** evaluation bills.

---

## 2. Billing Mechanics
Security Hub billing is calculated on a monthly consumption model:
1.  **Security Compliance Checks:** Billed per automated evaluation of a security control against an AWS resource.
2.  **Finding Ingestion Events:** Billed for receiving findings from third-party security integrations or internal services.
3.  **Hidden Dependency (AWS Config Rules):** Security Hub security standards rely on **AWS Config Rules** behind the scenes. Enabling Security Hub standards automatically enables AWS Config rules, which bill separately under AWS Config.

---

## 3. Key Cost Dimensions

### A. Security Check Pricing (us-east-1 Volume Tiers)
*   **First 100,000 security checks/month:** **$0.0010 per check** ($1.00 per 1,000 checks).
*   **Next 400,000 security checks/month:** **$0.0008 per check** ($0.80 per 1,000 checks).
*   **Above 500,000 security checks/month:** **$0.0005 per check** ($0.50 per 1,000 checks).

### B. Finding Ingestion Events
*   **The Rate:** **$0.00003 per finding event** ($0.03 per 1,000 findings).
*   *Note:* The first 10,000 finding ingestion events per month per account are **free ($0.00)**.

### C. The AWS Config Multiplier (Hidden Cost Trap)
*   Each control in a Security Hub standard (e.g. `[S3.1] S3 buckets should prohibit public read access`) invokes an **AWS Config Rule evaluation**.
*   If you enable 4 security standards (e.g., *AWS Foundational Security Best Practices*, *CIS AWS Foundations Benchmark*, *PCI-DSS*, and *NIST SP 800-53*):
    *   Security Hub evaluates the same S3 bucket 4 times against identical rules.
    *   You pay Security Hub 4 times AND you pay **AWS Config Rule Evaluation fees ($0.001 per evaluation)** 4 times!
    *   *Result:* Enabling redundant standards quadruples your Security Hub + AWS Config bill.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Dimension | Usage Tier | Rate (us-east-1) | Price per 10,000 Checks / Events |
|-------------------|------------|------------------|----------------------------------|
| **Compliance Checks**| 0 - 100,000 | **$0.00100** | **$10.00** |
| **Compliance Checks**| 100,000 - 500,000 | **$0.00080** | **$8.00** |
| **Compliance Checks**| 500,000+ | **$0.00050** | **$5.00** |
| **Finding Ingestion**| 10,000+ | **$0.00003** | **$0.30** |

---

## 5. AWS Free Tier Coverage
*   **Security Hub Free Trial:** 30-day free trial for all new accounts, including unlimited security checks and finding ingestions during the trial window.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Enabling All Security Standards Simultaneously:** Activating every available standard in Security Hub. This runs duplicate checks across identical resources, multiplying Security Hub and AWS Config costs by 400%.
*   **Enabling Security Hub in Ephemeral Regions:** Enabling Security Hub in unused AWS regions (e.g. `ap-east-1` or `sa-east-1`) where you have zero production resources. Default global resources (like IAM) get evaluated in every active region, incurring duplicate charges.
*   **Monitoring Sandbox & Developer Accounts with High Compliance Suites:** Running full PCI-DSS compliance checks on temporary developer sandbox accounts.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enable ONLY "AWS Foundational Security Best Practices":**
    *   Disable redundant standards like *CIS AWS Foundations Benchmark v1.2* or *PCI-DSS* (unless explicitly mandated by external auditors).
    *   *Why:* The **AWS Foundational Security Best Practices** standard covers 95%+ of all CIS controls.
    *   **The Savings:** Slashing redundant standards cuts Security Hub AND AWS Config evaluation costs by **60–75%**.
2.  **Disable Security Hub in Unused AWS Regions:**
    *   Go to the AWS Region Selector.
    *   Disable Security Hub in all AWS regions where your organization does not deploy infrastructure.
    *   **The Savings:** Prevents duplicate IAM and global resource evaluations across multiple regional endpoints.
3.  **Disable Specific Non-Critical Controls:**
    *   Review the Security Hub controls list.
    *   Disable controls for services you do not use (e.g. disable Redshift controls if your account uses zero Redshift clusters).
    *   This stops AWS Config from running evaluations on non-existent resources.
4.  **Consolidate Multi-Account Security via AWS Organizations:** Set up a centralized **Security Hub Delegated Administrator** account. This aggregates findings centrally, allowing you to manage compliance controls from a single pane of glass without logging into individual child accounts.
