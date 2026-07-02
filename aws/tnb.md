# AWS Service Cost Research: AWS Telco Network Builder (TNB)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Telco Network Builder (TNB) is an AWS orchestration service designed specifically for telecommunications service providers. It automates the deployment, lifecycle management, and scaling of 5G and LTE cellular network functions (such as 5G Core, Radio Access Network - RAN, and IP Multimedia Subsystem - IMS) on AWS infrastructure (EKS, EC2) using standard European Telecommunications Standards Institute (ETSI) NFV standards. TNB translates telecom descriptors (VNFD, NSD) into cloud infrastructure templates. TNB is billed per network function instance-hour.

---

## 2. Billing Mechanics
1.  **Managed Network Function Item (MNFI) Hours:** Billed per hour for each active network function deployed and managed by TNB ($0.035 per MNFI per hour = $0.84 per day per network function).
2.  **API Requests:** First 45,000 API calls per month are 100% free; billed at $0.0025 per API request thereafter.
3.  **Underlying AWS Infrastructure:** Standard EC2, EKS, ELB, and S3 charges apply separately for running cellular workloads.

---

## 3. Key Cost Dimensions

| TNB Component | Free Tier Allowance | Rate (us-east-1) | Price for 100 Active Network Functions |
|---------------|---------------------|------------------|-----------------------------------------|
| **MNFI Instance Hours** | None | **$0.035 / MNFI-hr** | **$2,520.00 / month** ($3.50/hr) |
| **TNB API Calls** | 45,000 calls / month | **$0.0025 / call** | $2.50 per 1,000 extra calls |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **MNFI Hourly Rate:** $0.035 per network function instance-hour ($25.20 per month per MNFI).
*   **API Request Rate:** $0.0025 per call after 45,000 free calls.

---

## 5. AWS Free Tier Coverage
*   **AWS Telco Network Builder Free Tier:** Includes **45,000 free API calls per month** indefinitely for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Leaving Idle Non-Production Test Network Functions Running:** Leaving staging/test 5G Core network function instances active 24/7 when testing is complete ($25.20/mo per MNFI plus underlying EKS/EC2 cluster compute fees).

---

## 7. Actionable Cost Optimization Strategies
1.  **Automate Dynamic Scale-In of Off-Peak Cellular Network Functions:**
    *   Configure TNB lifecycle hooks to automatically scale down non-essential subscriber session network functions during off-peak overnight hours (1:00 AM – 5:00 AM).
    *   **The Savings:** Reduces MNFI billing by **20–30%**.
2.  **Teardown Dev/Test Telecom Topologies Automatically:** Use CI/CD pipeline hooks to terminate test TNB network package deployments at the end of daily validation runs.
