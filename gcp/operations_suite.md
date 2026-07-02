# Cloud Logging & Monitoring (Operations Suite) Cost Optimization

Google Cloud's Operations Suite (formerly Stackdriver) provides logging, monitoring, tracing, and profiling. Because logging and metrics are automatically collected from every VM, GKE pod, and API call, ingestion volumes can grow silently and trigger massive bills. Standard logs are billed at **$0.50 per GB**, which is highly expensive for static text. This file covers tactics for optimizing Operations Suite costs.

---

## 1. Billing Components & Pricing

Operations Suite billing centers on two main areas:

### A. Cloud Logging
1. **Log Ingestion:** Billed per GiB ingested ($0.50/GB). The first **50 GB per billing account per month is free**.
2. **Log Retention:** Ingested logs are stored for **30 days for free** in the default `_Default` bucket. Retaining logs longer than 30 days is charged per GB/month ($0.010/GB/month).

### B. Cloud Monitoring
1. **System Metrics:** Standard GCP resource metrics (VM CPU, disk latency, network throughput) are collected **completely free**.
2. **Custom / Third-Party Metrics:** Ingested custom metric samples (e.g. application metrics, Prometheus metrics) are billed per million samples ingested after the free tier of 150 MiB.

---

## 2. Core Cost-Reduction Tactics

### A. Set Logs Router Exclusion Filters (Immediate Cost Reduction)
By default, the Logs Router routes all generated logs to the `_Default` bucket, charging ingestion for everything. You can write **Exclusion Filters** to discard low-value, high-volume logs at the routing level (before ingestion billing occurs).
* **High-Volume Targets to Exclude:**
  1. **HTTP Load Balancer Access Logs:** Exclude successful `HTTP 200` requests and health checks. Keep only errors (`HTTP 4xx` and `5xx`).
  2. **Kubernetes Health Probes:** Exclude `kube-probe` or readiness/liveness logs.
  3. **Application Debug Logs:** Ensure production environments are configured to log at `INFO` or `WARN` levels. Create an exclusion rule to drop any accidental `DEBUG` logs.
* **Exclusion Filter Syntax Example (Load Balancer Health Checks):**
  ```query
  resource.type="http_load_balancer"
  jsonPayload.statusDetails="handled_by_routing_rule"
  httpRequest.userAgent="GoogleHC/1.0"
  ```

### B. Route Compliance Logs to GCS Archive Storage
* **The Waste:** Keeping raw audit or access logs in the Cloud Logging bucket for 365 days or longer. Cloud Logging storage rates are high, and the query interface is not optimized for multi-year searches.
* **The Solution:**
  1. Set the retention of your standard `_Default` logging bucket to the free **30 days**.
  2. Create a Log Router Sink to export compliance logs to a **Cloud Storage (GCS)** bucket.
  3. Apply a GCS Lifecycle policy to immediately transition logs to **Coldline** or **Archive** storage ($0.0012/GB/month).
* **The Benefit:** Cuts long-term compliance storage costs by **over 80%**. If you need to analyze old logs, you can query the GCS bucket directly using BigQuery External Tables.

### C. Create Log-Based Metrics for Alerting
* **The Waste:** Storing millions of raw "successful action" logs in order to run queries or alerts when actions fail.
* **The Solution:** Create a **Log-Based Metric** (a counter or distribution) that scans log streams, extracts values (e.g., status codes), and saves them as numerical metrics in Cloud Monitoring.
* **Action:** Once the metric is defined, set an exclusion filter to discard the raw logs. You keep the ability to alert on failures (via the cheap metric) without paying to store the millions of raw logs.

### D. Enforce Structured JSON Logging
* Unstructured text logs are hard to filter. Structured logs (JSON format) allow you to write highly specific exclusion filters based on key-value attributes (e.g., dropping logs where `jsonPayload.user_id` matches test accounts).

---

## 3. Operations Suite Audit Checklist

1. [ ] **Top Log Emitters Identification:** Query Cloud Billing reports or run a log volume query to find the top 3 projects/services emitting the highest log volumes.
2. [ ] **Exclusion Filter Implementation:** Ensure active exclusion filters are running on all load balancer and Kubernetes subnets.
3. [ ] **Clean Up Log Sinks:** Delete duplicate log sinks to avoid paying multiple ingestion charges for the same log line.
4. [ ] **Standardize Retention:** Verify that all active log buckets (excluding compliance) have retention set to the free 30 days.
5. [ ] **Audit Compliance Exports:** Confirm that all multi-year log retention requirements are handled via exports to GCS Archive or BigQuery rather than Logging buckets.
6. [ ] **Audit Custom Metrics:** Review Custom Metrics in Cloud Monitoring. Delete metrics with high cardinality (e.g., metrics tagged with unique user IDs) which inflate ingestion costs.
