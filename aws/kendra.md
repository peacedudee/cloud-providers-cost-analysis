# AWS Service Cost Research: Amazon Kendra

> **Status:** ⚠️ Deprecated / Legacy Support only (Effective July 30, 2026)
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Kendra is an intelligent enterprise search service powered by machine learning. It allows organizations to search across various content repositories (such as SharePoint, S3, Salesforce, and databases) using natural language queries.
*   **Critical Cost Notice:** Amazon Kendra will **no longer accept new customers starting July 30, 2026**. Existing deployments can continue to run, but AWS recommends migrating search and generative RAG (Retrieval-Augmented Generation) architectures to **Amazon Bedrock Knowledge Bases** (coupled with OpenSearch, RDS, or third-party vector databases).

---

## 2. Billing Mechanics
Amazon Kendra is billed on a provisioned capacity hourly model:
1.  **Index Hourly Rate:** Billed per hour for each active index, regardless of whether it is queried.
2.  **Storage / Capacity Units:** Extra storage and query throughput capacity units are billed hourly.
3.  **Connector Usage:** Data connectors (e.g. Salesforce, SharePoint) have minor hourly run fees.

---

## 3. Key Cost Dimensions

### A. Index Edition Hourly Rates (us-east-1)
*   **Developer Edition Index (For POCs/Development only):**
    *   *Rate:* **$1.125 per hour** (~$810.00/month).
    *   *Limits:* Bounded to 10,000 documents and 4,000 queries/day. No high availability (multi-AZ).
*   **GenAI Enterprise Edition Index:**
    *   *Rate:* **$0.32 per hour** (~$230.40/month).
    *   *Limits:* Scaled down for lightweight RAG queries.
*   **Enterprise Edition Index (For high-scale production):**
    *   *Rate:* **$1.40 per hour** (~$1,008.00/month).
    *   *Limits:* Includes multi-AZ high availability. Standard limits of 100,000 documents and 40,000 queries/day.

### B. Index Scaling capacity (Enterprise Edition Only)
If you exceed base limits, you must purchase additional units:
*   **Storage Capacity Unit (SCU):** Adds 100,000 documents. Billed at **$0.35 per hour** (~$252.00/month) per SCU.
*   **Query Capacity Unit (QCU):** Billed at **$0.14 per hour** (~$100.80/month) per QCU.

### C. The Idle Index Cost
Because Kendra index pricing is provisioned, you pay for the index 24/7. An idle Developer index left forgotten in a test account bills **$810.00/month** even if it processes 0 queries.

---

## 4. Detailed Pricing Rates (us-east-1)

| Kendra Edition | Base Hourly Rate | Base Monthly Cost | Included Documents | Included Queries / Day |
|----------------|------------------|-------------------|--------------------|------------------------|
| **Developer** | **$1.125** | **$810.00** | 10,000 | 4,000 (No HA) |
| **GenAI Enterprise**| **$0.320** | **$230.40** | 10,000 | Variable |
| **Enterprise** | **$1.400** | **$1,008.00** | 100,000 | 40,000 (Multi-AZ) |

---

## 5. AWS Free Tier Coverage
*   **Developer Edition:** 750 free hours for the first 30 days (new Kendra accounts).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Forgotten Sandbox Indexes:** Creating a Developer Index to test search features, then leaving it provisioned for months. This bills **$810.00/month** per index.
*   **Using Enterprise Index for Development:** Launching an Enterprise index ($1,008.00/month) in development or staging environments where high availability and 100,000 document quotas are unnecessary.
*   **Duplicate Index Syncs:** Provisioning multiple separate indexes for different document directories when metadata filtering within a single index would suffice.

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate to Amazon Bedrock Knowledge Bases:**
    *   Proactively transition search and RAG workloads to **Amazon Bedrock Knowledge Bases**.
    *   Pair Bedrock with **Amazon RDS (PostgreSQL with pgvector)**.
    *   **The Savings:** Bypasses Kendra's high flat-rate index fees ($810 to $1,008/month), paying only standard RDS storage/compute rates (starting at ~$15/month).
2.  **Delete Kendra Indexes When Idle:**
    *   Do not leave Kendra indexes running in non-production.
    *   Kendra index data sources and sync mappings can be saved as configurations. Delete the index and only provision it when active testing is needed.
3.  **Enforce Metadata Filtering over Multiple Indexes:** Consolidate multiple corporate indexes into a single **Enterprise Index**. Use document metadata tags (e.g. `Department=HR`, `Group=Finance`) to restrict search query scopes rather than spinning up separate $1,008/mo index endpoints.
4.  **Audit Storage Capacity Units (SCUs):** Monitor document volumes. Ensure you are not overpaying for active SCUs ($0.35/hour) if your document catalog has dropped below threshold limits.
