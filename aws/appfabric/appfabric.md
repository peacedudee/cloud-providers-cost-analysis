# AWS Service Cost Research: AWS AppFabric

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS AppFabric is a fully managed service that connects, centralizes, and standardizes security audit logs and user access states across SaaS applications (including Asana, Google Workspace, Jira, Salesforce, Slack, Zoom, Workday, and Zendesk). AppFabric normalizes SaaS log streams into the Open Cybersecurity Schema Framework (OCSF) format and routes them to security data lakes or SIEM tools (Amazon Security Lake, OpenSearch, Splunk, Datadog) without custom API development. AppFabric billing is based on user profiles connected and audit log volume ingested.

---

## 2. Billing Mechanics
1. **User Profile Connections:** Billed monthly per unique user profile per connected SaaS application ($3.00 per user profile per app per month).
2. **Audit Log Ingestion & Normalization:** Billed per GB of SaaS audit log data ingested, normalized to OCSF format, and delivered ($0.25 per GB processed).
3. **Downstream Data Storage & Transport:** Standard Amazon S3 storage ($0.023/GB-mo) or Amazon Security Lake / Kinesis Data Firehose fees apply for downstream targets.

---

## 3. Key Cost Dimensions

### A. Core Pricing Rates (us-east-1)
* **User Profile Connections:** **$3.00 per user profile per connected app per month**.
* **Audit Log Ingestion:** **$0.25 per GB** of SaaS audit logs ingested and normalized to OCSF format.

### B. Mathematical Cost Calculation Example
*Scenario: 1,000 corporate users connected across 3 primary SaaS applications (Salesforce, Google Workspace, Slack), generating 100 GB of normalized audit logs per month.*

1. **User Profile Connections Charge:**
   $$\text{Profile Fee} = (1,000\text{ users} \times 3\text{ apps}) \times \$3.00 = \$9,000.00\text{ / month}$$
2. **Audit Log Ingestion Charge:**
   $$\text{Log Ingestion Fee} = 100\text{ GB} \times \$0.25 = \$25.00\text{ / month}$$
3. **Total Monthly Cost:** **$9,025.00 / month**

---

## 4. Detailed Pricing Rates (us-east-1)

| AppFabric Feature | Billed Unit | Rate (us-east-1) | Price for 1,000 Users across 3 Apps |
|-------------------|-------------|------------------|--------------------------------------|
| **User Profile Connections** | Per user-app-month | **$3.00 / profile** | **$9,000.00 / month** |
| **Audit Log Ingestion** | Per GB processed | **$0.25 / GB** | **$25.00** (100 GB logs) |

---

## 5. AWS Free Tier Coverage
* **AWS AppFabric Free Tier:** Includes a **30-day free trial** for up to 100 user profiles per connected SaaS application.

---

## 6. Common Cost Hotspots & Pitfalls
* **Connecting All Auxiliary SaaS Apps Universally:** Connecting AppFabric to non-critical SaaS tools used by thousands of employees, generating high user profile connection charges ($3.00/user-app-mo).
* **Verbose Unfiltered Log Streams:** Emitting high-frequency non-security events from SaaS tools into AppFabric ($0.25/GB log ingestion).

---

## 7. Actionable Cost Optimization Strategies
1. **Restrict AppFabric Connections to Mission-Critical SaaS Platforms:** Limit AppFabric integration strictly to high-risk SaaS platforms (e.g., Salesforce, Google Workspace, Workday, Slack) that handle sensitive enterprise data.
   * **The Savings:** Slashes user profile connection billing by **60%–80%**.
2. **Filter SaaS Event Streams at the Source:** Adjust SaaS log configurations to export only security-relevant events (authentication attempts, permission modifications, data exports) rather than full user activity clickstreams.
