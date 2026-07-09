# AWS Service Cost Research: AWS Cloud9

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Cloud9 is a cloud-based Integrated Development Environment (IDE) that allows developers to write, run, and debug code via a web browser. It includes a code editor, debugger, embedded terminal, and pre-packaged tooling for popular languages (Python, Node.js, Go, Java, C++) and the AWS CLI. The Cloud9 IDE software interface is **100% free**, while underlying compute and storage resources are billed at standard AWS rates.

---

## 2. Billing Mechanics
1. **Cloud9 IDE Application Fee:** **100% Free ($0.00)**. Zero user licensing fees or workspace setup charges.
2. **Underlying EC2 Compute Instance:** Billed hourly per provisioned EC2 instance hosting the environment (e.g., `t4g.small` or `t3.small`).
3. **Underlying EBS Storage Volume:** Billed monthly per GB of EBS storage provisioned for the environment root volume (e.g., 10 GB GP3).
4. **Auto-Hibernation:** When enabled, Cloud9 automatically stops the underlying EC2 instance after 30 minutes of inactivity, pausing compute charges.

---

## 3. Key Cost Dimensions

| Feature / Resource | Billing Metric | Rate (us-east-1) | Monthly Cost (Auto-Hibernate vs 24/7) |
|--------------------|----------------|------------------|---------------------------------------|
| **Cloud9 IDE Service** | Per user | **Free ($0.00)** | **$0.00** |
| **EC2 `t4g.small` (Graviton)** | Per instance-hour | **$0.0168 / hr** | **$1.34 / mo** (40 hrs/wk) vs $12.26 (24/7) |
| **EBS Storage (10 GB GP3)** | Per GB-month | **$0.08 / GB-mo** | **$0.80 / month** |

---

## 4. Detailed Pricing Rates (us-east-1)

* **Cloud9 IDE Interface Rate:** $0.00 per month.
* **`t4g.small` Instance Rate:** $0.0168 per hour (~$12.26/mo if running 24/7; ~$1.34/mo if auto-hibernated).
* **GP3 EBS Volume Rate:** $0.08 per GB-month.

---

## 5. AWS Free Tier Coverage
* **AWS Cloud9:** Free tier applies via standard EC2 and EBS free tier allowances (750 free `t2.micro`/`t3.micro` hours and 30 GB EBS per month for new accounts).

---

## 6. Common Cost Hotspots & Pitfalls
* **Disabling Auto-Hibernation:** Disabling auto-hibernation on Cloud9 environments, leaving underlying EC2 instances running 24/7 ($12.00–$15.00/mo) when developers only work 2 hours a day.

---

## 7. Actionable Cost Optimization Strategies
1. **Enforce 30-Minute Auto-Hibernation (Default):**
   * Ensure the **Auto-Hibernate setting** is set to 30 minutes during environment creation.
   * *Mechanism:* The EC2 instance automatically stops 30 minutes after the browser tab is closed, halting compute billing ($0.00/hr while stopped).
   * **The Savings:** Slashes EC2 compute charges by **85%** ($1.34/mo vs $12.26/mo per developer).
2. **Select ARM Graviton Instances (`t4g.small`):** Choose `t4g.small` ARM instances instead of `t3.small` when creating environments for 20% lower hourly compute rates.
3. **Right-Size EBS Storage Volumes (10 GB):** Allocate 10 GB to 20 GB of GP3 storage for root volumes. Avoid allocating 100+ GB unless heavy dataset processing is required.
