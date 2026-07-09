# Cost-Cutting Playbook: AWS App Runner
> **Companion File:** [app_runner.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/app_runner/app_runner.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS App Runner provides a fully managed environment for containerized web applications, billing on a pay-as-you-go model for active vCPU, provisioned memory, and code builds. Importantly, App Runner is currently in **maintenance mode**, meaning the most significant long-term cost and operational strategy is migrating workloads to Fargate or Amplify Gen 2. In the interim, immediate cost reductions can be achieved by rightsizing memory configurations (which run 24/7), switching to image-based deployments, and optimizing concurrency settings to minimize scale-out events.

## Strategy Categories
### 1. Waste Elimination
### 2. Rightsizing
### 3. Commitment Discounts
### 4. Architecture Changes
### 5. Scheduling & Auto-Scaling
### 6. Pricing Model Optimization
### 7. Network & Data Transfer Optimization
---
## Cross-Service Synergies
- **Amazon ECR:** Moving builds to CI/CD pipelines and deploying directly from ECR eliminates App Runner build charges.
- **AWS Compute Savings Plans:** Apply to App Runner compute usage, lowering the effective hourly rate.
- **Amazon CloudFront:** Caching responses at the edge reduces the number of requests hitting App Runner, minimizing active vCPU time.
- **AWS Fargate / Amazon ECS:** The recommended migration path for long-term architectural stability and cost control.
---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
Identify precise usage metrics for `Active vCPU`, `Provisioned Memory`, and `Build Minutes` under the App Runner product code.
### B. CloudWatch Metrics
Analyze `ConcurrentRequestsPerInstance`, `ActiveInstances`, `CPUUtilization`, and `MemoryUtilization` to inform rightsizing.
### C. AWS Config / Trusted Advisor
Identify unattached VPC connectors, idle App Runner services, or missing Savings Plan coverage.
### D. Company Policies
Understand CI/CD requirements, approved build systems, and migration timelines away from maintenance-mode services.
### E. IaC (Optional)
Review Terraform or CloudFormation scripts for `AutoScalingConfiguration` and `InstanceConfiguration` parameter tuning.
---
## Output Schema
### Finding Record (JSON)
### Summary Report Table

#### 1. APPRUNNER-Delete Unused App Runner Services
- **What:** Identify and terminate App Runner services that receive zero traffic over a 30-day period.
- **Why It Saves Money:** Even when idle, App Runner charges for provisioned memory 24/7 ($5.11/month per GB). Deleting idle services eliminates this baseline cost.
- **Implementation Steps:** 
  1. Review CloudWatch `Requests` metrics for App Runner services.
  2. Identify services with zero or negligible requests over the last 30 days.
  3. Backup necessary configuration or code.
  4. Delete the App Runner service via AWS Console or CLI.
- **Estimated Savings:** 100% of the service cost
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch metric analysis

#### 2. APPRUNNER-Clean Up Abandoned VPC Connectors
- **What:** Remove App Runner Custom VPC connectors that are no longer attached to any active services.
- **Why It Saves Money:** While connectors themselves may not incur direct hourly fees, associated ENIs and NAT Gateway dependencies can accrue data processing or idle resource costs.
- **Implementation Steps:** 
  1. List all VPC connectors in the App Runner console.
  2. Identify connectors not linked to an active service.
  3. Delete the unused connectors.
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Visibility into VPC connector attachments

#### 3. APPRUNNER-Consolidate Low-Traffic Microservices
- **What:** Merge multiple low-traffic App Runner applications into a single App Runner service using internal routing.
- **Why It Saves Money:** Each App Runner service incurs its own 24/7 provisioned memory cost. Consolidating them means paying the memory baseline for one service instead of several.
- **Implementation Steps:** 
  1. Identify low-traffic services with compatible technology stacks.
  2. Refactor code to handle multiple routes within a single container.
  3. Deploy the unified container to a single App Runner service.
  4. Terminate the legacy individual services.
- **Estimated Savings:** 20-50%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compatible application architectures

#### 4. APPRUNNER-Reduce CloudWatch Logs Retention
- **What:** Lower the retention period for App Runner application and service logs in CloudWatch.
- **Why It Saves Money:** CloudWatch Logs charges $0.03 per GB per month for storage. App Runner defaults to indefinitely retaining logs.
- **Implementation Steps:** 
  1. Navigate to the CloudWatch Logs console.
  2. Locate the App Runner log groups (`/aws/apprunner/...`).
  3. Edit the retention setting from "Never expire" to 14 or 30 days based on compliance needs.
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Log retention policy agreement

#### 5. APPRUNNER-Clean Up Stale ECR Images
- **What:** Delete old, untagged, or unused container images in Amazon ECR that were deployed to App Runner.
- **Why It Saves Money:** ECR storage costs $0.10 per GB per month. Accumulating images from frequent App Runner deployments bloats storage costs.
- **Implementation Steps:** 
  1. Navigate to Amazon ECR.
  2. Implement ECR Lifecycle Policies to expire untagged images after a set number of days.
  3. Apply policies to retain only the last N tagged images.
- **Estimated Savings:** 5-10%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** ECR Lifecycle Policy permissions

#### 6. APPRUNNER-Downsize Over-Provisioned Memory
- **What:** Reduce the allocated memory in the Instance Configuration for services not fully utilizing their RAM.
- **Why It Saves Money:** Memory is billed 24/7 to keep the instance warm. Reducing memory from 2 GB to 1 GB saves ~$5.11 per instance per month continuously.
- **Implementation Steps:** 
  1. Monitor the `MemoryUtilization` metric in CloudWatch.
  2. Identify services consistently peaking below 50% memory usage.
  3. Update the App Runner service configuration to a lower memory tier.
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch MemoryUtilization data

#### 7. APPRUNNER-Downsize Over-Provisioned vCPU
- **What:** Reduce the allocated vCPU for applications that are not CPU-bound.
- **Why It Saves Money:** vCPU is billed when instances are active ($0.064/vCPU-hour). Lowering the vCPU allocation proportionally reduces the active compute charge.
- **Implementation Steps:** 
  1. Review the `CPUUtilization` metric in CloudWatch during active periods.
  2. Identify services utilizing only a small fraction of their allocated vCPU.
  3. Reconfigure the service with a lower vCPU setting.
- **Estimated Savings:** 15-25%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch CPUUtilization data

#### 8. APPRUNNER-Optimize Concurrency Auto Scaling (MaxConcurrency)
- **What:** Increase the `MaxConcurrency` setting (up to the limit of 200) for workloads that can handle more simultaneous requests per instance.
- **Why It Saves Money:** A low concurrency limit forces App Runner to prematurely spin up new instances under load, multiplying active vCPU and memory charges. Higher concurrency packs more traffic into fewer instances.
- **Implementation Steps:** 
  1. Evaluate application performance under load (e.g., I/O bound vs CPU bound).
  2. Edit the Auto Scaling configuration for the service.
  3. Increase the `MaxConcurrency` value safely while monitoring latency.
- **Estimated Savings:** 20-40%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Load testing data

#### 9. APPRUNNER-Restrict MaxSize Auto Scaling Limits
- **What:** Lower the `MaxSize` instance limit in the Auto Scaling configuration to prevent unbounded scaling during traffic spikes or DDoS attempts.
- **Why It Saves Money:** Prevents runaway costs by capping the maximum number of instances App Runner can provision, limiting maximum possible active compute charges.
- **Implementation Steps:** 
  1. Determine the maximum expected legitimate traffic for the service.
  2. Calculate the number of instances required to serve that traffic.
  3. Update the `MaxSize` parameter in the Auto Scaling configuration to this calculated ceiling.
- **Estimated Savings:** Variable (Cost Avoidance)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Baseline traffic analysis

#### 10. APPRUNNER-Purchase Compute Savings Plans
- **What:** Apply AWS Compute Savings Plans to cover App Runner usage.
- **Why It Saves Money:** Compute Savings Plans provide discounts up to 17% on App Runner compute rates in exchange for a 1- or 3-year hourly spend commitment.
- **Implementation Steps:** 
  1. Use AWS Cost Explorer's Savings Plans recommendations tool.
  2. Verify App Runner consistent usage baseline.
  3. Purchase a Compute Savings Plan (No Upfront or Partial Upfront).
- **Estimated Savings:** 10-17%
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Stable App Runner workloads

#### 11. APPRUNNER-Migrate to AWS Fargate / ECS
- **What:** Re-platform App Runner workloads to Amazon ECS on AWS Fargate.
- **Why It Saves Money:** App Runner is in maintenance mode. Fargate offers more granular scaling controls, deeper Savings Plan discounts, Spot capacity integration, and eliminates the rigid 24/7 memory provisioning requirements.
- **Implementation Steps:** 
  1. Export the container image currently used by App Runner.
  2. Create an ECS Task Definition mapping the same environment variables and ports.
  3. Deploy an ECS Service on Fargate behind an Application Load Balancer.
  4. Cut over DNS traffic and decommission App Runner.
- **Estimated Savings:** 20-40%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** ECS/Fargate architecture knowledge

#### 12. APPRUNNER-Migrate to AWS Amplify Gen 2
- **What:** Move suitable web applications (Next.js, Nuxt, etc.) from App Runner to AWS Amplify Gen 2 hosting.
- **Why It Saves Money:** Amplify Gen 2 is the recommended modern path for full-stack web apps on AWS, offering highly optimized pricing for SSR (Server-Side Rendering) compared to App Runner's container model.
- **Implementation Steps:** 
  1. Evaluate application compatibility with Amplify Hosting.
  2. Connect the source code repository to AWS Amplify.
  3. Configure build settings and deploy the application.
  4. Route traffic to Amplify and sunset the App Runner service.
- **Estimated Savings:** 30-50%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Web framework compatibility

#### 13. APPRUNNER-Switch to Image-Based Deployments from ECR
- **What:** Move from code-based deployments (App Runner pulling from GitHub) to image-based deployments pulling pre-built images from Amazon ECR.
- **Why It Saves Money:** App Runner charges $0.005 per build minute for code deployments. Offloading builds to external CI/CD pipelines (which may run on free tiers or more cost-effective compute) eliminates this fee.
- **Implementation Steps:** 
  1. Configure GitHub Actions, GitLab CI, or AWS CodeBuild to build the Docker image.
  2. Push the resulting image to Amazon ECR.
  3. Reconfigure the App Runner service to pull from the ECR repository instead of source code.
- **Estimated Savings:** 5-15%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** External CI/CD pipeline

#### 14. APPRUNNER-Disable Automatic Builds for Non-Production
- **What:** Turn off automatic deployment triggers on every commit for Dev/QA App Runner environments.
- **Why It Saves Money:** Prevents unnecessary build minute charges ($0.005/min) and deployment cycles caused by minor or frequent commits.
- **Implementation Steps:** 
  1. Open the App Runner service configuration.
  2. Navigate to the deployment settings.
  3. Change the deployment trigger from "Automatic" to "Manual" for non-prod environments.
- **Estimated Savings:** 2-10%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Developer agreement on manual deployment cadence

#### 15. APPRUNNER-Implement CloudFront Caching
- **What:** Place Amazon CloudFront in front of the App Runner service to cache static assets and common API responses.
- **Why It Saves Money:** Caching reduces the volume of requests hitting the App Runner container. This minimizes the time instances spend in the "Active" state, directly reducing the $0.064/vCPU-hour active compute charges.
- **Implementation Steps:** 
  1. Create a CloudFront distribution.
  2. Set the App Runner default domain as the Custom Origin.
  3. Configure caching behaviors for static paths (e.g., `/images/*`, `/css/*`).
  4. Update DNS to point to the CloudFront distribution.
- **Estimated Savings:** 10-30%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application supports caching headers

#### 16. APPRUNNER-Pause Non-Production Services Automatically
- **What:** Use AWS CLI or SDKs via AWS Lambda and EventBridge to pause App Runner services in Dev/Test environments during non-business hours.
- **Why It Saves Money:** Pausing a service completely stops billing for provisioned memory and active compute, resulting in a ~65% cost reduction for environments only needed 40 hours a week.
- **Implementation Steps:** 
  1. Write a Lambda function using `boto3` to call `pause_service` and `resume_service`.
  2. Configure an EventBridge cron rule to pause at 6 PM Friday and resume at 8 AM Monday.
  3. Assign IAM permissions to the Lambda function.
- **Estimated Savings:** 60-70% (on applied environments)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Identified non-production App Runner ARNs

#### 17. APPRUNNER-Offload Background Processing to SQS/Lambda
- **What:** Remove asynchronous or long-running background tasks from the App Runner web container.
- **Why It Saves Money:** Keeping the App Runner container active for long background tasks accrues active vCPU charges. Lambda bills in millisecond increments and is much cheaper for event-driven asynchronous processing.
- **Implementation Steps:** 
  1. Decouple background tasks from the main web application.
  2. Have App Runner push messages to an SQS queue.
  3. Configure an AWS Lambda function to consume the queue and execute the task.
- **Estimated Savings:** 15-30%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application refactoring capabilities

#### 18. APPRUNNER-Enable Response Compression
- **What:** Configure the application running inside App Runner to compress HTTP responses (gzip/brotli).
- **Why It Saves Money:** Reduces the size of data transmitted out to the internet, lowering standard AWS Data Transfer Out ($0.09/GB) costs.
- **Implementation Steps:** 
  1. Enable compression middleware in the application framework (e.g., Express.js `compression` package, or NGINX configuration).
  2. Deploy the updated container to App Runner.
- **Estimated Savings:** 2-10% (on data transfer)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Web framework compression support

#### 19. APPRUNNER-Use VPC Endpoints for Backend Connectivity
- **What:** Deploy Gateway and Interface VPC Endpoints for App Runner to communicate with internal AWS services (like S3, DynamoDB, or RDS) without traversing a NAT Gateway.
- **Why It Saves Money:** If App Runner is configured with a Custom VPC connector that routes internet-bound traffic through a NAT Gateway, talking to AWS services incurs NAT Data Processing charges ($0.045/GB). VPC Endpoints bypass the NAT Gateway.
- **Implementation Steps:** 
  1. Identify AWS services the App Runner application connects to.
  2. Create VPC Endpoints (e.g., S3 Gateway Endpoint) in the VPC attached to the App Runner connector.
  3. Ensure VPC route tables direct traffic to the endpoints.
- **Estimated Savings:** 5-20%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Custom VPC Connector configured
