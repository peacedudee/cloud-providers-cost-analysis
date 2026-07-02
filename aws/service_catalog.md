# AWS Service Cost Research: AWS Service Catalog

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Service Catalog allows organizations to create, manage, and govern portfolios of pre-approved cloud resources and IT services. It enables workforce users to provision infrastructure self-service (using CloudFormation or Terraform templates) while ensuring compliance with corporate security and cost policies. Service Catalog is billed on a low per-API call consumption model.

---

## 2. Billing Mechanics
1.  **API Requests:** Billed per API call made to AWS Service Catalog (`$0.0007 per API call`).
2.  **Free Tier:** First 1,000 API calls per month are **100% Free ($0.00)** per account/region.
3.  **No Base Subscription:** No catalog hosting fees, portfolio storage fees, or user license fees.

---

## 3. Key Cost Dimensions

| Feature / Metric | Billing Unit | Rate (us-east-1) | Price for 10,000 API Calls |
|------------------|--------------|------------------|----------------------------|
| **First 1,000 API Calls**| Per account-month | **Free ($0.00)** | **$0.00** |
| **Additional API Calls** | Per API call | **$0.0007** | **$7.00** |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **API Call Rate:** $0.0007 per call ($0.70 per 1,000 calls).
*   **Monthly Free Allowance:** 1,000 calls/month per region.

---

## 5. AWS Free Tier Coverage
*   **AWS Service Catalog Free Tier:** Includes **1,000 free API calls per month** per account/region indefinitely.

---

## 6. Common Cost Hotspots & Pitfalls
*   **High-Frequency Automated Polling Scripts:** Running aggressive CI/CD scripts or custom internal portals that query Service Catalog APIs (`DescribePortfolio`, `ListLaunchPaths`) thousands of times per hour without local caching.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enforce Pre-Approved Cost-Optimized Templates:**
    *   Use Service Catalog to restrict developer deployments to pre-approved CloudFormation templates.
    *   *How:* Lock down template parameters so developers can only launch cost-efficient instance sizes (e.g. `t4g.small` or `c6g.medium`) or mandatory Spot Instance configurations.
    *   **The Savings:** Prevents developers from launching unapproved, expensive instances (`m5.12xlarge`).
2.  **Cache API Calls in Internal Portals:** Implement local in-memory caching (TTL 1 hour) for Service Catalog API responses in internal IT portals to stay within the free 1,000 API calls monthly tier.
