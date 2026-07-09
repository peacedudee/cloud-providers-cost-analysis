# AWS Service Cost Research: Amazon MWAA (Managed Workflows for Apache Airflow)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon MWAA (Managed Workflows for Apache Airflow) is a managed orchestration service for Apache Airflow that makes it easy to setup, operate, and scale end-to-end data pipelines in Python. MWAA manages Airflow web servers, schedulers, workers, and metadata databases automatically. MWAA environments are stateful and run continuously 24/7; they **cannot be paused or stopped**. MWAA is billed per environment instance-hour, worker auto-scaling hours, and metadata storage.

---

## 2. Billing Mechanics
1. **Environment Base Hourly Fee (Continuous 24/7 Billing):** Billed hourly based on environment size (`mw1.small`, `mw1.medium`, `mw1.large`, `mw1.xlarge`). Environment base fees apply continuously as long as the environment is provisioned.
2. **Worker Auto-Scaling:** Additional worker instances auto-scale during task execution and are billed per worker-hour ($0.055 to $0.44/hr).
3. **Meta-Database Storage:** Billed per GB of database storage ($0.10 per GB-month).
4. **No Free Tier:** MWAA has no permanent free tier.

---

## 3. Key Cost Dimensions

### A. Environment Base Hourly Rates (us-east-1 continuous 24/7)
* **`mw1.small` (1 Scheduler, 2 Workers max base):** **$0.49 per hour** (**$357.70 per month** base).
* **`mw1.medium` (2 Schedulers, 10 Workers max base):** **$0.99 per hour** (**$722.70 per month** base).
* **`mw1.large` (2 Schedulers, 10 Workers max base):** **$1.99 per hour** (**$1,452.70 per month** base).
* **`mw1.xlarge` (2 Schedulers, 10 Workers max base):** **$3.99 per hour** (**$2,912.70 per month** base).

---

## 4. Detailed Pricing Rates (us-east-1)

| MWAA Environment Class | Hourly Rate | Monthly Base Cost (24/7) | Target Workload Volume |
|------------------------|-------------|--------------------------|------------------------|
| **`mw1.small`** | **$0.49** | **$357.70** | Up to 50 DAGs |
| **`mw1.medium`** | **$0.99** | **$722.70** | 50 to 250 DAGs |
| **`mw1.large`** | **$1.99** | **$1,452.70** | 250 to 1,000 DAGs |
| **`mw1.xlarge`** | **$3.99** | **$2,912.70** | 1,000+ DAGs |

---

## 5. AWS Free Tier Coverage
* **Amazon MWAA Free Tier:** None. Continuous base hourly fees ($0.49/hr minimum) apply immediately upon environment creation.

---

## 6. Common Cost Hotspots & Pitfalls
* **Environment Sprawl (Multi-Environment Over-Provisioning):**
  * Provisioning separate MWAA environments for every team, business unit, or developer ($357.70/mo base each).
  * *Math:* 5 small MWAA environments =
    $$5 \times \$357.70\text{ / month} = \mathbf{\$1,788.50\text{ / month base (\$21,462/year)}}$$
    in idle infrastructure fees.
* **Using MWAA for Simple Sequential Workflows:** Deploying a $357.70/mo Airflow cluster for simple daily ETL jobs that run in 2 minutes.

---

## 7. Actionable Cost Optimization Strategies
1. **Consolidate DAGs into Shared Environments:**
   * Consolidate multiple team DAGs into a single shared `mw1.small` or `mw1.medium` MWAA environment.
   * Use Airflow Role-Based Access Control (RBAC) and DAG folder structures to isolate permissions.
   * **The Savings:** Saves $357.70/month for every consolidated environment.
2. **Use AWS Step Functions Express Workflows for Simple Pipelines:**
   * For simple sequential ETL pipelines or microservice orchestration, use **AWS Step Functions Express Workflows ($1.00/M requests)** instead of MWAA.
   * *Why:* Step Functions is 100% serverless and scales down to $0.00 when idle.
   * **The Savings:** Slashes data orchestration costs from $357.70/mo down to under $5.00/mo.
3. **Optimize Airflow Worker Auto-Scaling Ranges:** Set minimum workers (`min-workers`) to 1 on `mw1.small` environments to prevent idle worker fees during periods of low activity.
