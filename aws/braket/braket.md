# AWS Service Cost Research: Amazon Braket

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Braket is a fully managed quantum computing service designed to help researchers, scientists, and software developers build, test, and execute quantum algorithms. It provides a unified development environment using the open-source Amazon Braket SDK, managed Jupyter Notebooks, and direct execution access to physical Quantum Processing Units (QPUs) from hardware providers (IonQ, Rigetti, Oxford Quantum Circuits, QuEra) as well as managed cloud quantum circuit simulators (SV1, DM1, TN1). Amazon Braket is billed per quantum task and per shot.

---

## 2. Billing Mechanics
Amazon Braket billing consists of two primary categories:
1. **Quantum Processing Unit (QPU) Hardware Tasks:**
   * *Task Request Fee:* Billed flat per quantum task submitted ($0.30 per task).
   * *Shot Execution Fee:* Billed per circuit execution shot ($0.00035 to $0.01 per shot depending on QPU provider).
2. **Quantum Circuit Simulators (SV1 / DM1 / TN1):**
   * Billed per minute of simulator execution time ($0.075 per minute = $4.50 per hour), billed per second with a 3-second minimum.
3. **Braket Managed Notebooks:** Billed at standard Amazon SageMaker notebook instance rates (e.g., `ml.t3.medium` at $0.05/hour).

---

## 3. Key Cost Dimensions

| Quantum Provider | Quantum Technology | Per-Task Request Fee | Per-Shot Execution Fee | Price for 100 Tasks (1,000 Shots Each) |
|------------------|--------------------|----------------------|------------------------|----------------------------------------|
| **Managed Simulator (SV1)** | Cloud Simulator | **$0.00** | **$0.075 / minute** | **$0.38** (5 mins execution) |
| **Rigetti** | Superconducting | **$0.30** | **$0.00035 / shot** | **$65.00** ($30 task + $35 shot) |
| **IonQ** | Trapped Ion | **$0.30** | **$0.01000 / shot** | **$1,030.00** ($30 task + $1,000 shot) |
| **QuEra** | Neutral Atom | **$0.30** | **$0.01000 / shot** | **$1,030.00** ($30 task + $1,000 shot) |

---

## 4. Detailed Pricing Rates (us-east-1)

* **Task Request Rate:** $0.30 per QPU task.
* **Simulator Execution Rate:** $0.075 per minute ($4.50 per hour).
* **Rigetti Shot Rate:** $0.00035 per shot.
* **IonQ / QuEra Shot Rate:** $0.01000 per shot.

---

## 5. AWS Free Tier Coverage
* **Amazon Braket Free Tier:** Includes **1 hour of free simulator execution time per month** (SV1 / DM1 / TN1 combined) indefinitely for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
* **Submitting Single-Shot Quantum Tasks ($0.30 Task Fee Trap):**
  * Submitting 1,000 separate quantum tasks containing 1 shot each instead of batching 1,000 shots into 1 task.
  * *Math:* 1,000 tasks × ($0.30 task fee + $0.01 shot fee) = **$310.00** vs 1 task × $0.30 + 1,000 shots × $0.01 = **$10.30!**

---

## 7. Actionable Cost Optimization Strategies
1. **Batch Circuit Executions into Single Quantum Tasks:**
   * Configure quantum algorithms to request multiple shots (e.g., `shots=1000`) within a single task submission.
   * *Benefit:* Amortizes the $0.30 task request fee across all 1,000 shots.
   * **The Savings:** Slashes task request overhead by **96%**.
2. **Test & Debug Circuits on Local Python SDK Simulators First:**
   * Run circuit logic locally using `LocalSimulator()` in Python ($0.00) or on Braket SV1 ($0.075/min) to verify gates before dispatching to physical QPUs.
   * **The Savings:** Eliminates thousands of dollars in wasted QPU trial executions.
