# AWS Service Cost Research: Amazon MQ

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon MQ is a managed message broker service for open-source message brokers Apache ActiveMQ and RabbitMQ. It makes it easy to migrate existing enterprise message brokers to the cloud without rewriting messaging code (supporting industry-standard APIs like JMS, NMS, AMQP, STOMP, MQTT, and WSS). Amazon MQ provisions dedicated single-instance or multi-AZ active/standby broker nodes. It is a provisioned service billed per broker instance-hour and storage volume.

---

## 2. Billing Mechanics
1.  **Broker Instance Hourly Fee:** Billed hourly per provisioned broker node based on instance class (`t3.micro`, `m5.large`, `m5.xlarge`) and deployment mode (Single-AZ vs. Multi-AZ Active/Standby).
2.  **Broker Storage:** Billed per GB of message storage per month (ActiveMQ EFS storage vs. RabbitMQ EBS GP3 storage).
3.  **Data Egress:** Standard AWS cross-AZ and internet data egress rates apply.

---

## 3. Key Cost Dimensions

### A. Broker Instance Hourly Pricing (us-east-1)
*   **`mq.t3.micro` (Single-AZ):** **$0.03 per hour** (~$21.90 per month).
*   **`mq.m5.large` (Single-AZ):** **$0.268 per hour** (~$195.64 per month).
*   **`mq.m5.large` (Multi-AZ Active/Standby - 2 Nodes):** **$0.536 per hour** (~$391.28 per month).
*   **`mq.m5.xlarge` (Multi-AZ Active/Standby):** **$1.072 per hour** (~$782.56 per month).

### B. Storage Pricing
*   **ActiveMQ EFS Storage:** **$0.30 per GB-month**.
*   **RabbitMQ / ActiveMQ EBS GP3 Storage:** **$0.10 per GB-month**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Broker Class | Deployment Mode | Hourly Rate | Monthly Base Cost (24/7) |
|--------------|-----------------|-------------|--------------------------|
| **`mq.t3.micro`** | Single-AZ | **$0.030** | **$21.90** |
| **`mq.m5.large`** | Single-AZ | **$0.268** | **$195.64** |
| **`mq.m5.large`** | Multi-AZ (Active/Standby)| **$0.536** | **$391.28** |
| **`mq.m5.xlarge`**| Multi-AZ (Active/Standby)| **$1.072** | **$782.56** |

---

## 5. AWS Free Tier Coverage
*   **Amazon MQ Free Tier:** Includes **750 free hours** of single-instance `mq.t3.micro` broker usage and 1 GB of storage per month for 12 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Running Multi-AZ `mq.m5.large` Brokers in Non-Production:** Launching Multi-AZ Active/Standby broker clusters ($391.28/mo base) in developer sandbox environments.
*   **Choosing Amazon MQ for New Cloud-Native Architectures:** Selecting Amazon MQ ($195–$391/mo base) for brand-new cloud applications instead of serverless messaging (SQS/SNS).

---

## 7. Actionable Cost Optimization Strategies
1.  **Use Amazon SQS / SNS for New Cloud-Native Microservices:**
    *   If you are building new applications (rather than migrating legacy JMS/AMQP code), use **Amazon SQS ($0.40/M reqs)** or **SNS ($0.50/M reqs)**.
    *   *Why:* Serverless SQS/SNS scales to $0.00 when idle, whereas Amazon MQ runs continuous 24/7 instance fees ($195.64+/month).
    *   **The Savings:** Eliminates $2,300+ in annual fixed instance charges per environment.
2.  **Use Single-AZ `mq.t3.micro` ($21.90/mo) in Non-Production:** Downgrade dev/test message brokers to single-instance `mq.t3.micro` nodes ($21.90/mo) to save **94%** compared to production multi-AZ brokers.
3.  **Migrate ActiveMQ Storage to EBS GP3:** For RabbitMQ or ActiveMQ EBS deployments, select **EBS GP3 storage ($0.10/GB-mo)** over EFS ($0.30/GB-mo) to save 66% on storage fees.
