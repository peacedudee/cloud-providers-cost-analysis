# Google SecOps (Chronicle) Cost Optimization & Research

Google SecOps (formerly Chronicle SIEM and SOAR) is a cloud-native security information and event management (SIEM) and security orchestration, automation, and response (SOAR) platform. Because Chronicle ingests security telemetry from across your hybrid cloud, managing **log ingestion volume** and filtering out low-signal, high-frequency logs is critical to prevent massive ingestion billing.

---

## 1. Google SecOps Billing Components

Chronicle pricing is typically structured under two models:
1. **Ingestion Volume (Credit-based):** Billed based on the volume of raw log data ingested into the platform. A purchased subscription provides a set amount of ingestion credits.
2. **Package-Based Subscription:** Subscriptions (Standard, Enterprise, Enterprise Plus) define available features. You purchase an annual ingestion capacity, and exceeding this limit results in Usage Overage charges.

---

## 2. Core Cost-Optimization Levers

### A. Apply Exclusion Filters at the GCP Log Router
* **The Cost Trap:** Forwarding all Google Cloud logs (including raw VPC Flow Logs, GCS Data Access Logs, and Cloud SQL Query Logs) to Chronicle. These logs generate terabytes of data daily, resulting in huge ingestion bills for little security value.
* **The Solution:** Filter before sending.
* **Action:** In your Google Cloud **Log Router**, configure sinks that forward logs to Chronicle. Use exclusion filters to drop low-signal logs:
  * Exclude `dataRead` and `dataWrite` audits from GCS, unless auditing a highly sensitive bucket.
  * Exclude successful network connections (`ACCEPT`) from VPC Flow Logs, and only forward `REJECT` logs to track firewall blockages.
  * Exclude standard database query logs and only forward authentication failures.

### B. Segment Ingestion by Environment
* **Action:** Restrict Chronicle ingestion to production environments and corporate control planes. Avoid ingesting system logs, container console logs, or microservice debug traces from development, testing, and staging projects, as these generate high volumes of noise.

### C. Consolidate and Deduplicate Log Feeds
* **Action:** If you are forwarding logs from GKE clusters, ensure you are not duplicate-logging (e.g., forwarding container stdout via Fluentd AND forwarding node system logs, capturing the same error twice).

---

## 3. Google SecOps Audit Checklist

1. [ ] **Log Router Exclusion Filters:** Verify that active log sinks forwarding to Chronicle have filters excluding high-volume data plane access logs.
2. [ ] **Production Ingestion Focus:** Confirm that no staging, QA, or sandbox debug logs are routed to the SIEM.
3. [ ] **Flow Logs Filter:** Ensure that only firewall `REJECT` logs are forwarded to Chronicle rather than complete VPC flow traces.
4. [ ] **Ingestion Capacity Monitoring:** Audit actual daily log ingestion against your purchased annual capacity limits to prevent Usage Overage charges.
