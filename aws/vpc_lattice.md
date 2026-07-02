# AWS Service Cost Research: AWS VPC Lattice

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS VPC Lattice is a fully managed application-layer (Layer 7) service that consistently connects, secures, and monitors communication between your microservices across different VPCs and accounts. It handles HTTP, HTTPS, and gRPC traffic natively, managing load balancing, routing, and authentication (via IAM). Because VPC Lattice separates service routing from the underlying network layer, it is an alternative to deploying complex Transit Gateways or PrivateLink meshes, featuring a usage-based cost structure.

---

## 2. Billing Mechanics
VPC Lattice is billed on a monthly cycle based on three dimensions:
1.  **Service Hourly Fee:** Billed per hour for each active service registered in the VPC Lattice directory.
2.  **Data Processed:** Billed per GB of traffic processed through the VPC Lattice network.
3.  **Request Volume:** Billed per million HTTP/HTTPS/gRPC requests processed.

---

## 3. Key Cost Dimensions

### A. Service Hourly Fee (us-east-1 Tiers)
A "service" in VPC Lattice represents a logical component (e.g. `billing-service`) containing target groups and routing rules.
*   **The Rate:** **$0.025 per hour** (~$18.25/month) per service.
*   *Tiered Discount:* Billed at $0.025/hour for the first 300,000 service-hours in a month, dropping to $0.015/hour at higher usage levels.
*   *Comparison:* A baseline service in VPC Lattice ($18.25/mo) is **50% cheaper** than NAT Gateway ($36.50/mo) and **78% cheaper** than PrivateLink Interface endpoints across 2 AZs ($36.50/mo), making L7 routing highly competitive for multi-VPC microservice clusters.

### B. Data Processed (L7 Routing Tax)
*   **The Rate:** **$0.025 per GB** of data processed.
*   *Comparison:* This rate is **44% cheaper** than NAT Gateway ($0.045/GB), but **25% more expensive** than Transit Gateway ($0.02/GB) or PrivateLink ($0.01/GB).

### C. Request Volume Charges
*   **The Rate:** **$0.10 per million requests** processed.
*   *Note:* Extremely low-cost tier that only becomes a significant factor for ultra-high-throughput transactional APIs.

---

## 4. Detailed Pricing Rates (us-east-1 Baseline)

| Cost Component | Base Rate | Monthly / Volume Basis | Notes |
|----------------|-----------|------------------------|-------|
| **Service Hour** | **$0.0250 / hour** | ~$18.25 / service | First 300,000 service-hours |
| **Data Processed**| **$0.0250 / GB** | Per GB of data routed | Egress charges separate |
| **Request Volume**| **$0.1000 / Million**| HTTP / HTTPS / gRPC requests | First 100 Billion requests |

---

## 5. AWS Free Tier Coverage
VPC Lattice offers a generous **12-month free tier** for new accounts:
*   **Service Hours:** **750 service-hours** per month.
*   **Data Processed:** **10 GB** of data processed per month.
*   **Requests:** **100,000 requests** per month.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Provisioning Stale/Unused Services:** Registering many microservices in the VPC Lattice service directory and leaving them active when the backend targets are stopped, accumulating $0.025/hour per service.
*   **High-Volume Large Payload Transfers:** Routing raw data pipelines (large CSVs, media streams, backups) through VPC Lattice. Since it charges a $0.025/GB processing fee, L7 routing of large file blocks is expensive compared to standard VPC Peering ($0.00/GB).
*   **Leaving Autoscale Testing Active:** Spinning up hundreds of temporary dev/test service registries and forgetting to delete them.

---

## 7. Actionable Cost Optimization Strategies
1.  **Differentiate L7 Microservices from L4 Data Pipes:**
    *   Route lightweight REST, gRPC, and HTTP APIs through **VPC Lattice** to leverage L7 security and low service-hour rates.
    *   Route heavy data transfers, database replication, and bulk log uploads through **VPC Peering** to bypass the $0.025/GB processing tax.
2.  **Delete Inactive Services:** Set up automation scripts to scan your VPC Lattice service directory and delete services that have registered `0` active target endpoints or received zero requests in 14 days.
3.  **Consolidate Services under Shared Domain Routing:** Instead of registering 10 separate minor sub-services:
    *   Register a **single parent service** (e.g. `internal-api`) in VPC Lattice.
    *   Define path-based routing rules (e.g. `/users` goes to TargetGroup-A, `/orders` to TargetGroup-B) inside that single service.
    *   **The Savings:** Consolidating 10 services down to 1 parent service cuts your fixed hourly charges from $182.50/month to **$18.25/month**, a **90% direct savings**.
4.  **Leverage Free Tier in Non-Prod:** Utilize the 750 free service-hour allowance to run non-production service meshes for free by restricting registries to a single service.
