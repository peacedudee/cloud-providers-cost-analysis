# AWS Service Cost Research: AWS Mainframe Modernization

> **Status:** ✅ Research Complete (Managed Runtime Closed to New Customers)
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Mainframe Modernization helps enterprise customers migrate, modernize, run, and operate legacy mainframe workloads (written in COBOL, PL/I, JCL, or Assembler) on AWS. It supports two primary modernization pathways:
1. **Automated Refactoring (via AWS Blu Age):** Converts legacy COBOL/PL/I code into cloud-native Java and Spring Boot applications.
2. **Replatforming (via Micro Focus / Enterprise Server):** Emulates and rehosts existing COBOL code in an AWS runtime environment with minimal code changes.

* **Managed Runtime Sunset Notice:** The legacy AWS Mainframe Modernization **Managed Runtime Environment is closed to new customers**. For ongoing modernization efforts, AWS provides **AWS Transform for mainframe** and **Self-Managed Runtime** options on standard EC2 infrastructure.

---

## 2. Billing Mechanics
1. **AWS Transform for mainframe Runtime:** Billed per AWS CPU core-hour (**$0.31 per CPU core-hour**) for managed transformation runtime deployments.
2. **Self-Managed Runtimes (EC2 + Software Licensing):**
   * *Compute:* Billed at standard EC2 instance rates (e.g. `m5` or `m6i` instance families).
   * *Software Licensing:* AWS Blu Age or Micro Focus runtime licenses billed per core/instance.
3. **Underlying Storage & Databases:** Billed at standard rates for provisioned Amazon Aurora, PostgreSQL, EFS, or S3 storage attached to the mainframe application.

---

## 3. Key Cost Dimensions

### A. AWS Transform for mainframe Runtime Rates
* **Core Unit Rate:** **$0.31 per CPU core-hour**.
* **Monthly Cost Math (24/7 Runtime):**
  * *2 vCPU Core Environment:* $0.31 x 2 x 730 hrs = **$452.60 / month**.
  * *4 vCPU Core Environment:* $0.31 x 4 x 730 hrs = **$905.20 / month**.
  * *8 vCPU Core Environment:* $0.31 x 8 x 730 hrs = **$1,810.40 / month**.
  * *16 vCPU Core Environment:* $0.31 x 16 x 730 hrs = **$3,620.80 / month**.

### B. Self-Managed EC2 Runtime Pattern
* Compute runs on standard EC2 instances (`m5.large`, `m6i.xlarge`) with optional Savings Plans / Reserved Instance discounts.

---

## 4. Detailed Pricing Rates (us-east-1)

| Deployment Model | Compute Metric | Rate (us-east-1) | Monthly Base Cost (24/7) |
|------------------|----------------|------------------|--------------------------|
| **AWS Transform Core** | Per CPU core-hour | **$0.310** | **$226.30 per core/mo** |
| **Self-Managed EC2 (`m5.large`)** | Per instance-hour | $0.096 (EC2 On-Demand) | $70.08 + License |
| **Self-Managed EC2 (`m6i.2xlarge`)**| Per instance-hour | $0.384 (EC2 On-Demand) | $280.32 + License |

---

## 5. AWS Free Tier Coverage
* **AWS Mainframe Modernization:** No free tier available. Usage of transformation tooling and runtimes is billed based on core hours or underlying EC2 compute.

---

## 6. Common Cost Hotspots & Pitfalls
* **Running Non-Production Mainframe Runtimes 24/7:**
  * Keeping staging mainframe runtime environments running 24/7 when developers only test during business hours.
* **Choosing Replatforming Over Refactoring for Long-Term Systems:** Replatforming legacy COBOL code, preserving dependency on specialized emulation runtime engines indefinitely instead of refactoring to open cloud-native code.

---

## 7. Actionable Cost Optimization Strategies
1. **Refactor COBOL to Cloud-Native Java (AWS Blu Age Pattern):**
   * Choose the **Automated Refactoring** pattern to convert legacy COBOL into standard Java/Spring Boot code.
   * *Why:* Once refactored to Java, applications can run on standard Amazon EKS containers, AWS Fargate, or EC2 Graviton (`t4g`) instances.
   * **The Savings:** Slashes infrastructure fees by **70–90%** compared to specialized mainframe runtimes or legacy IBM MIPS licensing.
2. **Schedule Non-Production Mainframe Runtime Teardowns:** Use AWS Lambda / SSM State Manager scripts to stop or delete dev/staging runtime instances during nights and weekends.
   * **The Savings:** Saves 65% on non-production compute costs.
3. **Apply Compute Savings Plans on Self-Managed EC2 Runtimes:** Commit to 1-year or 3-year Compute Savings Plans for production self-managed EC2 instances running mainframe workloads to achieve up to **72% savings**.
