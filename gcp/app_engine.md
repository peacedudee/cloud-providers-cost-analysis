# App Engine (GAE) Cost Optimization & Research

Google App Engine (GAE) is a fully managed Platform-as-a-Service (PaaS) model. While GAE was a pioneer in serverless, its pricing structure has rigid scaling rules and legacy components that can make it far more expensive than modern alternatives like Cloud Run. This file covers App Engine cost mechanics, optimization tricks, and migration advice.

---

## 1. Environment Comparison & Billing Mechanics

App Engine operates in two distinct environments with completely different cost characteristics:

### A. Standard Environment
* **Billing Unit:** Billed by **Instance Hours**.
* **Scale-to-Zero:** Yes. If no requests are received, it scales down to 0 instances and you pay $0.
* **Daily Free Quota:** Google provides 28 free instance-hours per day for `F1` instance classes, which is enough to run one small web service 24/7 for free.
* **Pricing Classes:**
  * **Frontend Instances (Automatic Scaling):** F1, F2, F4, F4_1G.
  * **Backend Instances (Basic/Manual Scaling):** B1, B2, B4, B8.
  * *Example:* An `F2` instance class costs exactly 2x the rate of `F1`, and `F4` costs 4x the rate of `F1`.

### B. Flexible Environment
* **Billing Unit:** Billed by the **actual compute resources (vCPU, RAM, Persistent Disk) provisioned**, similar to GCE VMs.
* **Scale-to-Zero:** **No**. App Engine Flex requires a minimum of 1 instance running 24/7 (and Google recommends 2 instances for high availability).
* **Cost Risk:** High. Because it cannot scale to zero, a low-traffic application in App Engine Flex will cost at least $40–$80/month per service, whereas in Standard or Cloud Run it would cost close to $0.

---

## 2. Core Cost-Reduction Tactics

### A. Scale down to Zero in Non-Production
* **App Engine Standard:** Ensure scaling parameters are configured to scale to 0 when idle.
* **App Engine Flexible:** Since Flex cannot scale to 0, **delete the service or stop the version** in staging/development environments when not in use. You can use Cloud Build or custom automation scripts to deploy during working hours and tear it down after hours.

### B. Optimize standard `app.yaml` Scaling Parameters
By default, GAE's automatic scaling can be overly aggressive, keeping idle instances running.
* **Minimize Idle Instances:**
  * In GAE Standard, set `min_idle_instances: 0` (or `1` if you must prevent cold starts).
  * Set `max_idle_instances: 1` or `2`. If this is omitted, App Engine will keep a high buffer of idle instances to handle traffic spikes, which costs money.
* **Tune Scaling Triggers:**
  * Adjust `target_cpu_utilization` (default is 0.6). Raising this to `0.75` or `0.80` will delay the spin-up of new instances, ensuring your existing instances are packed more efficiently.
  * Adjust `target_throughput_utilization` and `max_concurrent_requests` to ensure instances are fully utilized before scaling out.

### C. Downsize Instance Classes
* Developers often scale up from `F1` (600MHz, 256MB RAM) to `F2` or `F4` to resolve out-of-memory (OOM) errors or performance bottlenecks.
* **Action:** Investigate application profiling (e.g. Node.js garbage collection or Java memory leak adjustments) to fit inside `F1` or `F2` shapes, rather than paying 4x GAE instance pricing.

### D. Budget Guardrails: Capping Scale
* Set `max_instances` in the `automatic_scaling` section of your `app.yaml`.
* **Action:** Set a safety cap (e.g., `max_instances: 5`) to prevent runaway scaling costs in case of scraping bots, heavy traffic spikes, or infinite loops.

---

## 3. The Modern Solution: Migration to Cloud Run

The single most effective way to optimize App Engine costs in 2026 is to **migrate to Cloud Run**.

### Why Cloud Run is More Cost-Effective:
1. **True Serverless Pricing:** Cloud Run bills per millisecond of request processing (down to the ms), whereas App Engine Standard bills for instance uptime (where containers remain warm and billable for up to 15 minutes after the last request).
2. **Multi-Concurrency:** A single Cloud Run container can process up to 1,000 concurrent requests. App Engine Standard runtimes are historically single-threaded or low-concurrency, requiring more instances for the same traffic.
3. **No Minimum Cost for Dockerized Apps:** If you are running Docker containers in App Engine Flex, migrating them to Cloud Run immediately eliminates the 24/7 VM billing and enables scale-to-zero.
4. **Flexible Sizing:** Cloud Run allows custom combinations of vCPU and memory. App Engine forces you into rigid class definitions (e.g. F1, F2).

---

## 4. GAE Audit Checklist

1. [ ] **Identify App Engine Flex Workloads:** Audit all App Engine applications to locate "Flexible" services and flag them for immediate migration to Cloud Run.
2. [ ] **Set `max_instances`:** Ensure every Standard application has a `max_instances` safety cap.
3. [ ] **Tune Idle Instances:** Verify `min_idle_instances` is set to `0` and `max_idle_instances` is capped (e.g., to `1` or `2`) in non-production environments.
4. [ ] **Instance Class Auditing:** Identify applications running on `F4` or `F4_1G` classes and profile memory to see if they can be downsized.
5. [ ] **Disable Unused Projects:** Go to App Engine -> Settings -> Disable Application for any legacy or abandoned apps.
