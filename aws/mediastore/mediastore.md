# AWS Service Cost Research: AWS Elemental MediaStore

> **Status:** ❌ Discontinued Service
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Elemental MediaStore was an AWS storage service optimized for media origination. It provided high-performance, low-latency, and consistent read-after-write behavior for live video streaming workflows.
* **Discontinuation Notice:** **AWS Elemental MediaStore was officially DISCONTINUED on November 13, 2025**. The service is no longer operational for new or existing workflows.
* **Migration Paths:** AWS recommends migrating legacy MediaStore video origination workflows to:
  1. **Amazon S3:** Recommended for standard live streaming origination workflows (leveraging S3's strong consistency and high throughput).
  2. **AWS Elemental MediaPackage:** Recommended for advanced video workflows requiring packaging (HLS, DASH, CMAF), DRM, or cross-region failover.

---

## 2. Historical Billing Mechanics (For Audit Reference)
*Before discontinuation, MediaStore was billed on:*
1. **Media Storage:** $0.023 per GB-month.
2. **GET / HEAD Requests:** $0.004 per 10,000 HTTP read requests ($0.40 per 1M).
3. **PUT / POST / DELETE Requests:** $0.05 per 10,000 HTTP write requests ($5.00 per 1M).

---

## 3. Recommended Alternative Services & Rates

| Migration Target | Ideal Use Case | Cost Model | Key Advantage |
|------------------|----------------|------------|---------------|
| **Amazon S3** | Standard video file storage & origination | $0.023/GB-month + request fees | Strong consistency & lower storage cost |
| **AWS Elemental MediaPackage** | Advanced ABR packaging & DRM | $0.05/GB ingest + $0.04/GB egress | Dynamic packaging, DRM, multi-AZ failover |

---

## 4. Migration & Cost Optimization Strategies
1. **Migrate Live Video Origins to Amazon S3:** For basic HLS/DASH segment storage, migrate to Amazon S3 Standard ($0.023/GB-month). Front S3 with Amazon CloudFront to cache video segment reads at the edge.
2. **Migrate Multi-Format Pipelines to MediaPackage:** For streams requiring on-the-fly packaging into multiple formats or DRM encryption, transition to AWS Elemental MediaPackage.
