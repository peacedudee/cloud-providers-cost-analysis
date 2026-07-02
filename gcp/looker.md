# Looker Cost Optimization & Research

Looker is an enterprise business intelligence (BI) and data analytics platform. Looker uses a hybrid billing model consisting of a **flat-rate hosting instance subscription** and **per-user licensing fees**. Because Looker queries the underlying database (typically BigQuery) directly, inefficient dashboard designs and un-cached explores can generate massive database query bills.

---

## 1. Looker Billing Components

Looker costs are driven by:
1. **Platform Subscription:** A flat monthly fee for hosting the Looker instance (Standard, Enterprise, or Embed editions).
2. **User Licenses:** Billed per user per month:
   * **Developer User:** High cost. Permitted to write LookML models.
   * **Standard User:** Medium cost. Permitted to create Explores, dashboards, and schedules.
   * **Viewer User:** Low cost. Permitted only to view existing dashboards and filter reports.
3. **Database Scan Costs:** Billed by the target database (e.g. BigQuery bytes scanned) when users load dashboards or run queries.

---

## 2. Core Cost-Optimization Levers

### A. Establish Strict Looker Caching (Datagroups)
* **The Cost Trap:** When a developer creates a dashboard with 20 tiles, and caching is disabled, every time a user opens or refreshes the page, Looker sends 20 separate queries to BigQuery. If 50 users load this dashboard, BigQuery processes 1,000 queries.
* **The Solution:** Use **Datagroups** to cache query results.
* **Action:** Define Datagroups in LookML and apply them to Explores and dashboards. Cache results for 4–24 hours, or trigger cache updates only when ETL pipelines finish loading new data.
* **The Benefit:** Looker serves subsequent dashboard loads directly from its internal cache, dropping BigQuery scan costs to **$0**.

### B. Prune and Downgrade User Licenses
* **The Waste:** Paying for active Developer or Standard licenses for users who only log in once a month to view static reports, or keeping licenses active for ex-employees.
* **Action:**
  1. Audit Looker's internal "User Activity" reports monthly.
  2. Disable accounts that have not logged in for 30 days.
  3. Downgrade Standard users to **Viewer** licenses if they only consume dashboards and do not build custom reports.

### C. Optimize Persistent Derived Table (PDT) Triggers
PDTs are temporary tables built inside your database to speed up complex queries.
* **The Waste:** Configuring PDTs to rebuild every hour using a time-based trigger (`sql_trigger_value: SELECT CURDATE()`), even if new data is only loaded once a day.
* **Action:** Link PDT triggers to database updates rather than time intervals (e.g. `sql_trigger_value: SELECT MAX(etl_run_id) FROM etl_log`).

---

## 3. Looker Audit Checklist

1. [ ] **Datagroup Cache Verification:** Review LookML models and verify that caching datagroups are applied to all highly-trafficked Explores.
2. [ ] **License Audit:** Deactivate inactive user profiles and downgrade under-utilized accounts to Viewer licenses.
3. [ ] **PDT Trigger Optimization:** Confirm PDT rebuild triggers align with actual ETL load schedules to prevent redundant database writes.
4. [ ] **Dashboard Row/Tile Optimization:** Limit the number of tiles per dashboard (ideally < 15) to prevent browser bottlenecks and query storms.
