# Cost-Cutting Playbook: AWS CodeBuild
> **Companion File:** [codebuild.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/codebuild/codebuild.md)
> **Last Updated:** July 2026

---

## Executive Summary
AWS CodeBuild is a serverless CI/CD build service billed per minute based on the compute tier and architecture used. Cost optimization in CodeBuild relies on three primary pillars: drastically reducing build execution time through aggressive caching, right-sizing the compute footprint (e.g., moving to ARM Graviton instances), and eliminating wasteful build runs. By implementing strict timeouts, layer caching, and optimized compute architectures, organizations can often reduce CodeBuild expenditure by 60% to 80%.

## Strategy Categories

### 1. Waste Elimination

#### CB-1. Enforce Strict Build Timeouts
- **What:** Configure the `timeoutInMinutes` property in CodeBuild project settings to a sensible limit (e.g., 15-20 minutes).
- **Why It Saves Money:** Prevents hung test scripts or stuck processes from burning build-minutes up to the default maximum (which can be hours), eliminating runaway costs.
- **Implementation Steps:**
  1. Audit current average build times in CloudWatch.
  2. Set `timeoutInMinutes` to 1.5x the average build time.
  3. Deploy the timeout configuration via IaC (Terraform/CloudFormation).
- **Estimated Savings:** 5-10% (prevents catastrophic billing spikes)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Known baseline build duration.

#### CB-2. Enable Local Docker Layer Caching
- **What:** Enable local caching for CodeBuild projects that build Docker images.
- **Why It Saves Money:** CodeBuild bills per build-minute. Rebuilding unchanged Docker layers takes significant time. Caching reduces build times by up to 80%, slashing per-minute billing proportionally.
- **Implementation Steps:**
  1. Edit CodeBuild project settings.
  2. Under Artifacts, select "Cache" and choose "Local".
  3. Check "Docker layer cache".
  4. Ensure Dockerfile is optimized to utilize layered caching.
- **Estimated Savings:** 40-80%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Docker-based builds.

#### CB-3. Enable S3 Caching for Dependencies
- **What:** Configure an S3 cache for language-specific package managers (`node_modules`, `.m2`, `pip`).
- **Why It Saves Money:** Downloading gigabytes of dependencies on every build adds 2-5 minutes of billed time. Caching dependencies in S3 cuts this down to seconds.
- **Implementation Steps:**
  1. Create an S3 bucket in the same region for caching.
  2. Update CodeBuild project cache type to "Amazon S3".
  3. Specify cache paths in the `buildspec.yml` file.
- **Estimated Savings:** 20-50%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Projects with heavy third-party dependencies.

#### CB-4. Implement Commit Skipping (e.g., `[skip ci]`)
- **What:** Configure Source Providers (GitHub/GitLab/Bitbucket) and CodeBuild webhooks to ignore documentation-only commits.
- **Why It Saves Money:** Prevents CodeBuild from spinning up and charging build-minutes for changes to READMEs or markdown files that do not require compilation or testing.
- **Implementation Steps:**
  1. Add webhook filter groups in CodeBuild to exclude patterns (e.g., `^.*\.md$`).
  2. Educate developers to use `[skip ci]` in commit messages for trivial changes.
- **Estimated Savings:** 2-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Source provider integration.

#### CB-5. Remove Stale CodeBuild Projects
- **What:** Identify and delete CodeBuild projects that have not executed a build in over 90 days.
- **Why It Saves Money:** While idle CodeBuild projects don't cost money directly, they often harbor unmanaged S3 artifacts, CloudWatch log groups, and unused IAM roles which do incur costs.
- **Implementation Steps:**
  1. Query CodeBuild API to list projects.
  2. Check last run timestamp.
  3. Deprecate and delete stale projects and their associated S3/CloudWatch resources.
- **Estimated Savings:** 1-5% (indirect savings via S3/CloudWatch)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Visibility into project usage.

### 2. Rightsizing

#### CB-6. Downsize Over-Provisioned Compute Tiers
- **What:** Migrate projects running on `build.general1.large` ($0.020/min) or `medium` ($0.010/min) to `build.general1.small` ($0.005/min) if CPU/RAM utilization is low.
- **Why It Saves Money:** A small instance costs 75% less than a large instance. Many shell scripts and small Go/Node apps do not require 8 vCPUs.
- **Implementation Steps:**
  1. Monitor CodeBuild CloudWatch metrics (CPUUtilized, MemoryUtilized).
  2. Identify projects using < 3GB RAM and < 2 vCPUs on large/medium tiers.
  3. Downgrade compute type in project settings.
- **Estimated Savings:** 50-75% per downsized project
- **Risk Level:** Medium (risk of OOM if downsized too aggressively)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch utilization metrics.

#### CB-7. Optimize Dockerfiles for Faster Builds
- **What:** Restructure Dockerfiles to copy package definition files (`package.json`) before application code, optimizing layer caching.
- **Why It Saves Money:** Shorter build times equal fewer billed build-minutes.
- **Implementation Steps:**
  1. Audit slow Docker builds.
  2. Rearrange `COPY` and `RUN` commands to maximize cache hits.
  3. Use multi-stage builds to reduce final image size and build complexity.
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Knowledge of Docker caching semantics.

#### CB-8. Consolidate Micro-Builds
- **What:** Group related micro-tasks (e.g., linting, unit testing, security scanning) into a single CodeBuild run rather than fanning out to multiple parallel CodeBuild projects.
- **Why It Saves Money:** CodeBuild charges for every minute, but spinning up multiple parallel builds incurs container provisioning time for each. Consolidating eliminates the overlapping setup tax.
- **Implementation Steps:**
  1. Combine `buildspec.yml` phases for related tasks.
  2. Run tools sequentially or in parallel shell jobs within the same CodeBuild instance.
- **Estimated Savings:** 10-20%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Independent build steps that can run in a single environment.

### 3. Commitment Discounts

#### CB-9. Adopt CodeBuild Reserved Capacity Fleets
- **What:** Use Reserved Capacity Fleets if you run consistent, high-volume, continuous builds.
- **Why It Saves Money:** AWS CodeBuild offers provisioned reserved fleets which can be more cost-effective for 24/7 heavy CI/CD pipelines compared to on-demand pricing, by offering a lower baseline hourly rate for a steady fleet of instances.
- **Implementation Steps:**
  1. Analyze historical concurrency and build volume.
  2. Provision a CodeBuild Reserved Capacity Fleet.
  3. Route high-frequency projects to the reserved fleet.
- **Estimated Savings:** 15-25%
- **Risk Level:** Medium (risk of underutilizing the reserved fleet)
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Predictable, high-volume build traffic.

### 4. Architecture Changes

#### CB-10. Migrate Linux Builds to ARM Graviton (`build.arm1`)
- **What:** Switch CodeBuild compute architecture from x86 to ARM Graviton.
- **Why It Saves Money:** `build.arm1.small` costs **$0.0034/min**, which is **32% cheaper** than `build.general1.small` ($0.005/min). Furthermore, Graviton processors often compile code faster.
- **Implementation Steps:**
  1. Ensure the application code and dependencies support ARM architecture (common for Java, Node.js, Python, Go).
  2. Update CodeBuild Environment settings to use `ARM_CONTAINER`.
  3. Select `build.arm1.small` or `build.arm1.large`.
- **Estimated Savings:** 32%+
- **Risk Level:** Medium (requires application compatibility testing)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Cross-platform or ARM-compatible codebase.

#### CB-11. Use Custom Pre-baked Build Images
- **What:** Create a custom Docker image containing all necessary build tools (Terraform, Node, AWS CLI, Maven) instead of using the standard AWS provided image and installing tools during the build.
- **Why It Saves Money:** Eliminates 1-3 minutes of `apt-get install` or `npm install -g` overhead on every single build execution.
- **Implementation Steps:**
  1. Create a Dockerfile with required tools.
  2. Build and push to Amazon ECR.
  3. Configure CodeBuild to use this custom ECR image as its environment.
- **Estimated Savings:** 15-30%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** ECR repository.

#### CB-12. Transition Simple Scripts to AWS Lambda
- **What:** Move trivial automation scripts (e.g., cron jobs, S3 file zipping, DB migrations) out of CodeBuild and into AWS Lambda.
- **Why It Saves Money:** Lambda bills by the millisecond and offers a massive free tier (1M requests, 400k GB-seconds). CodeBuild has a slower startup and bills a minimum of 1 minute (or per second after).
- **Implementation Steps:**
  1. Identify CodeBuild projects that execute in < 1 minute and do not require Docker.
  2. Refactor scripts into Lambda functions.
  3. Trigger via EventBridge instead of CodeBuild webhooks.
- **Estimated Savings:** 90-100%
- **Risk Level:** Medium (requires code refactoring)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Script fits within Lambda limits (15 min, 10GB RAM).

### 5. Scheduling & Auto-Scaling

#### CB-13. Schedule Nightly Batch Builds Instead of Per-Commit
- **What:** For non-critical pipelines (e.g., heavy integration tests, E2E browser tests), trigger builds on a nightly schedule rather than on every single commit.
- **Why It Saves Money:** Running a 30-minute test suite once a night instead of 20 times a day for every developer push drastically reduces build-minutes.
- **Implementation Steps:**
  1. Remove webhook triggers for heavy E2E CodeBuild projects.
  2. Set up an EventBridge rule to trigger the build daily at 2:00 AM.
- **Estimated Savings:** 40-70% (on testing pipelines)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Agreement on feedback loop timing.

#### CB-14. Avoid Concurrent Build Limit Sprawl
- **What:** Limit the maximum number of concurrent builds for a project to prevent burst billing and CI gridlock.
- **Why It Saves Money:** By queueing builds, you force newer commits to supersede older ones. If a developer pushes 5 commits in 5 minutes, limiting concurrency allows you to cancel the queued intermediate builds and only build the latest.
- **Implementation Steps:**
  1. Configure concurrent build limits in CodeBuild project settings.
  2. Enable "Batch configuration" to combine or cancel redundant in-flight builds.
- **Estimated Savings:** 10-15%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** N/A

### 6. Pricing Model Optimization

#### CB-15. Maximize the AWS Free Tier
- **What:** Ensure that small, utility builds utilize the `build.general1.small` instance type to consume the 100 free build-minutes allocated per month per account.
- **Why It Saves Money:** The first 100 minutes on `build.general1.small` are completely free ($0.00). Using a `medium` or `large` instance bypasses this free tier.
- **Implementation Steps:**
  1. Identify short, frequent builds.
  2. Lock them to `build.general1.small`.
- **Estimated Savings:** Up to $0.50/month per account (trivial, but mathematically optimized)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** N/A

#### CB-16. Utilize CodeBuild Spot Fleet (Via EC2 Fleets)
- **What:** Run CodeBuild on Amazon EC2 Spot instances using CodeBuild fleets.
- **Why It Saves Money:** Spot instances can offer up to 90% savings over on-demand EC2 prices. By routing CodeBuild to a Spot-backed fleet, you drastically reduce compute costs.
- **Implementation Steps:**
  1. Create a CodeBuild fleet configured to use EC2 Spot instances.
  2. Assign non-critical or fault-tolerant builds to this fleet.
- **Estimated Savings:** 50-70%
- **Risk Level:** High (Spot instances can be interrupted)
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Fault-tolerant build processes.

### 7. Network & Data Transfer Optimization

#### CB-17. Route CodeBuild VPC Traffic via Gateway Endpoints
- **What:** When CodeBuild is connected to a VPC, ensure S3 and DynamoDB traffic routes through VPC Gateway Endpoints instead of a NAT Gateway.
- **Why It Saves Money:** NAT Gateway data processing costs $0.045/GB. Downloading gigabytes of S3 dependencies through a NAT Gateway gets expensive. VPC Gateway Endpoints for S3 are free.
- **Implementation Steps:**
  1. Create a VPC Gateway Endpoint for S3 in the VPC used by CodeBuild.
  2. Ensure route tables for CodeBuild subnets point S3 traffic to the endpoint.
- **Estimated Savings:** 100% of S3 data transfer costs (recovers $0.045/GB)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CodeBuild running inside a VPC.

#### CB-18. Keep CodeBuild and ECR/S3 in the Same Region
- **What:** Ensure CodeBuild runs in the same AWS Region as the S3 buckets (cache/artifacts) and ECR repositories (images) it accesses.
- **Why It Saves Money:** Cross-region data transfer costs $0.01 to $0.02 per GB. Pulling a 1GB Docker image from a different region on every build incurs unnecessary network charges.
- **Implementation Steps:**
  1. Audit CodeBuild project regions.
  2. Verify ECR and S3 bucket regions match.
  3. Relocate resources if mismatched.
- **Estimated Savings:** Recovers $0.01-$0.02 per GB transferred
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region architecture.

---

## Cross-Service Synergies
- **Amazon ECR:** Storing pre-baked build images in ECR drastically reduces CodeBuild execution time.
- **Amazon S3:** S3 acts as a high-speed, low-cost cache for dependency layers, replacing expensive NAT Gateway egress traffic.
- **AWS Lambda:** Lambda serves as a cheaper, millisecond-billed alternative for trivial scripts currently wasting the 1-minute minimum CodeBuild charge.
- **Amazon CloudWatch:** Essential for monitoring `CPUUtilized` and `MemoryUtilized` to facilitate rightsizing.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
- Query `line_item_product_code = 'AWSCodeBuild'` to analyze usage by `product_instance_type` and `product_operating_system`.

### B. CloudWatch Metrics
- `BuildDuration`: To identify candidates for caching and timeout enforcement.
- `CPUUtilized` & `MemoryUtilized`: Crucial for rightsizing from large to small tiers.

### C. AWS Config / Trusted Advisor
- Audit CodeBuild project configurations for cache enablement and architecture types (ARM vs x86).

### D. Company Policies
- Determine acceptable feedback loop times (e.g., nightly vs per-commit builds).

### E. IaC (Optional)
- Review Terraform/CloudFormation code to ensure `timeoutInMinutes` and caching are enforced at the module level.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "CB-001",
  "strategy_id": "CB-11",
  "resource_id": "arn:aws:codebuild:us-east-1:123456789012:project/my-backend-build",
  "account_id": "123456789012",
  "finding_type": "Architecture Change",
  "description": "CodeBuild project is running on build.general1.large (x86). Can be migrated to build.arm1.large for 32% savings.",
  "monthly_savings_usd": 125.50,
  "effort_level": "Medium",
  "status": "Open"
}
```

### Summary Report Table

| Project Name | Current Tier | Recommended Tier | Cache Enabled? | Monthly Savings | Risk Level |
|--------------|--------------|------------------|----------------|-----------------|------------|
| backend-api  | general1.large | arm1.large       | No (Add S3)    | $125.50         | Medium     |
| frontend-web | general1.medium| general1.small   | Yes (Local)    | $45.00          | Low        |
| cron-job     | general1.small | Migrate to Lambda| N/A            | $10.20          | Medium     |
```
