# AWS Service Cost Research: AWS Mainframe Modernization

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Mainframe Modernization is a managed service that helps enterprise customers migrate, modernize, run, and operate legacy mainframe workloads (written in COBOL, PL/I, JCL, or Assembler) on AWS. It supports two primary modernization pathways:
1.  **Automated Refactoring (via AWS Blu Age):** Converts legacy COBOL/PL/I code into cloud-native Java and Spring Boot applications.
2.  **Replatforming (via Micro Focus / Enterprise Server):** Emulates and rehosts existing COBOL code in a managed AWS runtime environment with minimal code changes.

---

## 2. Billing Mechanics
1.  **Managed Runtime Environment Hourly Fee:** Billed hourly per provisioned runtime environment instance based on compute class (`m2.m5.large`, `m2.m5.xlarge`, `m2.m5.2xlarge`).
2.  **Mainframe Refactoring Tooling & Runtime:** Software licenses for Blu Age runtime or Micro Focus enterprise engines are included in the hourly runtime fee.
3.  **Underlying Storage & Database:** Billed at standard rates for provisioned Amazon Aurora, PostgreSQL, EFS, or S3 storage attached to the runtime.

---

## 3. Key Cost Dimensions

### A. Managed Runtime Environment Rates (us-east-1)
*   **`m2.m5.large` Runtime (2 vCPU, 8 GB RAM):** **$0.895 per hour** (**$653.35 per month** base).
*   **`m2.m5.xlarge` Runtime (4 vCPU, 16 GB RAM):** **$1.790 per hour** (**$1,306.70 per month** base).
*   **`m2.m5.2xlarge` Runtime (8 vCPU, 32 GB RAM):** **$3.580 per hour** (**$2,613.40 per month** base).
*   **`m2.m5.4xlarge` Runtime (16 vCPU, 64 GB RAM):** **$7.160 per hour** ($5,226.80 per month base).

---

## 4. Detailed Pricing Rates (us-east-1)

| Runtime Class | CPU / Memory | Hourly Rate | Monthly Base Cost (24/7) |
|---------------|--------------|-------------|--------------------------|
| **`m2.m5.large`** | 2 vCPU, 8 GB | **$0.895** | **$653.35** |
| **`m2.m5.xlarge`** | 4 vCPU, 16 GB| **$1.790** | **$1,306.70** |
| **`m2.m5.2xlarge`**| 8 vCPU, 32 GB| **$3.580** | **$2,613.40** |
| **`m2.m5.4xlarge`**| 16 vCPU, 64 GB| **$7.160** | **$5,226.80** |

---

## 5. AWS Free Tier Coverage
*   **AWS Mainframe Modernization Free Tier:** None. Continuous managed runtime fees apply upon environment creation.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Running Non-Production Mainframe Runtimes 24/7:**
    *   Keeping `m2.m5.2xlarge` developer/staging mainframe runtime environments ($2,613.40/mo) running 24/7 when developers only test during business hours.
*   **Choosing Replatforming Over Refactoring for Long-Term Systems:** Replatforming legacy COBOL code, preserving dependency on specialized mainframe runtime environments ($653–$5,226/mo base) indefinitely.

---

## 7. Actionable Cost Optimization Strategies
1.  **Refactor COBOL to Cloud-Native Java (AWS Blu Age Pattern):**
    *   Choose the **Automated Refactoring** pattern to convert legacy COBOL into standard Java/Spring Boot code.
    *   *Why:* Once refactored to Java, applications can run on standard Amazon EKS containers, AWS Fargate, or EC2 Graviton (`t4g`) instances.
    *   **The Savings:** Slashes infrastructure fees by **70–90%** compared to specialized mainframe runtimes or legacy IBM MIPS licensing.
2.  **Schedule Non-Production Mainframe Runtime Environment Teardowns:** Use AWS Lambda / SSM State Manager scripts to stop or delete dev/staging `m2` runtime environments during nights and weekends.
    *   **The Savings:** Saves 65% on non-production mainframe runtime costs (~$1,700/mo saved per staging environment).
