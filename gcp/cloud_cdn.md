# Cloud CDN Cost Optimization & Research

Google Cloud CDN (Content Delivery Network) uses Google's global edge network to serve content close to users. By caching assets at POPs (Points of Presence), you reduce latency and offload traffic from your origin servers. While CDN is an excellent cost-saving tool overall, a low cache hit ratio or unoptimized cache keys can result in high "cache fill" charges and origin compute waste.

---

## 1. Cloud CDN Billing Components

Cloud CDN costs are determined by:
1. **Cache Egress (per GB):** Billed when edge caches deliver assets to users. Egress from CDN edge is significantly cheaper than egress directly from Compute Engine VMs or Cloud Storage buckets to the internet.
2. **Cache Fill (per GB):** Billed when a cache miss occurs and the edge must fetch the asset from the origin (GCS bucket or VM). This incurs origin egress fees and load balancer processing fees.
3. **HTTP/HTTPS Requests:** Billed per million lookups.
4. **Peering Pricing (2026 update):** Google Cloud has adjusted pricing for peering channels (Direct Peering, Carrier Peering, CDN Interconnect). Organizations utilizing customized CDN interconnect paths to third-party CDNs (like Cloudflare or Fastly) must review rate adjustments.

---

## 2. Core Cost-Reduction Tactics

### A. Maximize the Cache Hit Ratio (CHR)
Every time a user requests an asset and GCDN fails to find it (cache miss), it triggers a **cache fill** from your origin. If your CHR is 10%, you are paying for both the storage/egress of GCS/VMs and the CDN, negating any savings. Aim for a CHR of **85% or higher**.
* **Extend Time-To-Live (TTL):** Set aggressive cache lifetimes on static assets (e.g. images, fonts, JS, CSS) using the `Cache-Control` header:
  ```http
  Cache-Control: public, max-age=31536000, immutable
  ```
* **Optimize Cache Keys:**
  * By default, GCDN includes protocol, host, and query strings in the cache key.
  * **The Trap:** If your static asset URLs contain dynamic query parameters (e.g., `/logo.png?session_id=123`), GCDN treats each request as a separate asset, causing 100% cache misses.
  * **Action:** Configure the cache key to **exclude query strings, cookies, or HTTP headers** that do not affect the returned content.

### B. Enable Dynamic Compression (Brotli/Gzip)
* Cloud CDN can automatically compress text-based static content (HTML, CSS, JavaScript) at the edge before sending it to the client.
* **Action:** Enable dynamic compression on the load balancer backend settings.
* **The Benefit:** Brotli/Gzip compression reduces the size of the transferred files by **60% to 80%**, leading to immediate, direct savings on Cache Egress GB pricing.

### C. Optimize Cache Fill Regions
* When cache fills occur, data is transferred from your origin to the CDN edge.
* **Action:** Co-locate your origin (e.g., Cloud Storage bucket or VM instance group) in the region closest to where the bulk of your users reside. If your users are in Europe, but your origin bucket is in `us-central1`, every cache fill will incur intercontinental network charges.

### D. Implement Negative Caching
* If your site experiences broken links (returning 404s) or unimplemented features (returning 501 errors), bots crawling the site will continuously hit your origin servers.
* **Action:** Enable **negative caching** to cache specific supported HTTP error codes (like 404, 301, 405, 501) for a short period (e.g., 120 seconds for 404s) to shield backends from repeated requests. Note: Cloud CDN does not support negative caching for 502 or 503 gateway/server errors.

---

## 3. Cloud CDN Audit Checklist

1. [ ] **Cache Hit Ratio (CHR) Audit:** Review the CHR of all GCDN-enabled backend services in Cloud Monitoring. Flag services with CHR < 80%.
2. [ ] **Cache Key Optimization:** Inspect URL query strings to ensure dynamic values (session IDs, tracking tokens) are excluded from the cache keys.
3. [ ] **Compression Check:** Verify that Dynamic Compression is toggled `On` for all CDN-enabled backends.
4. [ ] **peering Audit (2026 Rates):** If using CDN Interconnect to third-party CDNs, review contract pricing following the May 2026 price changes.
5. [ ] **TTL Verification:** Inspect HTTP response headers of static assets to confirm `Cache-Control` max-age is set appropriately.
