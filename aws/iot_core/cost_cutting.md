# Cost-Cutting Playbook: AWS IoT Core
> **Companion File:** [iot_core.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/iot_core/iot_core.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS IoT Core provides a highly scalable platform for managing device connections and message routing, but its granular, pay-as-you-go pricing model (metered in 5KB/1KB chunks and per-million operations) can lead to significant bill shock if not optimized. The primary drivers of excessive cost are high-frequency, un-batched telemetry messaging, unnecessary Device Shadow updates, and inefficient use of the standard pub/sub broker for one-way data ingestion. This playbook outlines 19 actionable strategies to optimize AWS IoT Core spending, leveraging architectural shifts like Basic Ingest, edge computing, and payload rightsizing to achieve up to 98% savings on high-volume fleets.

## Strategy Categories

### 1. Waste Elimination

#### 1. Eliminate Orphaned or Zombie Device Connections
- **What:** Identify and disconnect retired, malfunctioning, or orphaned devices that maintain an active connection to the IoT Gateway but do not transmit useful telemetry.
- **Why It Saves Money:** Connectivity is billed at $0.08 per 1 Million connection minutes. Devices stuck in rapid reconnect loops or maintaining dead connections accrue ongoing time-based fees without delivering value.
- **Implementation Steps:**
  1. Monitor CloudWatch metrics `Connect.Success` and `Connect.ClientError`.
  2. Implement AWS IoT Device Defender to detect anomalous connection patterns (e.g., constant reconnecting).
  3. Revoke certificates or disable policies for identified zombie devices.
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS IoT Device Defender, CloudWatch metrics.

#### 2. Clean Up Unused IoT Rules
- **What:** Audit and remove active IoT Rules Engine rules that trigger on topics but have no downstream actions, or pipe data to deprecated backend services.
- **Why It Saves Money:** The Rules Engine bills $0.15 per 1 Million rules triggered, even if the subsequent action fails or the data is no longer utilized by the business.
- **Implementation Steps:**
  1. Review all IoT rules in the AWS Console.
  2. Check CloudWatch metrics `RuleExecution` and `ActionExecution` for each rule.
  3. Disable or delete rules that have 0 successful actions or are routing to unused resources.
- **Estimated Savings:** 2-10%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch logs for IoT Rules.

#### 3. Reduce MQTT Keep-Alive Traffic
- **What:** Increase the MQTT keep-alive interval for devices that do not require near-instantaneous disconnect detection.
- **Why It Saves Money:** Frequent keep-alive (ping/pong) messages consume bandwidth and device battery, and while PINGREQ/PINGRESP packets are not billed as standard messages, minimizing them reduces connection overhead and potential reconnects that trigger Rules/Lifecycle events.
- **Implementation Steps:**
  1. Audit device firmware MQTT client configurations.
  2. Increase the keep-alive interval from default (e.g., 30s) to 10-20 minutes, depending on the application's tolerance for delayed offline detection.
- **Estimated Savings:** < 1%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Firmware update capabilities (OTA).

#### 4. Disable Excessive IoT Core CloudWatch Logging
- **What:** Reduce the default logging level of AWS IoT Core from `DEBUG` to `INFO` or `ERROR` for production workloads.
- **Why It Saves Money:** High-volume IoT workloads generate massive amounts of log data. CloudWatch Logs charges $0.50 per GB ingested. Detailed `DEBUG` logging for millions of devices can cost thousands of dollars.
- **Implementation Steps:**
  1. Go to AWS IoT Core Settings.
  2. Under "Logs", change the default logging role level to `ERROR` or `WARN`.
  3. Create specific, resource-targeted logging overrides for debugging individual devices only when necessary.
- **Estimated Savings:** 5-15% (on CloudWatch bill)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

### 2. Rightsizing

#### 5. Batch Telemetry Messages on Edge Devices
- **What:** Buffer telemetry data locally on the edge device/micro-controller and publish combined payloads every 60 seconds instead of sending 1-second pings.
- **Why It Saves Money:** Messages are metered in 5 KB increments. A 50-byte message incurs the same $1.00/1M fee as a 4.9 KB message. Consolidating 60 x 50-byte pings into one 3 KB payload reduces message volume by 98.3%.
- **Implementation Steps:**
  1. Update device firmware to store telemetry points in local memory.
  2. Construct an array of telemetry data points.
  3. Publish the batched array every 60 seconds or when the payload approaches 5 KB.
- **Estimated Savings:** 80-98%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Device edge memory capacity, OTA update capability.

#### 6. Minimize Device Shadow Updates
- **What:** Update Device Shadows only when a state change actually occurs, rather than performing periodic time-based updates.
- **Why It Saves Money:** Device Shadow operations are billed at $1.25 per 1 Million operations. Periodic updates for unchanged state incur massive fees (e.g., updating every minute = $54/device/month).
- **Implementation Steps:**
  1. Modify device logic to cache current desired/reported state.
  2. Only publish an update to `$aws/things/thingName/shadow/update` when the local state diverges from the cached cloud state.
- **Estimated Savings:** 40-90% (on Shadow costs)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Firmware update capability.

#### 7. Optimize Device Shadow Payload Size
- **What:** Break up large monolithic Device Shadows into multiple Named Shadows, and only update the specific shadow that has changed.
- **Why It Saves Money:** Device Shadows are metered in 1 KB chunks. If you have a 4 KB shadow and update a 10-byte field, you are billed for 4 chunks (4 x $1.25/1M ops). Using a 500-byte Named Shadow for frequently changing fields reduces this to 1 chunk.
- **Implementation Steps:**
  1. Analyze Device Shadow JSON documents.
  2. Separate static/slow-changing metadata from fast-changing telemetry/state.
  3. Implement Named Shadows (e.g., `thingName/shadow/name/fast_changing`).
- **Estimated Savings:** 50-75% (on Shadow costs)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application logic refactoring.

#### 8. Implement Payload Compression (CBOR/Protobuf)
- **What:** Replace verbose JSON payloads with dense binary serialization formats like Protocol Buffers (Protobuf) or CBOR.
- **Why It Saves Money:** Messages are metered in 5 KB chunks. Compressing data allows you to pack more batched telemetry into a single 5 KB chunk, effectively reducing the total number of metered messages.
- **Implementation Steps:**
  1. Integrate a CBOR/Protobuf library in device firmware.
  2. Serialize telemetry locally.
  3. Ensure backend IoT Rules are configured with AWS Lambda or a decoder action to deserialize the binary payload before downstream processing.
- **Estimated Savings:** 20-50%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Firmware update, backend decoder implementation.

### 3. Commitment Discounts

#### 9. Consolidate Billing for Volume Tiers
- **What:** Use AWS Organizations Consolidated Billing to aggregate all IoT Core usage across multiple accounts (e.g., Dev, Staging, multiple Production accounts).
- **Why It Saves Money:** IoT Core messaging provides volume discounts (Tier 1: $1.00/1M, Tier 2: $0.80/1M, Tier 3: $0.70/1M). Aggregating usage pushes the organization into the cheaper tiers faster.
- **Implementation Steps:**
  1. Ensure all AWS accounts utilizing IoT Core are linked under a single AWS Organization payer account.
  2. Verify that billing features are enabled.
- **Estimated Savings:** 10-25%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** AWS Organizations.

#### 10. Negotiate Private Pricing Addendum (PPA)
- **What:** Negotiate a custom discount directly with AWS for massive-scale IoT deployments (typically >$10k/month in IoT spend).
- **Why It Saves Money:** AWS will often provide custom unit price discounts for IoT Core in exchange for a committed annual spend across the AWS account.
- **Implementation Steps:**
  1. Forecast 12-36 month IoT Core usage growth.
  2. Contact the AWS Account Manager.
  3. Sign a Private Pricing Addendum for IoT Core components.
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** High existing or committed AWS spend.

### 4. Architecture Changes

#### 11. Leverage Basic Ingest for One-Way Telemetry
- **What:** Modify devices to publish telemetry directly to `$aws/rules/<rule-name>/<topic>` instead of a standard pub/sub topic.
- **Why It Saves Money:** Basic Ingest bypasses the pub/sub message broker, eliminating the $1.00/1M messaging fee entirely ($0.00 cost). You only pay the Rules Engine fee ($0.15/1M triggered + $0.15/1M actions). This is a massive ~70% cost reduction.
- **Implementation Steps:**
  1. Identify devices that only send data to the cloud (no other devices subscribe to their topics).
  2. Change the MQTT publish topic on the device firmware to use the Basic Ingest format.
  3. Ensure the corresponding IoT Rule exists to handle the ingestion.
- **Estimated Savings:** 70% (on Messaging)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** OTA update capability, telemetry is strictly device-to-cloud.

#### 12. Filter Telemetry at the Edge (AWS IoT Greengrass)
- **What:** Deploy AWS IoT Greengrass on edge gateway devices to process, filter, and aggregate data locally before sending it to the cloud.
- **Why It Saves Money:** Instead of sending raw 100Hz vibration data to IoT Core, Greengrass can run a local ML model or threshold rule to only transmit anomalies to the cloud. This drastically cuts message volume and associated IoT Core messaging/rules costs.
- **Implementation Steps:**
  1. Provision edge hardware capable of running Greengrass.
  2. Deploy Lambda functions or Greengrass components to the edge.
  3. Route high-frequency sensors to Greengrass locally instead of directly to the cloud.
- **Estimated Savings:** 50-95%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Capable edge hardware (gateways/IPCs).

#### 13. Optimize Rules Engine SQL Filtering
- **What:** Write stricter SQL `WHERE` clauses in IoT Rules to filter out messages *before* an action is executed.
- **Why It Saves Money:** Rules Engine triggers cost $0.15/1M, but executing an action costs an additional $0.15/1M. Filtering heavily in the SQL statement prevents downstream actions (and costs from services like Lambda, Kinesis, DynamoDB) from triggering unnecessarily.
- **Implementation Steps:**
  1. Review all IoT Core Rules (`SELECT * FROM 'topic'`).
  2. Add strict `WHERE` conditions (e.g., `WHERE temperature > 80`).
- **Estimated Savings:** 10-40% (on Actions and downstream services)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Knowledge of payload schema.

#### 14. Replace Direct Messaging with Device Shadows for Rare Config Updates
- **What:** Use Device Shadows instead of pub/sub MQTT messages for infrequent configuration changes.
- **Why It Saves Money:** If a device frequently wakes up and subscribes to a topic to check for config updates, it generates messaging costs. Shadows provide a persistent, easily queryable state that devices can check upon boot/wake, reducing the need for constant topic polling.
- **Implementation Steps:**
  1. Migrate configuration data to the `desired` block of a Device Shadow.
  2. Update firmware to retrieve the shadow on boot rather than subscribing to a config topic permanently.
- **Estimated Savings:** 5-15%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Refactoring cloud backend and device firmware.

### 5. Scheduling & Auto-Scaling

#### 15. Dynamically Adjust Device Reporting Intervals
- **What:** Make device reporting frequency state-aware (e.g., report every 10 seconds when a motor is running, but every 1 hour when it is off).
- **Why It Saves Money:** Sending static, unchanged data at a high frequency wastes 5 KB messaging chunks and Rules Engine actions. Slowing down idle reporting directly cuts message volume.
- **Implementation Steps:**
  1. Add state-detection logic to device firmware.
  2. Implement variable sleep/publish timers based on state.
- **Estimated Savings:** 30-60%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Firmware update capabilities.

#### 16. Disconnect Non-Critical Devices During Idle Periods
- **What:** For devices with predictable active hours (e.g., office equipment, daytime industrial sensors), disconnect the MQTT connection entirely during off-hours.
- **Why It Saves Money:** Saves the $0.08 per 1M minutes connectivity fee. 12 hours disconnected per day halves the connectivity cost.
- **Implementation Steps:**
  1. Program devices with local real-time clocks or scheduled sleep modes.
  2. Have devices gracefully disconnect (MQTT DISCONNECT) at the end of the shift and sleep until morning.
- **Estimated Savings:** 30-50% (on Connectivity)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Use cases that tolerate offline periods.

### 6. Pricing Model Optimization

#### 17. Region Arbitrage for IoT Core Deployments
- **What:** Deploy IoT Core endpoints in lower-cost AWS regions if strict latency requirements are not necessary.
- **Why It Saves Money:** AWS IoT Core pricing varies slightly by region. For example, some regions might charge slightly more for messaging or connectivity compared to `us-east-1` or `us-east-2`.
- **Implementation Steps:**
  1. Compare IoT Core pricing across target operating regions.
  2. Provision the IoT Core registry and endpoints in the cheapest acceptable region.
  3. Ensure cross-region data transfer costs don't negate the savings if backend services are in a different region.
- **Estimated Savings:** 5-10%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Latency tolerance, data residency compliance.

### 7. Network & Data Transfer Optimization

#### 18. Optimize VPC Endpoint Data Transfer
- **What:** Use VPC endpoints appropriately, but beware of cross-AZ data transfer costs when routing massive volumes of IoT Rules Engine actions into a VPC (e.g., to RDS or Kafka).
- **Why It Saves Money:** Routing rules engine traffic to a VPC can incur $0.01/GB data transfer fees across Availability Zones. Ensuring VPC endpoints are deployed regionally or optimizing the architecture can reduce these hidden networking costs.
- **Implementation Steps:**
  1. Review architecture mapping IoT Core to VPC resources.
  2. Ensure resources are deployed efficiently across AZs to minimize cross-AZ NAT/endpoint fees.
- **Estimated Savings:** 1-5% (on Networking bill)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC architecture knowledge.

#### 19. Minimize Wildcard Topic Subscriptions
- **What:** Restrict devices from subscribing to broad wildcard topics (e.g., `company/+/metrics/#`).
- **Why It Saves Money:** Subscribing to a wildcard means the broker will fan-out messages to that device unnecessarily. Standard message delivery to a subscribed client incurs standard messaging fees. Over-fanning messages causes exponential cost growth.
- **Implementation Steps:**
  1. Audit device subscriptions.
  2. Implement strict IoT Policies that deny wildcard subscriptions (`iot:Subscribe`).
  3. Force devices to subscribe only to their specific `thingName` topics.
- **Estimated Savings:** 10-40%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** IoT Policy updates.

---

## Cross-Service Synergies
- **AWS Lambda:** Triggering Lambda from IoT Rules can get expensive. Consider using native IoT Rules integrations (e.g., direct to DynamoDB, S3, or Kinesis) to bypass Lambda execution costs entirely.
- **Amazon S3 / Kinesis Data Firehose:** Use IoT Rules to batch data into Firehose -> S3 for cheap long-term storage and analytics, bypassing expensive hot databases for raw telemetry.
- **AWS IoT Greengrass:** Pushes computation to the edge, drastically reducing the data ingested into IoT Core and consequently saving downstream processing and storage costs across the AWS ecosystem.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
Required to analyze specific usage components (Connectivity, Messaging, Rules, Shadow) and identify the specific AWS accounts/regions driving costs. Look for `ProductCode: AWSIoT`.
### B. CloudWatch Metrics
Essential for observing device behavior: `Connect.Success`, `PublishIn.Success`, `RuleExecution`, `ActionExecution`.
### C. AWS Config / Trusted Advisor
Used to audit active IoT Rules, Device Shadows, and attached IoT Policies for overly permissive wildcard usage.
### D. Company Policies
Data retention policies dictate how long telemetry must be kept and how quickly it needs to be available, driving decisions around Edge filtering and batching.
### E. IaC (Optional)
Terraform or CloudFormation scripts to identify how IoT Rules and integrations are provisioned and to bulk-apply optimizations like logging reductions.

---

## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "IOTC-001",
  "resource_id": "arn:aws:iot:us-east-1:123456789012:rule/ProcessTelemetry",
  "strategy_id": "IOTC-BASIC-INGEST",
  "potential_savings_monthly": 4500.00,
  "confidence_score": 0.95
}
```

### Summary Report Table
| Finding ID | Strategy | Resource | Est. Savings | Risk |
|------------|----------|----------|--------------|------|
| IOTC-001 | Leverage Basic Ingest | Fleet-A-Telemetry | $4,500.00 | Low |
| IOTC-002 | Batch Edge Messages | Fleet-B-Sensors | $18,200.00 | Med |
| IOTC-003 | Delete Unused Rules | Rule-LegacyApp | $320.00 | Low |
| IOTC-004 | Optimize Shadow Size | ThingGroup-HVAC | $1,150.00 | Med |
