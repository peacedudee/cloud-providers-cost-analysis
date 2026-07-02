# Looker Studio Cost Optimization & Research

Looker Studio (formerly Google Data Studio) is an easy-to-use, serverless data visualization tool. Standard Looker Studio is **100% free**, while Looker Studio Pro charges a flat per-user licensing fee. However, because it queries underlying data engines (like BigQuery, Cloud SQL, Spanner) in real-time, inefficiently designed dashboards can trigger thousands of database queries, generating massive downstream billing spikes.

---

## 1. Looker Studio Billing Components

1. **Looker Studio (Standard):** Free. Unlimited dashboards, unlimited sharing.
2. **Looker Studio Pro:** Billed per user per month (adds organizational governance, enterprise sharing, and support).
3. **Data Source Query Fees (The Real Cost):** Every filter change, date range adjustment, or page swap in a Looker Studio report sends SQL queries directly to your data source. In BigQuery, this is billed at **$5.00 per TB scanned** unless cached.

---

## 2. Core Cost-Optimization Levers

### A. Deploy BigQuery BI Engine
BigQuery BI Engine is a fast, in-memory analysis service that integrates natively with Looker Studio.
* **Action:** Allocate a small memory reservation (e.g., 2 GB to 10 GB) of BI Engine capacity in your Google Cloud console for the BigQuery projects/datasets queried by your dashboards.
* **The Benefit:** BI Engine caches data in memory. Looker Studio queries that hit the BI Engine cache are processed in milliseconds and **charge $0 for data scans**, completely eliminating query costs for active dashboards.

### B. Use the "Extract Data" Connector
* **The Waste:** A dashboard querying a raw 500 GB transaction history table directly, scanning the entire table every time a user refreshes the page or changes a filter.
* **The Solution:** Use **Extract Data**.
* **Action:** In Looker Studio, add a data source using the "Extract Data" connector. Configure it to pull only the columns and rows needed for the dashboard on a daily schedule.
* **The Benefit:** The dashboard queries the static extracted snapshot (stored for free inside Looker Studio), bypassing the live BigQuery table completely and eliminating database query charges.

### C. Enforce Query Caching
* Looker Studio caches query results by default for 15 minutes to 4 hours.
* **Action:** Keep the "Cache" setting enabled under the Data Source connection properties. Instruct users not to click the "Refresh Data" button repeatedly in the report UI, as this forces Looker Studio to bypass the cache and run new, expensive queries in BigQuery.

---

## 3. Looker Studio Audit Checklist

1. [ ] **BI Engine Reservation:** Confirm BI Engine reservations are configured for all BigQuery datasets queried by active Looker Studio dashboards.
2. [ ] **Extract Data Connector Use:** Identify dashboards querying large raw tables and migrate them to the Extract Data connector where real-time sync is not required.
3. [ ] **Cache Setting Check:** Ensure data source cache settings are set to the longest acceptable interval.
4. [ ] **Scheduled Email Reports:** Review scheduled PDF/email delivery of reports. If dashboards are emailed daily to 100 people, ensure they use cached data rather than generating 100 separate raw query runs.
