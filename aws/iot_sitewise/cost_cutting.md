# Cost-Cutting Playbook: AWS IoT SiteWise
> **Companion File:** [iot_sitewise.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/iot_sitewise/iot_sitewise.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS IoT SiteWise provides powerful industrial data modeling, ingestion, and monitoring capabilities, but costs can skyrocket due to high-frequency sensor ingestion and mismanaged storage tiers. This playbook outlines 20 actionable strategies to optimize IoT SiteWise spend, focusing heavily on edge processing, ingestion buffering, lifecycle policies, and architectural adjustments. Proper implementation can reduce ingestion and storage costs by over 90% in high-volume industrial environments.

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
- **AWS IoT Greengrass:** Hosts SiteWise Edge components to enable local processing and aggregation before cloud ingestion.
- **AWS IoT Events:** Triggers alarms based on SiteWise properties; unused alarms should be cleaned up.
- **Amazon S3:** Serves as the Cold Tier for SiteWise storage, reducing long-term archival costs.
- **Amazon CloudWatch:** Monitors SiteWise ingestion rates and active users to detect anomalies or billing spikes.
- **AWS PrivateLink (VPC Endpoints):** Reduces data transfer out / NAT Gateway costs for telemetry sent from VPCs.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- **LineItem/ProductCode:** `AWSIoTSiteWise`
- **LineItem/Operation:** `Ingest`, `Storage`, `Compute`, `PortalUser`
- **Pricing/usagetype:** e.g., `USW2-IngestedMessages`, `USW2-HotStorage-GB-Month`

### B. CloudWatch Metrics
- `IngestedMessages` / `IngestedDataSize`
- `ComputedValues`
- Active Gateway metrics

### C. AWS Config / Trusted Advisor
- Inventory of SiteWise Gateways, Asset Models, Assets, and Portals.

### D. Company Policies
- Data retention requirements (Hot vs. Warm vs. Cold).
- Dashboard and monitoring access policies.

### E. IaC (Optional)
- CloudFormation / Terraform templates for SiteWise asset definitions and portal configurations.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "IOTSW-001",
  "strategy_category": "Architecture Changes",
  "resource_id": "gateway-12345",
  "issue": "Raw telemetry ingested directly to cloud",
  "recommendation": "Deploy SiteWise Edge for local aggregation",
  "estimated_savings_monthly": 259800.00,
  "level_of_effort": "Medium"
}
```

### Summary Report Table
| Finding ID | Category | Resource | Recommendation | Est. Savings | Effort |
|---|---|---|---|---|---|
| IOTSW-001 | Architecture | factory-gateway | Deploy SiteWise Edge for local aggregation | $259K+ | Medium |
| IOTSW-002 | Rightsizing | sitewise-storage | Configure Hot-to-Warm lifecycle policy | $800 | Low |

---

#### 1. Clean Up Unused Assets and Asset Models
- **What:** Identify and delete asset models and assets that are no longer receiving data or actively used in dashboards.
- **Why It Saves Money:** Stale properties may still incur storage costs, and cleaning up simplifies the hierarchy and prevents accidental data ingestion to dead endpoints.
- **Implementation Steps:**
  1. Audit assets and properties using AWS CLI/SDK.
  2. Identify assets with zero ingested messages over 30 days.
  3. Delete orphaned assets and their associated models.
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Visibility into asset data flow.

#### 2. Deactivate Dormant SiteWise Monitor Users
- **What:** Remove users who haven't logged into SiteWise Monitor recently.
- **Why It Saves Money:** You are billed $10.00 per unique active user per month. Removing inactive users eliminates this overhead.
- **Implementation Steps:**
  1. Review IAM Identity Center / SiteWise Monitor access logs.
  2. Identify users inactive for >30 days.
  3. Remove user access from SiteWise Portals.
- **Estimated Savings:** 5-15% of portal costs
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Audit logs for user access.

#### 3. Purge Orphaned IoT Events Alarms
- **What:** Delete IoT Events alarms connected to SiteWise properties that are no longer relevant.
- **Why It Saves Money:** Alarms cost $0.10 per active alarm per month, plus computation and evaluation charges.
- **Implementation Steps:**
  1. Review configured alarms in AWS IoT Events.
  2. Verify if the underlying SiteWise properties are still active.
  3. Delete alarms that are obsolete.
- **Estimated Savings:** <5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Access to IoT Events console.

#### 4. Disable Unused Asset Properties
- **What:** Disable telemetry ingestion for specific equipment properties that are not utilized in analytics or dashboards.
- **Why It Saves Money:** Avoids the $1.00 per 1M message ingestion fee and subsequent storage costs for useless data.
- **Implementation Steps:**
  1. Interview plant operators to determine critical vs. non-critical metrics.
  2. Stop pushing non-critical metric data from the factory edge.
  3. Remove properties from the Asset Model in SiteWise.
- **Estimated Savings:** 10-30%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Stakeholder alignment on required metrics.

#### 5. Optimize Telemetry Measurement Frequency
- **What:** Reduce the sampling rate of sensors (e.g., from 10ms to 1s or 10s) at the source PLC or gateway.
- **Why It Saves Money:** Sending fewer messages directly reduces the $1.00/1M message ingestion cost and associated storage.
- **Implementation Steps:**
  1. Analyze dashboard refresh rates and analytics requirements.
  2. Reconfigure PLC or edge gateway to lower the polling/publishing frequency.
- **Estimated Savings:** 50-90% of ingestion costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Hardware/PLC configuration access.

#### 6. Implement Storage Lifecycle Policies
- **What:** Automatically transition data from Hot Tier to Warm Tier, and eventually to Cold Tier (S3).
- **Why It Saves Money:** Hot Tier ($0.30/GB-month) is 10x more expensive than Warm Tier ($0.029/GB-month). Cold storage in S3 is even cheaper.
- **Implementation Steps:**
  1. Navigate to SiteWise Settings.
  2. Enable Warm Tier storage and set a retention period for the Hot Tier (e.g., 30 days).
  3. Enable Cold Tier storage and map to an S3 bucket with its own Glacier lifecycle rules.
- **Estimated Savings:** 80-90% of storage costs
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** S3 bucket for cold data.

#### 7. Downgrade Edge Gateway Packs
- **What:** Use the free Data Collection Pack instead of the $200/month Data Processing Pack if local computations aren't needed.
- **Why It Saves Money:** Saves a flat $200 per active gateway per month.
- **Implementation Steps:**
  1. Assess if the gateway performs local metric aggregation, transformations, or hosts local dashboards.
  2. If only forwarding telemetry, change the gateway configuration to the Data Collection Pack.
- **Estimated Savings:** $200/month per gateway
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Audit of edge gateway workloads.

#### 8. Tune Metric Computation Intervals
- **What:** Increase the time interval for metric calculations (e.g., compute averages every 15 minutes instead of every 1 minute).
- **Why It Saves Money:** Computations are billed at $0.50 per 1M calculated values. Fewer intervals mean fewer computations.
- **Implementation Steps:**
  1. Review asset models in SiteWise.
  2. Edit metrics to use larger time windows.
- **Estimated Savings:** 10-40% of compute costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of reporting requirements.

#### 9. Leverage AWS Enterprise Discount Program (EDP)
- **What:** Secure global discounts on AWS IoT services via an EDP commitment.
- **Why It Saves Money:** While SiteWise has no specific Reserved Instances, EDP provides a blanket percentage discount across all AWS usage.
- **Implementation Steps:**
  1. Consolidate AWS spend to meet EDP thresholds ($1M+ annual).
  2. Negotiate EDP terms with AWS account team.
- **Estimated Savings:** 5-15% overall
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** High overall AWS spend.

#### 10. Implement SiteWise Edge for Local Industrial Aggregation
- **What:** Deploy the Data Processing Pack on a local gateway to compute metrics at the edge and only send aggregated data to the cloud.
- **Why It Saves Money:** Slashes ingestion volume (e.g., 260B messages to 3.6M messages), turning a $260,000 ingestion bill into $3.60, vastly outweighing the $200 edge pack fee.
- **Implementation Steps:**
  1. Provision a local industrial PC or gateway.
  2. Install AWS IoT Greengrass and SiteWise Edge.
  3. Configure local models and set cloud sync to only upload aggregated metrics.
- **Estimated Savings:** 90-99%+
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Local hardware availability and networking.

#### 11. Implement Buffered Ingestion for Non-Critical Telemetry
- **What:** Use buffered ingestion rather than near real-time ingestion to pack data into 5KB increments.
- **Why It Saves Money:** Buffered ingestion packs 60 data points or 5KB per metered increment, whereas real-time uses 10 data points or 1KB, effectively reducing ingestion charges by up to 6x.
- **Implementation Steps:**
  1. Update MQTT publishing topics or SiteWise API calls to use the buffered ingestion endpoints.
  2. Ensure applications tolerate the minor buffering latency.
- **Estimated Savings:** Up to 80% of ingestion costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Code/firmware update capabilities.

#### 12. Filter Noise/Jitter Data Before Ingestion
- **What:** Drop duplicate, invalid, or noisy data points at the edge before they are transmitted to SiteWise.
- **Why It Saves Money:** Prevents paying $1.00/1M messages and storage fees for useless or redundant data points (e.g., temperature hasn't changed).
- **Implementation Steps:**
  1. Implement a deadband filter or delta-compression script in an AWS Greengrass Lambda function on the edge gateway.
  2. Only send data if it deviates significantly from the last reading.
- **Estimated Savings:** 20-50%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Greengrass or custom edge logic.

#### 13. Maximize Payload Efficiency for Ingestion Increments
- **What:** Batch telemetry into precisely sized payloads to fully utilize the 1KB (real-time) or 5KB (buffered) metered increments.
- **Why It Saves Money:** Sending a 100-byte message costs the same as a 1KB message in real-time mode. Packing 10 readings into a single 1KB message avoids paying 10x the price.
- **Implementation Steps:**
  1. Modify device firmware or edge gateway to accumulate readings.
  2. Send payloads close to the 1KB or 5KB threshold.
- **Estimated Savings:** 50-90% of ingestion costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Payload size profiling.

#### 14. Offload High-Volume Raw Telemetry to Timestream/S3
- **What:** Send high-frequency raw data directly to Amazon Timestream or S3, and only send specific structured metrics to SiteWise.
- **Why It Saves Money:** S3 is vastly cheaper for raw binary telemetry storage. SiteWise is a premium service for structured asset models and should not be used as a raw data dump.
- **Implementation Steps:**
  1. Use AWS IoT Core topic rules to route raw data to S3 or Timestream.
  2. Only route business-critical KPIs to SiteWise.
- **Estimated Savings:** 60-80%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Data lake architecture.

#### 15. Optimize Asset Hierarchy to Minimize Redundant Computations
- **What:** Flatten overly deep asset hierarchies where computations simply roll up identical values without adding insight.
- **Why It Saves Money:** Computations are charged per calculated value. Complex multi-layered rollups multiply the number of computations.
- **Implementation Steps:**
  1. Review asset models for unnecessary intermediary rollup levels.
  2. Simplify models to only compute metrics at the necessary parent levels.
- **Estimated Savings:** 10-25% of compute costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of the physical asset layout.

#### 16. Schedule Edge Gateway Synchronizations During Off-Peak
- **What:** Only synchronize historical/buffered data from the edge to the cloud during off-peak windows.
- **Why It Saves Money:** While not directly reducing SiteWise unit costs, this reduces peak bandwidth requirements and potential network throughput charges.
- **Implementation Steps:**
  1. Configure edge buffers to hold non-critical data.
  2. Set a cron schedule to push data overnight.
- **Estimated Savings:** Indirect (Bandwidth/Network)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Edge local storage capacity.

#### 17. Pause Computations for Out-of-Service Equipment
- **What:** Stop sending data and halt computations for machines that are under maintenance or decommissioned.
- **Why It Saves Money:** Prevents the system from computing "zero" or "offline" metrics continuously, saving ingestion and compute fees.
- **Implementation Steps:**
  1. Integrate maintenance schedules (CMMS) with IoT rules.
  2. Dynamically disable telemetry forwarding or asset properties when a machine is marked out of service.
- **Estimated Savings:** 1-5%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Integration with maintenance systems.

#### 18. Leverage the SiteWise Free Tier for PoCs
- **What:** Utilize the AWS Free Tier (10,000 metrics/messages, 1GB storage for 2 months) for Proof of Concepts instead of dedicated dev accounts.
- **Why It Saves Money:** Prevents incurring immediate costs while testing industrial architectures.
- **Implementation Steps:**
  1. Use a fresh AWS account for the PoC.
  2. Strictly monitor usage against free tier limits.
- **Estimated Savings:** 100% of PoC costs (up to limit)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** New AWS Account.

#### 19. Separate Hot Dashboard Data from Historical Analytics Data
- **What:** Only keep data necessary for live operational dashboards in the Hot Tier. Export analytics data to S3 (Cold Tier) or Athena for reporting.
- **Why It Saves Money:** Avoids using SiteWise Warm/Hot tiers for heavy long-term analytical queries.
- **Implementation Steps:**
  1. Adjust Hot Tier retention to exactly match dashboard lookback windows (e.g., 7 days).
  2. Perform heavy analytical BI queries using Amazon Athena against the S3 Cold Tier data.
- **Estimated Savings:** 30-50% of storage/query costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Cold Tier export configured.

#### 20. Use VPC Endpoints (AWS PrivateLink) for SiteWise
- **What:** If gateways or EC2 instances within a VPC send data to SiteWise, use a VPC Endpoint rather than routing through a NAT Gateway.
- **Why It Saves Money:** NAT Gateways charge ~$0.045 per GB of data processed. VPC Endpoints for SiteWise are cheaper and avoid public data transfer fees.
- **Implementation Steps:**
  1. Create an Interface VPC Endpoint for AWS IoT SiteWise in the VPC.
  2. Ensure edge devices in the VPC resolve the SiteWise API locally.
- **Estimated Savings:** Variable (Network Transfer Costs)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC networking topology.
