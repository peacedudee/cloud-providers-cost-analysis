# AWS Service Cost Research: AWS DevOps Guru

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS DevOps Guru is a machine learning-powered service that helps developers and operators improve application availability and performance. It automatically analyzes operational data, metrics, logs, and events from your AWS resources to identify anomalous behavior (e.g. latency spikes, resource exhaustion, database connection leaks) and recommends remediation actions. DevOps Guru is serverless and bills on a pay-as-you-go resource coverage model.

---

## 2. Billing Mechanics
DevOps Guru bills monthly based on the active resources it monitors:
1.  **Resource Hours Analyzed:** Billed hourly per monitored AWS resource.
2.  **Resource Groups:** Monitored resources are categorized into Group A (cheap/lightweight resources) and Group B (heavy/compute-intensive resources).
3.  **Active Monitoring Rule:** You are only billed for resources that are "active" (producing metrics, logs, or events) during the billing hour. Idle resources are not billed.
4.  **API Requests:** Charged for custom API dashboard or metric exports.

---

## 3. Key Cost Dimensions

### A. Resource-Hour Pricing (us-east-1 Tiers)
*   **Group A Services (e.g., AWS Lambda, S3 Buckets, DynamoDB tables, SQS queues, SNS topics):**
    *   *Rate:* **$0.0028 per resource-hour** (~$2.04 per active resource per month).
*   **Group B Services (e.g., Amazon EC2 instances, RDS databases, ECS/EKS tasks, API Gateways, NAT Gateways, ALB/ELB load balancers):**
    *   *Rate:* **$0.0042 per resource-hour** (~$3.06 per active resource per month).

### B. Math Example (Small Production Stack)
If you monitor an application stack consisting of:
*   4 Lambda functions (Group A): 4 × $2.04 = $8.16/month.
*   2 DynamoDB tables (Group A): 2 × $2.04 = $4.08/month.
*   2 RDS PostgreSQL databases (Group B): 2 × $3.06 = $6.12/month.
*   1 Application Load Balancer (Group B): 1 × $3.06 = $3.06/month.
*   *Total Monthly DevOps Guru Cost:* **$21.42/month**.

### C. API Calls
*   **The Rate:** **$0.00004 per API call** (e.g. `DescribeAnomaly`, `ListInsights`).

---

## 4. Detailed Pricing Rates (us-east-1)

| Resource Group | Monitored AWS Services | Rate per Resource-Hour | Approximate Rate per Month (24/7 Active) |
|----------------|------------------------|-------------------------|------------------------------------------|
| **Group A** | Lambda, S3, DynamoDB, SQS, SNS | **$0.0028** | **$2.044** |
| **Group B** | EC2, RDS, ECS, EKS, API Gateway, ALB, NAT | **$0.0042** | **$3.066** |

---

## 5. AWS Free Tier Coverage
*   **DevOps Guru Free Tier:** **7,200 resource-hours** free per month for Group A services, and **3,600 resource-hours** free per month for Group B services (first 3 months).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Enabling DevOps Guru Across All Account Resources:** Checking the "All resources in the current AWS account" option when setting up DevOps Guru. This automatically analyzes all sandbox, developer, and legacy testing resources, inflating bills with low-value alerts.
*   **Accumulating Costs from Ephemeral Resources:** Tracking dynamic scaling groups (e.g. hundreds of auto-scaling container tasks or EC2 nodes) that run briefly but generate active resource hours.
*   **Monitoring Inactive Stacks:** Leaving DevOps Guru enabled on legacy CloudFormation stacks or tags that are no longer actively developed or maintained.

---

## 7. Actionable Cost Optimization Strategies
1.  **Define Strict Resource Coverage (Using Tags or CloudFormation):**
    *   Do not select account-wide monitoring.
    *   Select **Filter by CloudFormation Stacks** or **Filter by AWS Tags**.
    *   Only apply monitoring to production tags (e.g. `Environment=Production`) or core critical application stacks.
    *   **The Savings:** Cuts monitoring bills by 60–90% by bypassing dev/test resources.
2.  **Prune Non-Critical Services from Tag Scopes:** If you have an application stack, evaluate if you need to monitor secondary components like SQS queues or S3 buckets. If not, exclude them from monitored tags to save $2.04/month per resource.
3.  **Disable Coverage on Auto-Scaling Compute Tiers:** For clusters with large auto-scaling spikes (e.g. ECS/EKS clusters), monitor the Load Balancer (ALB) and RDS database, but exclude individual container tasks to prevent variable resource hour billing.
4.  **Enforce Account Lifecycle Automation:** Clean up inactive CloudFormation stacks or tag groups in developer accounts to prevent idle resource monitoring charges.
