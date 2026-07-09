# AWS Service Cost Research: AWS DevOps Guru

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS DevOps Guru is a machine learning-powered service that helps developers and operators improve application availability and performance. It automatically analyzes operational data, metrics, logs, and events from your AWS resources to identify anomalous behavior (e.g. latency spikes, resource exhaustion, database connection leaks) and recommends remediation actions. DevOps Guru is serverless and bills on a pay-as-you-go resource coverage model.

---

## 2. Billing Mechanics
DevOps Guru bills monthly based on the active resources it monitors:
1.  **Resource Hours Analyzed:** Billed hourly per monitored AWS resource.
2.  **Resource Groups:** Monitored resources are categorized into Group A (lightweight resources) and Group B (heavy/compute-intensive resources).
3.  **Active Monitoring Rule:** You are only billed for resources that are "active" (producing metrics, logs, or events) during the billing hour. Idle resources are not billed.
4.  **API Requests:** Charged for custom API dashboard or metric exports.
5.  **Free Scoping Tools:** AWS Resource Groups (used to organize and scope the resources monitored by DevOps Guru) is a **100% free service ($0.00)**.

---

## 3. Key Cost Dimensions

### A. Resource-Hour Pricing (us-east-1 Tiers)
*   **Group A Services (e.g., AWS Lambda, S3 Buckets, DynamoDB tables, SQS queues, SNS topics):**
    *   *Rate:* **$0.0028 per resource-hour** (~$2.04 per active resource per month).
*   **Group B Services (e.g., Amazon EC2 instances, RDS databases, ECS/EKS tasks, API Gateways, NAT Gateways, ALB/ELB load balancers):**
    *   *Rate:* **$0.0042 per resource-hour** (~$3.06 per active resource per month).

### B. Example Monthly Cost Calculation
*Workload: A small production application stack is organized using AWS Resource Groups (Free). The stack contains 4 active Lambda functions (Group A), 2 active DynamoDB tables (Group A), 2 active RDS PostgreSQL databases (Group B), and 1 Application Load Balancer (Group B) running 24/7.*

*   **Group A Resource Cost:**
    $$\text{Lambda Fee} = 4\text{ resources} \times \$2.04\text{ / month} = \$8.16$$
    $$\text{DynamoDB Fee} = 2\text{ resources} \times \$2.04\text{ / month} = \$4.08$$
*   **Group B Resource Cost:**
    $$\text{RDS Fee} = 2\text{ resources} \times \$3.06\text{ / month} = \$6.12$$
    $$\text{ALB Fee} = 1\text{ resource} \times \$3.06\text{ / month} = \$3.06$$
*   **Total Monthly Cost:** **$21.42/month**

---

## 4. Detailed Pricing Rates (us-east-1)

| Resource Group | Monitored AWS Services | Rate per Resource-Hour | Approximate Rate per Month (24/7 Active) |
|----------------|------------------------|-------------------------|------------------------------------------|
| **Group A** | Lambda, S3, DynamoDB, SQS, SNS | **$0.0028** | **$2.044** |
| **Group B** | EC2, RDS, ECS, EKS, API Gateway, ALB, NAT | **$0.0042** | **$3.066** |
| **API Calls** | Custom dashboard queries | **$0.00004** | Per API request |

---

## 5. AWS Free Tier Coverage
*   **DevOps Guru Free Tier:** **7,200 resource-hours** free per month for Group A services, and **3,600 resource-hours** free per month for Group B services for the first 3 months. Includes 10,000 free API calls per month during the trial.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Account-Wide Monitoring Enabled:** Checking the "All resources in the current AWS account" option when setting up DevOps Guru. This automatically analyzes all sandbox, developer, and legacy testing resources, inflating bills with low-value alerts.
*   **Accumulating Costs from Ephemeral Auto-Scaling Tasks:** Tracking dynamic scaling groups (e.g. hundreds of auto-scaling container tasks or EC2 nodes) that run briefly but generate active resource hours.
*   **Integrated SNS Alerts Surcharges:** Forgetting that sending DevOps Guru anomaly notifications via Amazon SNS incurs standard SNS message delivery fees.

---

## 7. Actionable Cost Optimization Strategies
1.  **Define Strict Resource Coverage Using Free AWS Resource Groups:**
    *   Do not select account-wide monitoring.
    *   Create a free **AWS Resource Group** filtered by specific production tags (e.g. `Environment=Production`) or core critical application CloudFormation stacks.
    *   Configure DevOps Guru to monitor only this specific Resource Group.
    *   **The Savings:** Bypasses dev/test resource billing, cutting monitoring bills by **60–90%**.
2.  **Disable Monitored Coverage on Auto-Scaling Compute Tiers:**
    *   For clusters with large auto-scaling spikes (e.g. ECS/EKS clusters), monitor the Load Balancer (ALB) and RDS database, but exclude individual container tasks to prevent variable resource hour billing.
3.  **Prune Lightweight Services from Monitored Scopes:**
    *   If you have a large microservice pipeline, evaluate if you need to monitor secondary components like SQS queues or S3 buckets. Excluding them from monitored tags saves $2.04/month per resource.
