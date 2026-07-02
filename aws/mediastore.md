# AWS Service Cost Research: AWS Elemental MediaStore

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Elemental MediaStore is an AWS storage service optimized for media origination. It provides the high performance, low-latency, and consistent read-after-write consistency required for live video streaming workflows. MediaStore acts as an origin datastore for live video chunks (HLS, DASH), providing predictable low-latency HTTP GET and PUT operations. MediaStore is billed based on storage volume and HTTP request volume.

---

## 2. Billing Mechanics
1.  **Media Storage:** Billed per GB of video segments stored in MediaStore containers ($0.023 per GB-month).
2.  **GET / HEAD Requests:** Billed per 10,000 HTTP read operations ($0.004 per 10,000 requests = $0.40 per 1 Million requests).
3.  **PUT / POST / DELETE Requests:** Billed per 10,000 HTTP write operations ($0.05 per 10,000 requests = $5.00 per 1 Million requests).

---

## 3. Key Cost Dimensions

| MediaStore Component | Billed Metric | Rate (us-east-1) | Price for 100 GB / 1M Requests |
|----------------------|---------------|------------------|--------------------------------|
| **Media Storage** | Per GB-month | **$0.023 / GB-mo** | **$2.30 / month** |
| **GET Requests** | Per 10k requests | **$0.004 / 10k** | **$0.40** |
| **PUT Requests** | Per 10k requests | **$0.050 / 10k** | **$5.00** |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Storage Rate:** $0.023 per GB-month.
*   **GET Request Rate:** $0.004 per 10,000 requests ($0.40 per 1 Million).
*   **PUT Request Rate:** $0.05 per 10,000 requests ($5.00 per 1 Million).

---

## 5. AWS Free Tier Coverage
*   **AWS Elemental MediaStore Free Tier:** Includes **50 GB of storage** and 1,000,000 HTTP requests per month for 12 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Serving Viewers Directly from MediaStore Origin:** Directing viewer video players to fetch HLS segments straight from MediaStore without an edge CDN, generating millions of origin GET requests ($0.40/M) and origin bandwidth fees.

---

## 7. Actionable Cost Optimization Strategies
1.  **Front MediaStore Containers with Amazon CloudFront:**
    *   Route all live streaming playback traffic through **Amazon CloudFront**.
    *   *Why:* CloudFront edge locations cache video segment chunks, serving 99.9% of viewer requests at the edge.
    *   **The Savings:** Slashes MediaStore origin GET request billing by **99.9%**.
2.  **Set Lifecycle Container Policies for Auto-Deletion:**
    *   Define a MediaStore Container Policy to automatically expire and delete video segment files older than 24 hours.
    *   **The Savings:** Prevents old live stream fragments from accumulating in storage.
