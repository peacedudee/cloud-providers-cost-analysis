# Cost-Cutting Playbook: AWS Elastic Beanstalk
> **Companion File:** [elastic_beanstalk.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/elastic_beanstalk/elastic_beanstalk.md)
> **Last Updated:** July 2026

---
## Executive Summary
AWS Elastic Beanstalk itself is free to use, meaning all associated costs stem from the underlying resources it provisions on your behalf (EC2, ALB, EBS, RDS, S3). Therefore, optimizing Elastic Beanstalk revolves around configuring its environment settings to provision the most cost-effective architecture for your workload. By addressing common pitfalls like running load-balanced environments in development, unbounded S3 version history, and aggressive scaling triggers, organizations can drastically reduce their Beanstalk footprint without sacrificing developer velocity.

## Strategy Categories
### 1. Waste Elimination
Strategies focused on identifying and deleting unused or orphaned resources provisioned by Beanstalk.

### 2. Rightsizing
Strategies to ensure the compute, storage, and database resources provisioned match the actual application requirements.

### 3. Commitment Discounts
Financial strategies leveraging AWS discounting mechanisms (Savings Plans, RIs) for underlying Beanstalk resources.

### 4. Architecture Changes
Modifying environment configurations and application architecture to use more cost-effective services or modes.

### 5. Scheduling & Auto-Scaling
Tuning how Beanstalk reacts to traffic and implementing time-based start/stop mechanisms.

### 6. Pricing Model Optimization
Leveraging alternative consumption models like Spot Instances to run workloads cheaper.

### 7. Network & Data Transfer Optimization
Reducing bandwidth costs and optimizing data paths between Beanstalk and clients/other services.

---
## Cross-Service Synergies
*   **EC2 & EBS:** Beanstalk heavily relies on EC2 and EBS. Optimizations applied here directly reduce Beanstalk environment costs.
*   **Application Load Balancer (ALB):** Managing ALB usage (especially avoiding it in non-prod) creates significant savings.
*   **Amazon S3:** Beanstalk application versions are stored in S3. S3 lifecycle management is required to prevent unbounded storage costs.
*   **Amazon RDS:** While Beanstalk can provision RDS, decoupling it is a best practice for both cost management and data safety.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
Used to identify costs tagged with `elasticbeanstalk:environment-name` to separate Beanstalk-driven EC2/ALB/S3 costs from standard resources.
### B. CloudWatch Metrics
Essential for analyzing EC2 CPUUtilization, ALB RequestCount, and Auto Scaling metrics to determine rightsizing and scaling tuning opportunities.
### C. AWS Config / Trusted Advisor
Helps identify single-instance versus load-balanced environments, idle load balancers, and unutilized EC2 instances.
### D. Company Policies
Understanding the risk tolerance for using Spot instances and the requirements for high availability in lower environments.
### E. IaC (Optional)
CloudFormation, Terraform, or `.ebextensions` files to review how environment configuration is defined and applied.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "EB-001",
  "strategy": "Convert Non-Prod to Single-Instance Environments",
  "environment_id": "e-example123",
  "environment_name": "dev-app-env",
  "current_architecture": "LoadBalanced",
  "recommended_architecture": "SingleInstance",
  "estimated_monthly_savings": 25.00
}
```

### Summary Report Table

| Strategy | Risk Level | Est. Savings | Implementation Scope |
|----------|------------|--------------|----------------------|
| Convert Non-Prod to Single-Instance Environments | Low | 30-50% | Engineer/DevOps |
| Purge Stale Application Versions | Low | 10-20% | Engineer/DevOps |
| Migrate to Graviton Instances | Medium | ~20% | Engineer/DevOps |
| Utilize Spot Instances | Medium | Up to 90% | Engineer/DevOps |
| Decouple RDS Databases | Medium | Variable | FinOps Team, Engineer/DevOps |

---

#### 1. Convert Non-Prod to Single-Instance Environments
- **What:** Change the Environment Type for Development, QA, and Staging environments from "Load balanced" to "Single instance".
- **Why It Saves Money:** Eliminates the ~$16.42/month base charge for the Application Load Balancer and halves the minimum compute cost by dropping the required instances from 2 to 1.
- **Implementation Steps:**
  1. Open the Elastic Beanstalk console.
  2. Navigate to the target non-prod environment.
  3. Go to Configuration > Capacity.
  4. Change "Environment type" to "Single instance".
  5. Apply the changes (this will rebuild the environment).
- **Estimated Savings:** 30-50% per environment
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Environment must not require load balancing or SSL termination at the ALB level for testing.

#### 2. Purge Stale Application Versions (Lifecycle Policy)
- **What:** Configure an Application Version Lifecycle rule to automatically delete old deployment bundles stored in S3.
- **Why It Saves Money:** Beanstalk keeps every uploaded ZIP file in S3 forever by default. Deleting old versions saves $0.023 per GB-month of accumulated storage.
- **Implementation Steps:**
  1. Open the Elastic Beanstalk console.
  2. Go to the Application settings (not the environment).
  3. Under "Application version lifecycle", enable the policy.
  4. Set it to retain a maximum number of versions (e.g., 50) or a maximum age (e.g., 90 days).
  5. Check "Delete source bundle from S3".
- **Estimated Savings:** 10-20% of S3 costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 3. Migrate to Graviton (ARM) Instances
- **What:** Change the EC2 instance types in the Beanstalk configuration to Graviton-based instances (e.g., `t4g`, `c7g`, `m7g`).
- **Why It Saves Money:** Graviton instances provide up to 20% lower cost and up to 40% better price performance compared to equivalent x86 instances.
- **Implementation Steps:**
  1. Ensure your application dependencies and platform branch support ARM64.
  2. In the Beanstalk environment configuration, navigate to Instances or Capacity.
  3. Select Graviton instance types (e.g., `t4g.small`).
  4. Deploy and test the environment to ensure compatibility.
- **Estimated Savings:** ~20% of compute costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application code and runtime must be compatible with ARM64 architecture.

#### 4. Decouple RDS Databases
- **What:** Remove RDS databases managed directly within the Beanstalk environment and provision them independently via the RDS console.
- **Why It Saves Money:** While not a direct savings, decoupled databases prevent accidental deletion, allow independent scaling, and allow the use of RDS Reserved Instances without risking the reservation if the Beanstalk environment is recreated.
- **Implementation Steps:**
  1. Take a final snapshot of the Beanstalk-managed RDS instance.
  2. Restore the snapshot as a new, independent RDS instance.
  3. Update the Beanstalk environment properties (environment variables) to point to the new RDS endpoint.
  4. Remove the RDS instance from the Beanstalk configuration.
- **Estimated Savings:** Variable (Enables RI purchases and independent rightsizing)
- **Risk Level:** High (Data migration required)
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Maintenance window for database cutover.

#### 5. Implement Scheduled Scaling for Non-Prod
- **What:** Use Auto Scaling time-based scheduling to shut down load-balanced Beanstalk environments during non-working hours (e.g., nights and weekends).
- **Why It Saves Money:** Reduces compute costs by nearly 70% (168 hours in a week vs 40-50 working hours).
- **Implementation Steps:**
  1. Navigate to the Beanstalk environment's Capacity configuration.
  2. Add time-based scaling rules.
  3. Set a rule to scale Min/Max to 0 at 6:00 PM on weekdays.
  4. Set a rule to scale Min/Max to 1 or 2 at 8:00 AM on weekdays.
- **Estimated Savings:** 60-70% of compute costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Environment must be load-balanced (Single-instance environments do not support ASG scheduling directly; they require custom EventBridge/Lambda solutions).

#### 6. Utilize Amazon EC2 Spot Instances
- **What:** Configure the Beanstalk Auto Scaling group to use Spot instances for a portion or all of its compute capacity.
- **Why It Saves Money:** Spot instances offer up to 90% discounts compared to On-Demand prices for EC2 instances.
- **Implementation Steps:**
  1. Go to the environment's Capacity configuration.
  2. Under "Fleet composition", select "Combine purchase options and instances".
  3. Configure the base capacity for On-Demand and set the remaining percentage to Spot.
  4. Provide multiple instance types to increase Spot availability.
- **Estimated Savings:** Up to 90% on the Spot portion of compute
- **Risk Level:** Medium (Instance interruption)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload must be stateless, fault-tolerant, and capable of handling instance terminations.

#### 7. Tune Auto Scaling Group Thresholds
- **What:** Adjust the scaling triggers (e.g., CPU Utilization) to be less aggressive.
- **Why It Saves Money:** Prevents the environment from rapidly provisioning unnecessary instances during minor, temporary traffic spikes.
- **Implementation Steps:**
  1. Navigate to the Capacity configuration.
  2. Locate the "Scaling triggers" section.
  3. Increase the Upper threshold (e.g., from 40% to 70% CPU) and adjust the Lower threshold accordingly.
  4. Increase the "Breach duration" to ensure the spike is sustained before scaling.
- **Estimated Savings:** 10-30% of compute costs during burst periods
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Historical CloudWatch metrics to determine safe thresholds.

#### 8. EC2 Instance Type Downsizing
- **What:** Reduce the size of the EC2 instances running the application (e.g., moving from `t3.large` to `t3.medium`).
- **Why It Saves Money:** Smaller instances have lower hourly rates. Many web applications are over-provisioned by default.
- **Implementation Steps:**
  1. Analyze CloudWatch CPU and memory metrics for the environment.
  2. Identify environments running with peak utilization below 20%.
  3. Modify the environment configuration to select a smaller instance type.
  4. Apply the update and monitor performance.
- **Estimated Savings:** 50% per size reduction step
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Solid understanding of the application's CPU and memory requirements.

#### 9. Reduce Minimum ASG Instance Count
- **What:** Lower the "Min instances" setting in the capacity configuration if it is unnecessarily high.
- **Why It Saves Money:** The minimum instance count dictates the baseline compute cost. Reducing it directly cuts the baseline spend.
- **Implementation Steps:**
  1. Go to the Capacity configuration.
  2. Locate the "Min instances" setting.
  3. If currently set > 2 for an application with low baseline traffic, reduce it to 1 or 2.
- **Estimated Savings:** Linear reduction based on removed instances
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application must be able to handle cold starts if traffic suddenly increases from a low baseline.

#### 10. Terminate Abandoned EB Environments
- **What:** Identify and completely terminate Beanstalk environments that are no longer used.
- **Why It Saves Money:** Stops all billing for the associated EC2, ALB, EBS, and any attached resources.
- **Implementation Steps:**
  1. Audit Beanstalk environments by looking at network traffic and recent deployment dates.
  2. Confirm with the owning team that the environment is defunct.
  3. In the Beanstalk console, select the environment, click "Actions", and select "Terminate environment".
- **Estimated Savings:** 100% of the environment's cost
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Proper communication with engineering teams to avoid deleting active services.

#### 11. Optimize Deployment Policies (Avoid Unnecessary Capacity)
- **What:** For lower environments, switch the deployment policy from "Immutable" or "Traffic Splitting" to "Rolling" or "All at once".
- **Why It Saves Money:** Immutable and Blue/Green deployments temporarily double the compute capacity of the environment during the deployment window. Using simpler deployment methods prevents this temporary spike in EC2 charges.
- **Implementation Steps:**
  1. Go to Configuration > Rolling updates and deployments.
  2. For non-prod environments, set the Deployment policy to "All at once" or "Rolling".
  3. Apply the changes.
- **Estimated Savings:** Eliminates temporary double-billing during deployments
- **Risk Level:** Low (in Non-Prod)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Acceptance of potential downtime during deployment in non-prod.

#### 12. Apply Compute Savings Plans
- **What:** Purchase Compute Savings Plans to cover the EC2 compute usage generated by Beanstalk.
- **Why It Saves Money:** Savings Plans offer up to 72% discount on compute usage compared to On-Demand rates in exchange for a 1- or 3-year commitment.
- **Implementation Steps:**
  1. Analyze stable, baseline EC2 usage in AWS Cost Explorer.
  2. Determine the optimal hourly commitment in dollars.
  3. Purchase a Compute Savings Plan via AWS Cost Management.
  4. The discount automatically applies to the EC2 instances launched by Beanstalk.
- **Estimated Savings:** 20-72% of compute costs
- **Risk Level:** Medium (Financial commitment)
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Stable baseline usage and leadership approval for long-term commitments.

#### 13. Optimize ALB Access Logs (S3 Storage)
- **What:** If ALB access logs are enabled for the Beanstalk environment, configure an S3 lifecycle policy to transition or delete the logs after a set period.
- **Why It Saves Money:** ALB logs generate significant S3 storage over time. Deleting them or moving them to Glacier reduces storage fees.
- **Implementation Steps:**
  1. Identify the S3 bucket storing the ALB logs for the environment.
  2. Navigate to the S3 console and open the bucket.
  3. Go to the Management tab and create a Lifecycle rule.
  4. Set the rule to delete objects or transition them to Glacier after 30/90 days.
- **Estimated Savings:** 50-90% of log storage costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compliance team approval on log retention requirements.

#### 14. EBS Volume Optimization
- **What:** Modify the EBS volume type and size attached to the Beanstalk EC2 instances. Change older `gp2` volumes to `gp3`.
- **Why It Saves Money:** `gp3` volumes are up to 20% cheaper than `gp2` volumes and offer independent IOPS/throughput scaling.
- **Implementation Steps:**
  1. Navigate to the Beanstalk environment Configuration > Instances.
  2. In the Root volume settings, change the Volume type to `gp3`.
  3. Ensure the Volume size is not excessively large (e.g., default is often higher than needed for stateless web apps).
- **Estimated Savings:** ~20% of EBS costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application does not require specialized high-performance storage (io1/io2).

#### 15. Integrate Amazon CloudFront
- **What:** Place an Amazon CloudFront distribution in front of the Elastic Beanstalk environment.
- **Why It Saves Money:** CloudFront caches static assets at edge locations, reducing the number of requests hitting the ALB and EC2 instances, which lowers Data Transfer Out, ALB LCU charges, and compute load.
- **Implementation Steps:**
  1. Go to the CloudFront console and create a new distribution.
  2. Set the Elastic Beanstalk environment URL as the Origin.
  3. Configure cache behaviors for static assets (images, CSS, JS).
  4. Update DNS to route traffic to the CloudFront distribution.
- **Estimated Savings:** 20-40% on Data Transfer and compute scaling
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application must serve cacheable static content.

#### 16. Implement VPC Endpoints (Gateway/Interface)
- **What:** If the Beanstalk application communicates heavily with S3 or DynamoDB, route that traffic through a VPC Gateway Endpoint rather than a NAT Gateway.
- **Why It Saves Money:** Data transfer through a VPC Gateway Endpoint to S3/DynamoDB is free, avoiding the $0.045/GB charge for processing data through a NAT Gateway.
- **Implementation Steps:**
  1. Open the VPC console.
  2. Navigate to Endpoints and create a new Gateway Endpoint for S3.
  3. Associate the endpoint with the route tables used by the Beanstalk private subnets.
- **Estimated Savings:** Eliminates NAT Gateway data processing fees for S3 traffic
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Beanstalk instances must be deployed in private subnets utilizing a NAT Gateway.

#### 17. Clean Up Orphaned RDS Snapshots
- **What:** Identify and delete old, unnecessary RDS snapshots that were created by Beanstalk when old environments were terminated.
- **Why It Saves Money:** RDS backup storage beyond the free allocation (which equals 100% of the active database size) incurs monthly fees.
- **Implementation Steps:**
  1. Go to the RDS console and select Snapshots.
  2. Filter by snapshots associated with deleted Beanstalk environments (often containing `eb-` in the name).
  3. Delete snapshots that exceed the required retention period.
- **Estimated Savings:** $0.095 per GB-month of snapshot storage
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Confirmation that the snapshot data is no longer needed for compliance or recovery.
