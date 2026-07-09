# Cost-Cutting Playbook: AWS Ground Station
> **Companion File:** [ground_station.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/ground_station/ground_station.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Ground Station is a highly specialized, fully managed service for satellite communications, billed primarily by contact minute and bandwidth tier (Narrowband vs Wideband). While the core pricing model is straightforward, optimizing costs requires meticulous orchestration of scheduled contacts, rigid adherence to cancellation policies, and strategic integration with downstream AWS services (like EC2 and S3) where the data is processed and stored. This playbook outlines 18 actionable strategies to eliminate waste from missed cancellations, rightsize contact windows, leverage commitment discounts, and streamline the network architecture that ingests satellite data.

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
- **Amazon EC2:** EC2 instances are often used as Software Defined Radios (SDR) or data processors during contacts. Optimizing Ground Station means also optimizing the EC2 footprint (Spot, Auto Scaling, Rightsizing).
- **Amazon S3:** Raw satellite imagery requires immense storage. Lifecycle policies and Intelligent-Tiering are critical for cost-effective retention.
- **AWS Data Transfer:** Routing data out of Ground Station effectively without incurring cross-region penalties or NAT Gateway fees is essential.
---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
Identify `AWSGroundStation` product codes, operation types (Narrowband vs Wideband), and usage types for contact minutes.
### B. CloudWatch Metrics
Monitor EC2 CPU/Memory utilization during contacts and network throughput from Ground Station ENIs.
### C. AWS Config / Trusted Advisor
Audit Mission Profiles and Ephemeris data configurations.
### D. Company Policies
Review satellite priority levels, acceptable data latency, and weather cancellation thresholds.
### E. IaC (Optional)
Terraform or CloudFormation templates defining Mission Profiles, EC2 receiver instances, and automated scheduling Lambda functions.
---
## Output Schema
### Finding Record (JSON)
### Summary Report Table

#### 1. Cancel Unneeded Passes >15 Minutes in Advance
- **What:** Manually or automatically cancel scheduled antenna contacts more than 15 minutes before the start time if the pass is no longer needed (e.g., due to weather or satellite health).
- **Why It Saves Money:** Cancellations made within 15 minutes of the start time incur a 100% penalty charge, costing up to $21.78/minute for Wideband, even if no data is transmitted.
- **Implementation Steps:** 
  1. Monitor satellite health and weather conditions prior to scheduled passes.
  2. Implement an automated script (e.g., Lambda) to evaluate pass necessity at the T-20 minute mark.
  3. Call `CancelContact` API if the pass is deemed unnecessary.
- **Estimated Savings:** 100% of the cost for unused contacts.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Automated API integration for Ground Station scheduling.

#### 2. Terminate Unused Ephemeris Data and Configurations
- **What:** Delete outdated Ephemeris data, unused Mission Profiles, and deprecated satellite configurations in AWS Ground Station.
- **Why It Saves Money:** While Ground Station primarily bills for contact minutes, keeping the control plane clean prevents accidental scheduling of retired satellites, which would incur contact minute charges.
- **Implementation Steps:** 
  1. Audit active satellites vs. those configured in Ground Station.
  2. Delete Ephemeris data for retired assets.
  3. Remove unused Mission Profiles to prevent scheduling errors.
- **Estimated Savings:** 1-5% (avoidance of erroneous scheduling)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Inventory of active satellites.

#### 3. Lifecycle Ground Station Raw Data in S3
- **What:** Implement S3 Lifecycle rules on the buckets receiving raw Wideband satellite data to transition it to cheaper storage tiers (Glacier) after processing.
- **Why It Saves Money:** Satellite imagery is massive. Storing raw Level-0 data in S3 Standard long-term costs $0.023/GB, whereas Glacier Deep Archive is $0.00099/GB (95% cheaper).
- **Implementation Steps:** 
  1. Identify S3 buckets used as Ground Station data delivery destinations.
  2. Create lifecycle rules to transition data to Glacier Deep Archive after 7-14 days (post-processing).
  3. Delete raw data after legal retention periods expire.
- **Estimated Savings:** 70-90% on S3 storage costs for satellite data.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of data processing pipelines.

#### 4. Delete Unused Mission Profiles
- **What:** Remove inactive or duplicate Mission Profiles from Ground Station.
- **Why It Saves Money:** Ensures operators don't accidentally schedule contacts using suboptimal or incorrect profiles, reducing the risk of paying for failed downlinks that must be retried.
- **Implementation Steps:** 
  1. List all `MissionProfiles` via AWS CLI.
  2. Identify profiles not associated with recent successful contacts.
  3. Delete obsolete profiles.
- **Estimated Savings:** <1% (Error prevention)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None

#### 5. Optimize Pass Sizing and Elevation Angles
- **What:** Schedule contacts only for the portion of the orbital pass where the satellite is at an optimal elevation angle to guarantee high signal quality, rather than the entire horizon-to-horizon duration.
- **Why It Saves Money:** Ground Station bills per contact minute. Scheduling the low-elevation "flat horizons" where signal quality is too poor for data transfer wastes minutes at the full $21.78 (Wideband) rate.
- **Implementation Steps:** 
  1. Analyze historical signal-to-noise ratio (SNR) metrics against elevation angles.
  2. Determine the minimum viable elevation angle for data transfer.
  3. Adjust the contact scheduling algorithm to only request time windows within this viable angle.
- **Estimated Savings:** 10-25% of contact minute costs per pass.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Capability to calculate and predict elevation angles.

#### 6. Rightsize Downlink EC2 Instances
- **What:** Downsize EC2 instances used as Software Defined Radios (SDR) or initial data processors during satellite contacts.
- **Why It Saves Money:** Ground Station requires EC2 instances to receive data streams. If these instances are over-provisioned (e.g., using `c5.24xlarge` when a `c5.4xlarge` suffices), you pay for unused compute capacity.
- **Implementation Steps:** 
  1. Review CloudWatch CPU and Memory metrics for receiver EC2 instances during active contacts.
  2. Identify instances peaking below 40% utilization.
  3. Modify the Auto Scaling group or instance type to a smaller size.
- **Estimated Savings:** 30-50% of EC2 compute costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch metrics enabled.

#### 7. Downgrade from Wideband to Narrowband Where Possible
- **What:** Use Narrowband (<=2 MHz) instead of Wideband (>2 MHz) for contacts that only require command, telemetry, and tracking (TT&C) without massive payload downlinks.
- **Why It Saves Money:** Narrowband is $9.90/minute On-Demand, while Wideband is $21.78/minute. Downgrading saves 54% per minute.
- **Implementation Steps:** 
  1. Review the data rate requirements for each scheduled pass.
  2. Update Mission Profiles to use Narrowband for purely health/telemetry passes.
  3. Schedule contacts using the optimized Mission Profiles.
- **Estimated Savings:** 54% per contact minute on eligible passes.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Satellite must support distinct telemetry-only transmission modes.

#### 8. Purchase Reserved Contact Minutes for Predictable Orbits
- **What:** Commit to a 1-Year Ground Station Reserved plan for satellites with predictable, daily sun-synchronous orbits.
- **Why It Saves Money:** A 1-Year Reserved commitment slashes the per-minute contact rate by exactly 50% for both Narrowband ($4.95/min) and Wideband ($10.89/min).
- **Implementation Steps:** 
  1. Analyze 3-6 months of historical Ground Station billing to determine the baseline of consistent monthly contact minutes.
  2. Project future satellite constellation growth or decommission schedules.
  3. Purchase the Reserved commitment via the AWS Console or API.
- **Estimated Savings:** 50% on baseline contact minutes.
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Predictable, long-term satellite operations.

#### 9. Purchase Savings Plans for Downlink EC2 Processing
- **What:** Apply Compute Savings Plans to the EC2 instances required to receive and process Ground Station data streams.
- **Why It Saves Money:** Saves up to 66% compared to On-Demand EC2 pricing for the compute layer of the Ground Station architecture.
- **Implementation Steps:** 
  1. Baseline the EC2 usage strictly associated with Ground Station receivers.
  2. Purchase a Compute Savings Plan covering the steady-state hourly commitment.
- **Estimated Savings:** 20-66% on EC2 costs.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Stable compute architecture.

#### 10. Utilize S3 Intelligent-Tiering for Telemetry Archives
- **What:** Move Ground Station output data buckets to S3 Intelligent-Tiering to automatically optimize storage costs for data with unknown or changing access patterns.
- **Why It Saves Money:** Eliminates the operational overhead of lifecycle rules while automatically moving unaccessed satellite telemetry to Infrequent Access (saving 50%) or Archive tiers.
- **Implementation Steps:** 
  1. Configure S3 buckets receiving GS data to use Intelligent-Tiering by default.
  2. Enable Archive Access tiers in the bucket configuration.
- **Estimated Savings:** 20-50% on S3 storage costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None

#### 11. Route Data Directly to S3 (instead of EC2)
- **What:** Where supported by the payload and processing pipeline, configure Ground Station to deliver data asynchronously directly to Amazon S3 rather than streaming it synchronously to an EC2 instance.
- **Why It Saves Money:** Eliminates the need to provision, run, and pay for always-on or perfectly-timed EC2 instances just to catch the data stream, drastically reducing compute costs.
- **Implementation Steps:** 
  1. Review payload processing requirements.
  2. Update Ground Station Dataflow Endpoint Groups to target S3 buckets.
  3. Trigger downstream serverless processing (Lambda) based on `s3:ObjectCreated` events.
- **Estimated Savings:** 100% of receiver EC2 compute costs.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Architecture must support asynchronous S3 delivery.

#### 12. Process Ground Station Data with Spot Instances
- **What:** Use Amazon EC2 Spot Instances for post-processing satellite data (e.g., image rendering, photogrammetry) once it has been safely downlinked and stored.
- **Why It Saves Money:** Spot instances provide up to 90% discounts compared to On-Demand prices for fault-tolerant, interruptible workloads like batch image processing.
- **Implementation Steps:** 
  1. Decouple the real-time downlink (which needs On-Demand/Reserved EC2) from the post-processing pipeline.
  2. Place post-processing tasks in an SQS queue.
  3. Spin up Spot Instances to consume and process the queue.
- **Estimated Savings:** 70-90% on post-processing compute costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Decoupled, fault-tolerant processing architecture.

#### 13. Automate Pass Cancellations based on Cloud Cover/Weather
- **What:** Integrate weather forecasting APIs with the Ground Station scheduling engine to automatically cancel Earth Observation (EO) optical satellite passes if cloud cover exceeds acceptable thresholds.
- **Why It Saves Money:** Prevents paying $21.78/minute for Wideband contacts that only download pictures of clouds. By canceling >15 minutes in advance, the charge is $0.
- **Implementation Steps:** 
  1. Build a Lambda function that triggers 30 minutes before a scheduled pass.
  2. Query a weather API for cloud cover at the target imaging location.
  3. If cloud cover > threshold, call `CancelContact` API.
- **Estimated Savings:** 10-30% of total contact costs for optical EO satellites.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Access to reliable weather API and automated scheduling.

#### 14. Auto-Scale EC2 Receivers based on Contact Schedules
- **What:** Dynamically scale EC2 receiver instances up before a scheduled Ground Station contact and scale them down immediately after the contact ends.
- **Why It Saves Money:** Avoids paying for 24/7 EC2 instances when satellite contacts may only occur a few times a day for 10-15 minutes at a time.
- **Implementation Steps:** 
  1. Hook into Amazon EventBridge for Ground Station `Contact Status Change` events.
  2. Trigger Step Functions or Lambda to provision/start EC2 instances exactly 5 minutes before the `PREPASS` state.
  3. Terminate/stop instances after the `POSTPASS` state completes.
- **Estimated Savings:** 80-95% of receiver EC2 costs.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Infrastructure as Code and event-driven automation.

#### 15. Use Kinesis Data Streams On-Demand vs Provisioned
- **What:** If routing Ground Station data through Kinesis, switch to Kinesis Data Streams On-Demand mode if satellite passes are infrequent and highly sporadic.
- **Why It Saves Money:** Provisioned mode charges per shard hour 24/7. On-Demand mode scales automatically and only charges for data written/read, which is cheaper for bursty satellite passes separated by hours of silence.
- **Implementation Steps:** 
  1. Review Kinesis shard utilization metrics.
  2. Toggle the capacity mode from Provisioned to On-Demand in the AWS Console.
- **Estimated Savings:** 40-70% on Kinesis costs for bursty workloads.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Data volume must match On-Demand pricing benefits.

#### 16. Consolidate Satellites into Shared Mission Profiles
- **What:** Standardize configurations so multiple satellites in a constellation share a single Mission Profile where possible, rather than maintaining 1:1 mapping.
- **Why It Saves Money:** Reduces the engineering and operational overhead of managing Ground Station infrastructure, indirectly saving engineering hours and reducing the likelihood of scheduling misconfigurations.
- **Implementation Steps:** 
  1. Audit existing Mission Profiles for overlapping parameters.
  2. Create a unified profile for similar satellite buses.
  3. Update scheduling logic to reference the shared profile.
- **Estimated Savings:** Indirect (OpEx savings).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Satellites must have identical RF and processing requirements.

#### 17. Optimize Data Transfer Costs using VPC Endpoints for S3
- **What:** Deploy a VPC Gateway Endpoint for S3 in the VPC where Ground Station EC2 receivers are located.
- **Why It Saves Money:** If EC2 receivers push massive Wideband data to S3 over a NAT Gateway, it incurs a $0.045/GB NAT processing charge. A VPC Gateway Endpoint for S3 is free, eliminating this data transfer cost entirely.
- **Implementation Steps:** 
  1. Go to VPC Console -> Endpoints.
  2. Create a Gateway Endpoint for S3.
  3. Attach it to the route tables of the private subnets hosting the EC2 receivers.
- **Estimated Savings:** $0.045 per GB of satellite data processed.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Private subnets routing S3 traffic via NAT.

#### 18. Avoid Cross-Region Data Transfer from Ground Station
- **What:** Ensure that the AWS Region where the Ground Station antenna is located matches the AWS Region of the VPC and S3 buckets processing the data.
- **Why It Saves Money:** Streaming data from a Ground Station antenna in `eu-central-1` to a processing VPC in `us-east-1` incurs cross-region data transfer fees ($0.02/GB), which adds up quickly for high-throughput Wideband streams.
- **Implementation Steps:** 
  1. Review the Dataflow Endpoint Groups.
  2. Ensure the destination VPC is in the same region as the Ground Station facility.
  3. If cross-region transfer is necessary, process data locally first to compress/filter it before transferring across regions.
- **Estimated Savings:** $0.02 per GB transferred.
- **Risk Level:** Medium
- **Implementation Scope:** Architect/Engineer
- **Prerequisites:** Multi-region architecture review.
