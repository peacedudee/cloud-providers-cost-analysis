# AWS Service Cost Research: AWS Lambda

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Lambda is a serverless, event-driven compute service that runs code in response to triggers without provisioning or managing servers. You pay only for what you consume, making it highly cost-effective for variable or low-traffic workloads, but potentially expensive if not optimized at high scale.

---

## 2. Billing Mechanics
Lambda charges are calculated using a pure pay-as-you-go billing model based on two primary dimensions:
1. **Requests:** The number of times your functions are triggered or executed.
2. **Duration:** The time it takes for your code to execute, measured in milliseconds (rounded up to 1 ms), multiplied by the memory allocated to the function.
3. **Volume-Tiered Duration:** Duration rates decrease automatically at higher monthly compute volumes (over 6 Billion GB-seconds).

---

## 3. Key Cost Dimensions

### A. Request Charges
* Billed at a flat rate of **$0.20 per 1 million requests** ($0.0000002 per request) across all architectures.
* Every trigger—including failed executions, timeouts, or retries—counts as a request.

### B. Duration Charges (Compute GB-seconds & Tiers)
Duration is calculated from the moment your code begins executing until it returns or terminates, rounded up to the nearest **1 millisecond**.
* Memory allocation ranges from 128 MB to 10,240 MB (10 GB). CPU power scales proportionally (1,769 MB = 1 full vCPU).
* **Architecture Choice & Volume Tiers (us-east-1):**
  * **x86 Architecture:**
    * Tier 1 (First 6 Billion GB-s/month): **$0.0000166667 per GB-s**.
    * Tier 2 (Next 9 Billion GB-s/month): **$0.0000150000 per GB-s**.
    * Tier 3 (Over 15 Billion GB-s/month): **$0.0000133334 per GB-s**.
  * **ARM (Graviton) Architecture (20% Discount):**
    * Tier 1 (First 7.5 Billion GB-s/month): **$0.0000133334 per GB-s**.
    * Tier 2 (Next 11.25 Billion GB-s/month): **$0.0000120001 per GB-s**.
    * Tier 3 (Over 18.75 Billion GB-s/month): **$0.0000106667 per GB-s**.

### C. Provisioned Concurrency (PC)
For workloads requiring low-latency startup (avoiding cold starts), you can pre-warm functions.
* **Concurrency Fee:** Charged for provisioned capacity while active (**$0.015 per GB-hour** allocated).
* **Execution Duration Surcharge/Discount:** Running inside PC pre-warmed environments uses lower duration rates ($0.0000097222/GB-s x86, $0.0000077778/GB-s ARM).

### D. Ephemeral Storage (`/tmp` space)
* Includes **512 MB of free ephemeral storage** in `/tmp`.
* Allocations up to 10 GB above 512 MB are billed at **$0.0000000309 per GB-second**.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Dimension | x86 Tier 1 Rate | ARM (Graviton) Tier 1 Rate | Notes |
|----------------|-----------------|----------------------------|-------|
| **Requests** | $0.20 per 1M | $0.20 per 1M | Billed for all invocations |
| **Duration (Tier 1)** | $0.0000166667 / GB-s | $0.0000133334 / GB-s | First 6B / 7.5B GB-s/mo |
| **Duration (Tier 2)** | $0.0000150000 / GB-s | $0.0000120001 / GB-s | Next 9B / 11.25B GB-s/mo |
| **Duration (Tier 3)** | $0.0000133334 / GB-s | $0.0000106667 / GB-s | Over 15B / 18.75B GB-s/mo |
| **Provisioned Concurrency** | $0.015 per GB-hour | $0.015 per GB-hour | Flat fee for warm instances |
| **Duration (with PC)** | $0.0000097222 / GB-s | $0.0000077778 / GB-s | Rate during active PC runs |
| **Additional Storage** | $0.0000000309 / GB-s | $0.0000000309 / GB-s | For `/tmp` allocation > 512 MB |

### Example Monthly Cost Calculation
*Workload: A function with 512 MB memory runs 10 million times a month. Average execution time is 200 ms (0.2s).*

* **Compute GB-seconds:**
  $$\text{Usage} = 10,000,000 \times 0.2\text{s} \times 0.5\text{GB} = 1,000,000\text{ GB-seconds}$$
* **x86 Duration Cost:**
  $$1,000,000 \times \$0.0000166667 = \$16.67$$
* **x86 Request Cost:**
  $$10\text{M} \times \$0.20/\text{M} = \$2.00$$
* **Total x86 Cost:** **$18.67/month**

* **ARM (Graviton) Duration Cost:**
  $$1,000,000 \times \$0.0000133334 = \$13.33$$
* **ARM Request Cost:** **$2.00**
* **Total ARM Cost:** **$15.33/month** (Saves 18% overall)

---

## 5. AWS Free Tier Coverage
* **Requests:** 1 million free requests per month (always free).
* **Duration:** 400,000 GB-seconds of compute time per month (always free).

---

## 6. Common Cost Hotspots & Pitfalls
* **Over-allocating Memory:** Defaulting to 1024 MB or 2048 MB when code requires 128 MB or 256 MB, multiplying costs unnecessarily.
* **Long-Running or Blocked Code:** Code that waits on external HTTP responses or database locks. Lambda bills for idle wait time.
* **Over-provisioned Provisioned Concurrency:** Keeping high concurrency (e.g. 100 warm instances) for sporadic workloads.
* **Infinite Loops / Recursive Triggers:** S3 upload triggers writing back to the same bucket, generating millions of executions in hours.

---

## 7. Actionable Cost Optimization Strategies
1. **Migrate to Graviton (ARM):** Change function architecture setting to `arm64` for an immediate **20% direct price reduction**.
2. **Optimize Memory with Lambda Power Tuning:** Benchmark execution times across memory settings. Increasing memory often makes functions run faster, resulting in lower total GB-second costs.
3. **Refactor Async Wait Tasks:** Use **AWS Step Functions** or callback patterns instead of idling inside Lambda while waiting for external tasks.
4. **Configure Short Execution Timeouts:** Set aggressive timeouts (5–10s) to prevent runaway loops from hitting the 15-minute max execution limit.
5. **Autoscale Provisioned Concurrency:** Use Application Auto Scaling to adjust provisioned concurrency based on business hours.
