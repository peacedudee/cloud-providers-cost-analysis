# AWS Service Cost Research: AWS Fault Injection Service (FIS)

> **Status:** ✅ Research Complete
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
*   *Math Example:* If an experiment runs 2 concurrent actions (e.g., CPU stress + network latency) for 15 minutes:
    $$\text{Actions: } 2\text{ actions} \times 15\text{ minutes} = 30\text{ action-minutes}$$
    $$30 \times \$0.10 = \$3.00\text{ for the experiment run}$$
*   **Generated Experiment Report:** **$5.00 per report**.

---

## 4. Detailed Pricing Rates (us-east-1)

| FIS Component | Billing Unit | Rate (us-east-1) | Price for 1-Hour Experiment Run |
|---------------|--------------|------------------|---------------------------------|
| **Action Execution** | Per action-minute | **$0.10** | **$6.00** (per 60-min action) |
| **Experiment Report**| Per report | **$5.00** | $5.00 flat |

---

## 5. AWS Free Tier Coverage
*   **AWS Fault Injection Service:** **No free tier** is available. Any executed experiment action incurs $0.10/action-minute charges.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Running Continuous Long-Duration Chaos Experiments:**
    *   Configuring an FIS experiment action to run continuously for 24 hours (1,440 minutes) during background load testing.
    *   *Math:* $0.10 \times 1,440\text{ minutes} = \mathbf{\$144.00\text{ per single experiment action run!}}$
*   **Generating Unnecessary Experiment Reports:** Automatically requesting $5.00 generated experiment reports for minor 2-minute test runs.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enforce Short Action Durations (5 to 15 Minutes):**
    *   Configure experiment actions with tight duration limits (e.g., 5 to 15 minutes).
    *   *Why:* 5 to 15 minutes is more than enough time to test whether auto-scaling groups or health checks detect and recover from faults.
    *   **The Savings:** Slashes experiment execution costs to **$0.50 to $1.50 per run**.
2.  **Set Up Automated Stop Conditions (Safeguards):**
    *   Configure **Stop Conditions** in FIS attached to CloudWatch Alarms (e.g. stop experiment if application latency exceeds 5,000ms).
    *   **The Savings:** Prevents prolonged fault injection and limits billable action minutes if an experiment goes wrong.
3.  **Skip Generated Reports for Non-Production Tests:** Rely on CloudWatch dashboards ($0.00) to inspect experiment metrics during routine testing instead of generating $5.00 FIS reports.
