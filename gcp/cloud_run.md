# Cloud Run Cost Optimization & Research

Google Cloud Run is a managed compute platform that enables you to run stateless containers in a serverless environment. While it scales down to zero automatically, misconfigurations in concurrency, CPU allocation, and scaling limits can result in surprisingly high bills. This file outlines key cost-reduction tactics for Cloud Run.

---

## 1. Cloud Run Billing Models & CPU Allocation

Choosing the right CPU allocation mode is the single most critical decision for Cloud Run costs:

| Billing Mode | CPU Allocation | Ideal Traffic Pattern | Cost Profile |
| :--- | :--- | :--- | :--- |
| **Request-based** (Default) | Allocated only during request processing | Spiky, low-volume, or unpredictable traffic. | Billed only when requests are active (down to the nearest ms). Zero cost when idle. |
| **Instance-based** | CPU is always allocated | Steady-state, high-volume, or latency-sensitive traffic. | Billed for the entire lifetime of the container instance. Base unit rates for vCPU/RAM are exactly **25% cheaper** than request-based, plus there are no per-request fees, leading to potentially higher savings. |

### Decision Matrix:
* **Use Request-Based if:** Your service receives intermittent traffic with long periods of silence (e.g., internal webhooks, cron jobs, low-traffic APIs).
* **Use Instance-Based if:** Your service has a constant stream of requests. Since the unit price is cheaper, if your container utilization is high, instance-based billing becomes significantly more cost-effective.

---

## 2. Core Cost-Reduction Tactics

### A. Tune Concurrency Settings
Concurrency determines the maximum number of requests that can be sent to a single container instance simultaneously.
* **The Default:** Cloud Run defaults to **80 concurrent requests** per container.
* **The Waste:** If your application is I/O-bound (e.g., Node.js, Go, Python async, or Java reactive, where threads spend most of their time waiting for database queries or API responses), the container CPU will remain largely idle while handling 80 requests.
* **Optimization:**
  * Increase concurrency (up to 250 or more, max is 1000) for I/O-bound services. This allows a single instance to handle more requests, preventing Cloud Run from spinning up new (and expensive) container instances.
  * Reduce concurrency (e.g., to 10-20) only for CPU-bound services (e.g., video transcoding, complex machine learning inference) to avoid resource starvation.

### B. Optimize Min and Max Instances
* **`min-instances` (Cold Start Mitigation):**
  * Setting `min-instances > 0` keeps instances warm even when there is no traffic.
  * **The Cost:** If you use request-based billing, you pay for the vCPU and RAM of these idle warm instances at a **reduced idle rate** (approx. 50-70% discount). If using instance-based billing, you pay the full rate.
  * **Action:** Keep `min-instances = 0` for development, staging, internal tools, and non-latency-sensitive workloads. Use minimal warm instances (e.g., `1`) only for critical, user-facing endpoints.
* **`max-instances` (Cost Guardrail):**
  * By default, Cloud Run scales up to 100 instances (or more). A sudden traffic spike, DDoS attack, or infinite loop in a calling service can quickly spin up hundreds of instances, causing a billing spike.
  * **Action:** Set a strict `max-instances` cap (e.g., `10` or `20`) on all services based on your actual maximum traffic expectations.

### C. Enable Startup CPU Boost
* Startup CPU Boost temporarily allocates extra CPU during container startup (cold start), which is then revoked once the container is ready.
* **The Cost Benefit/Trade-off:** By accelerating container startup, you reduce the time GCR spends booting your app, improving cold start latency. However, **Startup CPU Boost is not free**. You are billed for the extra boosted CPU amount during the startup time plus 10 seconds after. It may increase or decrease your overall bill depending on how much the startup time is reduced.

### D. Region Selection (Tier 1 vs Tier 2)
* Cloud Run has two pricing tiers depending on the region.
* **Tier 1 (Cheaper):** e.g., `us-central1`, `us-east1`, `us-east4`, `us-west1`, `europe-west1`, `asia-east1`, `asia-northeast1`.
* **Tier 2 (approx. 10-15% more expensive):** e.g., `europe-west2` (London), `europe-west3` (Frankfurt), `asia-southeast1` (Singapore).
* **Action:** Whenever data gravity and network latency permit, deploy Cloud Run services in Tier 1 regions.

---

## 3. Hidden & Trap Costs

* **Unauthenticated Inbound Traffic:**
  * Publicly accessible Cloud Run URLs can be scraped or hit by malicious botnets. Every incoming request, even if it returns a `401 Unauthorized` or `404 Not Found`, wakes up the container and incurs request and compute billing.
  * **Action:** Require authentication for all internal microservices using IAM. For public services, deploy a **Cloud Armor** WAF policy in front of your HTTP Load Balancer to filter out bad bots before they reach Cloud Run.
* **Background Thread Activities:**
  * In request-based billing, once a response is returned, the CPU is throttled. If your code schedules background tasks (e.g., sending an email or writing a log asynchronously after returning a response), the tasks will execute extremely slowly or fail, or you will be billed for extended execution time.
  * **Action:** If you must use background tasks, switch to **Instance-based billing** or use a task queue like **Cloud Tasks** to invoke another endpoint.

---

## 4. GCR Audit Checklist

1. [ ] **CPU Mode Check:** Match billing mode (Request-based vs. Instance-based) to the traffic shape of each service.
2. [ ] **Max Instances Cap:** Ensure every service has a non-default `max-instances` limit.
3. [ ] **Dev/Staging Min Instances:** Verify `min-instances = 0` in all non-production environments.
4. [ ] **Concurrency Profiling:** Run load tests to determine the maximum concurrency your container can handle without latency degradation.
5. [ ] **Startup CPU Boost:** Enable for all services with a cold start time greater than 3 seconds.
6. [ ] **Region Verification:** Ensure services are deployed in Tier 1 regions unless regional data storage compliance requires otherwise.
