# Pub/Sub Cost Optimization & Research

Google Cloud Pub/Sub is an asynchronous messaging service designed to be highly reliable and scalable. While it is billed strictly based on data volume, inefficient message packaging, long retention periods, and unnecessary message routing can escalate costs. 

---

## 1. Important Platform Update (2026)

> [!IMPORTANT]
> **Pub/Sub Lite Discontinuation:** Google Cloud Pub/Sub Lite will be permanently shut down on **March 18, 2026**. Organizations that previously utilized Pub/Sub Lite for cheaper, partition-based messaging must migrate their pipelines to standard Pub/Sub immediately.

---

## 2. Pub/Sub Billing Mechanics

Standard Pub/Sub billing is composed of three main factors:
1. **Throughput (Data Volume):** Charged per GB of data ingested and delivered.
   * **The 1 KB Minimum Rule:** Every publish, pull, or push request carries a **minimum charge of 1 KB of data**, regardless of actual message size. If you publish a 100-byte message, you are billed for 1,000 bytes.
2. **Message Storage (Retention):** Retaining messages on topics or in subscriptions (to replay them later or to buffer them for slow subscribers) is billed per GB/month.
   * **The Free Zone:** The first 24 hours of message retention is **free**. You are only charged for retaining messages beyond 24 hours.
3. **Network Egress:** Moving data across regions or zones between publishers, Pub/Sub servers, and subscribers incurs standard GCP networking egress fees.

---

## 3. Core Cost-Reduction Tactics

### A. Enable Message Batching at the Publisher
Because of the 1 KB minimum billing rule, sending single, small messages to Pub/Sub is highly expensive.
* **The Math:** Publishing 100 messages of 100 bytes individually = 100 KB billed. Batching those 100 messages into a single publish request = 10 KB billed (a **90% savings**).
* **Action:** Ensure that all publisher applications are configured to buffer and batch messages (based on size, count, or a delay threshold like 50ms) before sending them to the API. Google Cloud Client Libraries handle batching natively but require proper configuration parameter tuning.

### B. Use Claim Check Pattern for Large Payloads
Pub/Sub is designed for fast, lightweight messaging. Publishing large images, files, or JSON blobs (up to 10 MB maximum) generates enormous throughput bills.
* **Tactic (Claim Check):**
  1. Write the large payload to a **Cloud Storage (GCS)** bucket.
  2. Publish a tiny Pub/Sub message containing only the GCS file reference (the URI).
  3. The subscriber receives the reference and downloads the data directly from GCS.
* **The Benefit:** GCS storage and internal data transfers are far cheaper per GB than Pub/Sub throughput.

### C. Minimize Message Retention Settings
By default, Pub/Sub retains messages on subscriptions for 7 days.
* **Action:** If your subscriber applications consume messages immediately and do not require historical message replays, change the message retention setting on both topics and subscriptions to **24 hours or less** (remembering that the first 24 hours are free).
* **The Benefit:** Eliminates the storage capacity charges for unacknowledged or acknowledged messages held in Pub/Sub memory.

### D. Implement Subscription Filters
* **The Waste:** A subscriber receives all messages from a topic and then uses application code to filter out 90% of them. You are billed by Pub/Sub for delivering 100% of the messages to that subscriber.
* **Action:** Use **Pub/Sub Subscription Filters**. Define filter expressions (e.g. `attributes.event_type = "payment_success"`) directly on the subscription configuration.
* **The Benefit:** Pub/Sub filters the messages at the server side. You are only billed for the data volume of messages that actually match the filter and are delivered to the subscriber.

### E. Use BigQuery Direct Subscriptions for Data Ingestion
* **The Waste:** Traditionally, to stream Pub/Sub data into BigQuery, organizations ran a continuous **Cloud Dataflow** pipeline. You pay for both the Dataflow VMs and the Pub/Sub delivery fees.
* **Action:** For simple ingestion, use the native **BigQuery Subscription** type. This allows Pub/Sub to write messages directly to a BigQuery table without spinning up Dataflow or any intermediate compute.

---

## 4. Pub/Sub Audit Checklist

1. [ ] **Lite Migration Check:** Identify and migrate all legacy Pub/Sub Lite topics to Standard Pub/Sub before the March 18, 2026 turn-down.
2. [ ] **Publisher Batching Validation:** Review publisher configurations to ensure message batching thresholds are active.
3. [ ] **Retention Setting Review:** Audit topics and subscriptions. Reduce message retention from the default 7 days to 24 hours (or less) where replay is not required.
4. [ ] **Client-side Filtering:** Locate subscriptions performing local filtering. Replace with server-side Pub/Sub filters.
5. [ ] **Dataflow Replacement:** Identify simple Pub/Sub-to-BigQuery pipelines and replace the Dataflow workers with direct BigQuery Subscriptions.
6. [ ] **Large Payload Check:** Verify that payloads > 100 KB are stored in GCS and passed via reference.
