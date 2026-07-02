# AWS Service Cost Research: AWS License Manager

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS License Manager makes it easier to manage software licenses (from vendors such as Microsoft, Oracle, SAP, and IBM) across AWS and on-premises environments. It enables administrators to create customized licensing rules that emulate their commercial license agreements, track license usage, enforce hard or soft limits on instance launches, and automate license reporting during vendor audits. License Manager for AWS workloads is provided at **no additional charge**.

---

## 2. Billing Mechanics
1.  **AWS Resource License Tracking (EC2, RDS):** **100% Free ($0.00)**. No setup, rule configuration, or tracking fees for licenses running on AWS.
2.  **On-Premises Managed Server Tracking:** Billed hourly per on-premises server registered and tracked via SSM Agent.
3.  **User-Based Subscriptions:** Subscriptions for Microsoft Office or Visual Studio managed via License Manager bill per user/month according to vendor pricing.

---

## 3. Key Cost Dimensions

| Feature / Workspace | Billing Metric | Rate (us-east-1) | Cost Impact |
|---------------------|----------------|------------------|-------------|
| **AWS License Tracking (EC2/RDS)**| Per instance | **Free ($0.00)** | **$0.00** |
| **On-Premises Server Tracking** | Per instance-hour | **$0.0069** | ~$5.00 / instance/mo |
| **User-Based Subscriptions (Office)**| Per user / month | Vendor Rates | Subscription dependent |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **License Manager Base Fee:** $0.00 per month.
*   **AWS Workload License Rules:** $0.00.

---

## 5. AWS Free Tier Coverage
*   **AWS License Manager:** Always 100% free for all AWS cloud workloads with no license count limits.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Over-Licensing vCPUs on High-Memory Instances (The SQL Server Core Trap):**
    *   Deploying memory-intensive databases (e.g. Microsoft SQL Server Enterprise or Oracle Database) on large EC2 instances (like `r5.4xlarge` with 16 vCPUs and 128 GB RAM).
    *   Commercial SQL Server Enterprise edition is licensed at **~$3,700 per vCPU core**. Licensing 16 vCPUs costs:
        $$16\text{ vCPUs} \times \$3,700 = \$59,200.00\text{ in software licenses!}$$
*   **Exceeding Commercial License Limits (Audit Fines):** Launching auto-scaled EC2 instances that breach your Bring Your Own License (BYOL) agreement caps, resulting in severe vendor audit penalties.

---

## 7. Actionable Cost Optimization Strategies
1.  **Use EC2 "Optimize CPU" to Slash Commercial License Fees:**
    *   Use the EC2 **Optimize CPU** feature when launching database instances requiring high RAM but low CPU capacity.
    *   *Example:* Launch an `r5.4xlarge` (128 GB RAM) but specify `CpuOptions: { CoreCount: 2, ThreadsPerCore: 2 }` (activating only 4 vCPUs out of 16).
    *   **The Savings:** Reduces SQL Server license fees from 16 cores ($59,200) down to 4 cores ($14,800), saving **$44,400.00 (75% discount)** while preserving 100% of the 128 GB RAM!
2.  **Enforce Hard License Limits (`Hard Limit = True`):**
    *   Configure AWS License Manager rules with `Hard Limit = True`.
    *   If an Auto Scaling Group attempts to launch an EC2 instance that would exceed your available BYOL core licenses, License Manager automatically blocks the launch.
    *   **The Savings:** Prevents unbudgeted BYOL license overages and audit non-compliance fines.
3.  **Automate BYOL vs License Included (LI) Cost Comparison:** Use License Manager dashboard metrics to audit whether converting workloads from License Included (LI) to Bring Your Own License (BYOL) with Dedicated Hosts reduces total cost of ownership.
