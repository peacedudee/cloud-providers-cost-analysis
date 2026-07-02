# AWS Service Cost Research: AWS IoT Greengrass

> **Status:** ✅ Research Complete (Greengrass V2 Active)
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS IoT Greengrass is an open-source IoT edge runtime and cloud service that helps you build, deploy, and manage device software at the edge. Greengrass enables edge devices (Raspberry Pi, industrial PCs, edge gateways) to run local Lambda functions, Docker containers, local messaging, data caching, and ML inference directly on physical hardware without constant cloud connectivity.
*   **Version Status Notice:** Support for **AWS IoT Greengrass V1 was officially discontinued on June 1, 2026**. All new and existing edge deployments use **Greengrass V2**.

---

## 2. Billing Mechanics
1.  **Active Greengrass Core Device Monthly Fee:** Billed monthly per active core gateway device that connects to the Greengrass cloud control plane ($0.16 per active core device per month).
2.  **Free Tier:** First **3 active core devices per month** are **100% FREE ($0.00)** indefinitely.
3.  **Cloud Messaging & Data Storage:** Local messaging between edge devices connected to a Greengrass core is 100% free ($0.00). Standard IoT Core messaging rates apply only when data is synced to the AWS cloud.

---

## 3. Key Cost Dimensions

| Greengrass Feature | Free Tier Allowance | Rate (us-east-1) | Price for 1,000 Edge Gateways |
|--------------------|---------------------|------------------|--------------------------------|
| **Active Core Devices** | **First 3 devices free**| **$0.16 / device-mo**| **$159.52 / month** (997 paid) |
| **Local Edge Messaging**| Unlimited local | **Free ($0.00)** | **$0.00** |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Active Core Device Rate:** $0.16 per device per month ($1.92 per device per year).
*   **Local Container & ML Execution:** $0.00.

---

## 5. AWS Free Tier Coverage
*   **AWS IoT Greengrass Free Tier:** First **3 active core devices per month** free indefinitely for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Streaming Raw Unfiltered Edge Sensor Data to the Cloud:** Using Greengrass as a dumb pipe to stream raw high-frequency sensor data directly to AWS IoT Core ($1.00/M msgs), incurring high cloud messaging and storage bills instead of processing data locally.

---

## 7. Actionable Cost Optimization Strategies
1.  **Filter and Aggregate Sensor Data Locally at the Edge:**
    *   Deploy local Python/Lambda components on the Greengrass core to process raw millisecond sensor readings locally.
    *   Only stream 1-minute anomaly or aggregated statistical summaries to AWS IoT Core.
    *   **The Savings:** Reduces cloud IoT Core messaging costs by **95%+** while maintaining continuous local operations during internet outages.
2.  **Leverage Free Local Inter-Device Communication:** Configure local client devices (sensors/cameras) to publish MQTT messages directly to the Greengrass Core locally for $0.00 messaging charges.
