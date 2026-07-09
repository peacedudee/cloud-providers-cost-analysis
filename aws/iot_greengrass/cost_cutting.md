# Cost-Cutting Playbook: AWS IoT Greengrass
> **Companion File:** [iot_greengrass.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/iot_greengrass/iot_greengrass.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS IoT Greengrass brings local processing, messaging, and ML inference to edge devices. While the service itself is exceptionally cost-effective at $0.16 per active core device per month, the true financial impact lies in how it interacts with the broader AWS ecosystem (especially AWS IoT Core, CloudWatch, and Kinesis). The primary objective of Greengrass cost optimization is using edge compute to aggressively filter, aggregate, and compress data locally, thereby drastically reducing cloud data ingestion, messaging, and storage costs, while ensuring decommissioned devices are cleanly removed from billing.

## Strategy Categories

### 1. Waste Elimination

#### 1. Audit and Decommission Retired Core Devices
- **What:** Identify and cleanly uninstall physical IoT gateways that are retired but still occasionally connecting to AWS.
- **Why It Saves Money:** Any device that authenticates even once during a calendar month incurs the full $0.16 charge. Phantom devices artificially inflate the bill.
- **Implementation Steps:** 
  1. Export a list of all active Greengrass Core devices from the AWS Console.
  2. Cross-reference with the physical hardware inventory to identify retired devices.
  3. Stop the Greengrass nucleus service on retired devices and wipe configurations.
  4. Delete the core device definition from the AWS IoT Greengrass console.
- **Estimated Savings:** 1-5% of Greengrass base costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Device inventory tracking, IAM access to IoT Core/Greengrass.

#### 2. Clean Up Orphaned Greengrass V1 Resources
- **What:** Remove lingering resources from deprecated Greengrass V1 deployments.
- **Why It Saves Money:** Greengrass V1 was discontinued in June 2026. Orphaned resources (Groups, definitions, associated Lambda aliases) cause clutter and potential residual charges if associated with other active services (e.g., lingering Lambda provisions).
- **Implementation Steps:**
  1. Scan AWS account for legacy V1 Groups and deployments.
  2. Verify all active devices have been successfully migrated to V2.
  3. Delete obsolete V1 resources via CLI or console.
- **Estimated Savings:** <1%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Migration to Greengrass V2 completed.

#### 3. Disable Unnecessary Edge Component Logging
- **What:** Reduce the verbosity of Greengrass component logs (e.g., changing from DEBUG to INFO/WARN) sent to Amazon CloudWatch.
- **Why It Saves Money:** CloudWatch Logs ingestion costs $0.50/GB. Verbose logging from thousands of edge devices can quickly become expensive.
- **Implementation Steps:**
  1. Review logging configuration in the Greengrass nucleus component and custom components.
  2. Set log level to `INFO` or `WARN` for production deployments.
  3. Configure log rotation and local storage limits to prevent excessive cloud uploads.
- **Estimated Savings:** 10-30% of CloudWatch Logs costs for IoT
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Centralized log management strategy.

### 2. Rightsizing

#### 4. Hardware Rightsizing for Edge Gateways
- **What:** Profile CPU and memory usage of Greengrass components to select more cost-effective edge hardware (e.g., cheaper ARM-based industrial PCs).
- **Why It Saves Money:** While not an AWS cloud cost, over-provisioning edge hardware across thousands of physical sites carries massive CapEx waste.
- **Implementation Steps:**
  1. Deploy Greengrass telemetry components to monitor local CPU/RAM utilization.
  2. Analyze peak resource usage over a 30-day period.
  3. Adjust procurement specs for future edge gateway purchases.
- **Estimated Savings:** 10-20% CapEx reduction
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps | Procurement/Leadership
- **Prerequisites:** Edge telemetry visibility.

#### 5. Rightsize Component Memory Limits
- **What:** Enforce strict memory limits on local Docker containers and Lambda functions running on Greengrass.
- **Why It Saves Money:** Prevents memory leaks in custom components from crashing the core device or requiring larger edge hardware, stabilizing operations and reducing maintenance costs (fewer truck rolls).
- **Implementation Steps:**
  1. Review historical memory consumption of edge components.
  2. Configure memory limits in the component recipe (`MemorySize`).
- **Estimated Savings:** Indirect (reduces truck rolls/hardware costs)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Knowledge of component performance profiles.

### 3. Commitment Discounts

#### 6. Negotiate Private Pricing for Large Fleets
- **What:** Secure custom volume discounts for Greengrass Core devices if operating fleets exceeding 10,000 devices.
- **Why It Saves Money:** AWS offers private pricing agreements (PPAs) that discount the $0.16 per device/month standard rate for massive enterprise deployments.
- **Implementation Steps:**
  1. Forecast expected active device count over the next 1-3 years.
  2. If the fleet exceeds 10,000 active devices, contact your AWS Account Manager.
  3. Negotiate a discounted per-device rate as part of an EDP or standalone PPA.
- **Estimated Savings:** 10-25% on base device charges
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** >10,000 active Greengrass Core devices.

### 4. Architecture Changes

#### 7. Filter and Aggregate Sensor Data Locally
- **What:** Deploy local edge components (Lambda/Python) to process raw high-frequency sensor readings locally instead of using Greengrass as a "dumb pipe".
- **Why It Saves Money:** Cloud messaging to IoT Core costs $1.00/million messages. Streaming 100ms sensor data directly to the cloud generates astronomical bills. Sending 1-minute summaries reduces message volume massively.
- **Implementation Steps:**
  1. Identify high-frequency data streams being sent to IoT Core.
  2. Develop a local component to compute averages, max/min, or moving averages.
  3. Publish only the aggregated summary payload to the cloud topic.
- **Estimated Savings:** 90-99% on AWS IoT Core messaging costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Edge devices have sufficient CPU for data processing.

#### 8. Leverage Free Local Inter-Device Communication
- **What:** Use the Greengrass local MQTT broker (Moquette) for communication between client devices on the same local network.
- **Why It Saves Money:** Local messaging between edge devices connected to a Greengrass core is 100% free, whereas bouncing those messages through the AWS cloud incurs IoT Core messaging rates.
- **Implementation Steps:**
  1. Deploy the `aws.greengrass.clientdevices.mqtt.Moquette` component.
  2. Configure client devices to connect to the local Core IP instead of the AWS IoT Core endpoint.
  3. Update authorization policies to permit local pub/sub.
- **Estimated Savings:** 100% of local M2M messaging costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Client devices on the same LAN as the Greengrass Core.

#### 9. Implement Edge Anomaly Detection (ML)
- **What:** Run machine learning models locally on Greengrass to detect anomalies and only send data to the cloud when an anomaly occurs.
- **Why It Saves Money:** Eliminates the need to send "normal" steady-state data to the cloud, slashing IoT Core, Kinesis, and cloud storage costs, while local ML execution is free.
- **Implementation Steps:**
  1. Train an anomaly detection model using SageMaker.
  2. Compile the model for edge hardware using SageMaker Neo.
  3. Deploy the model to Greengrass using edge ML components.
  4. Modify data pipelines to conditionally publish to the cloud only on anomaly triggers.
- **Estimated Savings:** 80-95% on cloud ingestion and storage
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Edge hardware with sufficient ML inference capacity (CPU/GPU/NPU).

#### 10. Use Stream Manager for Batched Cloud Uploads
- **What:** Utilize the Greengrass Stream Manager component to cache data locally and export it to AWS in optimized batches (e.g., to Kinesis or S3).
- **Why It Saves Money:** Reduces the number of API calls, Kinesis PUT requests, or S3 object creations. Batching is significantly cheaper than streaming individual records.
- **Implementation Steps:**
  1. Deploy the `aws.greengrass.StreamManager` component.
  2. Configure a stream to collect local data.
  3. Set up an export configuration to Kinesis Data Streams or S3 with high batch sizes.
- **Estimated Savings:** 40-70% on cloud ingestion API costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application tolerance for batched/delayed data delivery.

#### 11. Process Video Streams at the Edge
- **What:** Analyze RTSP camera feeds locally using Greengrass and computer vision models, rather than streaming raw video to Kinesis Video Streams.
- **Why It Saves Money:** Streaming 24/7 HD video to the cloud consumes massive bandwidth and incurs high KVS ingestion costs. Local processing is free; only metadata (e.g., bounding boxes, alerts) is sent to the cloud.
- **Implementation Steps:**
  1. Deploy local computer vision models on Greengrass.
  2. Connect IP cameras to the local Greengrass core.
  3. Extract metadata and publish to IoT Core; drop the raw video frames.
- **Estimated Savings:** 90-99% on KVS and data transfer costs
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Edge devices capable of video decoding and ML inference.

### 5. Scheduling & Auto-Scaling

#### 12. Smart Data Synchronization based on Connectivity
- **What:** Implement edge logic to cache data when the core device is on an expensive cellular connection (LTE/5G) and sync only when connected to cheap/free Wi-Fi or Ethernet.
- **Why It Saves Money:** Cellular data plans for IoT fleets can be prohibitively expensive. Deferring non-critical telemetry uploads saves on ISP/cellular bills.
- **Implementation Steps:**
  1. Implement a component that monitors local network interface status.
  2. Use Stream Manager to cache telemetry data indefinitely.
  3. Enable cloud export only when the preferred (Wi-Fi) interface is active.
- **Estimated Savings:** 50-80% on cellular data costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Dual-network capable edge hardware.

### 6. Pricing Model Optimization

#### 13. Maximize AWS IoT Greengrass Free Tier (Non-Prod)
- **What:** Ensure that development and testing environments leverage the AWS Free Tier, which covers the first 3 active core devices per month for 12 months.
- **Why It Saves Money:** Avoids paying the $0.16/device fee for simple QA or sandbox tests.
- **Implementation Steps:**
  1. Isolate dev/test deployments into separate AWS accounts.
  2. Ensure each dev account runs no more than 3 active core devices.
- **Estimated Savings:** $0.48 / month per dev account
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Multi-account strategy (AWS Organizations).

#### 14. Eliminate "Always-On" Cloud Connections for Offline Scenarios
- **What:** If a Greengrass device is designed to run entirely offline (e.g., purely local automation), disable its cloud connection features to prevent it from authenticating with AWS.
- **Why It Saves Money:** A core device is only billed $0.16 if it connects to the cloud in a given month. Purely offline devices incur $0.00 in Greengrass fees.
- **Implementation Steps:**
  1. Identify devices that only perform local tasks (no cloud sync required).
  2. Configure the Greengrass nucleus to operate in offline mode or block outbound AWS endpoints at the firewall level.
- **Estimated Savings:** 100% of Greengrass base costs for those specific devices
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload strictly requires no cloud monitoring or OTA updates.

### 7. Network & Data Transfer Optimization

#### 15. Compress Payloads Before Cloud Synchronization
- **What:** Use standard compression (GZIP, Snappy) on data payloads before publishing them to AWS IoT Core or uploading to S3.
- **Why It Saves Money:** IoT Core meters messages in 5 KB increments. Compressing a 12 KB payload down to 4 KB reduces the billed message count from 3 to 1.
- **Implementation Steps:**
  1. Implement a local compression component in Python or Node.js.
  2. Route all edge data through this component before publishing to the cloud.
  3. Deploy a cloud-side Lambda function to decompress payloads upon arrival.
- **Estimated Savings:** 50-70% on IoT Core messaging costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compression algorithms supported by edge CPU.

#### 16. Optimize MQTT Topic Structures for Shadow Updates
- **What:** Use compact MQTT topic names and minimal JSON structures for device shadow updates.
- **Why It Saves Money:** AWS IoT Core billing is based on total payload size (including topic name overhead). Extremely long topic names and verbose JSON keys consume unnecessary bandwidth and message increments.
- **Implementation Steps:**
  1. Review Device Shadow JSON schemas.
  2. Minify JSON keys (e.g., use `"t"` instead of `"temperature"`).
  3. Ensure shadow updates only send changed fields (deltas) rather than the entire state document.
- **Estimated Savings:** 10-30% on IoT Core data ingestion
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Client application modifications to handle minified schemas.

---

## Cross-Service Synergies
- **IoT Core:** Greengrass is the ultimate cost optimization tool for IoT Core. By processing data locally, Greengrass acts as a massive cost-shield, preventing high-frequency edge data from incurring IoT Core's $1.00/M message fees.
- **Amazon S3 / Kinesis:** Using Greengrass Stream Manager to batch and compress edge data before sending to S3 or Kinesis significantly reduces API `PUT` request costs and data transfer charges.
- **Amazon SageMaker:** Training models in the cloud and deploying them via SageMaker Neo to Greengrass shifts expensive cloud inference costs to free local edge compute.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query for `lineItem/ProductCode` = `AWSIoTGreengrass`.
- Look for `lineItem/Operation` indicating `CoreDeviceActive`.
- Correlate with IoT Core (`AWSIoT`) messaging volume in the same region.

### B. CloudWatch Metrics
- Monitor `MessagePublished` metrics on IoT Core to quantify the volume of data arriving from Greengrass gateways.
- Review Greengrass component CPU and Memory utilization metrics (if exported).

### C. AWS Config / Trusted Advisor
- Check for Greengrass Core devices that have not checked in over 30 days but are still defined in the registry (potential candidates for decommissioning).

### D. Company Policies
- Determine the minimum required data granularity for business operations (e.g., "do we really need 100ms vibration data in the cloud, or is a 5-minute summary sufficient?").

### E. IaC (Optional)
- Review Greengrass V2 deployment manifests/recipes to audit component configurations and Stream Manager batching settings.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "IOTGG-001",
  "service": "AWS IoT Greengrass",
  "strategy_category": "Architecture Changes",
  "finding_description": "Raw sensor data is streamed directly to IoT Core without local edge aggregation.",
  "risk_level": "Medium",
  "estimated_savings_percentage": 95,
  "effort_level": "Medium",
  "actionable_recommendation": "Deploy a local Greengrass component to compute 1-minute aggregates of sensor data and publish only summaries to AWS IoT Core."
}
```

### Summary Report Table

| Finding ID | Strategy | Impact | Effort | Owner |
|------------|----------|--------|--------|-------|
| IOTGG-001 | Audit and Decommission Retired Core Devices | Low | Low | DevOps |
| IOTGG-002 | Clean Up Orphaned Greengrass V1 Resources | Low | Low | DevOps |
| IOTGG-003 | Disable Unnecessary Edge Component Logging | Low | Low | DevOps |
| IOTGG-004 | Hardware Rightsizing for Edge Gateways | High | Medium | Procurement |
| IOTGG-005 | Rightsize Component Memory Limits | Low | Low | DevOps |
| IOTGG-006 | Negotiate Private Pricing for Large Fleets | High | High | FinOps |
| IOTGG-007 | Filter and Aggregate Sensor Data Locally | High | Medium | DevOps |
| IOTGG-008 | Leverage Free Local Inter-Device Communication | High | Low | DevOps |
| IOTGG-009 | Implement Edge Anomaly Detection (ML) | High | High | DevOps |
| IOTGG-010 | Use Stream Manager for Batched Cloud Uploads | Medium | Medium | DevOps |
| IOTGG-011 | Process Video Streams at the Edge | High | High | DevOps |
| IOTGG-012 | Smart Data Synchronization based on Connectivity | Medium | Medium | DevOps |
| IOTGG-013 | Maximize AWS IoT Greengrass Free Tier (Non-Prod) | Low | Low | FinOps |
| IOTGG-014 | Eliminate "Always-On" Cloud Connections for Offline Scenarios | Medium | Medium | DevOps |
| IOTGG-015 | Compress Payloads Before Cloud Synchronization | Medium | Low | DevOps |
| IOTGG-016 | Optimize MQTT Topic Structures for Shadow Updates | Low | Low | DevOps |
