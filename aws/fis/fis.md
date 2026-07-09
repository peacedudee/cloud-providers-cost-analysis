# AWS Service Cost Research: AWS Fault Injection Service (FIS)

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Fault Injection Service (FIS) is a fully managed chaos engineering service for running controlled fault injection experiments on AWS workloads. It enables engineering teams to test application resiliency by injecting real-world faults—such as high CPU load, memory exhaustion, network latency, API throttling, EC2 instance terminations, or AZ outages. FIS is serverless, billing based on experiment action execution minutes.

---

## 2. Billing Mechanics
AWS FIS billing is calculated monthly based on experiment runtime:
1.  **Action Execution Duration:** Billed per minute that each experiment action is active ($0.10 per action-minute).
2.  **Multi-Account Target Accounts:** Charges apply per action-minute for each targeted account in multi-account experiments.
3.  **Generated Experiment Reports:** Billed per experiment summary report generated ($5.00 per report).

---

## 3. Key Cost Dimensions

### A. Action Execution Pricing (us-east-1)
*   **The Rate:** **$0.10 per action-minute** (prorated by the second).
*   **Experiment Report:** **$5.00 per report**.

---

## 4. Detailed Pricing Rates (us-east-1)

| FIS Component | Billing Unit | Rate (us-east-1) | Price for 1-Hour Experiment Run |
|---------------|--------------|------------------|---------------------------------|
| **Action Execution** | Per action-minute | **$0.10** | **$6.00** (per 60-min action) |
| **Experiment Report**| Per report | **$5.00** | $5.00 flat |

### Example Monthly Cost Calculation
*Workload: An engineering team runs chaos experiments twice a week (8 times per month). Each experiment injects 3 concurrent faults (actions): an EC2 CPU stress test, an API throttle, and a network latency injector. Each experiment runs for exactly 15 minutes. The team generates 1 summary report per run.*

*   **Action Minutes consumed per Experiment:**
    $$\text{Action Minutes} = 3\text{ actions} \times 15\text{ minutes} = 45\text{ action-minutes}$$
*   **Hourly Action Cost per Experiment:**
    $$\text{Action Cost} = 45 \times \$0.10 = \$4.50$$
*   **Monthly Action Cost (8 runs):**
    $$\text{Monthly Action Cost} = \$4.50 \times 8 = \$36.00$$
*   **Monthly Report Cost (8 reports):**
    $$\text{Monthly Report Cost} = 8\text{ reports} \times \$5.00 = \$40.00$$
*   **Total Monthly Spend:** **$76.00/month** (The generated reports contribute 53% of the total monthly spend).

---

## 5. AWS Free Tier Coverage
*   **AWS Fault Injection Service:** **No free tier** is available. Any executed experiment action incurs $0.10/action-minute charges.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Running Continuous Long-Duration Chaos Experiments:** Configuring an FIS experiment action to run continuously for 24 hours (1,440 minutes) during background load testing.
    *   *Math:* $0.10 \times 1,440\text{ minutes} = \mathbf{\$144.00\text{ per single experiment action run!}}$
*   **Generating Unnecessary Experiment Reports:** Automatically requesting $5.00 generated experiment reports for minor 2-minute test runs.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enforce Short Action Durations (5 to 15 Minutes):**
    *   Configure experiment actions with tight duration limits (e.g., 5 to 15 minutes). 5 to 15 minutes is more than enough time to test whether auto-scaling groups or health checks detect and recover from faults.
    *   **The Savings:** Slashes experiment execution costs to **$0.50 to $1.50 per run**.
2.  **Set Up Automated Stop Conditions (Safeguards):**
    *   Configure **Stop Conditions** in FIS attached to CloudWatch Alarms (e.g., stop experiment if application latency exceeds 5,000ms).
    *   **The Savings:** Prevents prolonged fault injection and limits billable action-minutes if an experiment goes wrong.
3.  **Skip Generated Reports for Non-Production Tests:**
    *   Rely on CloudWatch dashboards ($0.00) or open-source visualization tools to inspect experiment metrics during routine testing instead of generating $5.00 FIS reports.
