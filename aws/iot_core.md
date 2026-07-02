# AWS Service Cost Research: AWS IoT Core

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS IoT Core is a managed cloud platform that lets connected devices (smart home appliances, industrial sensors, automotive telemetry systems, medical devices) easily and securely interact with cloud applications and other devices. It supports billions of devices and trillions of messages, managing device connections via MQTT, HTTPS, and WebSockets. AWS IoT Core features an MQTT Message Broker, Device Gateway, Rules Engine, Device Shadow, and Device Registry. AWS IoT Core is billed on a granular per-component pay-as-you-go basis.

---

## 2. Billing Mechanics
AWS IoT Core billing is calculated across four independent usage components:
1.  **Connectivity:** Billed per minute of active device connection to the Device Gateway ($0.08 per 1 Million connection minutes = ~$0.042 per device per year connected 24/7).
2.  **Messaging:** Billed per 1 Million messages transmitted ($1.00 per 1 Million messages). Messages are metered in **5 KB increments** up to 128 KB.
3.  **Rules Engine:** Billed per 1 Million rules triggered ($0.15 per 1 Million rules) and actions applied ($0.15 per 1 Million actions).
4.  **Device Shadow & Registry:** Billed per 1 Million shadow or registry operations ($1.25 per 1 Million operations). Operations are metered in **1 KB increments**.

---

## 3. Key Cost Dimensions

### A. Core Component Pricing Rates (us-east-1)
*   **Connectivity Rate:** **$0.08 per 1 Million connection minutes**.
*   **Messaging Rate:** **$1.00 per 1 Million messages** (in 5 KB chunks).
*   **Rules Engine Rate:** **$0.15 per 1 Million rules triggered**.
*   **Device Shadow Rate:** **$1.25 per 1 Million shadow operations** (in 1 KB chunks).

### B. High-Volume Messaging Hazard
If 10,000 devices send a 100-byte telemetry message every second (un-batched):
$$10,000 \text{ devices} \times 86,400 \text{ sec/day} \times 30 \text{ days} = 25.92 \text{ Billion messages/month}$$
$$25.92 \text{ Billion} \times \$1.00 / \text{Million} = \mathbf{\$25,920.00\text{ / month!}}$$

---

## 4. Detailed Pricing Rates (us-east-1)

| IoT Core Component | Metering Increment | Rate (us-east-1) | Price for 10 Million Units |
|--------------------|--------------------|------------------|----------------------------|
| **Connectivity** | Per connection-min | **$0.08 / 1M mins** | **$0.80** |
| **Messaging** | 5 KB chunks | **$1.00 / 1M msgs** | **$10.00** |
| **Rules Engine** | Per rule triggered | **$0.15 / 1M rules**| **$1.50** |
| **Device Shadow** | 1 KB chunks | **$1.25 / 1M ops** | **$12.50** |

---

## 5. AWS Free Tier Coverage
*   **AWS IoT Core Free Tier:** Includes **2,250,000 connection minutes**, 500,000 messages, 250,000 rules triggered, and 250,000 shadow/registry operations per month for 12 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **1-Second Telemetry Pings (Un-Batched Messaging):**
    *   Configuring edge devices to send tiny 50-byte MQTT messages every 1 second instead of batching.
    *   *Why:* Because IoT Core meters in 5 KB chunks, a 50-byte message incurs the full 5 KB message fee, generating tens of thousands of dollars in messaging bills.

---

## 7. Actionable Cost Optimization Strategies
1.  **Batch Telemetry Messages on Edge Devices:**
    *   Buffer telemetry data locally on the micro-controller or gateway device and publish combined 5 KB metric payloads every 60 seconds instead of sending 1-second pings.
    *   **The Savings:** Reduces message volume by 98.3%, dropping monthly messaging costs from **$25,920.00 down to $432.00/month (98% reduction)**.
2.  **Tune MQTT Keep-Alive Intervals:** Increase MQTT keep-alive intervals from 30 seconds to 20 minutes to minimize keep-alive message traffic and connection overhead.
3.  **Minimize Device Shadow Updates:** Update Device Shadows only when state changes occur, avoiding continuous periodic shadow writes ($1.25/M ops).
