# AWS Service Cost Research: AWS Clean Rooms

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Clean Rooms enables organizations across collaborative sectors (such as advertising, financial services, retail, or healthcare) to analyze and collaborate on collective datasets securely without sharing raw underlying data. Partners run SQL queries, PySpark analytical jobs, or privacy-preserving machine learning models across shared tables while enforcing strict privacy rules. Clean Rooms is serverless, billing based on the compute processing capacity consumed during queries.

---

## 2. Billing Mechanics
AWS Clean Rooms bills monthly based on compute resources consumed by active queries and jobs:
1. **Clean Rooms Processing Units (CRPU-hours):** Billed per second for compute capacity used to execute SQL queries, PySpark analytical jobs, and Clean Rooms ML.
2. **Minimum Billing Windows:** Metered in per-second increments with specific minimum runtimes (60-second minimum for SQL/ML; 10-minute minimum for PySpark).
3. **Member Billing Allocation (Billing Splits):** Collaborations can designate which partner account pays for query execution, allowing compute bills to be routed to specific partner invoices.

---

## 3. Key Cost Dimensions

### A. Clean Rooms Processing Units (us-east-1 CRPU Rates)
* **The Rate:** **$0.027 per CRPU-hour** in `us-east-1`.
* **SQL Queries:** Billed per second with a **60-second minimum** per query execution.
* **PySpark Jobs:** Billed per second with a **10-minute minimum** charge per job execution.
* **Clean Rooms ML:** Billed at $0.027 per CRPU-hour for training and model hosting.

### B. Member Payment Rules (Flex Billing)
* **Default Payment:** The member executing the query pays for the CRPUs consumed.
* **Sponsor Payment Allocation:** Collaborations can be configured so that the **Data Owner** or a dedicated **Services Account** pays for all queries run by partner members.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Component | Compute Rate (per CRPU-hour) | Minimum Billed Duration | Billed Unit |
|-------------------|------------------------------|-------------------------|-------------|
| **SQL Queries** | **$0.0270** | **60 seconds** | Per-second execution |
| **PySpark Jobs** | **$0.0270** | **10 minutes** | Per-second execution |
| **Clean Rooms ML** | **$0.0270** | **60 seconds** | Model training & hosting |

---

## 5. AWS Free Tier Coverage
* **AWS Clean Rooms:** No free tier available. All queries and collaborative jobs generate immediate usage billing.

---

## 6. Common Cost Hotspots & Pitfalls
* **Unoptimized Multi-Billion Row Joins:** Executing queries across unindexed partner tables without filtering clauses, forcing Clean Rooms to spin up large CRPU clusters.
* **Frequent Short PySpark Jobs (10-Minute Minimum Trap):** Scheduling PySpark data prep scripts to execute every 2 minutes. The **10-minute minimum** billing block multiplies PySpark charges 5x.
* **Uncapped Partner Billing Splits:** Configuring a billing split where your account sponsors partner queries without setting strict CloudWatch budget caps.

---

## 7. Actionable Cost Optimization Strategies
1. **Configure Strict Query Constraints (Query Rules):** Set rules limiting allowed SQL operations (e.g., restricting to list intersections only) and blocking unindexed full-table scans.
2. **Batch PySpark Jobs to Avoid Minimum Run Penalties:** Batch analytical workloads in S3 and run PySpark jobs at longer intervals (e.g., daily or weekly) to bypass the 10-minute minimum charge penalty.
3. **Utilize Intermediate Tables for Cached Multi-Step Reports:** Write intermediate query outputs to a **Clean Rooms Intermediate Table** so downstream analytical queries read cached data rather than re-executing expensive joins.
4. **Set Up CloudWatch Budget Alarms on `CRPUConsumed`:** Configure CloudWatch alarms on the `CRPUConsumed` metric to automatically alert or suspend query permissions if a collaboration's cumulative usage breaches monthly budgets.
