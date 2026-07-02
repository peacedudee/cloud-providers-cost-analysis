# AWS Service Cost Research: AWS CloudFormation

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS CloudFormation is an Infrastructure-as-Code (IaC) service that allows developers to model, provision, and manage AWS and third-party resources declaratively using JSON or YAML templates. CloudFormation manages resource dependency ordering, stack creation, updates, and automated rollbacks. CloudFormation management functions for native AWS resources are provided at **no additional charge**.

---

## 2. Billing Mechanics
1. **Standard AWS Resource Provisioning (`AWS::*`):** **100% Free ($0.00)**. Zero charge for stack creation, updates, deletions, StackSets cross-account deployments, or Change Sets. You pay only for underlying AWS resources provisioned by the template.
2. **Third-Party Extension Handlers:** Operations using custom registry resource providers (outside `AWS::`, `Amazon::`, `AMZN::`, `Alexa::`, or `Custom::` namespaces) incur small handler invocation and execution duration charges.

---

## 3. Key Cost Dimensions

| Feature / Resource Namespace | Billing Metric | Rate (us-east-1) | Price for 1,000 Operations |
|------------------------------|----------------|------------------|----------------------------|
| **AWS Native Resources (`AWS::*`)** | Per stack operation | **Free ($0.00)** | **$0.00** |
| **Third-Party Extension Handlers** | Per handler call | **$0.0009** | **$0.90** |
| **Extension Execution Duration** | Per second (>30s) | **$0.00008** | $0.08 per 1,000 extra secs |

---

## 4. Detailed Pricing Rates (us-east-1)

* **CloudFormation Base Management Fee:** $0.00 per month.
* **Stack Creation / Updates / Deletions:** $0.00.
* **StackSets Cross-Account & Cross-Region Deployments:** $0.00.
* **Change Sets Generation:** $0.00.

---

## 5. AWS Free Tier Coverage
* **AWS CloudFormation:** Always 100% free for all native AWS resource templates with no time, template count, or stack size limits.

---

## 6. Common Cost Hotspots & Pitfalls
* **Orphaned Failed Stacks (`ROLLBACK_FAILED`):** Stacks that encounter errors during deployment and get stuck in `ROLLBACK_FAILED` or `CREATE_FAILED` states with partially provisioned resources.
  * *Impact:* Partially created resources (Load Balancers, NAT Gateways, EC2 instances, EKS clusters) continue billing hourly until the stack is manually deleted.
* **Unmanaged Drift in Non-Production Infrastructure:** Leaving stale sandbox stacks running for months without auditing active stack inventory.

---

## 7. Actionable Cost Optimization Strategies
1. **Automate Scheduled Stack Teardowns for Dev Environments:**
   * Deploy developer sandbox environments exclusively using CloudFormation templates.
   * Combine CloudFormation with AWS EventBridge / Lambda to execute automated stack deletions (`DeleteStack`) at the end of each workday (e.g., 7 PM).
   * **The Savings:** Eliminates overnight and weekend compute burn, saving up to **65%** on sandbox infrastructure fees.
2. **Alert on and Delete Failed Stacks (`ROLLBACK_FAILED`):** Configure CloudWatch alarms or EventBridge rules to detect failed stack deployments immediately, enabling automated stack cleanup before orphan resources accumulate charges.
3. **Use Drift Detection to Audit Infrastructure:** Run **CloudFormation Drift Detection** regularly to identify out-of-band manual changes and ensure all resources remain tracked and cleanly removable via IaC.
