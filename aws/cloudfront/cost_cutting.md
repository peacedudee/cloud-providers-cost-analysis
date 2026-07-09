# Cost-Cutting Playbook: Amazon CloudFront

> **Companion File:** [cloudfront.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/cloudfront/cloudfront.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon CloudFront is a fast, global Content Delivery Network (CDN). Because CloudFront egress rates ($0.085/GB baseline in US/EU) are lower than direct S3/EC2 internet egress ($0.090/GB), and data transfer *from* AWS origins to CloudFront is **100% FREE ($0.00)**, fronting web applications with CloudFront is inherently cost-effective. However, cost hotspots frequently arise from improper Price Class selection, high-cost regional edge routing, expensive edge compute (Lambda@Edge vs CloudFront Functions), paid cache invalidations, and unoptimized Origin Shield configurations.

This playbook details **16 actionable strategies** across five categories, delivering an estimated **25–60% reduction in CloudFront CDN spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Restrict Distribution Price Class to Price Class 100:** Eliminates expensive South America ($0.220/GB) and Australia ($0.170/GB) edge location surcharges for regional web apps.
2. **Migrate Lightweight Edge Scripts from Lambda@Edge to CloudFront Functions:** Cuts execution costs by **83%** and eliminates duration charges ($0.10/M vs $0.60/M + duration).
3. **Configure Long-TTL Cache Headers (`Cache-Control: max-age=31536000`):** Maximizes edge cache-hit ratio, drastically reducing backend origin requests.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Decommission Disabled or Unused CloudFront Distributions
- **What:** Locate and delete CloudFront distributions that have been disabled or have recorded zero requests over 30 days.
- **Why It Saves Money:** Cleans up administrative overhead, SSL certificate associations, and DNS alias clutter.
- **Implementation Steps:**
  1. List distributions with 0 requests via CloudWatch metric `Requests`.
  2. Disable distribution, wait for status `Deployed`, and delete: `aws cloudfront delete-distribution --id E1234567890`.
- **Estimated Savings:** Administrative hygiene and DNS cost prevention.
- **Risk Level:** Zero risk (for verified zero-traffic distributions).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Traffic verification over 30 days.

#### 2. Replace Paid Invalidations with Versioned Filenames (Cache Busting)
- **What:** Stop issuing blanket paid cache invalidations (`/*`) upon code deployments and adopt versioned asset filenames (e.g. `app.v2.js` or `main.a1b2c3.css`).
- **Why It Saves Money:** The first 1,000 invalidation paths per month are free; additional invalidations cost **$0.005 per path**. Issuing wildcard invalidations across 100,000 files during frequent CI/CD deploys costs **$500.00 per deployment**!
- **Implementation Steps:**
  1. Configure Webpack/Vite build tools to append content hashes to build artifact names.
  2. Set long TTLs (`max-age=31536000`) on hashed static assets.
  3. Cease calling `aws cloudfront create-invalidation` in CI/CD pipelines.
- **Estimated Savings:** 100% of paid invalidation fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Build pipeline support for file hashing.

---

### 2. Architecture Changes

#### 3. Restrict Distribution Scope to Price Class 100 for Regional Applications
- **What:** Change distribution configuration from `Price Class All` to `Price Class 100` (North America, Canada, Europe) for web applications primarily targeting Western users.
- **Why It Saves Money:** High-cost geographic regions bill data egress at significantly higher rates: South America costs **$0.220/GB** (2.5x US rate), Australia costs **$0.170/GB** (2x US rate). Serving traffic in high-cost regions inflates CDN bills unnecessarily.
- **Implementation Steps:**
  1. Analyze viewer geographic distribution in CloudFront console under Reports -> Top Locations.
  2. If South America / Asia / Australia traffic is < 1%, set `PriceClass = PriceClass_100`.
- **Estimated Savings:** 30–60% egress cost reduction for global traffic.
- **Risk Level:** Low (viewers in excluded regions are routed to nearest Price Class 100 edge location with slightly higher latency).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Geographic traffic analysis.

#### 4. Migrate Edge Scripts from Lambda@Edge to CloudFront Functions
- **What:** Refactor lightweight edge logic (URL rewrites, HTTP header manipulation, Geo-IP redirects, CORS headers) from Lambda@Edge to CloudFront Functions.
- **Why It Saves Money:** CloudFront Functions cost **$0.10 per million executions** with ZERO duration fees. Lambda@Edge costs **$0.60 per million requests** plus compute duration ($0.00005/GB-s). CloudFront Functions is **83% cheaper**.
- **Implementation Steps:**
  1. Re-write JavaScript edge logic using lightweight ECMAScript 5.1 CloudFront Functions runtime.
  2. Associate function with viewer-request or viewer-response event trigger.
  3. Remove Lambda@Edge triggers.
- **Estimated Savings:** 83% reduction in edge execution fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Script must execute within 1 ms runtime limit and not access network/body.

#### 5. Front All Public S3 & ALB Endpoints with CloudFront
- **What:** Route all public web traffic through CloudFront rather than serving users directly from S3 bucket URLs or Application Load Balancer DNS.
- **Why It Saves Money:**
  1. Data transfer from S3/ALB origins to CloudFront is **100% FREE ($0.00)**.
  2. CloudFront includes **1 TB/month of free internet egress**.
  3. CloudFront egress ($0.085/GB) is cheaper than direct S3 egress ($0.090/GB).
- **Implementation Steps:**
  1. Create CloudFront distribution with S3/ALB as origin.
  2. Restrict S3 bucket policy using Origin Access Control (OAC).
- **Estimated Savings:** 15–30% net data egress reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Domain DNS routing access.

#### 6. Optimize Origin Shield Placement
- **What:** Audit Origin Shield configuration; disable Origin Shield for single-region origins with high cache hit ratios or select the optimal single regional cache location.
- **Why It Saves Money:** Origin Shield acts as a centralized regional cache, but charges an additional request fee (**$0.0075 per 10,000 requests** in US/EU). If cache hit ratio is already high (> 95%), Origin Shield adds cost without benefit.
- **Implementation Steps:**
  1. Evaluate Origin Shield request metrics vs origin request reduction.
  2. Disable Origin Shield on high-cache-hit static distributions.
- **Estimated Savings:** 100% of Origin Shield request fees ($0.75 per 1 million requests).
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Origin request volume audit.

---

### 3. Storage & Data Tiering / Caching

#### 7. Enforce Long-TTL Cache Headers & Gzip/Brotli Compression
- **What:** Configure origin web servers to output optimal `Cache-Control` headers and enable automatic Brotli/Gzip compression in CloudFront cache behaviors.
- **Why It Saves Money:** 
  1. Automatic compression reduces asset payload sizes by 60–80%, directly dropping outbound gigabytes.
  2. High cache-hit ratios eliminate origin GET request charges ($0.005/1k on S3) and origin compute load.
- **Implementation Steps:**
  1. Set `Cache-Control: public, max-age=31536000, immutable` on static assets.
  2. Enable `Compress Objects Automatically = true` in Cache Behavior settings.
- **Estimated Savings:** 40–70% bandwidth and origin request cost reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Header configuration access on origin.

#### 8. Optimize Cache Key Customization (Minimize Query Strings / Headers)
- **What:** Configure Cache Policies to include only necessary query string parameters, cookies, and HTTP headers in the CloudFront Cache Key.
- **Why It Saves Money:** Forwarding all headers (e.g. `User-Agent`, `Authorization`) or random query parameters fragmentation the edge cache, forcing low cache hit ratios and inflating origin traffic.
- **Implementation Steps:**
  1. Use Managed Cache Policies (`CachingOptimized`).
  2. Create custom cache policy forwarding ONLY specific whitelist query parameters (e.g. `v`, `id`).
- **Estimated Savings:** Increases cache hit ratio from 50% to 95%+, cutting origin fees by half.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application HTTP request review.

---

### 4. Pricing Model Optimization

#### 9. Leverage CloudFront Security Savings Bundle
- **What:** Subscribe to the CloudFront Security Savings Bundle in exchange for a 1-year monthly usage commitment.
- **Why It Saves Money:** Provides up to **30% savings on CloudFront data transfer** and includes AWS WAF usage (up to 10% of commitment value) for free.
- **Implementation Steps:**
  1. Estimate baseline monthly CloudFront egress in Cost Explorer.
  2. Purchase CloudFront Security Savings Bundle commitment in CloudFront console.
- **Estimated Savings:** Up to 30% discount on data transfer + free AWS WAF.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** 1-year baseline spend stability.

#### 10. Maximize Always-Free Tier Egress Allowance (1 TB / Month)
- **What:** Consolidate multiple low-traffic static websites into a single AWS account to fully utilize the 1 TB/month always-free internet egress allowance and 10M free HTTP/HTTPS requests.
- **Why It Saves Money:** Always-Free Tier applies account-wide indefinitely, delivering $0.00 CDN bills for small applications.
- **Implementation Steps:**
  1. Audit multi-account distributions.
- **Estimated Savings:** $85.00/month saved per 1 TB of free tier utilization.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Account architecture review.

#### 11. Negotiate Committed Use Discounts (CUD) for High-Volume Egress (> 10 TB/Month)
- **What:** Contact AWS Sales to negotiate a custom CloudFront Committed Use Discount contract if outbound data transfer exceeds 10 TB/month.
- **Why It Saves Money:** Secures custom pricing rates below standard $0.085/GB tiered pricing.
- **Implementation Steps:**
  1. Aggregate multi-account CloudFront egress volume across Organization.
  2. Engage AWS Account Manager for custom pricing agreement.
- **Estimated Savings:** 30–50% custom enterprise discount.
- **Risk Level:** Low.
- **Implementation Scope:** Procurement / FinOps
- **Prerequisites:** > 10 TB/month global egress volume.

---

### 5. Network & Data Transfer Optimization

#### 12. Eliminate Unnecessary HTTPS Request Surcharges for Static Public Assets
- **What:** Consolidate multiple small static asset domain requests onto HTTP/2 or HTTP/3 single-domain connections.
- **Why It Saves Money:** HTTPS requests cost **$0.0100 per 10,000 requests** (vs HTTP at $0.0075). Multiplexing requests over HTTP/2 reduces total handshake connections.
- **Implementation Steps:**
  1. Enable HTTP/2 and HTTP/3 support on distribution viewer settings.
- **Estimated Savings:** 10–25% reduction in request fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Modern client support.

#### 13. Block Malicious & Bot Traffic with AWS WAF Rate Limiting
- **What:** Attach AWS WAF rules to CloudFront to block scraper bots, DDoS attacks, and high-velocity HTTP request floods at the edge.
- **Why It Saves Money:** Prevents malicious bot attacks from inflating CloudFront request fees ($0.01/10k) and bandwidth egress fees.
- **Implementation Steps:**
  1. Create AWS WAF Web ACL with Rate-Based Rule (e.g. limit 2,000 requests per 5 min per IP).
  2. Associate Web ACL with CloudFront distribution.
- **Estimated Savings:** Protects against $1,000s in unexpected DDoS billing spikes.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** WAF Web ACL configuration.

#### 14. Optimize Field-Level Encryption Usage
- **What:** Apply Field-Level Encryption ONLY to sensitive PII request fields (e.g. credit card numbers) rather than encrypting entire POST request bodies.
- **Why It Saves Money:** Field-Level Encryption carries a surcharge of **$0.02 per 10,000 requests**.
- **Implementation Steps:**
  1. Target specific JSON field parameters in Field-Level Encryption profile.
- **Estimated Savings:** 50–90% reduction in encryption surcharges.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** PII field mapping.

#### 15. Enforce Strict Origin Response Timeouts
- **What:** Reduce Origin Response Timeout from default 30 seconds down to 3–5 seconds.
- **Why It Saves Money:** Prevents CloudFront from holding open viewer connections and accumulating edge runtime state during backend origin database outages.
- **Implementation Steps:**
  1. Set `OriginResponseTimeout = 5` in Origin settings.
- **Estimated Savings:** Prevents cascading edge resource consumption.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Origin SLA review.

#### 16. Restrict Cache Invalidation Paths to Specific File Target Boundaries
- **What:** Train developers to invalidate explicit file paths (e.g. `/images/banner.jpg`) rather than broad subfolder wildcards (`/images/*`) when single files change.
- **Why It Saves Money:** Keeps total path count under the 1,000 free monthly invalidation path limit.
- **Implementation Steps:**
  1. Update deployment automation scripts to pass explicit file lists to `create-invalidation`.
- **Estimated Savings:** Keeps invalidations within the free allowance.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Deployment script update.

---

## Cross-Service Synergies

```
[ Amazon CloudFront CDN ] 
        │
        ├──(Free Origin Data Transfer)──> [ S3 / ALB Origin ] (Saves $0.09/GB direct egress)
        │
        ├──(Bundled Security)───────────> [ AWS WAF ] (Savings Bundle includes WAF credits)
        │
        └──(Edge Compute Refactor)──────> [ CloudFront Functions ] (Saves 83% vs Lambda@Edge)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `CloudFront-Out-Bytes`, `CloudFront-Requests-HTTP-1`, `CloudFront-Requests-HTTPS-Proxy`, `Invalidations`.
- `line_item_resource_id`: CloudFront Distribution ID (`E1234567890`).
- `product_from_location`: Viewer geographic region (`US`, `Europe`, `SouthAmerica`, `Australia`).

### B. CloudWatch Metrics & Diagnostic Reports
- CloudWatch `AWS/CloudFront` Metrics: `Requests`, `BytesDownloaded`, `BytesUploaded`, `4xxErrorRate`, `5xxErrorRate`, `CacheHitRate`.
- CloudFront Reports: Viewer Geography, Top Referrers, Cache Statistics.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "CF-PC-001",
  "service": "CloudFront",
  "category": "Architecture Changes",
  "resource_id": "E1A2B3C4D5E6F7",
  "resource_name": "marketing-static-assets-cdn",
  "account_id": "123456789012",
  "region": "global",
  "current_config": {
    "price_class": "PriceClass_All",
    "monthly_egress_gb": 45000,
    "top_viewer_regions": ["US: 85%", "EU: 12%", "SA: 2%", "AU: 1%"],
    "monthly_cost_usd": 4125.00
  },
  "recommended_config": {
    "price_class": "PriceClass_100",
    "projected_monthly_cost_usd": 3825.00
  },
  "financial_impact": {
    "monthly_savings_usd": 300.00,
    "annual_savings_usd": 3600.00,
    "savings_percentage": 7.3
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "97% of viewers are in US/EU; SA/AU viewers experience minor ~20ms latency addition without egress penalty."
  },
  "implementation": {
    "scope": "Engineer/DevOps",
    "effort_estimate": "15 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Architecture (Price Class / Origin)** | 12 | $14,200.00 | $3,550.00 | 25.0% | Low |
| **Edge Compute (CloudFront Functions)** | 6 | $2,800.00 | $2,324.00 | 83.0% | Low |
| **Pricing Model (Savings Bundle)** | 2 | $18,500.00 | $5,550.00 | 30.0% | Low |
| **Caching Optimization (Headers)** | 15 | $6,400.00 | $3,200.00 | 50.0% | Low |
| **Total** | **35** | **$41,900.00** | **$14,624.00** | **34.9%** | -- |
