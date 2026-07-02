# AWS Service Cost Research: AWS Resilience Hub

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Resilience Hub provides a central place to define, measure, and manage the resilience posture of your applications. It assesses your application architecture against defined Recovery Time Objectives (RTO) and Recovery Point Objectives (RPO), identifies resilience weaknesses (such as missing multi-AZ database replicas or unbacked S3 buckets), and generates automated recovery procedures. Resilience Hub is billed per registered described application per month.

---

## 2. Billing Mechanics
1.  **Described Application Monthly Fee:** Billed monthly per active described application registered in Resilience Hub ($15.00 per application-month, prorated daily).
2.  **Free Trial:** Includes 3 free described applications per account for the first 6 months.
3.  **No Underlying API Fees:** Running resilience assessment evaluations generates no extra per-assessment fees.

---

## 3. Key Cost Dimensions

| Feature / Usage Level | Billed Metric | Rate (us-east-1) | Price for 10 Applications / Month |
|-----------------------|---------------|------------------|-----------------------------------|
| **First 3 Applications (6 Mos)**| Per app-month | **Free ($0.00)** | **$0.00** |
| **Additional Applications** | Per app-month | **$15.00** | **$150.00 / month** |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Described Application Rate:** $15.00 per month ($0.50 per day per application).
*   **Initial Free Allowance:** 3 applications free for 6 months.

---

## 5. AWS Free Tier Coverage
*   **AWS Resilience Hub:** 3 described applications free per month for the first 6 months across your account.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Registering Ephemeral Microservices as Standalone Applications:** Registering every minor developer microservice or temporary staging stack as a separate "described application" in Resilience Hub, generating $15.00/month recurring fees for non-production workloads.

---

## 7. Actionable Cost Optimization Strategies
1.  **Register Only Mission-Critical Production Applications:**
    *   Limit Resilience Hub application registrations to Tier-1 production workloads where explicit RTO/RPO SLA compliance is required.
    *   Do not register non-production or sandbox stacks in Resilience Hub.
    *   **The Savings:** Avoids $15.00/app-month charges on non-critical infrastructure.
2.  **Consolidate Related Microservices into Single Application Groupings:** Group tightly coupled microservices (e.g. frontend API, auth service, and database) under a single combined Resilience Hub application definition rather than registering 5 separate applications ($75/mo vs $15/mo).
