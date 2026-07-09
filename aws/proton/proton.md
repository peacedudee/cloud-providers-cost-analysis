# AWS Service Cost Research: AWS Proton

> **Status:** ⚠️ Deprecated / Sunset Notice (Support ends October 7, 2026)
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Proton is an automated application delivery service for infrastructure-as-code (IaC) templates. It allows platform engineering teams to define, manage, and deploy standardized environment and service templates (combining CloudFormation, ECS, EKS, Serverless Lambda, and App Runner configurations) for developer self-service.
*   **Critical Status Notice:** AWS has announced the deprecation of AWS Proton. Support for AWS Proton will officially end on **October 7, 2026**.

---

## 2. Billing Mechanics
1.  **Core Orchestration Service:** **100% Free ($0.00)**. No fees for template creation, environment provisioning, or service updates.
2.  **Underlying AWS Resources:** Billed at standard rates for the infrastructure resources provisioned by Proton templates (e.g. CloudFormation stacks, ECS tasks, EKS clusters, load balancers, S3 template buckets).

---

## 3. Key Cost Dimensions

| Service Component | Billing Metric | Rate (us-east-1) | Cost Impact |
|-------------------|----------------|------------------|-------------|
| **Proton Orchestrator** | Per service/env | **Free ($0.00)** | **$0.00** |
| **Provisioned CloudFormation**| Per stack | Standard Rates | Driven by provisioned resources |
| **S3 Template Storage** | Per GB-month | $0.023 / GB-mo | Minimal storage fees |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **AWS Proton Service Fee:** $0.00 per month.
*   **Template Publishing:** $0.00.

---

## 5. AWS Free Tier Coverage
*   **AWS Proton:** Always 100% free for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Orphaned Environments Created via Proton:** Leaving old test environments provisioned via Proton active, where underlying ALBs ($18/mo), NAT Gateways ($32/mo), and EC2 instances continue billing hourly.

---

## 7. Actionable Cost Optimization Strategies
1.  **Prepare Migration Plan Before Deprecation (October 7, 2026):**
    *   Plan your platform engineering migration away from AWS Proton.
    *   Transition environment templates directly to **AWS CloudFormation StackSets**, **AWS CDK**, or **Terraform Modules** hosted in AWS CodeArtifact or GitHub.
    *   **The Benefit:** Ensures smooth platform operations before Proton API endpoints shut down.
2.  **Delete Orphaned Environments Before Migration:** Use the Proton console or CLI to delete unused development environments, automatically triggering CloudFormation stack teardowns to release underlying compute and networking resources.
