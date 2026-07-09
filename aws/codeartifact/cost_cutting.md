# Cost-Cutting Playbook: AWS CodeArtifact
> **Companion File:** [codeartifact.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/codeartifact/codeartifact.md)
> **Last Updated:** July 2026

---

## Executive Summary
AWS CodeArtifact is a highly cost-efficient, fully managed artifact repository service where costs are driven primarily by storage volume ($0.05/GB-month) and API requests ($0.05/10,000 requests), with generous free tiers (2 GB storage, 100k API requests/month). Cost optimization for CodeArtifact largely focuses on hygiene—aggressively purging obsolete snapshot builds, leveraging local CI caches to prevent redundant API polling, and ensuring strict in-region data routing via VPC Endpoints to avoid hidden NAT Gateway or Cross-Region data transfer fees. Implementing strict lifecycle policies and lockfiles can rapidly slash CodeArtifact spend by over 70%.

---

## Strategy Categories

### 1. Waste Elimination

#### 1. CA-1. Purge Stale CI/CD Snapshot Packages
- **What:** Implement automated scripts (via EventBridge or Lambda) to delete non-release, temporary snapshot packages (e.g., `1.0.0-SNAPSHOT`) older than 14-30 days.
- **Why It Saves Money:** Eliminates the accumulation of obsolete gigabytes of storage, saving $0.05 per GB-month.
- **Implementation Steps:** 
  1. Identify repositories containing snapshot builds.
  2. Write a Lambda function utilizing the `DeletePackageVersions` API.
  3. Schedule the Lambda via EventBridge to run weekly, targeting packages matching `*SNAPSHOT*` older than 30 days.
- **Estimated Savings:** 60-80% of CodeArtifact storage costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Strong naming convention separating snapshot and release builds.

#### 2. CA-2. Delete Orphaned and Empty Repositories
- **What:** Identify and delete CodeArtifact repositories and domains that were used for POCs or deprecated projects.
- **Why It Saves Money:** Stops recurring storage billing for cached upstream packages in forgotten repositories.
- **Implementation Steps:** 
  1. Audit CloudWatch metrics (`PackageCount`, `StorageBytes`) for inactive repositories.
  2. Verify with engineering teams if the repository is abandoned.
  3. Delete the repository via CLI/Console.
- **Estimated Savings:** 5-10% of overall CodeArtifact spend.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Visibility into project lifecycles.

#### 3. CA-3. Remove Failed Build Artifacts
- **What:** Ensure CI pipelines automatically clean up partial or broken artifact uploads if a pipeline fails mid-publish.
- **Why It Saves Money:** Prevents paying for useless, corrupted storage ($0.05/GB).
- **Implementation Steps:** 
  1. Update CI/CD pipeline scripts (e.g., Jenkinsfile, GitHub Actions).
  2. Add an `always()` or `on: failure` block to invoke package deletion for the specific build ID if the test suite fails post-publish.
- **Estimated Savings:** <5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CI/CD pipeline access.

### 2. Rightsizing

#### 4. CA-4. Exclude Extraneous Files from Packages
- **What:** Use `.npmignore`, `.dockerignore`, or `MANIFEST.in` to strictly exclude tests, documentation, raw media, and source code from published artifacts.
- **Why It Saves Money:** CodeArtifact charges per GB. Bloated packages take up significantly more space and cost more to store.
- **Implementation Steps:** 
  1. Audit largest packages in CodeArtifact.
  2. Add ignore files to their source repositories.
  3. Enforce package size checks in the CI pipeline before publishing.
- **Estimated Savings:** 20-40% of storage costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Knowledge of package manager manifest rules.

#### 5. CA-5. Minify and Compress Assets
- **What:** Compress binaries, zip assets, and minify JS/CSS before bundling them into packages published to CodeArtifact.
- **Why It Saves Money:** Reduces the overall GB footprint of the artifacts stored, cutting the $0.05/GB-month storage rate.
- **Implementation Steps:** 
  1. Integrate build tools (e.g., Webpack, gzip, upx) into the CI pipeline.
  2. Ensure minification runs *before* the package is zipped and published.
- **Estimated Savings:** 10-30% of storage costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compatibility with downstream consumers.

### 3. Commitment Discounts

#### 6. CA-6. Leverage AWS Enterprise Discount Program (EDP)
- **What:** While CodeArtifact has no specific Savings Plans, it is eligible for blanket EDP discounts.
- **Why It Saves Money:** Applies a flat percentage discount (typically 9-15%) across all CodeArtifact usage (storage and API).
- **Implementation Steps:** 
  1. Consolidate AWS billing via AWS Organizations.
  2. Negotiate an EDP contract with your AWS Account Manager.
- **Estimated Savings:** 9-15% total cost reduction.
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** High overall AWS spend (usually $1M+/year).

### 4. Architecture Changes

#### 7. CA-7. Centralize Upstream Caching (Proxy Repositories)
- **What:** Use a single, centralized CodeArtifact domain to proxy public registries (npmjs, PyPI, Maven Central) instead of fetching them from the internet directly on every build.
- **Why It Saves Money:** Caches packages locally in AWS. This drastically reduces outbound internet NAT Gateway data processing fees ($0.045/GB), replacing it with in-region CodeArtifact transfers (Free).
- **Implementation Steps:** 
  1. Create an external connection in CodeArtifact to the public upstream.
  2. Update all build agents to point to the CodeArtifact proxy endpoint instead of the public internet.
- **Estimated Savings:** High (Offsets massive NAT Gateway spend, transferring it to much cheaper CodeArtifact storage).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC configuration visibility.

#### 8. CA-8. Consolidate Multiple CodeArtifact Domains
- **What:** Consolidate isolated engineering teams into a single CodeArtifact Domain where possible.
- **Why It Saves Money:** Packages cached from external connections are deduplicated at the Domain level. Having multiple domains means paying to store the same public packages (e.g., React, Lodash) multiple times.
- **Implementation Steps:** 
  1. Audit existing CodeArtifact domains.
  2. Migrate distinct repositories into a single unified domain.
  3. Decommission legacy domains.
- **Estimated Savings:** 10-30% of storage costs (deduplication).
- **Risk Level:** Medium (Requires authentication updates).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Unified IAM strategy across teams.

#### 9. CA-9. Separate Snapshot and Release Repositories
- **What:** Architect distinct CodeArtifact repositories for stable releases vs. temporary nightly/snapshot builds.
- **Why It Saves Money:** Allows for aggressive, risk-free automated deletion scripts (see CA-1) to be applied to the snapshot repo without accidentally deleting production releases.
- **Implementation Steps:** 
  1. Create `project-release` and `project-snapshot` repositories.
  2. Route CI/CD publish steps conditionally based on git branch (main vs dev).
- **Estimated Savings:** Enables 60-80% storage savings from CA-1.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CI/CD branch logic.

### 5. Scheduling & Auto-Scaling

#### 10. CA-10. Event-Driven CI/CD Instead of Polling
- **What:** Stop CI/CD systems from aggressively polling CodeArtifact for new package versions every minute. Use AWS EventBridge rules to trigger builds on `PackageVersionCreated` events.
- **Why It Saves Money:** API requests are billed at $0.05 per 10k calls. Aggressive polling across hundreds of packages generates millions of empty API requests per month.
- **Implementation Steps:** 
  1. Configure an EventBridge rule listening to CodeArtifact publish events.
  2. Target the CI/CD pipeline (e.g., CodePipeline, Jenkins webhook).
  3. Disable polling in the CI/CD system.
- **Estimated Savings:** 50-90% of API Request costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** EventBridge integration capabilities.

### 6. Pricing Model Optimization

#### 11. CA-11. Implement Lockfiles strictly (`package-lock.json`)
- **What:** Commit lockfiles (`package-lock.json`, `yarn.lock`, `Pipfile.lock`) to source control so package managers know exactly which versions to download.
- **Why It Saves Money:** Without a lockfile, package managers spam the CodeArtifact API resolving complex dependency trees and querying available versions, driving up the API Request bill ($0.05/10k).
- **Implementation Steps:** 
  1. Ensure `npm ci` is used in CI environments instead of `npm install`.
  2. Mandate lockfile commits in PR reviews.
- **Estimated Savings:** 20-50% of API Request costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Standardized local development environments.

#### 12. CA-12. Leverage Local CI Runner Dependency Caching
- **What:** Utilize native CI caching mechanisms (e.g., GitHub Actions `actions/setup-node` caching, GitLab CI caching) to cache `node_modules` or `~/.m2` directly on the runner.
- **Why It Saves Money:** Prevents the CI pipeline from redownloading the exact same gigabytes of dependencies from CodeArtifact on every single commit, saving API calls and localized build time.
- **Implementation Steps:** 
  1. Update CI YAML configs to hash lockfiles and cache dependency directories.
  2. Only hit CodeArtifact when the lockfile hash changes.
- **Estimated Savings:** 40-70% of API Request costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Modern CI/CD tooling.

#### 13. CA-13. Maximize the AWS Free Tier
- **What:** Exploit the 2 GB storage and 100k API requests per month free tier inherently provided by AWS.
- **Why It Saves Money:** For small side projects, micro-teams, or POCs, carefully managing artifacts to stay under 2 GB means CodeArtifact is effectively 100% free ($0.00).
- **Implementation Steps:** 
  1. Apply hyper-aggressive retention policies (e.g., keep only the last 3 builds) to keep storage under 2 GB.
- **Estimated Savings:** 100% (for small workloads).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Small footprint workloads.

### 7. Network & Data Transfer Optimization

#### 14. CA-14. Enforce In-Region VPC Endpoints (PrivateLink)
- **What:** Configure AWS PrivateLink Interface VPC Endpoints for CodeArtifact inside the VPC where build agents (EC2, EKS, CodeBuild) reside.
- **Why It Saves Money:** Routing traffic over the public internet via NAT Gateways incurs a $0.045/GB data processing charge. VPC endpoints keep traffic private and bypass the NAT Gateway, while CodeArtifact to in-region AWS compute transfer is natively $0.00.
- **Implementation Steps:** 
  1. Create VPC Endpoints for CodeArtifact API and CodeArtifact Build.
  2. Ensure Security Groups allow inbound port 443 from build subnets.
- **Estimated Savings:** Saves $0.045 per GB of dependencies downloaded.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Private subnets and VPC management rights.

#### 15. CA-15. Colocate Build Runners and Repositories
- **What:** Ensure that CodeArtifact domains and repositories are deployed in the *exact same AWS Region* as the EC2, CodeBuild, or EKS clusters that consume them.
- **Why It Saves Money:** Cross-region data transfer costs ~$0.02 per GB. In-region data transfer from CodeArtifact to AWS compute is Free ($0.00). 
- **Implementation Steps:** 
  1. Audit the regions of build nodes vs CodeArtifact via AWS Config.
  2. If a mismatch exists, migrate the CodeArtifact domain to the build region or vice versa.
- **Estimated Savings:** Eliminates all cross-region data transfer costs (100% reduction).
- **Risk Level:** Medium (Requires data migration).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region architecture map.

#### 16. CA-16. Avoid Multi-Region Replication Unless strictly necessary
- **What:** Avoid replicating CodeArtifact repositories across multiple AWS regions just for high availability if your primary build runners are localized to one region.
- **Why It Saves Money:** You pay double the storage costs ($0.05/GB) and incur cross-region data transfer costs to keep the replicas in sync.
- **Implementation Steps:** 
  1. Evaluate the actual necessity of cross-region reads.
  2. If unnecessary, delete the replica repository.
- **Estimated Savings:** 50% of storage costs + data transfer savings.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Disaster Recovery (DR) policy review.

---

## Cross-Service Synergies

- **CodeBuild & CodePipeline:** Natively integrates to authenticate securely with CodeArtifact without passing hardcoded tokens. In-region transfer between CodeArtifact and CodeBuild is free.
- **VPC / PrivateLink:** Drastically reduces NAT Gateway data processing fees by keeping heavy dependency downloads on the private AWS network.
- **EventBridge & Lambda:** Forms the backbone of automated cost-saving scripts (like snapshot pruning) and event-driven architecture to stop API polling.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
- Query `line_item_product_code` = `AWSCodeArtifact`
- Filter `line_item_usage_type` for `StorageByteHours` (Storage) and `Request` (API calls).

### B. CloudWatch Metrics
- `PackageCount`: To identify abandoned or runaway repositories.
- `StorageBytes`: To track the physical size of repositories over time.
- `UnsuccessfulRequests`: To identify misconfigured build agents spamming the API.

### C. AWS Config / Trusted Advisor
- Check for the existence of CodeArtifact VPC Endpoints within build VPCs.
- Verify regional alignment between compute resources and CodeArtifact.

### D. Company Policies
- Data retention policies (How long must we legally keep old software builds?).
- CI/CD guidelines regarding snapshot vs release branching.

### E. IaC (Optional)
- Terraform/CloudFormation files to verify domain consolidation and proxy repository configurations.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "CA-1",
  "service": "AWS CodeArtifact",
  "strategy_name": "Purge Stale CI/CD Snapshot Packages",
  "category": "Waste Elimination",
  "estimated_savings_percentage": "60-80%",
  "risk_level": "Low",
  "effort_level": "Medium",
  "description": "Implement automated scripts to delete non-release, temporary snapshot packages older than 14-30 days."
}
```

### Summary Report Table

| ID | Strategy | Category | Est. Savings | Risk | Scope |
|----|----------|----------|--------------|------|-------|
| CA-1 | Purge Stale Snapshot Packages | Waste Elimination | 60-80% | Low | Engineer/DevOps |
| CA-2 | Delete Orphaned Repositories | Waste Elimination | 5-10% | Low | FinOps Team |
| CA-3 | Remove Failed Build Artifacts | Waste Elimination | <5% | Low | Engineer/DevOps |
| CA-4 | Exclude Extraneous Files | Rightsizing | 20-40% | Low | Engineer/DevOps |
| CA-5 | Minify and Compress Assets | Rightsizing | 10-30% | Low | Engineer/DevOps |
| CA-6 | Leverage AWS EDP | Commitment Discounts | 9-15% | Low | Procurement |
| CA-7 | Centralize Upstream Caching | Architecture Changes | High (NAT) | Low | Engineer/DevOps |
| CA-8 | Consolidate Domains | Architecture Changes | 10-30% | Medium | Engineer/DevOps |
| CA-9 | Separate Snapshot/Release Repos | Architecture Changes | Enabler | Low | Engineer/DevOps |
| CA-10 | Event-Driven CI/CD (No Polling) | Scheduling & Auto-Scaling | 50-90% (API) | Low | Engineer/DevOps |
| CA-11 | Implement Lockfiles strictly | Pricing Model | 20-50% (API) | Low | Engineer/DevOps |
| CA-12 | Local CI Runner Caching | Pricing Model | 40-70% (API) | Low | Engineer/DevOps |
| CA-13 | Maximize Free Tier | Pricing Model | 100% (Small)| Low | Engineer/DevOps |
| CA-14 | Enforce In-Region VPC Endpoints | Network Optimization | High (NAT) | Low | Engineer/DevOps |
| CA-15 | Colocate Build Runners & Repos | Network Optimization | 100% (Transfer)| Medium | Engineer/DevOps |
| CA-16 | Avoid Unnecessary Multi-Region | Network Optimization | 50% | Medium | Engineer/DevOps |
