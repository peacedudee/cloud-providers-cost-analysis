# AWS Service Cost Research: AWS IoT Core

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS IoT Core is a managed cloud platform that lets connected devices (smart home appliances, industrial sensors, automotive telemetry systems, medical devices) easily and securely interact with cloud applications and other devices. It supports billions of devices and trillions of messages, managing device connections via MQTT, HTTPS, and WebSockets. AWS IoT Core features an MQTT Message Broker, Device Gateway, Rules Engine, Device Shadow, and Device Registry. AWS IoT Core is billed on a granular per-component pay-as-you-go basis.

---

## 2. Billing Mechanics
AWS IoT Core billing is calculated across four independent usage components:
1. **Connectivity:** Billed per minute of active device connection to the Device Gateway ($0.08 per 1 Million connection minutes = ~$0.042 per device per year connected 24/7).
2. **Messaging:** Billed per 1 Million messages transmitted, with volume discount tiers starting above 1 Billion messages/month. Messages are metered in **5 KB increments** up to 128 KB.
   * *Basic Ingest:* Publish to `$aws/rules/<rule-name>/<topic>` to bypass the publish-subscribe broker for **$0.00 messaging cost** (only Rules Engine fees apply).
   * *Direct Messaging:* Bypasses pub/sub overhead to send point-to-point server-to-device messages. Charged at standard messaging rates.
3. **Rules Engine:** Billed per 1 Million rules triggered ($0.15 per 1 Million rules) and actions applied ($0.15 per 1 Million actions).
4. **Device Shadow & Registry:** Billed per 1 Million shadow or registry operations ($1.25 per 1 Million operations). Operations are metered in **1 KB increments**.

---

## 3. Key Cost Dimensions

### A. Core Component Pricing Rates (us-east-1)
* **Connectivity Rate:** **$0.08 per 1 Million connection minutes**.
* **Messaging Rates (Volume-Tiered):**
  * **First 1 Billion messages/month:** **$1.00 per 1 Million messages**.
  * **Next 4 Billion messages/month (1B to 5B):** **$0.80 per 1 Million messages**.
  * **Over 5 Billion messages/month:** **$0.70 per 1 Million messages**.
* **Rules Engine Rate:** **$0.15 per 1 Million rules triggered** and **$0.15 per 1 Million actions executed**.
* **Device Shadow Rate:** **$1.25 per 1 Million shadow operations** (in 1 KB chunks).

### B. High-Volume Messaging Hazard
If 10,000 devices send a 100-byte telemetry message every second (un-batched):
$$10,000 \text{ devices} \times 86,400 \text{ sec/day} \times 30 \text{ days} = 25.92 \text{ Billion messages/month}$$

Under the volume-tiered pricing model, the monthly cost is calculated as follows:
* Tier 1 (First 1 Billion): $1,000 \text{ Million} \times \$1.00 / \text{Million} = \$1,000.00$
* Tier 2 (Next 4 Billion): $4,000 \text{ Million} \times \$0.80 / \text{Million} = \$3,200.00$
* Tier 3 (Remaining 20.92 Billion): $20,920 \text{ Million} \times \$0.70 / \text{Million} = \$14,644.00$
* **Total Messaging Cost:** $\mathbf{\$1,000.00 + \$3,200.00 + \$14,644.00 = \$18,844.00 \text{ / month}}$

---

## 4. Detailed Pricing Rates (us-east-1)

| IoT Core Component | Metering Increment | Rate (us-east-1) | Price for 10 Million Units |
|--------------------|--------------------|------------------|----------------------------|
| **Connectivity** | Per connection-min | **$0.08 / 1M mins** | **$0.80** |
| **Messaging (Tier 1)** | 5 KB chunks | **$1.00 / 1M msgs** | **$10.00** |
| **Messaging (Tier 2)** | 5 KB chunks | **$0.80 / 1M msgs** | **$8.00** |
| **Messaging (Tier 3)** | 5 KB chunks | **$0.70 / 1M msgs** | **$7.00** |
| **Basic Ingest** | 5 KB chunks | **$0.00 / 1M msgs** | **$0.00** |
| **Rules Engine (Triggers)** | Per rule triggered | **$0.15 / 1M rules**| **$1.50** |
| **Rules Engine (Actions)** | Per action executed | **$0.15 / 1M actions**| **$1.50** |
| **Device Shadow & Registry** | 1 KB chunks | **$1.25 / 1M ops** | **$12.50** |

---

## 5. AWS Free Tier Coverage
* **AWS IoT Core Free Tier:** Includes the following allowances per month for the first 12 months for new accounts:
  * **2,250,000 connection minutes**
  * **500,000 messages**
  * **225,000 Registry or Device Shadow operations**
  * **250,000 rules triggered** and **250,000 actions applied**

---

## 6. Common Cost Hotspots & Pitfalls
* **1-Second Telemetry Pings (Un-Batched Messaging):**
  * Configuring edge devices to send tiny 50-byte MQTT messages every 1 second instead of batching.
  * *Why:* Because IoT Core meters in 5 KB chunks, a 50-byte message incurs the full 5 KB message fee, leading to massive messaging bills.
* **Unnecessary Device Shadow Updates:**
  * Performing periodic, time-based updates to Device Shadows even when there is no state change, incurring flat $1.25/M operations fees.

---

## 7. Actionable Cost Optimization Strategies
1. **Batch Telemetry Messages on Edge Devices:**
   * Buffer telemetry data locally on the micro-controller or gateway device and publish combined 5 KB metric payloads every 60 seconds instead of sending 1-second pings.
   * **The Savings:** Reduces message volume by 98.3%, dropping monthly messaging costs from **$18,844.00 down to $320.00/month (98.3% reduction)**.
2. **Leverage Basic Ingest for One-Way Telemetry:**
   * Publish data directly to `$aws/rules/<rule-name>/<topic>` to bypass the message broker when messages do not need to be fanned out to other MQTT clients.
   * **The Savings:** Eliminates the per-message fee ($1.00/M), leaving only Rules Engine fees ($0.15/M rules + $0.15/M actions), representing a **70% cost reduction** for telemetry ingestion.
3. **Tune MQTT Keep-Alive Intervals:**
   * Increase MQTT keep-alive intervals from 30 seconds to 20 minutes to minimize keep-alive message traffic and connection overhead.
4. **Minimize Device Shadow Updates:**
   * Update Device Shadows only when state changes occur, avoiding continuous periodic shadow writes ($1.25/M ops).
