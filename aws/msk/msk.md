# AWS Service Cost Research: Amazon MSK (Managed Streaming for Apache Kafka)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon MSK is a fully managed, highly available service that makes it easy to build and run applications that use Apache Kafka to process streaming data. It manages the provisioning, configuration, and maintenance of Apache Kafka clusters. MSK can be deployed in a Provisioned capacity model (where you manage broker instance types and count) or in a Serverless model. Because Kafka relies on active message brokers and storage replication, idle clusters are a common source of high fixed costs.

---

## 2. Billing Mechanics
Amazon MSK billing is structured based on the selected deployment model:
1. **MSK Provisioned Clusters:**
   * *Broker Node Hours:* Billed hourly per broker instance based on instance family and size (e.g. `kafka.t3.small`, `kafka.m5.large`, `kafka.m7g.large`).
   * *Storage:* Billed per GB-month for EBS storage provisioned per broker.
   * *Tiered Storage:* Billed per GB-month for offloaded message data stored in S3 ($0.060 per GB-month).
   * *Data Transfer:* Standard AWS cross-AZ data transfer fees apply to inter-broker replication traffic.
2. **MSK Serverless Clusters:**
   * *Cluster Hours:* A flat hourly charge per provisioned serverless cluster ($0.75 per cluster-hour = $547.50/mo base).
   * *Partition Hours:* Billed hourly for active partition counts ($0.0015 per partition-hour).
   * *Data Ingestion / Retrieval:* Billed per GB written ($0.10/GB) and read ($0.05/GB).
   * *Storage:* Billed per GB-month of message storage ($0.10/GB-mo).

---

## 3. Key Cost Dimensions

### A. MSK Provisioned Clusters (us-east-1)
* **Broker Instance Hours:** You pay per-second (with a 1-minute minimum) for each running broker.
  * *kafka.t3.small (Development Node):* **$0.0433 per hour** (~$31.60/month).
  * *kafka.m5.large (Baseline Production Node):* **$0.2100 per hour** (~$153.30/month).
* **Redundancy Multiplier:** Kafka clusters require a minimum of **3 brokers** spread across 3 Availability Zones for HA, which triples the base broker fee (e.g. 3 × $0.21/hr = **$459.90/month** base for `kafka.m5.large`).
* **Storage Capacity:** Billed at **$0.10 per GB-month** for provisioned EBS storage per broker.
* **Inter-Broker Replication Tax:** Kafka replicates messages across brokers in different AZs. Data transfer egress *into* MSK brokers is free, but **cross-AZ replication egress** between brokers is billed at standard rates (**$0.01 per GB**).

### B. MSK Serverless Clusters
* **Cluster Base Fee:** **$0.75 per cluster-hour** (in `us-east-1`), creating a flat idle base fee of:
  $$\$0.75 \times 730\text{ hours} = \mathbf{\$547.50\text{ / month flat}}$$
  *Warning: This flat base fee is higher than running 3 provisioned `kafka.t3.small` nodes ($94.80/mo).*
* **Partition Fee:** **$0.0015 per partition-hour** (minimum 1 partition).
* **Data Throughput:**
  * *Ingestion:* **$0.10 per GB** written.
  * *Retrieval:* **$0.05 per GB** read.
* **Storage Capacity:** **$0.10 per GB-month** (with a retention limit of up to 1 day).

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Provisioned Cluster Rate | Serverless Cluster Rate | Details |
|----------------|--------------------------|-------------------------|---------|
| **Base Compute** | **$0.2100 / broker-hr** (`m5.large`) | **$0.7500 / cluster-hr** | Flat hourly charge |
| **Partition Hour**| N/A | **$0.0015 / partition-hr** | Billed per partition |
| **EBS Storage** | **$0.1000 / GB-month** | **$0.1000 / GB-month** | Broker disk storage |
| **Tiered Storage (S3)**| **$0.0600 / GB-month** | N/A | Offloaded S3 retention |
| **Data Ingestion**| Free ($0.00) | **$0.1000 / GB** | Volume processed |
| **Data Retrieval**| Free ($0.00) | **$0.0500 / GB** | Volume read |

---

## 5. AWS Free Tier Coverage
* **Amazon MSK:** **No free tier** is available. Any initialized cluster generates standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
* **Deploying MSK Serverless for Low-Traffic Dev Apps:** Provisioning MSK Serverless for small testing workloads. The flat **$547.50/month cluster base fee** is highly inefficient for low-traffic endpoints.
* **EBS Over-Provisioning on Brokers:** Allocating large EBS volumes (e.g. 1 TB per broker) to handle historical messages, billing at $0.10/GB-month ($300/month for 3 brokers) when messages could be offloaded.
* **Unmonitored Partition Sprawl (Serverless):** Creating thousands of partitions in MSK Serverless. 1,000 partitions cost **$1,095.00/month** in partition-hour fees alone.

---

## 7. Actionable Cost Optimization Strategies
1. **Break-Even Sizing (Serverless vs. Provisioned):**
   * Use **Provisioned Mode with `kafka.t3.small` nodes** for development, testing, and QA environments (3 nodes = ~$95/month). Do not use Serverless (which starts at ~$548/month base).
   * Use **MSK Serverless** for production workloads that are highly variable and process **less than 2 TB of data per month** to avoid managing broker scaling.
   * Use **Provisioned Mode with `kafka.m5` or `kafka.m7g` nodes** for large, steady-state production workloads processing **more than 3 TB of data per month**.
2. **Tiered Storage (Offload EBS to S3):**
   * Enable **Tiered Storage** on your Provisioned MSK clusters.
   * Configure a short retention window (e.g., 2 hours or 12 hours) on local broker EBS storage ($0.10/GB).
   * Let MSK automatically push older message data to **Amazon S3 Tiered Storage ($0.060/GB-month)**, cutting storage costs by **40%**.
3. **Compress Payloads at the Producer:** Always enable message compression (e.g., **LZ4**, **Snappy**, or **Gzip**) on your Kafka producers. This reduces broker storage footprints, ingestion volume, and cross-AZ replication egress transfer charges by up to **50–80%**.
4. **Consolidate Partitions:** Review partition layouts. Avoid creating redundant partitions per topic. Keep partition counts aligned with consumer group concurrency capabilities.
