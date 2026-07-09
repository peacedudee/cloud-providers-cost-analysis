# AWS Service Cost Research: AWS Lake Formation

> **Status:** ✅ Research Complete (Governed Tables Deprecated)
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Lake Formation is a managed service that simplifies and automates the creation, securing, and management of data lakes. It consolidates security settings and access control policies (database-, table-, row-, column-, and cell-level permissions) across different AWS analytics engines. Because Lake Formation acts as a governance, data-sharing, and security orchestration layer, the core service itself is **100% free ($0.00)**.
* **Deprecation Notice (Governed Tables):** Support for Lake Formation **Governed Tables officially ended on December 31, 2024**, and associated APIs (transactions and metadata management for Governed Tables) were fully disabled by **February 17, 2025**. AWS recommends using open-source transactional table formats—specifically **Apache Iceberg**, **Apache Hudi**, or **Linux Foundation Delta Lake**—for ACID transactions and automatic compaction on S3 data lakes.

---

## 2. Billing Mechanics
Lake Formation operates on a zero-cost core governance model:
1. **Core Governance, Access Control & Data Sharing:** **100% Free ($0.00)**. There are no hourly charges or per-permission fees for managing fine-grained security policies or sharing datasets across accounts.
2. **Underlying AWS Infrastructure Services:** You pay standard rates for the resources used to store metadata, store data files, and execute queries:
   * *Data Storage:* Amazon S3 standard capacity and request rates.
   * *Data Catalog Metadata:* AWS Glue Data Catalog object and request rates.
   * *Query & Compute Engines:* Amazon Athena, Amazon EMR, Amazon Redshift, or AWS Glue ETL.

---

## 3. Key Cost Dimensions

### A. Fine-Grained Access Control (Row, Column & Cell Filtering)
* When executing queries with cell-level, row-level, or column-level permissions, Lake Formation dynamically filters data for the requesting principal.
* This filtering compute is executed by the underlying querying engine (e.g. Amazon Athena or Redshift Spectrum) and is billed at the standard engine rate (e.g., $5.00/TB scanned in Athena). No extra premium is added by Lake Formation.

### B. Open-Source Transactional Tables (Iceberg / Hudi / Delta)
* For ACID transactions, schema evolution, and time travel, Lake Formation integrates directly with Apache Iceberg tables in the Glue Data Catalog.
* Compaction and file optimization jobs are executed via AWS Glue ETL or Athena compaction jobs, billed at standard Glue DPU rates ($0.44/DPU-hour) or Athena query rates.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Billed Rate (us-east-1) | Details |
|----------------|-------------------------|---------|
| **Core Security & Permissions**| **Free ($0.00)** | Billed at $0.00 |
| **Cross-Account Data Sharing** | **Free ($0.00)** | Billed at $0.00 |
| **Governed Tables Feature** | **Deprecated ($0.00)** | Feature disabled as of Feb 2025 |
| **Glue Data Catalog Storage** | $1.00 / 100,000 objects | First 1M objects free/month |
| **Underlying S3 Storage** | Standard S3 rates | $0.023/GB-month (Standard) |
| **Query Compute** | Standard Athena/Redshift rates | e.g., $5.00/TB scanned |

---

## 5. AWS Free Tier Coverage
* **AWS Lake Formation:** Core service is 100% free indefinitely for all accounts.
* **Glue Data Catalog:** Includes **1 million free objects** and **1 million free requests** per month.

---

## 6. Common Cost Hotspots & Pitfalls
* **Small File Proliferation in S3 Data Lakes:**
  * Writing millions of tiny 10 KB files to S3 data lakes without compaction.
  * *Why:* Increases S3 PUT request fees ($0.005/1,000 requests) and dramatically increases Athena query scanning times and costs.
* **Over-Scanning Unpartitioned Datasets:**
  * Running queries against unpartitioned S3 folders, forcing Athena or Redshift Spectrum to scan the entire dataset for every query instead of using Lake Formation partition filtering.

---

## 7. Actionable Cost Optimization Strategies
1. **Adopt Apache Iceberg for Table Optimization & Compaction:**
   * Migrate legacy or unstructured tables to **Apache Iceberg** format registered in Glue Data Catalog.
   * Use Athena or Glue automated compaction jobs to combine small S3 objects into optimal **128 MB to 512 MB** files.
   * **The Savings:** Cuts S3 PUT request fees and reduces Athena query costs by up to **90%** through effective file pruning.
2. **Enforce Column and Partition Pruning:**
   * Use Lake Formation column-level permissions and Glue partition keys to restrict query scopes.
   * **The Savings:** Ensures query engines scan only relevant columns and partitions, directly reducing per-TB scanned query fees.
3. **Use Standard S3 Lifecycle Policies for Data Archival:**
   * Set lifecycle policies on underlying S3 buckets to transition raw or historical data lake partitions to **S3 Glacier Flexible Archive** ($0.0036/GB-mo) or **Glacier Deep Archive** ($0.00099/GB-mo).
