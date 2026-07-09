# Cost-Cutting Playbook: AWS Elemental MediaStore
> **Companion File:** [mediastore.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/mediastore/mediastore.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Elemental MediaStore was officially discontinued on November 13, 2025. Because the service is no longer operational, the primary cost-optimization focus shifts from managing active MediaStore resources to completing migrations, eliminating residual billing artifacts, tearing down legacy integrations (like CloudFront distributions and IAM roles pointing to dead endpoints), and optimizing the new replacement architectures (Amazon S3 and AWS Elemental MediaPackage). This playbook outlines strategies to ensure all legacy MediaStore waste is eradicated and that migrated workflows operate at maximum cost efficiency.

## Strategy Categories
### 1. Waste Elimination
#### MEDIAST-001. Identify and Resolve Any Residual MediaStore Billing Line Items
- **What:** Audit the AWS Cost and Usage Report (CUR) for any lingering "AWS Elemental MediaStore" charges due to incomplete teardowns, suspended accounts, or billing anomalies post-deprecation.
- **Why It Saves Money:** Eliminates 100% of the cost of a dead service that might be erroneously billed due to technical edge cases.
- **Implementation Steps:** 1. Query the AWS CUR for the MediaStore service code. 2. Open an AWS Support ticket if charges appear after November 13, 2025. 3. Request a refund for erroneous charges.
- **Estimated Savings:** 100% of residual charges
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Access to AWS CUR and AWS Billing Support.

#### MEDIAST-002. Purge Legacy CloudFront Distributions Fronting Discontinued MediaStore Origins
- **What:** Identify and delete CloudFront distributions that still have MediaStore endpoints configured as origins, which are now returning errors.
- **Why It Saves Money:** Avoids unnecessary DNS resolution costs, WAF evaluation costs (if attached), and potential request charges from automated scrapers hitting dead endpoints.
- **Implementation Steps:** 1. List CloudFront distributions. 2. Filter origins containing `*.data.mediastore.*.amazonaws.com`. 3. Delete distributions if the workflow is retired, or update to the new S3/MediaPackage origin.
- **Estimated Savings:** $1 - $50/month per unused distribution (plus associated WAF fees)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudFront read/write access.

#### MEDIAST-003. Remove Unused MediaStore IAM Policies and Roles
- **What:** Delete IAM roles, policies, and permissions boundaries that were explicitly created to grant access to MediaStore (e.g., `mediastore:PutObject`).
- **Why It Saves Money:** While IAM itself is free, cleaning up policies prevents security risks and reduces the operational overhead of IAM governance, indirectly saving engineering time.
- **Implementation Steps:** 1. Search IAM for policies containing `mediastore:*`. 2. Identify roles using these policies (like legacy MediaLive roles). 3. Detach and delete the policies.
- **Estimated Savings:** Indirect (Security/Ops overhead)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** IAM administrative access.

#### MEDIAST-004. Delete Abandoned AWS Elemental MediaLive Outputs Pointing to MediaStore
- **What:** Locate inactive or erroring MediaLive channels that are still configured to output to legacy MediaStore URLs.
- **Why It Saves Money:** Idle or paused MediaLive channels may still incur minor costs or block quotas. Removing dead outputs prevents operators from accidentally starting broken channels, which would incur MediaLive hourly fees.
- **Implementation Steps:** 1. Describe MediaLive channels. 2. Check destination settings for MediaStore URLs. 3. Delete or reconfigure the channels to S3 or MediaPackage.
- **Estimated Savings:** Prevents accidental $0.50 - $4.00/hour MediaLive run costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** MediaLive access.

#### MEDIAST-005. Clean Up Associated CloudWatch Logs and Metrics Alarms for MediaStore
- **What:** Delete CloudWatch alarms tracking MediaStore metrics and set retention policies or delete legacy MediaStore access/error logs.
- **Why It Saves Money:** CloudWatch alarms cost $0.10/alarm/month, and Log storage costs $0.03/GB-month.
- **Implementation Steps:** 1. Identify alarms with the `AWS/MediaStore` namespace. 2. Delete the alarms. 3. Identify Log Groups related to MediaStore. 4. Delete them or set a short TTL (e.g., 7 days).
- **Estimated Savings:** 100% of legacy alarm and log storage costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch access.

### 2. Rightsizing
#### MEDIAST-006. Rightsize Legacy Video Storage During Migration
- **What:** Instead of executing a 1-to-1 migration of all legacy MediaStore data to S3, aggressively purge old, unneeded HLS/DASH video segments.
- **Why It Saves Money:** Prevents paying $0.023/GB-month in S3 for stale live-streaming segments that are no longer requested by viewers.
- **Implementation Steps:** 1. Analyze video segment age prior to migration. 2. Delete segments older than the required DVR window (e.g., > 14 days). 3. Migrate only the active catalog.
- **Estimated Savings:** 30-80% of migration storage footprint
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Scripting access to MediaStore (if still accessible via support exception) or during the migration sync.

#### MEDIAST-007. Optimize S3 Bucket Lifecycle Rules for Migrated Content
- **What:** Implement lifecycle policies on the S3 buckets that replaced MediaStore to automatically expire old HLS/DASH segments.
- **Why It Saves Money:** Live video segments lose value rapidly. Deleting them automatically after the playback window saves S3 Standard storage costs ($0.023/GB-month).
- **Implementation Steps:** 1. Navigate to the S3 bucket acting as the new origin. 2. Create a lifecycle rule for prefixes containing video segments. 3. Set expiration to 7-30 days based on DVR requirements.
- **Estimated Savings:** 70-90% of long-term storage costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** S3 configuration access.

#### MEDIAST-008. Review MediaPackage Target Capacities
- **What:** For workflows migrated to MediaPackage, review the configured channel and packaging settings to ensure they are not over-provisioned (e.g., generating unused bitrates or DRM formats).
- **Why It Saves Money:** MediaPackage charges $0.05/GB ingested and $0.04/GB packaged/originated. Unused renditions increase both ingest and packaging costs.
- **Implementation Steps:** 1. Review MediaPackage endpoint configurations. 2. Analyze CloudFront logs to see which stream variants are actually requested. 3. Remove unused variants from the MediaPackage configuration.
- **Estimated Savings:** 10-25% of MediaPackage costs
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** MediaPackage and CloudFront log access.

### 3. Commitment Discounts
#### MEDIAST-009. Leverage S3 Storage Class Analysis for Long-Term Discounts on Migrated Assets
- **What:** For VOD assets migrated from MediaStore to S3, use S3 Storage Class Analysis to identify objects that can move to S3 Infrequent Access (IA) or Glacier.
- **Why It Saves Money:** S3 Standard is $0.023/GB-month, while S3 Standard-IA is $0.0125/GB-month, reducing storage costs by ~45%.
- **Implementation Steps:** 1. Enable S3 Storage Class Analysis on the destination bucket. 2. Wait 30 days for data. 3. Implement lifecycle transitions based on the analysis.
- **Estimated Savings:** 40-50% on storage of inactive assets
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** S3 Management access.

#### MEDIAST-010. Adopt CloudFront Security Savings Bundle for Migrated Edge Delivery
- **What:** Commit to a monthly CloudFront spend to receive a discount on data transfer for the CloudFront distributions fronting the new S3/MediaPackage origins.
- **Why It Saves Money:** Video streaming is highly bandwidth-intensive. The Savings Bundle provides up to 30% off CloudFront usage in exchange for a 1-year commitment.
- **Implementation Steps:** 1. Analyze consistent baseline CloudFront data transfer. 2. Purchase a CloudFront Security Savings Bundle in the AWS Billing console covering 70-80% of the baseline.
- **Estimated Savings:** Up to 30% on CloudFront egress
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Stable traffic patterns, Billing access.

### 4. Architecture Changes
#### MEDIAST-011. Migrate Standard Live Origination to Amazon S3
- **What:** Fully transition legacy MediaStore workloads that do not require advanced packaging to S3.
- **Why It Saves Money:** S3 Standard provides the same high throughput and strong read-after-write consistency required for HLS/DASH origination at $0.023/GB-month, serving as the official AWS recommended direct replacement.
- **Implementation Steps:** 1. Provision an S3 bucket. 2. Update encoder (e.g., MediaLive) destinations to the S3 bucket. 3. Point CloudFront origins to the S3 bucket.
- **Estimated Savings:** Maintains cost parity with MediaStore while ensuring service continuity
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Basic understanding of S3 and CDN integration.

#### MEDIAST-012. Migrate Advanced Video Workflows to AWS Elemental MediaPackage
- **What:** For complex workflows (DRM, multi-format out), transition from MediaStore to AWS Elemental MediaPackage.
- **Why It Saves Money:** While MediaPackage has a different pricing structure ($0.05/GB ingest), it eliminates the need to run multiple encoders for different formats, saving significant compute and encoding licensing costs upstream.
- **Implementation Steps:** 1. Configure a MediaPackage channel. 2. Output a single high-quality mezzanine stream from the encoder. 3. Let MediaPackage handle JIT packaging.
- **Estimated Savings:** 20-40% on upstream encoding compute costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of Just-In-Time (JIT) packaging.

#### MEDIAST-013. Consolidate Multiple Legacy Endpoints into Single S3/MediaPackage Architectures
- **What:** Instead of having a 1:1 mapping of legacy MediaStore containers to new buckets, consolidate multiple video channels into a single S3 bucket or shared MediaPackage channels with distinct prefixes.
- **Why It Saves Money:** Reduces operational overhead, simplifies CloudFront origin configurations, and aggregates storage metrics to make better lifecycle/tiering decisions.
- **Implementation Steps:** 1. Audit all legacy MediaStore endpoints. 2. Map multiple channel paths to a single S3 bucket (e.g., `/channel1/`, `/channel2/`). 3. Update IAM and CloudFront configurations accordingly.
- **Estimated Savings:** Indirect (Operational efficiency)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Architecture planning.

#### MEDIAST-014. Optimize Migration Data Transfer Using AWS DataSync
- **What:** If migrating off an on-prem or alternative interim storage solution back to S3 (post-MediaStore shutdown), use AWS DataSync for the bulk transfer.
- **Why It Saves Money:** DataSync optimizes network utilization and can transfer data faster and more reliably than custom scripts, reducing compute hours for migration instances.
- **Implementation Steps:** 1. Deploy DataSync agents if necessary. 2. Configure tasks to move video archives to the new S3 origin bucket. 3. Monitor and complete the transfer.
- **Estimated Savings:** Reduces EC2 migration compute time by up to 50%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS DataSync setup.

### 5. Scheduling & Auto-Scaling
#### MEDIAST-015. Schedule MediaLive Channels Pushing to Replacement Origins
- **What:** Turn off AWS Elemental MediaLive channels during non-broadcast hours (e.g., overnight for regional sports).
- **Why It Saves Money:** MediaLive charges an hourly rate for active channels. Stopping them when no live content is being generated saves up to $2.00-$4.00/hour.
- **Implementation Steps:** 1. Identify non-24/7 channels. 2. Use AWS EventBridge and Lambda to schedule StartChannel and StopChannel API calls.
- **Estimated Savings:** 30-50% of MediaLive costs for part-time streams
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Predictable broadcast schedules.

### 6. Pricing Model Optimization
#### MEDIAST-016. Transition to S3 Intelligent-Tiering for Migrated VOD Assets
- **What:** Use S3 Intelligent-Tiering for video segments that transition from Live to VOD and have unpredictable access patterns.
- **Why It Saves Money:** Automatically moves rarely accessed video chunks to lower-cost tiers (saving up to 68%) without retrieval fees or lifecycle management overhead.
- **Implementation Steps:** 1. Enable S3 Intelligent-Tiering on the destination bucket. 2. Monitor monitoring & automation fees ($0.0025 per 1,000 objects) to ensure it is cost-effective for the object count.
- **Estimated Savings:** 20-40% on storage costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Minimum object size of 128KB (small segments may not benefit).

### 7. Network & Data Transfer Optimization
#### MEDIAST-017. Force All End-User Video Playback Through CloudFront
- **What:** Ensure that end-users cannot access the new S3 or MediaPackage origins directly, forcing all traffic through Amazon CloudFront.
- **Why It Saves Money:** Data transfer out from S3 to the internet costs $0.09/GB. Data transfer from S3 to CloudFront is $0.00/GB, and CloudFront egress is cheaper (starting at $0.085/GB, often heavily discounted).
- **Implementation Steps:** 1. Implement Origin Access Control (OAC) on the S3 bucket. 2. Update bucket policies to deny direct internet access. 3. Verify video playback routes through the CDN.
- **Estimated Savings:** 10-50% on Data Transfer Out (depending on CDN discounts)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudFront and S3 IAM setup.

#### MEDIAST-018. Co-locate Encoders and Origins in the Same AWS Region
- **What:** Ensure that AWS Elemental MediaLive (or 3rd party encoders) pushes data to the S3 bucket or MediaPackage channel located in the exact same AWS Region.
- **Why It Saves Money:** Cross-region data transfer from EC2/MediaLive to S3 in another region incurs $0.01-$0.02/GB. In-region data transfer to S3 is free.
- **Implementation Steps:** 1. Audit the region of your encoders and storage origins. 2. Re-provision the S3 bucket/MediaPackage in the encoder's region if mismatched.
- **Estimated Savings:** 100% of cross-region ingest fees
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region deployment review.

#### MEDIAST-019. Maximize CloudFront Cache Hit Ratios for Video Segments
- **What:** Optimize Cache-Control headers on the replacement S3 bucket or MediaPackage origin to ensure video segments are cached as long as possible at the edge.
- **Why It Saves Money:** Every cache miss incurs an S3 GET request ($0.0004 per 1,000 requests) and potential internal data transfer. Maximizing cache hit ratio reduces origin load and associated request costs.
- **Implementation Steps:** 1. Set `Cache-Control: max-age=31536000` for immutable video segments (.ts, .m4s). 2. Set shorter TTLs (e.g., 2-4 seconds) for live manifests (.m3u8, .mpd).
- **Estimated Savings:** 10-30% on S3 GET request costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CDN caching knowledge.

#### MEDIAST-020. Review and Disable Legacy Cross-Region Replication
- **What:** Turn off any S3 Cross-Region Replication (CRR) rules or custom Lambda scripts that were previously syncing MediaStore data to backup regions if that disaster recovery model is no longer required in the new S3/MediaPackage architecture.
- **Why It Saves Money:** CRR incurs cross-region data transfer costs ($0.01-$0.02/GB) plus storage costs in the destination region.
- **Implementation Steps:** 1. Check the replacement S3 buckets for active replication rules. 2. Disable rules if a multi-region DR strategy is not business-mandated.
- **Estimated Savings:** 50% on storage and 100% on inter-region transfer costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** DR policy alignment.

---
## Cross-Service Synergies
- **CloudFront:** Essential for delivering video securely and cheaply, effectively replacing MediaStore's edge optimization.
- **Amazon S3:** The primary target for standard live origination post-MediaStore deprecation.
- **AWS Elemental MediaPackage:** The primary target for advanced streaming architectures (JIT packaging, DRM).
- **AWS IAM & CloudWatch:** Key services for cleaning up legacy dependencies, alarms, and unused permissions post-migration.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
Identify any lingering MediaStore line items, assess S3 storage costs for migrated origins, and analyze CloudFront data transfer out costs.
### B. CloudWatch Metrics
Review active alarms related to `AWS/MediaStore` namespace and monitor `CacheHitRate` on CloudFront to ensure new origins are heavily cached.
### C. AWS Config / Trusted Advisor
Find unused IAM policies and detect unattached CloudFront distributions originally pointing to MediaStore.
### D. Company Policies
Understand Video DVR retention windows to aggressively expire stale segments in S3.
### E. IaC (Optional)
Audit Terraform/CloudFormation for leftover MediaStore resources, legacy IAM roles, or hardcoded MediaStore endpoint URLs.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "MEDIAST-001",
  "service": "AWS Elemental MediaStore",
  "strategy_category": "Waste Elimination",
  "description": "Identify and Resolve Any Residual MediaStore Billing Line Items.",
  "savings_potential_percentage": 100,
  "effort_level": "Low"
}
```
### Summary Report Table
| Finding ID | Strategy | Category | Est. Savings | Risk |
|------------|----------|----------|--------------|------|
| MEDIAST-001 | Resolve Residual Billing | Waste Elimination | 100% | Low |
| MEDIAST-002 | Purge Legacy CloudFront | Waste Elimination | $1-$50/dist | Medium |
| MEDIAST-003 | Remove Unused IAM | Waste Elimination | Indirect | Low |
| MEDIAST-004 | Delete Abandoned Outputs | Waste Elimination | $0.50-$4/hr | Low |
| MEDIAST-005 | Clean Up CW Logs & Alarms | Waste Elimination | 100% of log cost | Low |
| MEDIAST-006 | Rightsize Legacy Video Storage | Rightsizing | 30-80% | Medium |
| MEDIAST-007 | Optimize S3 Lifecycle Rules | Rightsizing | 70-90% | Low |
| MEDIAST-008 | Review MediaPackage Targets | Rightsizing | 10-25% | High |
| MEDIAST-009 | Leverage S3 Storage Class Analysis | Commitment Discounts | 40-50% | Low |
| MEDIAST-010 | Adopt CloudFront Savings Bundle | Commitment Discounts | Up to 30% | Medium |
| MEDIAST-011 | Migrate to Amazon S3 | Architecture Changes | Cost Parity | Low |
| MEDIAST-012 | Migrate to MediaPackage | Architecture Changes | 20-40% compute | Medium |
| MEDIAST-013 | Consolidate Endpoints | Architecture Changes | Indirect | Low |
| MEDIAST-014 | Optimize Migration with DataSync | Architecture Changes | Compute reduction | Low |
| MEDIAST-015 | Schedule MediaLive Channels | Scheduling & Auto-Scaling | 30-50% | Medium |
| MEDIAST-016 | Transition to S3 Intelligent-Tiering | Pricing Model Optimization | 20-40% | Low |
| MEDIAST-017 | Force CDN Playback | Network & Data Transfer | 10-50% | High |
| MEDIAST-018 | Co-locate Encoders and Origins | Network & Data Transfer | 100% x-region fee | Low |
| MEDIAST-019 | Maximize Cache Hit Ratios | Network & Data Transfer | 10-30% | Low |
| MEDIAST-020 | Disable Legacy CRR | Network & Data Transfer | 50% storage | Medium |
