# Serverless Orchestration & Tooling (Workflows, Tasks, Scheduler, Eventarc, Source Repositories)

This research file covers Google Cloud's serverless orchestration, messaging, scheduling, and source control tools: **Cloud Workflows**, **Cloud Tasks**, **Cloud Scheduler**, **Eventarc**, and **Cloud Source Repositories**. These services operate on high-volume, pay-as-you-go structures. While they have generous free tiers, high-frequency executions and inefficient workflow loops can accumulate significant bills.

---

## 1. Billing Models & Pricing

* **Cloud Workflows:** Billed per step executed. First **5,000 steps per month are free**.
  * Internal steps: $0.01 per 1,000 steps.
  * External steps (HTTP calls to non-GCP endpoints): $0.05 per 1,000 steps.
* **Cloud Tasks:** Billed per million operations (task creation, execution, retries). First **1 million tasks per month are free**, then $0.40 per million.
* **Cloud Scheduler:** Billed per job per month (approx. $0.10 per job). First **3 jobs per billing account are free**.
* **Eventarc:** Billed per 1 million events routed (approx. $0.60 per million events). First **120,000 events per month are free**.
* **Cloud Source Repositories:** First **5 users are free**, then $1.00 per active user per month.

---

## 2. Core Cost-Optimization Levers

### A. Cloud Workflows: Avoid High-Iteration Loops
* **The Cost Trap:** Writing a Workflow that retrieves a list of 100,000 users and loops through them one-by-one inside the Workflow syntax. Each loop iteration represents multiple steps (assigning variables, calling endpoints, comparing values). Executing this single workflow run could generate 500,000 steps, costing $5.00 per run.
* **The Solution:** Offload heavy processing.
* **Action:** Use Workflows strictly as a high-level **orchestrator**. If you need to perform heavy looping, data parsing, or text formatting, write a lightweight Python or Node.js **Cloud Function** to process the array in memory, and invoke it from the workflow in a single step.

### B. Cloud Tasks: Optimize Retry Policies
* **The Waste:** A downstream microservice fails, and Cloud Tasks continues retrying the task 1,000 times on a fast loop. You are billed for every single retry operation.
* **Action:** Configure **Max Retries** and **Backoff Limits** in your queue settings. Limit retries to a maximum of 5–10 attempts with an exponential backoff configuration, and route failed tasks to a Dead Letter Queue (DLQ) for analysis rather than retrying indefinitely.
  ```yaml
  # Sample queue retry configuration
  retryConfig:
    maxAttempts: 5
    maxBackoff: 3600s
    minBackoff: 10s
  ```

### C. Cloud Scheduler & Source Repositories: Prune Active Users & Jobs
* **Action (Source Repositories):** Audit repository access lists. Remove ex-employees, service accounts, and read-only build agents from user access permissions. Every active user beyond the first 5 costs $1.00/month.
* **Action (Cloud Scheduler):** Delete temporary cron jobs used to trigger test alerts or migration scripts. Consolidate cron jobs. Instead of running 5 separate schedulers at identical times to trigger 5 Cloud Functions, run a single scheduler that triggers a Cloud Workflow to execute the tasks in sequence.

---

## 3. Serverless Tooling Audit Checklist

1. [ ] **Workflow Loop Check:** Scan Cloud Workflows definition YAMLs for heavy loops or iterations. Refactor them to execute array processing in Cloud Functions.
2. [ ] **Task Queue Retry Limits:** Audit Cloud Tasks queue configurations and set strict `maxAttempts` limits.
3. [ ] **Scheduler Job Purge:** Delete idle or obsolete Cloud Scheduler cron configurations.
4. [ ] **Source Repo Active Users:** Audit repository access lists and prune ex-employees to stay within the free 5-user tier.
5. [ ] **Eventarc Trigger Review:** Clean up Eventarc triggers when deleting corresponding destination endpoints (like Cloud Run or Cloud Functions) to prevent orphaned events.
