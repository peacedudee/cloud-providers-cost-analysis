# AWS Service Cost Research: AWS Amplify

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Amplify is a set of purpose-built tools and features that let frontend web and mobile developers quickly build full-stack applications on AWS. Amplify consists of:
* **Amplify Hosting:** A fully managed service for deploying and hosting static sites, SSR React, Vue, Next.js, and mobile web apps.
* **Amplify Gen 2 (Code-first Backend):** A TypeScript-driven developer experience powered by AWS CDK for defining backend capabilities including Auth (Cognito), Data (AppSync / DynamoDB), Storage (S3), and Functions (Lambda).

Amplify billing is divided between hosting usage (build minutes, storage, data served) and underlying backend AWS resources.

---

## 2. Billing Mechanics
1. **Amplify Hosting Build Minutes:** Billed per minute of build container execution during CI/CD deployments ($0.01 per build minute for standard compute).
2. **Hosting Data Storage:** Billed per GB of static assets and SSR application code stored in Amplify ($0.023 per GB-month).
3. **Hosting Data Served (Egress Bandwidth):** Billed per GB of data served to end users ($0.15 per GB served).
4. **Amplify Web Application Firewall (WAF) Integration:** Attaching AWS WAF to an Amplify application incurs a **$15.00 per month per app** management fee (prorated hourly), plus standard AWS WAF rule and request charges.
5. **Backend Services (Gen 1 / Gen 2):** Billed at standard rates for provisioned AWS resources (Cognito, AppSync, DynamoDB, S3, Lambda, IAM).

---

## 3. Key Cost Dimensions

### A. Amplify Hosting Rates (us-east-1)

| Amplify Hosting Feature | Free Tier Allowance (12 Mos) | Rate (us-east-1) | Price for 100 GB Served / 1k Mins |
|-------------------------|------------------------------|------------------|-----------------------------------|
| **Build Minutes (Standard)** | 1,000 build-mins / month | **$0.01 / minute** | **$10.00** |
| **Data Storage** | 5 GB / month | **$0.023 / GB-month** | $2.30 / month |
| **Data Served (Egress)** | 15 GB / month | **$0.15 / GB served** | **$15.00** |
| **Amplify WAF Integration** | N/A | **$15.00 / app-month** | $15.00 fixed + WAF usage |

---

## 4. Detailed Pricing Rates (us-east-1)

* **Build Rate:** $0.01 per build minute for standard build instances. Note that build time charges apply only to actual build execution, excluding instance initialization overhead.
* **Storage Rate:** $0.023 per GB-month.
* **Data Served Rate:** $0.15 per GB served.
* **WAF Integration:** $15.00/month per app feature charge + standard WAF Web ACL costs ($5.00/month per Web ACL + $1.00/month per rule + $0.60 per million requests).

---

## 5. AWS Free Tier Coverage
* **AWS Amplify Free Tier:** Includes **1,000 build minutes per month**, **5 GB storage per month**, and **15 GB data served per month** for 12 months for new AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
* **High Data Egress on Uncached Media Assets:** Hosting large uncompressed images, heavy assets, or video files directly on Amplify Hosting without CloudFront edge caching or S3 offloading, incurring $0.15/GB bandwidth charges on high-traffic sites.
* **Uncached Node Modules & Full Clean Builds:** Running clean `npm install` steps on every commit without caching `node_modules` or `.next/cache`, extending build times from 1.5 minutes up to 8–10 minutes per deployment.
* **Multiple Preview Branches:** Leaving automatic PR preview builds enabled across high-frequency branches, consuming hundreds of build minutes unnecessarily.
* **Unintended WAF Integration Charges:** Enabling AWS WAF protection on development/staging Amplify web apps, incurring the $15.00/app-month integration charge plus Web ACL fees for non-production environments.

---

## 7. Actionable Cost Optimization Strategies
1. **Cache Build Dependencies in `amplify.yml`:**
   * Add `node_modules` and framework build caches (e.g., `.next/cache`, `.nuxt`) to the build settings in `amplify.yml`.
   * *Why:* Reduces build time from 8 minutes down to 1.5 minutes per commit.
   * **The Savings:** Slashes build-minute costs by **up to 80%**.
2. **Offload Heavy Media Assets to Amazon S3 & CloudFront:**
   * Store images, videos, and large downloadable files in an Amazon S3 bucket fronted by CloudFront ($0.085/GB egress baseline with 1 TB free tier) rather than serving them directly via Amplify Hosting ($0.15/GB).
   * **The Savings:** Reduces data served egress costs by **43%**.
3. **Limit Automatic Branch Deployments:** Disable automatic builds for non-essential pull requests or dev branches in Amplify Console settings. Set up build triggers only for main/production branches.
4. **Disable WAF on Staging Apps:** Reserve Amplify WAF integration ($15/month + WAF fees) exclusively for production deployments.
