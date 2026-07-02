# Datastream Cost Optimization & Research

Google Cloud Datastream is a serverless, easy-to-use Change Data Capture (CDC) and replication service. It allows you to synchronize data reliably and at scale from databases (MySQL, PostgreSQL, Oracle, SQL Server) to Google Cloud Storage and BigQuery. Datastream billing is purely usage-based, making traffic control and regional placement key cost levers.

---

## 1. Datastream Billing Components

Datastream charges are calculated based on two primary dimensions:
1. **Data Volume Processed (per GB):** Billed based on the gigabytes of data replicated from your source database:
   * Rates vary depending on the database engine (e.g. MySQL/PostgreS rates are lower; Oracle replication rates carry a premium).
2. **Network Egress:** Standard egress rates apply if you transfer data across regions or clouds (e.g. replicating from a MySQL instance in `us-east1` to a BigQuery dataset in `europe-west1`).

---

## 2. Core Cost-Optimization Levers

### A. Filter Out Unneeded Schemas, Tables, and Columns
Datastream replicates everything by default if not configured otherwise.
* **The Waste:** Replicating temporary staging tables, logging tables, or large audit tables containing high-volume writes that have no value for downstream analytics.
* **Action:** In the Datastream connection settings:
  1. Explicitly select only the business-critical tables needed for analytics.
  2. Exclude high-write, temporary, or cache tables.
  3. Exclude large binary (BLOB) or long text columns from replication if they are not used in report queries.
* **The Benefit:** Directly cuts the GB data volume billed by Datastream.

### B. Route Within the Same Region (Zero Inter-Region Egress)
* **The Waste:** Replicating data from a Cloud SQL PostgreSQL instance in `us-central1` to a BigQuery dataset in `us-east4` via Datastream. You pay $0.01 per GB in network egress fees for every byte replicated.
* **Action:** Ensure the Datastream stream, the source database, and the target BigQuery/GCS landing zone are all provisioned in the **exact same region**.

### C. Restrict Development Stream Activity
* **Tactic:** In development and testing, run Datastream only during validation cycles. Disable streams when developers are not actively testing replication logic. Avoid syncing full production database copies to dev/test environments through Datastream.

---

## 3. Datastream Audit Checklist

1. [ ] **Stream Schema Filters:** Audit active streams. Verify that unneeded tables (log tables, cache tables) and binary columns are explicitly excluded.
2. [ ] **Regional Alignment:** Confirm that stream sources, stream instances, and target destinations (GCS/BigQuery) reside in the same region.
3. [ ] **Non-Prod Stream Stoppage:** Ensure development replication streams are paused when not actively in use.
