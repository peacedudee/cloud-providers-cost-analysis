# Secret Manager Cost Optimization & Research

Google Cloud Secret Manager provides a secure and convenient storage system for API keys, passwords, certificates, and other sensitive data. While Secret Manager is highly reliable, high-frequency access from applications and keeping a large history of old, unused secret versions can create unnecessary monthly billing.

---

## 1. Secret Manager Billing Components

Secret Manager billing is calculated based on two metrics:
1. **Active Secret Versions:** Billed per active secret version per month (approx. $0.06 per version).
   * **Free Tier:** 6 active secret versions per month at no cost.
   * Billed only for versions in the `Enabled` or `Disabled` state.
   * Versions in the `Destroyed` state do not carry storage charges.
2. **Secret Access Operations (API Calls):** Billed per 10,000 secret read accesses (approx. $0.03 per 10,000 operations).
   * **Free Tier:** 10,000 access operations per month at no cost.

---

## 2. Core Cost-Optimization Levers

### A. Implement In-Memory Caching (Reduce API Access Fees)
* **The Cost Trap:** An application container fetching a database password or API token from Secret Manager on every incoming HTTP request or database query, resulting in millions of API requests per day.
* **The Solution:** Cache secrets in application memory.
* **Action:**
  1. Fetch the secret once during application container startup.
  2. Store the secret value in application memory (e.g. static variable or local memory cache).
  3. Add a Time-To-Live (TTL) to the cache (e.g., refresh the secret once every 1 hour or 24 hours).
* **The Benefit:** Reduces API access requests to Secret Manager by **99.9%**, keeping operations billing at zero.

### B. Automatically Destroy Stale Secret Versions
* **The Waste:** Keeping dozens of historical secret versions active in the console. For example, if you rotate database passwords monthly, you accumulate active versions, all of which bill at $0.06/month.
* **Action:** Implement automation (or manual sweeps) to **Destroy** (not just Disable) old secret versions. Keep only the current active version and the immediately preceding version for fallback.
  ```bash
  # Destroy an old secret version to stop storage charges
  gcloud secrets versions destroy 1 --secret="my-database-password"
  ```

### C. Consolidate Secrets
* **The Waste:** Creating separate secrets for every micro-config variable (e.g., `db_host`, `db_user`, `db_port`, `db_name` as 4 separate secrets = $0.24/month and 4 separate API calls).
* **Action:** Consolidate related configurations into a single JSON-formatted secret payload (e.g., a single secret `db_credentials` containing all fields as a JSON object).

---

## 3. Secret Manager Audit Checklist

1. [ ] **Memory Caching Verification:** Ensure application code caches secrets locally rather than fetching them on every transactional loop.
2. [ ] **Old Version Destruction:** Scan secrets and programmatically destroy old, obsolete secret versions (version numbers older than the current - 1).
3. [ ] **JSON Consolidation:** Group small, related configuration parameters into single JSON secrets to minimize the total active secret version count.
