# AWS Service Cost Research: Amazon CloudFront

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon CloudFront is a fast, secure Content Delivery Network (CDN) service that globally distributes data, videos, applications, and APIs to end users with low latency. CloudFront caches content at edge locations worldwide. Because data transfer from AWS origins (such as Amazon S3, EC2, or Application Load Balancers) to CloudFront is **100% Free ($0.00)**, routing public application traffic through CloudFront is almost always cheaper than serving users directly from backend origin servers.

---

## 2. Billing Mechanics
CloudFront offers two primary pricing models:
1. **Pay-As-You-Go (Standard):** Billed based on data transfer egress volume, HTTP/HTTPS request counts, and edge compute execution runtimes.
2. **Flat-Rate Plans:** Bundled monthly subscription tiers combining CDN, WAF, and DNS query routing into a flat monthly charge.

Under the Pay-As-You-Go model, billing is driven by:
* **Outbound Data Transfer (Egress):** Billed per GB of data transferred out from CloudFront edge locations to the internet.
* **Request Fees:** Billed per 10,000 HTTP or HTTPS requests.
* **Origin Shield (Optional):** Request charges for utilizing a centralized regional caching layer.
* **Edge Compute:** Execution runtimes for CloudFront Functions or Lambda@Edge scripts.

---

## 3. Key Cost Dimensions

### A. Data Transfer Out (Egress - us-east-1 Baseline)
* **Internet Egress:** Tiered rates based on edge location geographic region. US/Canada/Europe edge locations offer the lowest rates:
  * First 10 TB/month: **$0.085 per GB** (compared to $0.09/GB for direct S3/EC2 internet egress).
  * Next 40 TB/month: **$0.080 per GB**.
* **Origin Ingress (Free):** Data transferred *from* AWS origin services (S3, EC2, ALB) *to* CloudFront edge locations is **100% Free ($0.00)**.

### B. Price Classes (Geographic Edge Filtering)
Restricting which global edge locations serve your content controls bandwidth costs:
* **Price Class 100:** Uses only low-cost regions (US, Canada, Europe). Most cost-effective tier.
* **Price Class 200:** Includes Price Class 100 plus Asia, Africa, and Latin America.
* **Price Class All:** Uses all global edge locations (including high-cost regions like South America at $0.220/GB).

### C. Request Charges (us-east-1)
* **HTTP Requests:** **$0.0075 per 10,000 requests**.
* **HTTPS Requests:** **$0.0100 per 10,000 requests**.

### D. Origin Shield
Acts as a centralized regional cache layer to minimize origin server load.
* **The Charge:** Billed per request forwarded through the shield (e.g., **$0.0075 per 10,000 requests** in US/Europe).

### E. Edge Compute (CloudFront Functions vs. Lambda@Edge)
* **CloudFront Functions (Lightweight header/URL rewrites):** Billed at **$0.10 per million executions** (no duration charges).
* **Lambda@Edge (Complex compute logic):** Billed at **$0.60 per million requests** plus compute duration charges (**$0.00005001 per GB-second**).

---

## 4. Detailed Pricing Rates (us-east-1 Pay-As-You-Go)

| Cost Component | US / Canada / Europe | Asia / Africa | South America | Australia |
|----------------|----------------------|---------------|---------------|-----------|
| **Egress (First 10 TB)** | **$0.085 / GB** | $0.120 / GB | $0.220 / GB | $0.170 / GB |
| **HTTP Requests (/10K)** | $0.0075 | $0.0090 | $0.0220 | $0.0090 |
| **HTTPS Requests (/10K)** | $0.0100 | $0.0120 | $0.0260 | $0.0125 |
| **Origin Shield (/10K)** | $0.0075 | $0.0090 | $0.0220 | $0.0090 |

---

## 5. AWS Free Tier Coverage
CloudFront features a generous **Always-Free Tier** (not restricted to the first 12 months):
* **Data Transfer:** **1 TB of outbound data transfer** to the internet per month.
* **Request Volume:** **10 million HTTP or HTTPS requests** per month.
* **Edge Compute:** **2 million CloudFront Function executions** per month.

---

## 6. Common Cost Hotspots & Pitfalls
* **Exposing Direct S3 / ALB URLs to Public Traffic:** Serving static assets directly from S3 buckets ($0.09/GB egress) rather than fronting with CloudFront ($0.085/GB egress baseline + 1 TB/mo free tier). Direct access bypasses edge caching and inflates origin S3 GET request charges.
* **Defaulting to Price Class All for Regional Web Apps:** Using all global edge locations for regional web apps serving minor traffic in high-cost regions (e.g., South America egress at $0.220/GB).
* **Using Lambda@Edge for Basic Header Rewrites:** Employing Lambda@Edge ($0.60/M + duration) for simple URL redirects or header injections that could run on CloudFront Functions ($0.10/M).

---

## 7. Actionable Cost Optimization Strategies
1. **Front All Public Assets with CloudFront:** Never expose direct S3 or ALB URLs. Point CloudFront to your S3 bucket or ALB as origin.
   * **Benefits:** Utilizes the **1 TB/mo always-free egress allowance**, eliminates origin egress charges ($0.00 origin-to-CloudFront), and reduces origin server compute.
2. **Restrict Regional Apps to Price Class 100:** For applications serving primarily North American and European users, set distribution configuration to **Price Class 100**.
3. **Migrate Edge Scripts to CloudFront Functions:** Transition lightweight URL redirects, geo-IP headers, and header manipulations from Lambda@Edge to **CloudFront Functions**.
   * **The Savings:** Cuts edge execution fees by **83%** and eliminates duration charges.
4. **Configure Proper Cache-Control Headers:** Set long TTL cache headers (`Cache-Control: max-age=31536000`) for static assets and use cache-busting version filenames (e.g., `app.v2.js`) to deploy updates instead of issuing paid cache invalidations.
