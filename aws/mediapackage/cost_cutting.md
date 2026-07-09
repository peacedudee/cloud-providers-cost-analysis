# Cost-Cutting Playbook: AWS Elemental MediaPackage
> **Companion File:** [mediapackage.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/mediapackage/mediapackage.md)
> **Last Updated:** July 2026

---
## Executive Summary
AWS Elemental MediaPackage is billed solely on data volume: data ingested from encoders ($0.05/GB) and data packaged/egressed to viewers or CDNs ($0.04/GB). Because there are no fixed hourly fees, cost optimization requires strict control over the data pipeline. The most critical cost-saving measure is placing a Content Delivery Network (CDN) like Amazon CloudFront in front of MediaPackage to offload packaging requests. This playbook details 15 strategies to eliminate waste, rightsize ingest payloads, optimize CDN caching, and refine streaming architectures to minimize data volume fees.

## Strategy Categories

### 1. Waste Elimination

#### 1. Delete Unused Live Channels
- **What:** Identify and delete MediaPackage live channels that are no longer actively receiving ingest data or serving viewers.
- **Why It Saves Money:** While MediaPackage doesn't charge hourly fees for channels, orphaned channels may still inadvertently receive "keep-alive" streams or test signals from upstream encoders, incurring $0.05/GB in ingest fees.
- **Implementation Steps:**
  1. Audit CloudWatch metrics (`IngestBytes`, `EgressBytes`) for all MediaPackage channels over the last 30 days.
  2. Identify channels with zero egress but non-zero ingest.
  3. Delete the channel and decommission the upstream encoder sending the data.
- **Estimated Savings:** 100% of wasted ingest costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS CLI/Console access

#### 2. Remove Orphaned Endpoints
- **What:** Delete MediaPackage endpoints (HLS, DASH, CMAF) that are no longer being queried by viewers or downstream CDNs.
- **Why It Saves Money:** Prevents accidental or unauthorized viewer requests from bypassing the CDN and hitting old endpoints directly, which incurs expensive $0.04/GB packaging charges.
- **Implementation Steps:**
  1. Review CloudWatch `EgressBytes` per endpoint.
  2. Identify endpoints with minimal, sporadic traffic that indicates direct requests rather than CDN origins.
  3. Delete unnecessary endpoints from the channel configuration.
- **Estimated Savings:** Variable (Prevents cost spikes)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Review of CDN origin configurations

#### 3. Delete Unused VOD Assets & Packaging Groups
- **What:** Clean up unused Video on Demand (VOD) assets and packaging groups in MediaPackage.
- **Why It Saves Money:** Reduces storage overhead (if applicable) and prevents accidental VOD packaging requests ($0.04/GB) for deprecated content.
- **Implementation Steps:**
  1. List all VOD assets via AWS CLI.
  2. Cross-reference assets with current content management system (CMS) catalogs.
  3. Delete orphaned or expired VOD assets and their associated packaging groups.
- **Estimated Savings:** 5-10% of VOD costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Content lifecycle policy

### 2. Rightsizing

#### 4. Optimize ABR Ladder to Reduce Ingest
- **What:** Tune the Adaptive Bitrate (ABR) ladder in the upstream encoder (e.g., MediaLive) to send only necessary renditions to MediaPackage.
- **Why It Saves Money:** MediaPackage charges $0.05/GB for ingested video. Dropping unnecessary high-bitrate renditions (e.g., an 8 Mbps 1080p stream if viewers only watch on mobile) directly reduces the ingest GB.
- **Implementation Steps:**
  1. Analyze viewer analytics to determine the most watched bitrates and resolutions.
  2. Remove unused or visually redundant renditions from the MediaLive output group.
  3. Ensure MediaLive is pushing the optimized ABR ladder to MediaPackage.
- **Estimated Savings:** 15-30% on ingest costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Viewer analytics data

#### 5. Shorten Time-Shifted Window (DVR Buffer)
- **What:** Reduce the start-over window (DVR buffer) configuration on MediaPackage endpoints.
- **Why It Saves Money:** While MediaPackage does not explicitly charge for internal storage, a massive 24-hour DVR window requires more segments to be indexed and managed. Shortening the window reduces internal processing and potential caching errors that lead to repeated origin requests.
- **Implementation Steps:**
  1. Review viewer behavior for time-shifted viewing.
  2. Update the endpoint configuration to reduce the `Start-over window` from 24 hours to 1-2 hours.
- **Estimated Savings:** Indirect savings on origin efficiency
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Time-shifted viewing metrics

#### 6. Consolidate Streaming Formats (Use CMAF)
- **What:** Standardize on Common Media Application Format (CMAF) for endpoints instead of generating separate HLS and DASH endpoints for the same content.
- **Why It Saves Money:** CMAF allows a single set of media segments to be referenced by both HLS and DASH manifests. This halves the number of unique segments the CDN must request from MediaPackage, drastically improving CDN cache hit ratios and reducing MediaPackage packaging/egress fees ($0.04/GB).
- **Implementation Steps:**
  1. Verify client player compatibility with CMAF.
  2. Create a CMAF endpoint on the MediaPackage channel.
  3. Migrate clients to use the CMAF endpoints and deprecate separate HLS/DASH endpoints.
- **Estimated Savings:** Up to 50% on packaging/egress costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CMAF-compatible video players

### 3. Commitment Discounts

#### 7. Negotiate AWS Private Pricing Agreements (EDP)
- **What:** Engage AWS for an Enterprise Discount Program (EDP) or a Private Pricing Term tailored to Media Services.
- **Why It Saves Money:** If your MediaPackage ingest and egress volumes exceed hundreds of TBs per month, AWS can offer custom volume discounts below the public $0.05/GB and $0.04/GB rates.
- **Implementation Steps:**
  1. Forecast 12-36 month MediaPackage data volumes.
  2. Contact your AWS Account Manager to request a Private Pricing Agreement for Media Services.
  3. Sign the commitment and apply the custom rates.
- **Estimated Savings:** 10-20% on all MediaPackage usage
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** High monthly AWS spend

### 4. Architecture Changes

#### 8. Always Front MediaPackage with CloudFront (CDN)
- **What:** Route 100% of viewer playback requests through an edge CDN like Amazon CloudFront instead of hitting MediaPackage directly.
- **Why It Saves Money:** Direct requests cost $0.04/GB for packaging. CloudFront caches the packaged segments. MediaPackage only packages a segment once per edge location. For 10,000 viewers, this reduces origin requests by 99.9%, replacing $800 direct packaging costs with $0.08 in origin packaging plus standard CDN egress rates.
- **Implementation Steps:**
  1. Create a CloudFront distribution pointing to the MediaPackage endpoint domain.
  2. Update video players to use the CloudFront URL.
  3. Implement strict security groups or origin access controls to block direct requests to MediaPackage.
- **Estimated Savings:** 90-99% on packaging costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudFront distribution setup

#### 9. Implement CloudFront Origin Shield
- **What:** Enable Origin Shield on the CloudFront distribution fronting MediaPackage.
- **Why It Saves Money:** Without Origin Shield, multiple CloudFront regional edge caches might request the same segment from MediaPackage independently. Origin Shield acts as a centralized caching layer, ensuring MediaPackage only receives ONE request globally per segment, further minimizing the $0.04/GB packaging fee.
- **Implementation Steps:**
  1. Identify the AWS region where MediaPackage is deployed.
  2. Enable Origin Shield in the CloudFront distribution settings and select the corresponding region.
- **Estimated Savings:** 10-20% on remaining packaging costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudFront fronting MediaPackage

#### 10. Optimize Cache-Control Headers for CDN
- **What:** Maximize the cache duration (TTL) for video segments and manifests.
- **Why It Saves Money:** Short TTLs cause the CDN to request the same segment from MediaPackage repeatedly. Increasing the TTL (e.g., setting immutable segments) maximizes CDN offload, preventing unnecessary $0.04/GB packaging fees.
- **Implementation Steps:**
  1. Analyze CloudFront cache hit ratios.
  2. Ensure MediaPackage is configured to send long Cache-Control headers for media segments (since they never change).
  3. Configure CloudFront behaviors to respect origin Cache-Control headers for segments.
- **Estimated Savings:** 5-15% on packaging costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudFront caching knowledge

#### 11. Migrate High-Traffic VOD to Static S3 Packaging
- **What:** Stop using MediaPackage VOD for long-tail or highly popular static Video on Demand content; instead, pre-package it with MediaConvert, store on S3, and serve via CloudFront.
- **Why It Saves Money:** MediaPackage VOD charges $0.04/GB for Just-in-Time packaging. If a VOD asset is watched millions of times, you pay for packaging continuously (on cache misses). Pre-packaging with MediaConvert has a one-time cost, and S3 to CloudFront data transfer is free (only paying CDN egress).
- **Implementation Steps:**
  1. Identify top-performing VOD assets in MediaPackage.
  2. Run a MediaConvert job to package these assets into HLS/DASH.
  3. Store outputs in S3 and route CloudFront to S3.
  4. Deprecate the MediaPackage VOD assets.
- **Estimated Savings:** 60-80% for high-traffic VOD
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS Elemental MediaConvert workflow

### 5. Scheduling & Auto-Scaling

#### 12. Stop Upstream Encoders (MediaLive) After Events
- **What:** Immediately stop upstream encoders sending data to MediaPackage once a live event concludes.
- **Why It Saves Money:** MediaPackage charges $0.05/GB for every byte ingested. If an encoder continues to send a slate or black screen after an event, you pay continuous ingest fees for no viewers.
- **Implementation Steps:**
  1. Integrate event scheduling systems with AWS Step Functions or Lambda.
  2. Trigger an API call to `StopChannel` on MediaLive precisely when the broadcast ends.
- **Estimated Savings:** 20-40% on ingest costs for event-based streams
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Automation framework (Lambda/EventBridge)

#### 13. Automate Pop-up Channel Teardown
- **What:** Delete MediaPackage channels entirely after ephemeral live events (e.g., sports broadcasts) are over.
- **Why It Saves Money:** Ensures zero chance of accidental data ingest and cleans up the AWS environment.
- **Implementation Steps:**
  1. Tag channels as "ephemeral" or "pop-up".
  2. Use EventBridge and Lambda to routinely scan for inactive pop-up channels and issue delete commands.
- **Estimated Savings:** Prevents accidental billing spikes
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Tagging strategy and Lambda functions

### 6. Pricing Model Optimization

#### 14. Use VOD Harvest Jobs Instead of Infinite DVR
- **What:** Instead of relying on a massive 14-day DVR window in MediaPackage for catch-up TV, use Harvest Jobs to export the live stream to S3 as a VOD asset.
- **Why It Saves Money:** Allows you to shorten the active DVR window on the live endpoint, shifting long-term catch-up viewing to a statically packaged VOD pipeline (via S3/CloudFront), bypassing ongoing MediaPackage fees for older content.
- **Implementation Steps:**
  1. Configure Harvest Jobs in MediaPackage to export completed live events.
  2. Store the harvested HLS in S3.
  3. Update the CMS to point viewers to the VOD S3 asset instead of the live DVR window after 2 hours.
- **Estimated Savings:** 15-25% for catch-up TV architectures
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** S3 and VOD delivery workflow

### 7. Network & Data Transfer Optimization

#### 15. Colocate Ingest Sources in the Same Region
- **What:** Ensure that the on-premises encoder, EC2 instance, or AWS Elemental MediaLive channel is in the exact same AWS Region as the MediaPackage channel.
- **Why It Saves Money:** Sending video data across AWS regions incurs cross-region data transfer fees ($0.01 - $0.02/GB) on top of the $0.05/GB MediaPackage ingest fee. Same-region ingest avoids these extra DTO charges.
- **Implementation Steps:**
  1. Audit the regions of all upstream encoders.
  2. Migrate MediaPackage channels or MediaLive channels to ensure they reside in the same region.
- **Estimated Savings:** 100% of cross-region data transfer fees
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region architecture review

---
## Cross-Service Synergies
- **CloudFront:** Essential for eliminating 99.9% of packaging fees via caching and Origin Shield.
- **MediaLive:** Tightly coupled; optimizing MediaLive ABR ladders directly reduces MediaPackage ingest costs.
- **MediaConvert & S3:** Offloads high-traffic VOD packaging from MediaPackage's Just-in-Time per-GB model to a cheaper static delivery model.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query `lineItem/ProductCode` for `AWSElementalMediaPackage`.
- Filter by `UsageType` containing `Ingest` and `Packaging`.

### B. CloudWatch Metrics
- `IngestBytes`: To track incoming data volumes from encoders.
- `EgressBytes`: To track origin packaging requests (should be minimal if CDN is working).

### C. AWS Config / Trusted Advisor
- Track endpoint configurations, specifically CDN authorization and direct-access prevention.

### D. Company Policies
- SLA requirements for live stream latency and DVR catch-up window lengths.

### E. IaC (Optional)
- Terraform/CloudFormation templates to verify CloudFront is deployed in front of all MediaPackage endpoints.

---
## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "MEDIAPK-001",
  "strategy_name": "Always Front MediaPackage with CloudFront (CDN)",
  "resource_id": "arn:aws:mediapackage:us-east-1:123456789012:channels/live-sports",
  "current_state": "Direct viewer access bypassing CDN",
  "recommended_state": "All traffic routed through CloudFront",
  "monthly_savings_usd": 1500.00,
  "effort_level": "Medium"
}
```

### Summary Report Table
| Finding ID | Strategy | Resource | Est. Savings | Risk Level |
|------------|----------|----------|--------------|------------|
| MEDIAPK-001 | CloudFront Fronting | live-sports | $1,500.00 | Low |
| MEDIAPK-002 | ABR Ladder Opt | news-channel | $350.00 | Medium |
