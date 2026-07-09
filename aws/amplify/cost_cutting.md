# Cost-Cutting Playbook: AWS Amplify
> **Companion File:** [amplify.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/amplify/amplify.md)
> **Last Updated:** July 2026

---

## Executive Summary
AWS Amplify provides a rapid development experience for full-stack applications, but its simplified billing model for hosting and abstraction of backend services can lead to hidden costs. Amplify hosting charges primarily on build minutes ($0.01/min), data storage ($0.023/GB-month), and data served ($0.15/GB). The backend components incur standard AWS service costs (Cognito, AppSync, DynamoDB, Lambda, etc.). The most effective cost optimization strategies for Amplify revolve around reducing build execution times, minimizing data egress through caching and S3 offloading, and preventing idle environments from accumulating costs.

## Strategy Categories

### 1. Waste Elimination

#### 1. Disable WAF on Staging Apps
- **What:** Remove AWS WAF integration from non-production Amplify application environments.
- **Why It Saves Money:** Amplify charges a flat $15.00/month management fee per app for WAF integration, in addition to standard WAF Web ACL, rule, and request fees. Running this on staging/dev environments is usually unnecessary.
- **Implementation Steps:** 
  1. Go to the Amplify Console.
  2. Select the staging/development application.
  3. Navigate to App settings > Security.
  4. Detach the WAF Web ACL.
- **Estimated Savings:** $15+ per app/month (100% of non-prod WAF fees).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** WAF is not a compliance requirement for non-prod environments.

#### 2. Limit Automatic PR and Branch Deployments
- **What:** Restrict Amplify to only automatically build and deploy essential branches (e.g., `main`, `staging`) rather than every pull request or developer branch.
- **Why It Saves Money:** At $0.01 per build minute, frequent commits across dozens of branches can rapidly consume build minutes.
- **Implementation Steps:**
  1. Open the Amplify Console.
  2. Go to App settings > Repository repository and branches.
  3. Disable "Preview" features for non-essential branches.
  4. Modify branch auto-build settings to only include specific branch patterns.
- **Estimated Savings:** 30-60% on build minute costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Developers can test locally using `amplify mock` or local dev servers.

#### 3. Delete Abandoned Amplify Apps and Environments
- **What:** Identify and delete Amplify applications or Gen 1/Gen 2 backend environments that are no longer in use.
- **Why It Saves Money:** Eliminates storage charges ($0.023/GB-month) for dormant hosting assets, as well as the fixed and idle costs of any provisioned backend resources (e.g., DynamoDB tables, Cognito user pools).
- **Implementation Steps:**
  1. Audit active projects with engineering teams.
  2. Run a script using the AWS CLI (`aws amplify list-apps`) to find apps with no recent deployments.
  3. Delete the app and associated backend environments via the Amplify Console or CLI.
- **Estimated Savings:** 100% of costs associated with abandoned apps.
- **Risk Level:** Medium (ensure apps are truly abandoned).
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Audit approval from engineering owners.

#### 4. Clean up Stale Backend CloudFormation Stacks
- **What:** Remove orphaned backend resources created by failed or abandoned Amplify Gen 1/Gen 2 deployments.
- **Why It Saves Money:** Failed environment deletions can leave behind DynamoDB tables, AppSync APIs, or RDS instances that continue to accrue hourly or storage charges.
- **Implementation Steps:**
  1. Review AWS CloudFormation for stacks starting with `amplify-`.
  2. Cross-reference stack names with active environments in the Amplify Console.
  3. Delete orphaned stacks manually or via CLI.
- **Estimated Savings:** Varies (potentially hundreds of dollars if NAT Gateways or RDS databases were provisioned).
- **Risk Level:** High (do not delete active production stacks).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Thorough validation of active environments.

### 2. Rightsizing

#### 5. Cache Build Dependencies in `amplify.yml`
- **What:** Configure the Amplify build specification to cache `node_modules` and framework-specific caches (like `.next/cache` or `.nuxt`).
- **Why It Saves Money:** Running a clean `npm install` on every commit can take 5-10 minutes. Caching drops build times to 1-2 minutes. At $0.01 per minute, this directly reduces compute costs.
- **Implementation Steps:**
  1. Edit `amplify.yml`.
  2. Under the `cache` section, add paths for `node_modules/**/*` and framework cache directories.
  3. Commit and push the changes.
- **Estimated Savings:** Up to 80% of build minute costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** App can successfully build from cached dependencies.

#### 6. Optimize SSR Bundle Sizes
- **What:** Reduce the size of Server-Side Rendered (SSR) application bundles (e.g., Next.js, Nuxt) deployed to Amplify.
- **Why It Saves Money:** Smaller bundles reduce the underlying Lambda execution time for SSR functions, reduce data storage requirements ($0.023/GB), and speed up build/deployment times.
- **Implementation Steps:**
  1. Run bundle analyzer plugins (e.g., `@next/bundle-analyzer`).
  2. Remove unused dependencies and implement code splitting.
  3. Optimize heavy libraries (e.g., replacing `moment.js` with `date-fns`).
- **Estimated Savings:** 10-20% on storage and SSR compute costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Next.js/Nuxt.js application.

#### 7. Restrict Logging Retention on Backend Lambdas
- **What:** Reduce the CloudWatch log retention period for Amplify-generated Lambda functions (SSR handlers, Auth triggers, AppSync resolvers) from "Never expire" to 7-14 days.
- **Why It Saves Money:** CloudWatch Logs charges $0.03 per GB per month for storage. High-traffic Amplify apps generate massive logs.
- **Implementation Steps:**
  1. Identify CloudWatch Log Groups associated with Amplify.
  2. Update the retention policy via the AWS Console or using a script (or configure via Amplify Gen 2 CDK overrides).
- **Estimated Savings:** 50-80% on CloudWatch Logs storage costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compliance requirements do not mandate indefinite log retention.

### 3. Commitment Discounts

#### 8. Compute Savings Plans for Amplify Backend Lambdas
- **What:** Purchase Compute Savings Plans to cover the Lambda usage generated by Amplify SSR hosting and backend API resolvers.
- **Why It Saves Money:** While Amplify Hosting itself (builds, storage, egress) is not covered by Savings Plans, the underlying AWS Lambda usage (for SSR and backend functions) and Fargate (if used) are covered, offering up to a 17% discount on Lambda.
- **Implementation Steps:**
  1. Analyze AWS Cost Explorer for Lambda usage linked to the Amplify application.
  2. Purchase a Compute Savings Plan commensurate with the stable baseline usage.
- **Estimated Savings:** Up to 17% on backend compute costs.
- **Risk Level:** Medium (requires 1 or 3-year commitment).
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Stable, predictable compute usage.

### 4. Architecture Changes

#### 9. Offload Heavy Media Assets to Native S3 & CloudFront
- **What:** Serve large uncompressed images, videos, and heavy downloadable assets from a dedicated Amazon S3 bucket fronted by CloudFront instead of hosting them within the Amplify app bundle.
- **Why It Saves Money:** Amplify charges $0.15/GB for data served. Native CloudFront charges a baseline of $0.085/GB (often lower with volume tiers) and includes a 1 TB/month Free Tier.
- **Implementation Steps:**
  1. Create an S3 bucket and CloudFront distribution.
  2. Move media assets out of the `public/` directory in the Amplify app.
  3. Update application URLs to point to the new CloudFront domain.
- **Estimated Savings:** ~43% reduction in data served egress costs.
- **Risk Level:** Medium (requires code changes and separate infrastructure management).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Media assets are a significant portion of egress.

#### 10. Eject Purely Static Sites to Direct S3 + CloudFront Hosting
- **What:** For simple, high-traffic static websites (HTML/CSS/JS without SSR or Amplify backend integrations), migrate hosting from Amplify directly to S3 and CloudFront.
- **Why It Saves Money:** Bypasses Amplify's $0.15/GB egress rate in favor of CloudFront's $0.085/GB rate, and eliminates the $0.01/min build fee by using GitHub Actions (often free) to push directly to S3.
- **Implementation Steps:**
  1. Provision an S3 bucket configured for web hosting and a CloudFront distribution.
  2. Set up a CI/CD pipeline (e.g., GitHub Actions) to sync the build output to S3 and invalidate the CloudFront cache.
  3. Update DNS records.
- **Estimated Savings:** 40-60% on total hosting costs.
- **Risk Level:** Medium (loss of Amplify Console convenience features).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application does not require Amplify SSR or managed CI/CD features.

#### 11. Migrate from SSR to Static Site Generation (SSG)
- **What:** Convert Server-Side Rendered (SSR) pages to Static Site Generation (SSG) or Incremental Static Regeneration (ISR) where real-time data fetching is not strictly required.
- **Why It Saves Money:** SSR triggers AWS Lambda executions for every page request, incurring compute costs. SSG pre-builds the pages, serving them purely as static assets from the edge, eliminating compute costs for reads.
- **Implementation Steps:**
  1. Refactor Next.js `getServerSideProps` to `getStaticProps`.
  2. Reconfigure routing and data fetching logic.
  3. Redeploy to Amplify.
- **Estimated Savings:** 90%+ reduction in SSR Lambda execution costs.
- **Risk Level:** Medium (affects data freshness).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Content does not require per-request personalization.

#### 12. Implement Monorepo Build Optimization
- **What:** For organizations using monorepos with multiple Amplify apps, configure conditional builds so that an app only builds when its specific directory changes.
- **Why It Saves Money:** Prevents App B from building (and consuming build minutes) when a developer only pushes a commit to App A.
- **Implementation Steps:**
  1. Use Amplify's monorepo support (e.g., `AMPLIFY_DIFF_DEPLOY`).
  2. Configure the Amplify Console build settings to define the `AMPLIFY_MONOREPO_APP_ROOT`.
- **Estimated Savings:** 50-80% on build minutes for monorepo setups.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application codebase is structured as a monorepo.

### 5. Scheduling & Auto-Scaling

#### 13. Auto-Scale Backend DynamoDB Capacities
- **What:** Configure auto-scaling or on-demand pricing for DynamoDB tables generated by Amplify Data / AppSync.
- **Why It Saves Money:** Amplify often provisions DynamoDB tables with default Provisioned capacity. If the app has variable traffic, paying for high provisioned throughput 24/7 wastes money.
- **Implementation Steps:**
  1. Access DynamoDB tables created by Amplify.
  2. Switch billing mode to On-Demand for unpredictable workloads, or enable Auto-Scaling for provisioned tables.
  3. In Amplify Gen 2, configure this natively via CDK in the backend definition.
- **Estimated Savings:** 40-70% on database costs for variable workloads.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of app traffic patterns.

### 6. Pricing Model Optimization

#### 14. Leverage AWS Free Tier Allowances for Development
- **What:** Ensure development, testing, and staging environments are utilizing accounts that qualify for the AWS Free Tier where possible.
- **Why It Saves Money:** The Free Tier provides 1,000 build minutes, 5 GB storage, and 15 GB data served per month for 12 months.
- **Implementation Steps:**
  1. Isolate dev/staging environments into a new AWS Account.
  2. Monitor usage to ensure it stays within Free Tier limits.
- **Estimated Savings:** Up to $25/month per account during the first year.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Account is eligible for Free Tier.

#### 15. Deploy Backends in Cheaper AWS Regions
- **What:** While Amplify Hosting distributes content globally at the edge, deploy the backend resources (Cognito, DynamoDB, Lambda) in lower-cost regions like `us-east-1` or `us-east-2`.
- **Why It Saves Money:** AWS services cost significantly more in regions like `sa-east-1` (São Paulo) or `ap-northeast-1` (Tokyo).
- **Implementation Steps:**
  1. Initialize the Amplify backend in a cost-effective region during the `amplify init` phase.
  2. (For existing projects) Migration requires data export/import and setting up a new environment, which is highly complex.
- **Estimated Savings:** 10-25% on backend infrastructure costs.
- **Risk Level:** High (if migrating existing apps); Low (for new apps).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Data residency laws allow hosting in the target region; latency is acceptable.

### 7. Network & Data Transfer Optimization

#### 16. Aggressive Browser Caching via `customHttp.yml`
- **What:** Set long-lived `Cache-Control` headers for static assets (images, CSS, JS) so end-user browsers do not re-request them on subsequent visits.
- **Why It Saves Money:** Prevents repeated downloads of the same assets, directly reducing the $0.15/GB Amplify Data Served charges.
- **Implementation Steps:**
  1. Open the Amplify Console or edit `customHttp.yml` in the repository.
  2. Add custom headers to apply `Cache-Control: public, max-age=31536000, immutable` for paths like `/**/*.png`, `/**/*.css`, `/**/*.js`.
- **Estimated Savings:** 20-50% on data served egress costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Asset filenames include content hashes (standard in React/Next/Vue) to prevent cache invalidation issues.

#### 17. Enable Image Optimization
- **What:** Utilize Next.js Image Optimization (or similar framework features) to compress, resize, and serve modern image formats (WebP/AVIF).
- **Why It Saves Money:** Reduces the byte size of images sent over the wire, cutting down on the $0.15/GB Data Served costs while improving page load speeds.
- **Implementation Steps:**
  1. Use the `next/image` component instead of standard `<img>` tags.
  2. Ensure the Amplify build supports Next.js image optimization (requires Next.js 11+ on Amplify).
- **Estimated Savings:** 30-50% on image-heavy egress costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Next.js application framework.

#### 18. Block Malicious / Bot Traffic using WAF Rate Limiting (Production)
- **What:** Implement AWS WAF with rate-limiting rules on the production Amplify application to block scrapers, botnets, and DDoS attempts.
- **Why It Saves Money:** Malicious traffic can consume massive amounts of egress bandwidth at $0.15/GB. The $15/month WAF fee + small rule fees easily pay for themselves by preventing egress spikes.
- **Implementation Steps:**
  1. Attach AWS WAF to the production Amplify app.
  2. Configure AWS Managed Bot Control rules and rate-based rules (e.g., limit IPs to 2000 requests / 5 mins).
- **Estimated Savings:** Protects against unpredictable egress spikes (potentially thousands of dollars during an attack).
- **Risk Level:** Medium (risk of false positives blocking legitimate users).
- **Implementation Scope:** Engineer/DevOps | SecOps
- **Prerequisites:** Application is on a custom domain.

---

## Cross-Service Synergies
- **CloudFront & S3:** Ejecting heavy media assets to S3/CloudFront reduces Amplify egress costs significantly.
- **AWS WAF:** Prevents excessive data transfer costs caused by malicious bot traffic.
- **CloudWatch:** Managing log retention ensures backend Lambda logging doesn't silently accrue high storage fees.
- **Savings Plans:** Backend compute generated by Amplify can be optimized alongside enterprise-wide EC2/Fargate/Lambda commitments.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query `lineItem/ProductCode` for `AWSAmplify`.
- Filter by `lineItem/UsageType` to identify the split between `BuildDuration`, `Storage`, and `DataTransfer-Out-Bytes`.

### B. CloudWatch Metrics
- Monitor Lambda execution durations and invocations for Amplify SSR functions.
- Monitor 4xx and 5xx errors to identify potential bot traffic driving up egress costs.

### C. AWS Config / Trusted Advisor
- Identify Amplify applications without recent build activity.
- Check DynamoDB tables created by Amplify for Provisioned vs On-Demand capacity configurations.

### D. Company Policies
- Determine internal standards for branch deployment (e.g., should PR previews be built by default?).
- Establish log retention compliance requirements.

### E. IaC (Optional)
- Review `amplify.yml` build configurations for caching instructions.
- Review Amplify Gen 2 CDK code (`amplify/backend.ts`) for resource provisioning logic.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "AMPLIFY-001",
  "category": "Waste Elimination",
  "strategy_name": "Disable WAF on Staging Apps",
  "resource_id": "arn:aws:amplify:us-east-1:123456789012:apps/d123456789",
  "estimated_monthly_savings": 15.00,
  "level_of_effort": "Low",
  "risk_level": "Low",
  "status": "Open"
}
```

### Summary Report Table

| ID | Category | Strategy | Est. Savings | Effort | Risk |
|---|---|---|---|---|---|
| AMPLIFY-001 | Waste Elimination | Disable WAF on Staging Apps | $15/app/mo | Low | Low |
| AMPLIFY-002 | Waste Elimination | Limit Automatic PR Deployments | 30-60% build cost | Low | Low |
| AMPLIFY-003 | Waste Elimination | Delete Abandoned Amplify Apps | 100% idle cost | Low | Medium |
| AMPLIFY-004 | Waste Elimination | Clean up Stale Backend Stacks | Varies | Medium | High |
| AMPLIFY-005 | Rightsizing | Cache Build Dependencies | Up to 80% build cost | Low | Low |
| AMPLIFY-006 | Rightsizing | Optimize SSR Bundle Sizes | 10-20% storage/compute | Medium | Low |
| AMPLIFY-007 | Rightsizing | Restrict Logging Retention | 50-80% log cost | Low | Low |
| AMPLIFY-008 | Commitment Discounts | Compute Savings Plans | Up to 17% backend | Low | Medium |
| AMPLIFY-009 | Architecture Changes | Offload Heavy Media to S3/CF | 43% egress cost | Medium | Medium |
| AMPLIFY-010 | Architecture Changes | Eject Static Sites to S3/CF | 40-60% total cost | Medium | Medium |
| AMPLIFY-011 | Architecture Changes | Migrate SSR to SSG | 90% compute cost | High | Medium |
| AMPLIFY-012 | Architecture Changes | Monorepo Build Optimization | 50-80% build cost | Low | Low |
| AMPLIFY-013 | Scheduling/Auto-Scaling | Auto-Scale DynamoDB | 40-70% DB cost | Low | Low |
| AMPLIFY-014 | Pricing Model | Leverage Free Tier for Dev | Up to $25/mo | Low | Low |
| AMPLIFY-015 | Pricing Model | Deploy Backends in Cheaper Regions | 10-25% backend cost | High | Low |
| AMPLIFY-016 | Network/Data Transfer | Aggressive Browser Caching | 20-50% egress cost | Low | Low |
| AMPLIFY-017 | Network/Data Transfer | Enable Image Optimization | 30-50% egress cost | Low | Low |
| AMPLIFY-018 | Network/Data Transfer | Block Malicious Traffic with WAF | Avoids massive spikes | Medium | Medium |
