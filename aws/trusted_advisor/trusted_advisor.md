# AWS Service Cost Research: AWS Trusted Advisor

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Trusted Advisor is an automated online tool that acts as your customized cloud expert, inspecting your AWS environment and making recommendations to follow AWS best practices. Trusted Advisor evaluates resources across 5 categories: **Cost Optimization**, **Security**, **Fault Tolerance**, **Performance**, and **Service Limits**. Access to Trusted Advisor check suites is tied to your AWS Support plan level.

---

## 2. Billing Mechanics
1.  **Core Checks (Basic/Developer Support):** **100% Free ($0.00)**. Includes 7 core security checks and all service limits checks for all AWS account holders.
2.  **Full Check Suite (Business & Enterprise Support):** Included at no additional fee beyond your standard **AWS Support Plan** subscription:
    *   *AWS Business Support:* Minimum $100.00/month (or 10%–3% of monthly AWS spend).
    *   *AWS Enterprise On-Demand / Enterprise Support:* Included in plan baseline.

---

## 3. Key Cost Dimensions

| AWS Support Tier | Trusted Advisor Check Access | Cost Impact |
|------------------|------------------------------|-------------|
| **Basic Support** | Core Security (7 checks) + Service Limits | **Free ($0.00)** |
| **Developer Support**| Core Security + Service Limits | Included in Dev Support ($29/mo) |
| **Business Support** | **Full Suite (Cost Optimization + All Checks)**| Included in Business Support |
| **Enterprise Support**| Full Suite + Organizational View + Priority | Included in Enterprise Support |

---

## 4. Key Cost Optimization Checks in Trusted Advisor
Subscribing to Business/Enterprise Support unlocks automated scans for:
*   **Idle Load Balancers:** Detects ALBs/NLBs with zero request traffic over 7 days ($18.00+/mo per idle balancer saved).
*   **Unattached EBS Volumes:** Detects EBS volumes not attached to any EC2 instance ($0.08–$0.125/GB-mo saved).
*   **Underutilized Amazon EC2 Instances:** Identifies instances with <10% CPU utilization over 14 days.
*   **Underutilized Amazon RDS Instances:** Identifies database instances with zero connections over 7 days.
*   **Amazon ElastiCache / Redshift Idle Clusters:** Identifies idle memory & data warehouse nodes.

---

## 5. AWS Free Tier Coverage
*   **AWS Trusted Advisor:** Basic checks and Service Limit warnings are always 100% free for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Paying for AWS Support Without Executing Trusted Advisor Findings:** Subscribing to AWS Business Support ($100.00+/month) but failing to review or remediate the **Cost Optimization findings**, leaving thousands of dollars in idle resources unaddressed.

---

## 7. Actionable Cost Optimization Strategies
1.  **Automate Remediation of Trusted Advisor Cost Findings:**
    *   Use **AWS EventBridge** to capture Trusted Advisor `Cost Optimization` check events.
    *   Trigger automated **AWS Systems Manager (SSM) Automation Runbooks** to:
        *   Automatically delete unattached EBS volumes older than 7 days.
        *   Stop idle EC2 instances with <5% CPU over 14 days.
    *   **The Savings:** Translates Trusted Advisor alerts into immediate automated bottom-line savings.
2.  **Use Organizational View for Enterprise Visibility:** If running AWS Organizations, enable **Trusted Advisor Organizational View** in the Payer Account to aggregate cost-saving recommendations across all 100+ child accounts into a single dashboard.
