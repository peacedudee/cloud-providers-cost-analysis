# AWS Service Cost Research: Amazon Route 53 Resolver

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Route 53 Resolver (also known as the ".2" resolver) is the default DNS server provided in every Amazon VPC. While standard VPC DNS queries are free, Route 53 Resolver features a hybrid network capability (Inbound and Outbound Resolver Endpoints) to bridge DNS queries between your cloud VPCs and your on-premises datacenters. Because resolver endpoints are billed per network interface hour, they represent a significant fixed cost in hybrid network architectures.

---

## 2. Billing Mechanics
Route 53 Resolver costs are billed on a monthly cycle based on the following dimensions:
1.  **Resolver Endpoint Hourly Fee:** Billed per hour for each IP address (elastic network interface) provisioned for inbound or outbound DNS resolution.
2.  **DNS Resolver Queries:** Billed per million recursive queries processed by the resolver endpoints.
3.  **DNS Firewall Queries (Optional):** Billed per million queries inspected by Route 53 Resolver DNS Firewall rules.

---

## 3. Key Cost Dimensions

### A. Inbound & Outbound Resolver Endpoints (us-east-1 Flat Fees)
Resolver endpoints require Elastic Network Interfaces (ENIs) with assigned IPs in your subnets.
*   **Hourly Base Fee:** **$0.125 per hour** per IP address.
*   **High-Availability Minimum:** AWS requires a minimum of **2 IP addresses** (in different Availability Zones) for each resolver endpoint (Inbound or Outbound).
*   **Endpoint Pair Baseline Cost:**
    *   *Single Inbound Resolver Endpoint:*
        $$2\text{ IP interfaces} \times 730\text{ hours} \times \$0.125 = \$182.50\text{ / month flat}$$
    *   *Resilient Hybrid Setup (Inbound + Outbound Pairs):*
        $$4\text{ IP interfaces} \times 730\text{ hours} \times \$0.125 = \$365.00\text{ / month flat}$$
    *   *Warning:* This base fee accumulates 24/7 even if no DNS queries are forwarded.

### B. DNS Resolver Query Fees
*   **The Charge:** Billed at a rate of **$0.40 per million queries** processed through the resolver endpoints.
*   *Note: Standard internal VPC DNS queries directed to the local default Route 53 Resolver IP (VPC CIDR + 2) are free ($0.00).*

### C. Route 53 Resolver DNS Firewall (Optional Security Layer)
Inspects DNS queries to block access to malicious domains (malware, phishing, botnets).
*   **Query Scanning Rate:** **$0.75 per million queries** inspected.
*   **Domain List Fee:** **$0.0075 per 10,000 domain names** matching in custom lists.
*   *Threat Feeds:* Managed AWS domain lists (like Malware or Botnet domains) are provided at no extra cost, but third-party intelligence feeds incur separate flat monthly subscriptions.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Base Rate | Unit / Details | Monthly Base Cost (Est.) |
|----------------|-----------|----------------|--------------------------|
| **Resolver Endpoint IP**| **$0.125 / hour** | Per interface ENI | ~$91.25 / IP-month |
| **Endpoint Pair (HA)** | **$0.250 / hour** | 2 IPs in 2 AZs | ~$182.50 / endpoint |
| **Inbound + Outbound** | **$0.500 / hour** | 4 IPs total (2 pairs)| **$365.00 / month flat**|
| **Resolver Queries** | **$0.400 / million**| Over resolver endpoint | Query volume |
| **DNS Firewall Scan** | **$0.750 / million**| Queries inspected | Query volume |

---

## 5. AWS Free Tier Coverage
*   **Route 53 Resolver:** No free tier is available. Endpoint hourly base charges begin immediately upon interface creation.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Deploying Endpoints in Every VPC:** Configuring separate inbound and outbound resolver endpoint pairs in multiple regional or developer VPCs. For example, deploying hybrid resolution in 5 VPCs generates:
    $$5\text{ VPCs} \times \$365.00 = \$1,825.00\text{ / month in flat idle charges}$$
*   **Scanning Local VPC DNS Traffic with DNS Firewall:** Configuring DNS Firewall rules to inspect every internal service-to-service lookup in your microservice network. Since internal lookups generate massive query volumes, billing at $0.75/million queries will drive up costs.

---

## 7. Actionable Cost Optimization Strategies
1.  **Centralize Hybrid DNS Resolver Endpoints:**
    *   Do not deploy resolver endpoints in multiple VPCs.
    *   Deploy a **single pair of Inbound Resolver IPs** and a **single pair of Outbound Resolver IPs** in a centralized **Services VPC** (or transit VPC).
    *   Link spoke VPCs to the central hub using VPC Peering or Transit Gateway.
    *   Associate the central Private Hosted Zones and Outbound Rules with the spoke VPCs.
    *   **The Savings:** Consolidating 5 VPC environments down to 1 central resolver hub saves **$1,460.00/month** in base charges.
2.  **Audit DNS Firewall Inspection Targets:**
    *   Configure DNS Firewall to inspect only external (internet-bound) DNS queries.
    *   Do not associate DNS Firewall rule groups with VPCs that handle strictly internal microservice-to-microservice workloads to avoid the $0.75/million query scan tax.
3.  **Adjust DNS Cache TTLs on On-Premises Servers:** For client queries sent from on-premises datacenters to AWS via Inbound Resolver endpoints, configure your local DNS forwarders (e.g. BIND, Active Directory) with long TTL cache settings (e.g., 1 hour or 24 hours). This reduces Inbound Resolver query volumes.
4.  **Delete Inactive Resolver Endpoints:** Audit the Route 53 Resolver console and delete any Outbound Rules or Endpoints that are not associated with active route tables.
