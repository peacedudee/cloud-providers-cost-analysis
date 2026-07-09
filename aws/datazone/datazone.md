# AWS Service Cost Research: Amazon DataZone

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon DataZone is a managed data management and governance service that makes it easy for data producers to catalog, discover, share, and govern data at scale across organizational boundaries. It provides a web portal where users can publish data assets, request access to datasets, and collaborate on projects. DataZone is serverless and billed on a pay-as-you-go consumption model, bypassing monthly per-user subscription fees.

---

## 2. Billing Mechanics
DataZone billing is calculated on a monthly cycle based on four consumption metrics:
1.  **API Requests:** Billed per 100,000 API calls made to the DataZone control plane (with a generous free tier).
2.  **Metadata Storage:** Billed per GB-month for data catalog metadata stored.
3.  **Compute Units:** Billed per compute unit utilized for automated metadata crawling, ingestion, search indexing, and lineage mapping.
4.  **AI Recommendations (Optional):** Billed based on input and output tokens consumed when using generative AI (powered by Amazon Bedrock) to automatically write business descriptions for assets.

---

## 3. Key Cost Dimensions

### A. API Requests (us-east-1)
*   **The Rate:** **$10.00 per 100,000 requests** ($0.0001 per request).
*   **Free Allocation:** The first **4,000 requests** per month are free.
*   *Note: Core administrative APIs (such as creating domains, projects, and executing basic search queries via the Web Portal) are **100% free** and do not count toward this limit.*

### B. Metadata Storage
*   **The Rate:** **$0.40 per GB-month**.
*   **Free Allocation:** The first **20 MB** of metadata storage per month is free.

### C. Compute Units
Compute capacity is consumed when crawling datasources or generating data lineage maps.
*   **The Rate:** **$1.776 per Compute Unit**.
*   **Free Allocation:** The first **0.2 Compute Units** per month are free.

### D. AI Metadata Recommendations
Uses generative AI (LLMs) to scan schemas and suggest business-friendly descriptions for tables and columns.
*   **Input Tokens:** **$0.015 per 1,000 tokens**.
*   **Output Tokens:** **$0.075 per 1,000 tokens**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Free Tier Allowance | Billed Rate (us-east-1) | Billed Unit |
|----------------|---------------------|--------------------------|-------------|
| **Control APIs** | Unlimited | **Free ($0.00)** | Administrative actions / Portal searches |
| **API Requests** | 4,000 requests / mo | **$10.00** | Per 100,000 requests |
| **Metadata Storage**| 20 MB / month | **$0.40** | Per GB-month |
| **Compute Units** | 0.2 units / month | **$1.776** | Per Compute Unit |
| **AI Input Tokens**| N/A | **$0.015** | Per 1,000 tokens |
| **AI Output Tokens**| N/A | **$0.075** | Per 1,000 tokens |

### Example Monthly Cost Calculation
*Workload: A team catalogs a new data warehouse. Over the month, the catalog generates 50,000 API requests, stores 2 GB of metadata, consumes 3 Compute Units during crawlers, and generates 2,000,000 input tokens and 500,000 output tokens for AI descriptions.*

1.  **API Request Cost:**
    $$\text{Requests Billed} = 50,000 - 4,000\text{ (Free)} = 46,000\text{ requests}$$
    $$\text{Request Fee} = \frac{46,000}{100,000} \times \$10.00 = \$4.60$$
2.  **Metadata Storage Cost:**
    *   Slightly under 20 MB is free, so we bill on the full 2 GB:
        $$\text{Storage Fee} = 2\text{ GB} \times \$0.40 = \$0.80$$
3.  **Compute Units Cost:**
    $$\text{Compute Billed} = 3.0 - 0.2\text{ (Free)} = 2.8\text{ Compute Units}$$
    $$\text{Compute Fee} = 2.8 \times \$1.776 = \$4.97$$
4.  **AI Recommendations Cost:**
    $$\text{Input Token Fee} = \frac{2,000,000}{1,000} \times \$0.015 = \$30.00$$
    $$\text{Output Token Fee} = \frac{500,000}{1,000} \times \$0.075 = \$37.50$$
    $$\text{Total AI Fee} = \$30.00 + \$37.50 = \$67.50$$
*   **Total Monthly Cost:** **$77.87/month**

---

## 5. AWS Free Tier Coverage
DataZone features a permanent **always-free tier** containing:
*   4,000 API requests per month.
*   20 MB of metadata storage per month.
*   0.2 Compute Units per month.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Continuous Crawler Metadata Syncs:** Scheduling metadata crawlers to scan massive, frequently changing tables (e.g. log directories) every hour. This consumes high Compute Units ($1.776/unit) and metadata storage.
*   **Massive AI Description Generation Runs:** Triggering AI recommendations for every individual column of a multi-thousand table database. Generating descriptions for thousands of columns can consume millions of input and output tokens, leading to high output token surcharges ($0.075/1k).

---

## 7. Actionable Cost Optimization Strategies
1.  **Restrict AI Metadata Generation to Core Assets:**
    *   Do not generate AI descriptions for all technical columns (like `id`, `created_at`, or `updated_at`).
    *   Run AI recommendations only for primary **table summaries** and key business-critical columns.
    *   **The Savings:** Reduces output token generation costs ($0.075/1k) by up to **80%**.
2.  **Decrease Metadata Ingestion Schedules:**
    *   Do not crawl your databases or S3 data lakes hourly. Schedule metadata syncs to run **once daily or weekly** (or trigger them only after major schema deployments) to stay within or near the **0.2 free Compute Unit allowance**.
3.  **Optimize Search Queries (Bypass API limits):**
    *   Encourage users to search for data assets through the **DataZone Web Portal** (which uses free control plane APIs) rather than writing custom scripts that poll the search APIs programmatically.
4.  **Consolidate Data Projects:**
    *   Keep your DataZone domain organized. Avoid creating separate "projects" for every developer. Consolidate developers under shared project domains to minimize duplicate metadata storage overheads.
