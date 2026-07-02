# AWS Service Cost Research: AWS Step Functions

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Step Functions is a serverless visual workflow orchestrator that makes it easy to sequence AWS Lambda functions, ECS tasks, and AWS services into complex, resilient applications. Step Functions manages state, checkpoints, retries, and error handling automatically. Step Functions offers two workflow types: **Standard Workflows** (long-running, up to 1 year, audited state history, exactly-once execution) and **Express Workflows** (high-volume, short-duration up to 5 minutes, at-least-once execution).

---

## 2. Billing Mechanics
1.  **Standard Workflows:** Billed per state transition ($0.025 per 1,000 state transitions).
2.  **Express Workflows:** Billed based on the number of execution requests ($1.00 per 1 Million requests) plus execution duration ($0.00001667 per GB-second).
3.  **Free Tier:** First **4,000 state transitions per month** on Standard Workflows are **100% Free ($0.00)**.

---

## 3. Key Cost Dimensions

### A. Standard Workflows (State Transition Model)
*   **The Rate:** **$0.025 per 1,000 state transitions** ($25.00 per 1 Million state transitions).
*   *Math:* A workflow with 10 state steps running 100,000 times/month:
    $$100,000 \times 10\text{ states} = 1,000,000\text{ state transitions}$$
    $$1,000,000 \times \$0.000025 = \mathbf{\$25.00\text{ / month}}$$

### B. Express Workflows (Request & Duration Model)
*   **Execution Requests:** **$1.00 per 1 Million requests**.
*   **Execution Duration:** **$0.00001667 per GB-second** ($0.06 per GB-hour).

---

## 4. Detailed Pricing Rates (us-east-1)

| Step Functions Workflow Type | Billing Unit | Rate (us-east-1) | Price for 1 Million Executions (10-state workflow) |
|------------------------------|--------------|------------------|---------------------------------------------------|
| **Standard Workflow** | Per 1k transitions | **$0.025** | **$250.00** (10M transitions) |
| **Express Workflow** | Per 1M requests + duration | **$1.00 / 1M + duration** | **~$2.50** (99% cheaper!) |

---

## 5. AWS Free Tier Coverage
*   **AWS Step Functions Free Tier:** Includes **4,000 free state transitions per month** on Standard Workflows indefinitely.

---

## 6. Common Cost Hotspots & Pitfalls
*   **High-Volume Data Processing Loops in Standard Workflows (The $75,000 Nightmare):**
    *   Iterating over millions of items (e.g., S3 batch processing or database records) inside a `Choice` or `Map` state loop in a Standard Workflow.
    *   *Math:* Processing 10 Million records/day through a 10-state loop = 100 Million state transitions/day = **$2,500.00/day ($75,000.00/month!)**.

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate High-Volume Data Pipelines to Express Workflows:**
    *   Switch short-duration (<5 minutes) data processing pipelines from Standard Workflows to **Express Workflows**.
    *   *Why:* Express Workflows bill per request ($1.00/M) + duration rather than per state transition ($25.00/M transitions).
    *   **The Savings:** Slashes high-volume workflow orchestration costs by **95–99%** ($2.50 vs $250.00 per 1M executions).
2.  **Use Distributed Map State for Large Datasets:**
    *   When processing large datasets (e.g. S3 files or CSVs), use **Step Functions Distributed Map State** in Express Mode.
    *   It launches parallel child executions in Express mode while maintaining progress visibility in the parent workflow.
3.  **Combine Minor States in Lambda Functions:** Combine trivial state transitions (e.g. 3 consecutive data transformation steps) into a single Lambda function execution to eliminate 2 billable state transitions per run.
