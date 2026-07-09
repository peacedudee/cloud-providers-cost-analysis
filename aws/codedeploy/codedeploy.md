# AWS Service Cost Research: AWS CodeDeploy

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS CodeDeploy is a fully managed deployment service that automates software deployments to compute targets including Amazon EC2 instances, AWS Fargate, AWS Lambda functions, and on-premises servers. CodeDeploy enables rapid release management while minimizing deployment downtime and managing update strategies (in-place updates, blue/green deployments, canary releases, and linear traffic shifting). CodeDeploy is **100% free** for all deployments to AWS cloud resources.

---

## 2. Billing Mechanics
1. **Deployments to AWS Compute (EC2, Fargate, Lambda, ECS):** **100% Free ($0.00)**. Zero deployment creation, agent, or update fees.
2. **Deployments to On-Premises Instances:** Billed per on-premises instance update ($0.02 per update).
3. **Underlying Infrastructure Costs:** Standard hourly rates apply for temporary EC2 replacement instances provisioned during blue/green deployments.

---

## 3. Key Cost Dimensions

| Deployment Target | Billed Metric | Rate (us-east-1) | Cost Impact |
|-------------------|---------------|------------------|-------------|
| **Amazon EC2 / ASG** | Per deployment | **Free ($0.00)** | **$0.00** |
| **AWS Fargate / ECS** | Per deployment | **Free ($0.00)** | **$0.00** |
| **AWS Lambda Functions** | Per deployment | **Free ($0.00)** | **$0.00** |
| **On-Premises Servers** | Per instance update | **$0.02** | $0.02 per server updated |

---

## 4. Detailed Pricing Rates (us-east-1)

* **AWS Resource Deployments:** $0.00.
* **On-Premises Instance Update Rate:** $0.02 per instance update per deployment.

---

## 5. AWS Free Tier Coverage
* **AWS CodeDeploy:** Always 100% free for all AWS cloud infrastructure deployments with no deployment count limits.

---

## 6. Common Cost Hotspots & Pitfalls
* **Leaving Original Blue Fleet Instances Running Post-Deployment:** Configuring blue/green deployments where the original (blue) instance fleet termination timer is set to days or never terminated, running double compute capacity.

---

## 7. Actionable Cost Optimization Strategies
1. **Leverage Free Canary & Linear Traffic Shifting for Lambda & ECS:**
   * Configure CodeDeploy **Canary (10% 10min)** or **Linear (10% every 1min)** deployment configurations for AWS Lambda and Fargate.
   * *Benefit:* CodeDeploy manages automated CloudWatch alarm rollbacks for **$0.00**, eliminating deployment outage risks without extra management fees.
2. **Set Short Termination Timers on Blue Fleet Instances:**
   * In EC2 Blue/Green deployment settings, configure the original (blue) instance group to terminate automatically **5 to 15 minutes** after traffic shifts to green.
   * **The Savings:** Prevents running duplicate EC2 instance fleets after deployment completes.
