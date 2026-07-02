# Vertex AI Search & Conversation Cost Optimization

Vertex AI Search and Conversation (formerly Generative AI App Builder) enables developers to build Google-quality search engines and chat assistants over their own enterprise data using LLMs and vector search. Because it integrates document indexing, optical character recognition (OCR), vector embeddings, and LLM orchestration, unchecked datasets and query storms can lead to high billing.

---

## 1. Billing Components & Pricing

Billing is split into several dimensions:
1. **Search Queries:** Billed per 1,000 queries (approx. $1.50 per 1,000 queries for Standard Search, up to $2.00+ for Enterprise Search with LLM summary generation).
2. **Conversation Chat Sessions:** Billed per chat session (or per user query in CX).
3. **Data Ingestion and Indexing:**
   * **Document Parsing/OCR:** Billed per page processed (e.g., extracting text from scanned PDF manuals).
   * **Vector Search Embeddings:** Billed for the underlying embeddings generation (tokens processed).
4. **Underlying LLM Calls (Gemini):** Billed separately for input/output tokens used to synthesize search answers.

---

## 2. Core Cost-Optimization Levers

### A. Pre-filter and Compress Documents before Ingestion
* **The Cost Trap:** Ingesting hundreds of 100 MB scanned PDFs directly into the data store, triggering OCR processing on every page (which bills at high per-page rates) and generating massive vector storage.
* **Action:**
  1. Extract raw text from files locally or using a lightweight serverless function before ingestion. Upload raw text files (`.txt` or `.json`) instead of PDFs.
  2. If PDFs are required, downsample images and remove blank/unneeded pages (like corporate templates, tables of contents, or index sections) prior to ingestion.
* **The Benefit:** Eliminates OCR parsing fees and decreases vector storage and token embedding costs.

### B. Implement Cache Layers for Popular Queries
* **The Waste:** If 1,000 users search for the exact same query (e.g. "company holiday schedule 2026") in your internal portal, and you run a live Enterprise Search on each load, you pay search query fees and LLM synthesis fees 1,000 times.
* **Action:** Implement a cache layer (e.g. Memorystore Redis or Memcached) in your application gateway. Store search query strings and their generated summaries for 4–12 hours.
* **The Benefit:** Bypasses live Vertex AI Search calls for repeated queries, reducing query fees by up to 60%.

### C. Downscale LLM Summarization Templates
* **Action:** In your Search config, when using search summaries, optimize the system instructions to request concise answers. Long-winded summaries from Gemini increase output token charges.
* **Tactic:** Limit search results summaries to maximum token limits (e.g. `maxOutputTokens: 256`).

---

## 3. Vertex AI Search & Conversation Audit Checklist

1. [ ] **Raw Text Ingestion Preference:** Confirm that new document imports utilize pre-extracted raw text files (`.json`, `.txt`) rather than scanned PDFs where possible to bypass OCR costs.
2. [ ] **Summary Query Caching:** Implement cache mechanisms in client applications for highly repeated search terms.
3. [ ] **Token Summary Constraints:** Review system prompts for summarization and enforce constraints on `maxOutputTokens`.
4. [ ] **Stale Data Store Cleanup:** Identify and delete unused data stores and search apps in sandbox and development environments.
