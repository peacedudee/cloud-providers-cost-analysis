# Cost-Cutting Playbook: AWS CodeDeploy
> **Companion File:** [codedeploy.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/codedeploy/codedeploy.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS CodeDeploy is highly cost-efficient by default, as deployments to AWS infrastructure (Amazon EC2, AWS Fargate, and AWS Lambda) are completely free. The primary direct cost is $0.02 per update for on-premises instance deployments. However, the architectural choices orchestrated by CodeDeploy—such as duplicating fleets for Blue/Green deployments, storing artifacts in Amazon S3, and transferring data across network boundaries—can drive substantial indirect costs. This playbook provides 17 actionable strategies to eliminate deployment waste, optimize underlying compute resources, and minimize networking and storage overhead associated with CI/CD pipelines.

## Strategy Categories
### 1. Waste Elimination
#### CD-01. Aggressive Blue Fleet Termination in Blue/Green Deployments
- **What:** Configure the original (blue) environment to terminate as quickly as safely possible after a successful traffic shift.
- **Why It Saves Money:** A Blue/Green deployment doubles your EC2 compute cost while both environments are active. Shortening the wait time from hours/days to minutes cuts duplicate compute billing.
- **Implementation Steps:**
  1. Open the CodeDeploy console and navigate to the Deployment Group.
  2. Under Environment configuration, select Blue/green deployment.
  3. Set "Terminate the original instances in the deployment group" to a short duration (e.g., 5-15 minutes).
  4. Ensure automated CloudWatch alarms validate the green fleet rapidly to allow early termination.
- **Estimated Savings:** 10-40% of deployment-related EC2 compute costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Robust automated health checks and CloudWatch alarms.

#### CD-02. Clean Up Unused CodeDeploy Applications and Deployment Groups
- **What:** Audit and delete legacy or orphaned CodeDeploy applications and deployment groups.
- **Why It Saves Money:** While CodeDeploy groups are free, they often point to orphaned Auto Scaling groups, ELBs, or legacy S3 artifacts that do incur ongoing charges.
- **Implementation Steps:**
  1. Use AWS Config or custom scripts to list CodeDeploy deployment groups.
  2. Identify groups with no deployment activity in the last 90 days.
  3. Validate if the associated target environments (EC2, ASG) are still required.
  4. Delete the unused deployment groups and their underlying infrastructure.
- **Estimated Savings:** $50 - $500+ per month in associated infrastructure
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Deployment history logs.

#### CD-03. Implement S3 Lifecycle Policies for Deployment Revisions
- **What:** Automatically transition or expire old CodeDeploy application revisions stored in Amazon S3.
- **Why It Saves Money:** Development teams push frequent revisions. Storing thousands of outdated multi-megabyte/gigabyte artifacts in S3 Standard adds unnecessary storage costs.
- **Implementation Steps:**
  1. Identify the S3 bucket used for CodeDeploy revisions.
  2. Create an S3 Lifecycle rule targeting the revision prefixes.
  3. Transition non-current versions to S3 Standard-IA after 30 days.
  4. Expire/Delete revisions older than 90 days.
- **Estimated Savings:** 50-80% of CodeDeploy S3 storage costs
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** S3 Bucket Administrator access.

#### CD-04. Remove Stale On-Premises Instances from Deployment Groups
- **What:** Deregister decommissioned or offline on-premises instances from CodeDeploy.
- **Why It Saves Money:** CodeDeploy charges $0.02 per on-premises instance update per deployment. Deploying to stale or offline targets incurs unnecessary update fees and inflates deployment times.
- **Implementation Steps:**
  1. List all on-premises instances registered to CodeDeploy (`aws deploy list-on-premises-instances`).
  2. Identify instances that consistently fail deployments or are offline.
  3. Deregister these instances using `aws deploy deregister-on-premises-instance`.
- **Estimated Savings:** 10-30% of direct CodeDeploy charges
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Visibility into on-premises fleet health.

### 2. Rightsizing
#### CD-05. Optimize Application Revision Bundle Size
- **What:** Reduce the size of the deployment artifact (zip/tar file) uploaded to S3 and downloaded by targets.
- **Why It Saves Money:** Smaller bundles reduce S3 storage costs, lower Data Transfer charges, and minimize NAT Gateway processing fees during instance downloads.
- **Implementation Steps:**
  1. Audit CI/CD pipelines creating the CodeDeploy bundles.
  2. Exclude `.git` folders, local testing modules (e.g., `node_modules` dev dependencies), and temporary build files.
  3. Compress artifacts efficiently before S3 upload.
- **Estimated Savings:** 20-50% on artifact storage and transfer
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Access to CI/CD pipeline configuration.

#### CD-06. Use In-Place Deployments Instead of Blue/Green for Non-Prod Environments
- **What:** Switch lower environments (Dev/Test/QA) to In-Place deployments.
- **Why It Saves Money:** Blue/Green creates a parallel duplicate environment, doubling EC2 costs during the process. In-Place updates existing instances, avoiding the 2x capacity cost spike.
- **Implementation Steps:**
  1. Review Deployment Group settings for Dev/Test environments.
  2. Change deployment type from Blue/Green to In-Place.
  3. Configure deployment configurations like `CodeDeployDefault.OneAtATime` to maintain partial capacity during the update.
- **Estimated Savings:** 50% of EC2 cost during deployment windows
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Environments that can tolerate minor capacity reduction during deployment.

### 3. Commitment Discounts
#### CD-07. Leverage Compute Savings Plans for Underlying EC2 Fleets
- **What:** Cover the underlying EC2 Auto Scaling groups managed by CodeDeploy with Compute Savings Plans.
- **Why It Saves Money:** The compute infrastructure CodeDeploy targets comprises the bulk of actual costs. Compute SPs reduce EC2 rates by up to 66%.
- **Implementation Steps:**
  1. Analyze EC2 usage in AWS Cost Explorer.
  2. Identify steady-state baselines of CodeDeploy target ASGs.
  3. Purchase a Compute Savings Plan covering the aggregate baseline.
- **Estimated Savings:** 20-66% of EC2 compute costs
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Stable production workloads.

#### CD-08. Utilize Spot Instances for Green Fleet ASGs
- **What:** Configure the Auto Scaling group used for CodeDeploy targets to use Mixed Instances Policies with Spot instances.
- **Why It Saves Money:** Spot instances provide up to 90% discount compared to On-Demand, drastically lowering the cost of running both the primary fleet and the duplicate green fleet.
- **Implementation Steps:**
  1. Update the Launch Template for the CodeDeploy ASG.
  2. Configure a Mixed Instances Policy specifying a high percentage of Spot instances for Dev/Test, or a safe baseline for Prod.
  3. Ensure CodeDeploy target groups support Spot interruptions gracefully.
- **Estimated Savings:** 50-80% of EC2 compute costs
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Stateless application architecture tolerant to instance interruptions.

### 4. Architecture Changes
#### CD-09. Migrate On-Premises Deployments to AWS Cloud
- **What:** Move workloads from on-premises servers to Amazon EC2, Fargate, or Lambda.
- **Why It Saves Money:** CodeDeploy charges $0.02 per update for on-premises servers, but is **100% free** for all AWS targets.
- **Implementation Steps:**
  1. Evaluate on-premises workloads managed by CodeDeploy.
  2. Architect a migration plan to EC2 or containerize for AWS Fargate.
  3. Update CodeDeploy Deployment Groups to target AWS resources.
- **Estimated Savings:** 100% of direct CodeDeploy per-update fees
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | Leadership
- **Prerequisites:** Cloud migration readiness.

#### CD-10. Implement Automated Rollbacks via CloudWatch Alarms
- **What:** Tie CodeDeploy to CloudWatch Alarms to automatically roll back failing deployments.
- **Why It Saves Money:** A failed deployment running on a green fleet can consume compute resources indefinitely if left hanging. Automated rollbacks rapidly terminate faulty resources and restore steady state.
- **Implementation Steps:**
  1. Create CloudWatch Alarms monitoring HTTP 5xx errors or CPU spikes.
  2. Edit the CodeDeploy Deployment Group.
  3. Under Advanced > Alarms, add the CloudWatch alarms and enable automated rollbacks.
- **Estimated Savings:** Prevents hours of wasted compute on failed deployments
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Accurate CloudWatch metrics for application health.

#### CD-11. Transition to AWS Fargate or Lambda Targets
- **What:** Refactor EC2-based monolithic deployments to serverless (Lambda) or containerized (Fargate) targets.
- **Why It Saves Money:** Serverless architectures scale to zero and you only pay for actual execution, avoiding the idle EC2 costs CodeDeploy otherwise manages. CodeDeploy for Lambda/Fargate remains free.
- **Implementation Steps:**
  1. Assess application for microservices/serverless refactoring.
  2. Containerize application or rewrite as Lambda functions.
  3. Update CodeDeploy to use ECS/Lambda deployment configurations.
- **Estimated Savings:** 30-70% reduction in overall compute costs
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application refactoring effort.

#### CD-12. Use S3 VPC Endpoints for Artifact Downloads
- **What:** Ensure EC2 instances download CodeDeploy artifacts from S3 via a VPC Gateway Endpoint.
- **Why It Saves Money:** If instances download artifacts via a NAT Gateway, you pay $0.045/GB in processing fees. S3 Gateway Endpoints are free and eliminate NAT data transfer costs.
- **Implementation Steps:**
  1. Navigate to the VPC console.
  2. Create an S3 Gateway Endpoint.
  3. Attach it to the route tables of the private subnets hosting CodeDeploy instances.
- **Estimated Savings:** $0.045 per GB of deployment artifacts
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Instances in private subnets.

### 5. Scheduling & Auto-Scaling
#### CD-13. Schedule Blue/Green Deployments During Off-Peak Hours
- **What:** Execute Blue/Green deployments during periods of minimum Auto Scaling group size.
- **Why It Saves Money:** CodeDeploy Blue/Green clones the current capacity of the ASG. Deploying when the ASG is at 2 instances (off-peak) instead of 20 instances (peak) drastically reduces the duplicate EC2 compute cost during the deployment window.
- **Implementation Steps:**
  1. Analyze ASG scale-in/scale-out patterns in CloudWatch.
  2. Identify low-traffic windows (e.g., 2 AM).
  3. Schedule CI/CD pipeline triggers or maintenance windows for these periods.
- **Estimated Savings:** Up to 90% of deployment-window compute costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Automated testing and CI/CD pipelines capable of unattended releases.

#### CD-14. Suspend Deployments to Idle Non-Prod Environments
- **What:** Pause CodeDeploy pipelines for Dev/Test environments during nights and weekends.
- **Why It Saves Money:** If lower environments are scaled to zero during off-hours to save EC2 costs, attempting deployments will fail or spin up unnecessary capacity.
- **Implementation Steps:**
  1. Implement AWS Instance Scheduler to shut down non-prod instances off-hours.
  2. Ensure CI/CD pipelines respect scheduling windows and do not trigger CodeDeploy outside working hours.
- **Estimated Savings:** ~65% of non-prod compute costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Environment scheduling automation.

### 6. Pricing Model Optimization
#### CD-15. Consolidate On-Premises Instances to Reduce Per-Instance Fees
- **What:** Consolidate micro-workloads running on many small on-premises instances into fewer, larger instances.
- **Why It Saves Money:** CodeDeploy bills $0.02 per instance update regardless of instance size. Deploying to 50 small instances costs $1.00 per release, while 5 large instances cost $0.10.
- **Implementation Steps:**
  1. Review on-premises server utilization.
  2. Consolidate application deployments onto fewer, higher-density servers.
  3. Deregister old instances from CodeDeploy.
- **Estimated Savings:** Direct correlation to instance count reduction
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Excess capacity on target on-premises servers.

#### CD-16. Maintain In-Region S3 Artifacts to Avoid Cross-Region Transfer Fees
- **What:** Ensure CodeDeploy deployment artifacts are stored in an S3 bucket in the same AWS region as the deployment targets.
- **Why It Saves Money:** Cross-region Data Transfer out to EC2 costs $0.01 to $0.02 per GB. In-region data transfer from S3 to EC2 is free.
- **Implementation Steps:**
  1. Review CodeDeploy application settings and CI/CD S3 upload steps.
  2. Ensure the S3 bucket region matches the CodeDeploy application region.
  3. Replicate artifacts locally if deploying globally.
- **Estimated Savings:** $0.01 - $0.02 per GB of artifacts
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region CI/CD architecture review.

### 7. Network & Data Transfer Optimization
#### CD-17. Deploy VPC Endpoints for CodeDeploy Commands
- **What:** Use AWS PrivateLink (VPC Interface Endpoints) for CodeDeploy API communication.
- **Why It Saves Money:** Prevents CodeDeploy Agent heartbeat and command polling traffic from routing through expensive NAT Gateways ($0.045/GB + hourly fees).
- **Implementation Steps:**
  1. Open the VPC console and create Interface Endpoints for `com.amazonaws.[region].codedeploy` and `com.amazonaws.[region].codedeploy-commands`.
  2. Attach them to the private subnets where target instances reside.
- **Estimated Savings:** Reduces NAT Gateway processing charges
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Target instances residing in private VPC subnets.

---
## Cross-Service Synergies
- **Amazon EC2 & Auto Scaling:** CodeDeploy orchestrates EC2 fleets. Optimizing ASG sizes directly reduces CodeDeploy Blue/Green duplicate capacity costs.
- **Amazon S3:** CodeDeploy heavily relies on S3 for revision artifacts. S3 lifecycle policies and VPC Gateway Endpoints govern the storage and transfer costs of these deployments.
- **AWS CloudWatch:** Automated CodeDeploy rollbacks depend on CloudWatch Alarms. Fine-tuning alarms prevents extended billing of broken green deployments.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- **LineItem/ProductCode:** `AWSCodeDeploy`
- **LineItem/Operation:** Filter for on-premises instance updates.

### B. CloudWatch Metrics
- **DeploymentSuccessRate:** High failure rates indicate wasted compute during retries.
- **EC2 Instance Counts:** Monitor ASG size spikes during Blue/Green deployment windows.

### C. AWS Config / Trusted Advisor
- Audit for unused CodeDeploy Applications or Deployment Groups.

### D. Company Policies
- Retention policies for CodeDeploy revision artifacts in S3.
- Standard termination wait times for Blue fleets.

### E. IaC (Optional)
- Review Terraform/CloudFormation for CodeDeploy Deployment Group configuration (In-Place vs Blue/Green, termination wait times).

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "CD-01",
  "strategy_name": "Aggressive Blue Fleet Termination",
  "service": "AWS CodeDeploy",
  "category": "Waste Elimination",
  "estimated_savings_percentage": "10-40%",
  "risk_level": "Medium",
  "action_required": "Reduce Blue fleet termination timer to 5-15 minutes.",
  "effort": "Low"
}
```
### Summary Report Table
| ID | Strategy | Category | Savings | Risk | Effort |
|----|----------|----------|---------|------|--------|
| CD-01 | Aggressive Blue Fleet Termination | Waste Elimination | 10-40% | Medium | Low |
| CD-02 | Clean Up Unused CodeDeploy Groups | Waste Elimination | Moderate | Low | Low |
| CD-03 | S3 Lifecycle Policies for Revisions | Waste Elimination | 50-80% | Low | Low |
| CD-04 | Remove Stale On-Premises Instances | Waste Elimination | 10-30% | Low | Low |
| CD-05 | Optimize Revision Bundle Size | Rightsizing | 20-50% | Low | Medium |
| CD-06 | In-Place Deployments for Non-Prod | Rightsizing | 50% | Low | Low |
| CD-07 | Compute Savings Plans for Targets | Commitment Discounts | 20-66% | Low | Low |
| CD-08 | Spot Instances for Green Fleet ASGs | Commitment Discounts | 50-80% | High | Medium |
| CD-09 | Migrate On-Premises to AWS | Architecture Changes | 100% | High | High |
| CD-10 | Automated Rollbacks via Alarms | Architecture Changes | Moderate | Low | Medium |
| CD-11 | Transition to Fargate or Lambda | Architecture Changes | 30-70% | High | High |
| CD-12 | S3 VPC Endpoints for Artifacts | Architecture Changes | Moderate | Low | Low |
| CD-13 | Schedule Deployments Off-Peak | Scheduling & Auto-Scaling | Up to 90% | Low | Medium |
| CD-14 | Suspend Deployments to Idle Envs | Scheduling & Auto-Scaling | ~65% | Low | Medium |
| CD-15 | Consolidate On-Premises Instances | Pricing Model Optimization | High | Medium | Medium |
| CD-16 | Maintain In-Region S3 Artifacts | Pricing Model Optimization | Moderate | Low | Low |
| CD-17 | VPC Endpoints for CodeDeploy | Network & Data Transfer Optimization | Moderate | Low | Low |
```
