# BigQuery Data Transfer Service Cost Research

Google Cloud BigQuery Data Transfer Service (DTS) automates data movement from SaaS applications (like Google Ads, YouTube, Salesforce) and cloud storage providers directly into BigQuery. While many transfers are free, third-party connectors and downstream storage charges can create unexpected costs if not governed properly.

---

## 1. DTS Billing Mechanics

DTS costs depend on the source of the data:
1. **Google Sources (Free):** Transfers from Google Ads, YouTube Channel Reports, Google Play, Google Merchant Center, and Cloud Storage are **100% free of transfer service charges**.
2. **Third-Party Connectors (Paid):** Transfers from third-party applications (e.g., Salesforce, HubSpot, Jira, Marketo, Salesforce Marketing Cloud) are billed:
   * Per transfer run or as a flat monthly rate per connector instance.
3. **BigQuery Storage & Query Fees:** You pay standard BigQuery storage rates for the tables populated by the transfers, and standard query scan fees when running downstream data processing or analytics.

---

## 2. Core Cost-Reduction Tactics

### A. Audit Third-Party Connector Run Frequency
* **The Waste:** Running a third-party paid connector on an hourly sync schedule, paying transfer run fees for incremental changes when the analytics team only reviews the dashboards once a day.
* **Action:** Adjust the transfer run schedule in the DTS console. Change the frequency from **Hourly** or **Every 4 Hours** to **Daily** (or **Weekly** for historical archive data).

### B. Partition and Cluster Landing Tables
* **The Issue:** DTS writes data to target BigQuery tables. If these landing tables are not optimized, downstream transformations and deduplication scripts must scan the entire table, generating high query fees.
* **Action:** Configure partition and clustering keys on your destination tables. For example, partition by the transfer ingestion date. This ensures downstream queries scan only the latest incremental data partition rather than historical tables.

### C. Reclaim Storage from Unused Transfers
* **Action:** Periodically review active transfer configurations. If a marketing campaign or project has ended, disable the corresponding DTS transfer. Retain the historical data if needed, but stop new ingestion runs to avoid additional storage growth.

---

## 3. BigQuery DTS Audit Checklist

1. [ ] **Connector Frequency Check:** Review schedules for all paid third-party connectors. Adjust run frequencies to daily/weekly where acceptable.
2. [ ] **Destination Optimization:** Verify that landing tables for high-volume transfers are partitioned and clustered by ingestion date or event time.
3. [ ] **Stale Transfer Audit:** Delete or pause transfer configurations for inactive marketing campaigns or retired tools.
4. [ ] **Data Retention Policies:** Apply table expiration policies (e.g., delete staging transfer tables after 30 days) to prevent permanent storage cost accumulation.
