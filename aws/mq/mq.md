# AWS Service Cost Research: Amazon MQ

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon MQ is a managed message broker service for open-source message brokers Apache ActiveMQ and RabbitMQ. It makes it easy to migrate existing enterprise message brokers to the cloud without rewriting messaging code (supporting industry-standard APIs like JMS, NMS, AMQP, STOMP, MQTT, and WSS). Amazon MQ provisions dedicated single-instance or multi-AZ active/standby broker nodes. It is a provisioned service billed per broker instance-hour and storage volume.
* **Deprecation Notice (`mq.t3.micro`):** The `mq.t3.micro` instance type has been **DEPRECATED** and is no longer available for provisioning new message brokers. AWS recommends upgrading new deployments to Graviton `mq.m7g` or `mq.m5` instance families.

---

## 2. Billing Mechanics
1. **Broker Instance Hourly Fee:** Billed hourly per provisioned broker node based on instance class (`mq.m7g.medium`, `mq.m5.large`, `mq.m5.xlarge`) and deployment mode (Single-AZ vs Multi-AZ Active/Standby vs Multi-AZ Cluster).
2. **Broker Storage:** Billed per GB of message storage per month (ActiveMQ EFS storage vs. RabbitMQ EBS GP3 storage).
3. **Data Egress:** Standard AWS cross-AZ and internet data egress rates apply.

---

## 3. Key Cost Dimensions

### A. Broker Instance Hourly Pricing (us-east-1 On-Demand)
* **`mq.m7g.medium` (Single-AZ Graviton):** **$0.144 per hour** (~$105.12 per month).
* **`mq.m5.large` (Single-AZ):** **$0.288 per hour** (~$210.24 per month).
* **`mq.m5.large` (Multi-AZ Active/Standby - 2 Nodes):** **$0.576 per hour** (~$420.48 per month).
* **`mq.m5.xlarge` (Multi-AZ Active/Standby):** **$1.152 per hour** (~$840.96 per month).

### B. Storage Pricing
* **ActiveMQ EFS Storage:** **$0.30 per GB-month**.
* **RabbitMQ / ActiveMQ EBS GP3 Storage:** **$0.10 per GB-month**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Broker Class | Deployment Mode | Hourly Rate | Monthly Base Cost (24/7) |
|--------------|-----------------|-------------|--------------------------|
| **`mq.m7g.medium`** | Single-AZ | **$0.144** | **$105.12** |
| **`mq.m5.large`** | Single-AZ | **$0.288** | **$210.24** |
| **`mq.m5.large`** | Multi-AZ (Active/Standby)| **$0.576** | **$420.48** |
| **`mq.m5.xlarge`**| Multi-AZ (Active/Standby)| **$1.152** | **$840.96** |

---

## 5. AWS Free Tier Coverage
* **Amazon MQ Free Tier:** Includes **750 free hours** of single-instance micro broker usage and 1 GB of storage per month for 12 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
* **Running Multi-AZ `mq.m5.large` Brokers in Non-Production:** Launching Multi-AZ Active/Standby broker clusters ($420.48/mo base) in developer sandbox environments.
* **Choosing Amazon MQ for New Cloud-Native Architectures:** Selecting Amazon MQ ($105–$420/mo base) for brand-new cloud applications instead of serverless messaging (SQS/SNS).

---

## 7. Actionable Cost Optimization Strategies
1. **Migrate New Broker Deployments to Graviton (`mq.m7g`):**
   * Select **`mq.m7g.medium` ($0.144/hr)** instead of `mq.m5.large` ($0.288/hr) for new message brokers.
   * **The Savings:** Cuts broker instance costs by **50% ($105.12/mo vs $210.24/mo)**.
2. **Use Amazon SQS / SNS for New Cloud-Native Microservices:**
   * If building new applications (rather than migrating legacy JMS/AMQP code), use **Amazon SQS ($0.40/M reqs)** or **SNS ($0.50/M reqs)**.
   * *Why:* Serverless SQS/SNS scales to $0.00 when idle, whereas Amazon MQ runs continuous 24/7 instance fees ($105.12+/month).
   * **The Savings:** Eliminates $1,200+ in annual fixed instance charges per environment.
3. **Use Single-AZ Brokers in Non-Production:** Downgrade dev/test message brokers to single-instance `mq.m7g.medium` nodes ($105.12/mo) to save **75%** compared to production multi-AZ brokers.
4. **Migrate ActiveMQ Storage to EBS GP3:** Select **EBS GP3 storage ($0.10/GB-mo)** over EFS ($0.30/GB-mo) to save 66% on storage fees.
