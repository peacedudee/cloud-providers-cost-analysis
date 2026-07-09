# AWS Service Cost Research: Amazon Route 53

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service. It connects user requests to infrastructure running in AWS (like EC2 instances, Elastic Load Balancers, or S3 buckets) and can also route users to external, non-AWS infrastructure. While DNS query fees appear minor, Route 53 hosted zones, traffic policies, and especially hybrid resolver endpoints can accumulate substantial base fees if not optimized.

---

## 2. Billing Mechanics
Route 53 bills monthly based on the following key metrics:
1.  **Hosted Zones:** A flat monthly fee per DNS zone hosted in Route 53 (public or private).
2.  **DNS Query Volume:** Charged per million queries processed, with prices scaling based on the routing policy (standard, latency, geo, or geoproximity).
3.  **Alias Queries (The Free Tier):** Queries mapped to specific AWS resources are processed at no charge.
4.  **Route 53 Resolver Endpoints:** Hourly fees per IP interface for hybrid (on-premises to cloud) DNS resolution.
5.  **Health Checks:** Monthly fees per active endpoint health check.

---

## 3. Key Cost Dimensions

### A. Hosted Zones (Flat Fees)
*   **Standard Hosted Zone:** **$0.50 per month** for the first 25 hosted zones.
*   **Volume Discount:** **$0.10 per month** for any hosted zone beyond the first 25.
*   *Note: If you delete a hosted zone within 12 hours of creation, the $0.50 monthly fee is waived.*

### B. DNS Queries (us-east-1)
*   **Standard Queries:** **$0.40 per million queries** (first 1 Billion queries/month).
*   **Latency and Geo-Routing Queries:** **$0.70 per million queries**.
*   **Geoproximity Routing Queries:** **$0.80 per million queries**.

### C. Alias Queries (AWS-to-AWS Free Billing)
*   **The Rule:** Route 53 does **not charge** for queries mapped to **Alias records** that point to supported AWS resources:
    *   Elastic Load Balancers (ALBs / NLBs)
    *   Amazon CloudFront distributions
    *   Amazon S3 bucket endpoints
    *   VPC Interface Endpoints
*   *Cost Saver:* Using Alias records instead of standard CNAME records for your endpoints routes traffic to AWS resources for **free**, saving $0.40 per million queries.

### D. Route 53 Resolver Endpoints (The Hybrid DNS Trap)
Used to route DNS queries between on-premises datacenters and AWS VPCs:
*   **The Fee:** **$0.125 per hour** per IP address allocated.
*   **High-Availability Minimum:** A standard, resilient deployment requires a minimum of 2 IP addresses (in 2 separate AZs) per endpoint.
*   **Base Cost:** Running one inbound resolver endpoint pair and one outbound resolver endpoint pair costs:
    $$4\text{ IP interfaces} \times 730\text{ hours} \times \$0.125 = \$365.00\text{ / month flat}$$
    This cost is billed regardless of whether the endpoints process any DNS queries.

### E. Health Checks
*   **AWS Resources:** **$0.50 per month** per active health check.
*   **Non-AWS Resources:** **$0.75 per month** per active health check.
*   *Surcharges:* Enabling advanced features like HTTPS checking, string matching, or fast interval checking (every 10 seconds) adds **$1.00 to $2.00/month** per check.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Monthly / Hourly Rate | Rate per Million Queries | Notes |
|----------------|-----------------------|--------------------------|-------|
| **Hosted Zone (<=25)**| $0.50 / month | N/A | Prorated monthly |
| **Hosted Zone (>25)** | $0.10 / month | N/A | Volume discount tier |
| **Standard Query** | N/A | **$0.40** | First 1 Billion queries |
| **Geo-Routing Query** | N/A | **$0.70** | Latency / Geolocation |
| **Alias Query** | **Free ($0.00)** | **Free ($0.00)** | Points to ELB/CloudFront/S3 |
| **Resolver Endpoint IP**| **$0.125 / hour** | $0.40 (Resolver queries) | ~$91.25/mo per IP |
| **Standard Health Check**| $0.50 / month | N/A | AWS resource check |

---

## 5. AWS Free Tier Coverage
*   **Route 53:** No free tier is available. All hosted zones and queries generate billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Using CNAME Records instead of Alias Records:** Configuring your domain to point to an Application Load Balancer or CloudFront distribution using a standard CNAME record. This forces Route 53 to bill you for every DNS query ($0.40/million) which could have been free under an Alias record.
*   **Orphaned Hosted Zones in Dev Accounts:** Creating a hosted zone for temporary developer testing or branch deployments and leaving the zone active. Each zone costs $0.50/month.
*   **Idle Resolver Endpoints:** Provisioning inbound and outbound resolver endpoints in multiple developer VPCs for hybrid testing and leaving them active, generating **$365.00/month** in base charges per VPC.
*   **Over-Monitoring with Health Checks:** Configuring fast-interval HTTPS health checks on hundreds of endpoints, building up high monthly recurring checking fees.

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate CNAME Records to Alias Records:** Audit your public hosted zones. Convert all standard CNAME records that point to AWS resources (ALBs, S3, CloudFront, PrivateLink) into **Alias (A/AAAA) records**. This instantly reduces DNS query fees to **$0** for those endpoints.
2.  **Consolidate Private Hosted Zones:** If you run multiple VPCs, do not create a separate Private Hosted Zone (PHZ) for each VPC. Create a **single Private Hosted Zone** and associate it with all your VPCs. This avoids paying $0.50/month for duplicate zones.
3.  **Consolidate Hybrid DNS Resolver Endpoints:** Instead of deploying resolver endpoints in every VPC, deploy a single shared pair of Inbound/Outbound Resolver Endpoints in a centralized **Services VPC**. Route DNS query traffic from other VPCs to this central hub using VPC Peering or Transit Gateway. This eliminates the **$365.00/month flat fee** per VPC.
4.  **Prune Unused Hosted Zones:** Set up an automation script to identify hosted zones that have received 0 queries over the past 90 days and delete them.
5.  **Audit Health Check Intervals:** Review active Route 53 health checks. Change checking intervals from fast (10 seconds) to standard (30 seconds) and disable HTTPS checks unless security audit compliance requires it.
