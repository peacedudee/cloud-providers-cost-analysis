# AWS Service Cost Research: AWS Schema Conversion Tool (SCT)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Schema Conversion Tool (SCT) is a free downloadable desktop application that automates heterogeneous database migrations. SCT converts existing database schemas (tables, views, indexes, keys, constraints), database code (stored procedures, functions, triggers, packages), and application SQL queries from commercial legacy database engines (Oracle, Microsoft SQL Server, IBM DB2, Teradata) into open-source target engines (Amazon Aurora PostgreSQL, Amazon Aurora MySQL, PostgreSQL, MySQL) or cloud data warehouses (Amazon Redshift).

---

## 2. Billing Mechanics
1.  **Software License:** **100% Free ($0.00)**. Free software download for Windows, macOS, and Linux.
2.  **Conversion Agents:** Free extraction and conversion agents for data warehouse migrations.

---

## 3. Key Cost Dimensions

| Feature / Tool | Billing Metric | Rate (us-east-1) | Price |
|----------------|----------------|------------------|-------|
| **AWS SCT Client App** | Per installation | **Free ($0.00)** | **$0.00** |
| **Schema & Code Conversion**| Per database | **Free ($0.00)** | **$0.00** |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Software Licensing Rate:** $0.00.

---

## 5. AWS Free Tier Coverage
*   **AWS Schema Conversion Tool:** Always 100% free for all users.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Failing to Automate Schema Conversion:** Manually rewriting complex SQL Server or Oracle PL/SQL stored procedures into PostgreSQL PL/pgSQL using external consulting hours, incurring tens of thousands in manual engineering fees when SCT automates 80–90% of conversions for free.

---

## 7. Actionable Cost Optimization Strategies
1.  **Eliminate Commercial Database Licensing Fees ($3,700+ / core):**
    *   Use AWS SCT to convert legacy Oracle or SQL Server schemas to **Amazon Aurora PostgreSQL** or **RDS MySQL**.
    *   *Why:* Commercial database software licenses (SQL Server Enterprise / Oracle Enterprise) cost ~$3,700+ per vCPU core.
    *   **The Savings:** Saves **$100,000+ per database node** in recurring commercial license fees.
2.  **Use SCT Extension Packs for Function Emulation:** Apply SCT Extension Packs to emulate specialized Oracle functions (e.g. `NVL`, `DECODE`, `SYSDATE`) in PostgreSQL automatically without refactoring source code.
