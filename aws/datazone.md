# AWS Service Cost Research: Amazon DataZone

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon DataZone is a managed data management and governance service that makes it easy for data producers to catalog, discover, share, and govern data at scale across organizational boundaries. It provides a web portal where users can publish data assets, request access to datasets, and collaborate on projects. DataZone is serverless and billed on a pay-as-you-go consumption model, bypassing monthly per-user subscription fees.

---

## 2. Billing Mechanics
DataZone billing is calculated on a monthly cycle based on four consumption metrics:
1.  **API Requests:** Billed per 100,000 API calls made to the DataZone control plane (with a generous free tier).
2.  **Metadata Storage:** Billed per GB-month for data catalog metadata stored.
3.  **DataZone Compute Units (DCU-hours):** Billed per hour for metadata search indexing and lineage processing.
4.  **AI Recommendations (Optional):** Billed per token for using generative AI (powered by Amazon Bedrock) to automatically write business descriptions for assets.

---

## 3. Key Cost Dimensions

### A. API Requests (us-east-1)
*   **The Rate:** **$10.00 per 100,000 requests** ($0.0001 per request).
*   **Free Allocation:** The first **4,000 requests** per month are free.
*   *Note: Core administrative APIs (such as creating domains, projects, and executing basic search queries) are **100% free** and do not count toward this limit.*

### B. Metadata Storage
*   **The Rate:** **$0.40 per GB-month**.
*   **Free Allocation:** The first **20 MB** of metadata storage per month is free.

### C. DataZone Compute (DCUs)
Compute capacity is consumed when crawling datasources or generating data lineage maps.
*   **The Rate:** **$0.55 per DCU-hour**.
*   **Free Allocation:** The first **0.2 DCUs** per month are free.

### D. AI Metadata Recommendations
Uses generative AI (LLMs) to scan schemas and suggest business friendly descriptions for tables and columns.
*   **The Charge:** Billed based on input and output tokens consumed (rates align with Amazon Bedrock API fees, e.g. **$0.015 per 1,000 tokens**).

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Free Tier Allowance | Billed Rate (us-east-1) | Billed Unit |
|----------------|---------------------|--------------------------|-------------|
| **Control APIs** | Unlimited | **Free ($0.00)** | Administrative actions |
| **API Requests** | 4,000 requests / mo | **$10.00** | Per 100,000 requests |
| **Metadata Storage**| 20 MB / month | **$0.40** | Per GB-month |
| **DCU Compute** | 0.2 DCU / month | **$0.55** | Per DCU-hour |
| **AI Token Usage**| N/A | Bedrock rates (token-based)| Per 1,000 tokens |

---

## 5. AWS Free Tier Coverage
DataZone features a permanent **always-free tier** containing:
*   4,000 API requests per month.
*   20 MB of metadata storage per month.
*   0.2 DCU-hours of compute per month.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Continuous Crawler Metadata Syncs:** Scheduling metadata crawlers to scan massive, frequently changing tables (e.g. log directories) every hour. This consumes high DCU compute hours ($0.55/hr) and metadata request volumes.
*   **Massive AI Description Generation Runs:** Triggering AI recommendations for every individual column of a multi-thousand table database. Generating descriptions for 100,000 columns can consume millions of tokens, leading to unexpected Bedrock charges.

---

## 7. Actionable Cost Optimization Strategies
1.  **Restrict AI Metadata Generation to Core Assets:**
    *   Do not generate AI descriptions for all technical columns (like `id`, `created_at`, or `updated_at`).
    *   Run AI recommendations only for primary **table summaries** and key business-critical columns.
2.  **Decrease Metadata Ingestion Schedules:** Do not crawl your databases or S3 data lakes hourly. Schedule metadata syncs to run **once daily or weekly** (or trigger them only after major schema deployments) to stay within the **0.2 free DCU-hour allowance**.
3.  **Optimize Search Queries (Bypass API limits):** Encourage users to search for data assets through the **DataZone Web Portal** (which uses free control plane APIs) rather than writing custom scripts that poll the search APIs programmatically.
4.  **Consolidate Data Projects:** Keep your DataZone domain organized. Avoid creating separate "projects" for every developer. Consolidate developers under shared project domains to minimize duplicate metadata storage overheads.
