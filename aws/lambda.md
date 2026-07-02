# AWS Service Cost Research: AWS Lambda

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Lambda is a serverless, event-driven compute service that runs code in response to triggers without provisioning or managing servers. You pay only for what you consume, making it highly cost-effective for variable or low-traffic workloads, but potentially expensive if not optimized at high scale.

---

## 2. Billing Mechanics
Lambda charges are calculated using a pure pay-as-you-go billing model based on two primary dimensions:
1.  **Requests:** The number of times your functions are triggered or executed.
2.  **Duration:** The time it takes for your code to execute, measured in milliseconds, multiplied by the memory allocated to the function.

---

## 3. Key Cost Dimensions

### A. Request Charges
*   Billed at a flat rate of **$0.20 per 1 million requests** ($0.0000002 per request) across all architectures.
*   Every trigger—including failed executions, timeouts, or retries—counts as a request.

### B. Duration Charges (Compute GB-seconds)
Duration is calculated from the moment your code begins executing until it returns or terminates, rounded up to the nearest **1 millisecond**.
*   The cost depends on the **allocated memory** (ranging from 128 MB to 10,240 MB). 
*   AWS automatically allocates proportional CPU power when you increase memory (e.g., at 1,769 MB, a function receives the equivalent of 1 full vCPU).
*   **Architecture Choice:**
    *   **x86 Architecture:** Standard compute pricing.
    *   **ARM (Graviton) Architecture:** Billed at a **~20% discount** compared to x86, representing a massive cost-saving lever for compiling/running code on ARM.

### C. Provisioned Concurrency (PC)
For workloads requiring low-latency startup (avoiding "cold starts"), you can pre-warm functions.
*   **Concurrency Fee:** Charged for the amount of concurrency provisioned and the time it is active (billed at **$0.015 per GB-hour** allocated).
*   **Execution Duration Discount:** When running executions inside pre-warmed provisioned environments, the duration charges are slightly lower:
    *   x86 Duration (PC active): $0.0000097222 per GB-second.
    *   ARM Duration (PC active): $0.0000077778 per GB-second.
*   *Note: PC introduces a fixed hourly cost similar to running a VM, which can lead to high idle charges if the provisioned capacity is underutilized.*

### D. Ephemeral Storage (`/tmp` space)
*   Every function includes **512 MB of free ephemeral storage** in the `/tmp` directory.
*   You can configure up to 10 GB of ephemeral storage.
*   Any allocated storage above 512 MB is billed at **$0.0000000309 per GB-second** of execution duration.

### E. Data Transfer
*   Standard data transfer charges apply for egress.
*   Data transfer from Lambda to resources in another Availability Zone (inter-AZ) within the same region or out to the internet is billed at standard EC2 egress rates.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Dimension | x86 Rate | ARM (Graviton) Rate | Notes |
|----------------|----------|---------------------|-------|
| **Requests** | $0.20 per 1M | $0.20 per 1M | Billed for all invocations |
| **Duration (Standard)** | $0.0000166667 / GB-s | $0.0000133334 / GB-s | Billed per ms of execution |
| **Provisioned Concurrency** | $0.015 per GB-hour | $0.015 per GB-hour | Flat fee for keeping environments warm |
| **Duration (with PC)** | $0.0000097222 / GB-s | $0.0000077778 / GB-s | Discounted rate for active PC runs |
| **Additional Storage** | $0.0000000309 / GB-s | $0.0000000309 / GB-s | For `/tmp` allocation above 512 MB |

### Example Monthly Cost Calculation
*Workload: A function with 512 MB memory runs 10 million times a month. Average execution time is 200 ms (0.2s).*

*   **Compute GB-seconds:**
    $$\text{Usage} = 10,000,000 \times 0.2\text{s} \times 0.5\text{GB} = 1,000,000\text{ GB-seconds}$$
*   **x86 Duration Cost:**
    $$1,000,000 \times \$0.0000166667 = \$16.67$$
*   **x86 Request Cost:**
    $$10\text{M} \times \$0.20/\text{M} = \$2.00$$
*   **Total x86 Cost:** **$18.67/month**

*   **ARM (Graviton) Duration Cost:**
    $$1,000,000 \times \$0.0000133334 = \$13.33$$
*   **ARM Request Cost:** **$2.00**
*   **Total ARM Cost:** **$15.33/month** (Save 18%)

---

## 5. AWS Free Tier Coverage
*   **Requests:** 1 million free requests per month (always free, applies to both new and existing accounts).
*   **Duration:** 400,000 GB-seconds of compute time per month (always free).
    *   *Example:* For a 512 MB function, this equals 800,000 seconds of run-time per month.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Over-allocating Memory:** Developers often default to allocating 1024 MB or 2048 MB when the code only requires 128 MB or 256 MB. Since duration pricing scales linearly with memory, this multiplies the cost unnecessarily.
*   **Long-Running or Blocked Code:** Code that waits on external APIs, database connections, or downstream HTTP responses. Lambda charges you for the time your function sits idle waiting for external responses.
*   **High provisioned concurrency with low traffic:** Provisioning high concurrency (e.g., keeping 100 environments warm) for a service that only gets sporadic requests. This creates high "idle" charges.
*   **Infinite Loops / Recursive Triggers:** A Lambda function triggered by an S3 upload that writes another file to the same bucket, triggering itself again. This can generate millions of runs in hours, leading to massive unexpected bills.
*   **SaaS Registry Storage (ECR Container Images):** When deploying Lambda functions as container images, the images are stored in **Amazon ECR** and billed at ECR storage rates ($0.10/GB-month). Keeping large base images and years of historic image versions creates hidden S3/ECR fees.
*   **Queue/Stream Polling API Costs (Event Source Mapping):** When Lambda polls a resource like an **SQS Queue** or **Kinesis Stream**, it executes API calls under the hood (e.g., `ReceiveMessage`). If SQS standard polling is left misconfigured with zero messages in the queue, continuous empty polling checks can accumulate millions of SQS API requests, driving up the SQS portion of the bill.

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate to Graviton (ARM):** Change the architecture setting of your Lambda functions from x86_64 to arm64. Most Node.js, Python, Java, and Go code runs out-of-the-box, providing a **20% direct price reduction**.
2.  **Optimize Memory with Lambda Power Tuning:** Use open-source tools like *AWS Lambda Power Tuning* (which runs your function at various memory levels and compares execution times and costs). Often, allocating *more* memory makes the function run so much faster that the total GB-second cost is actually *lower* than running with less memory.
3.  **Refactor Async Wait Tasks:** Instead of having a Lambda function wait for a long-running external process (like an ETL job or human approval), use **AWS Step Functions** or **event-driven callback patterns** to suspend execution and avoid paying for idle wait time.
4.  **Configure Timeouts Aggressively:** Set short execution timeouts (e.g., 5–10 seconds) on your functions unless they perform batch processing. This prevents run-away loops or blocked threads from running for the maximum 15 minutes and inflating the bill.
5.  **Use Autoscaling for Provisioned Concurrency:** Implement Application Auto Scaling rules to dynamically increase/decrease provisioned concurrency based on scheduled hours or active utilization metrics, minimizing waste during off-peak hours.
