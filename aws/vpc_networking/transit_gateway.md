# AWS Service Cost Research: AWS Transit Gateway

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Transit Gateway acts as a cloud router, connecting multiple Virtual Private Clouds (VPCs), VPN connections, and AWS Direct Connect links together through a central hub. It simplifies network topology by eliminating the need to manage complex mesh VPC Peering networks. However, Transit Gateway is a high-cost service because it bills on two distinct dimensions: a flat hourly fee per connection and a data-processing fee on all traffic passing through it.

---

## 2. Billing Mechanics
Transit Gateway is billed monthly based on the following dimensions:
1.  **Transit Gateway Attachment Fee:** A flat hourly rate billed for every resource connected (attached) to the Transit Gateway.
2.  **Data Processing Fee:** Billed per GB of data processed by the Transit Gateway.
3.  **Peering Charges:** Billed for connecting separate Transit Gateways across AWS regions.

---

## 3. Key Cost Dimensions

### A. Transit Gateway Attachment Fees (us-east-1)
Every network interface connected to the gateway incurs a flat fee:
*   **VPC Attachment:** **$0.05 per hour** (~$36.50/month) per VPC.
*   **VPN Attachment:** **$0.05 per hour** (~$36.50/month) per VPN.
*   **Direct Connect Attachment:** **$0.05 per hour** (~$36.50/month) per link.
*   *Scale Impact:* If you connect 20 VPCs and 5 VPNs to a Transit Gateway, your baseline idle cost is:
    $$25\text{ attachments} \times 730\text{ hours} \times \$0.05 = \$912.50\text{ / month flat}$$
    This is billed even if no network traffic is sent.

### B. Data Processing Fees
*   **The Surcharge:** AWS bills **$0.02 per GB** for all data processed through the Transit Gateway.
*   **Crucial Difference from VPC Peering:** Under standard VPC Peering, data transfer between VPCs in the *same* Availability Zone using private IPs is **free ($0.00/GB)**. Under Transit Gateway, **every gigabyte** sent across VPCs is taxed at **$0.02/GB**, regardless of whether the traffic stays within the same AZ.
*   *High-Volume Processing Pitfall:* Routing high-volume database replication or backup synchronizations (e.g. 50 TB/month) across VPCs via Transit Gateway adds:
    $$50,000\text{ GB} \times \$0.02 = \$1,000.00\text{ / month in processing fees}$$

### C. Transit Gateway Peering
Linking Transit Gateways in different regions (for global multi-VPC routing) incurs:
*   Standard **$0.05 per hour** attachment fee for the peering connection.
*   No Transit Gateway data processing fees are charged on the peering link (data is billed on the VPC attachments).
*   Standard AWS **Inter-Region Data Transfer Egress fees** ($0.01 to $0.02 per GB) apply.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Component | Hourly Attachment Rate | Data Processing Rate (/GB) | Monthly Base Cost (per unit) |
|-------------------|-------------------------|----------------------------|------------------------------|
| **VPC Connection** | **$0.05 / hour** | **$0.0200** | ~$36.50 |
| **VPN Connection** | **$0.05 / hour** | **$0.0200** | ~$36.50 |
| **Direct Connect** | **$0.05 / hour** | **$0.0200** | ~$36.50 |
| **Transit Gateway Peering**| **$0.05 / hour** | Free ($0.00) | ~$36.50 |

---

## 5. AWS Free Tier Coverage
*   **AWS Transit Gateway:** No free tier is available. Any provisioned gateway and attachments generate standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Routing High-Volume Database Syncs / Backups:** Connecting analytical databases (like Redshift, Elasticsearch, or database replicas) in different VPCs via Transit Gateway. The $0.02/GB processing fee can quickly reach thousands of dollars per month.
*   **Dev/Test VPC Attachment Proliferation:** Attaching every transient development, testing, and branch VPC to the Transit Gateway, building up substantial monthly connection fees.
*   **Cross-AZ Routing Overheads:** Routing traffic through a Transit Gateway attachment that is not optimized for Availability Zone alignment, potentially incurring inter-AZ data transfer fees ($0.01/GB) on top of the $0.02/GB processing fee.

---

## 7. Actionable Cost Optimization Strategies
1.  **Use VPC Peering for High-Volume Links:** Identify VPC pairs that exchange large volumes of data (e.g. app server to database VPC, or file servers). Set up a direct **VPC Peering connection** between them. 
    *   **The Savings:** VPC Peering has **$0.00/GB data processing fees** (Transit Gateway is $0.02/GB) and **$0 base hourly fees** (Transit Gateway is $0.05/hour). This bypasses Transit Gateway entirely for that specific link.
2.  **Consolidate Dev/Test VPC Attachments:** Instead of attaching all non-production VPCs individually:
    *   Merge development, testing, and staging environments into a single **Non-Production VPC** separated by subnets and security groups.
    *   Attach only this single VPC to the Transit Gateway, saving $0.10/hour (~$73.00/month) per consolidated VPC attachment.
3.  **Delete Inactive VPC Attachments:** Audit the Transit Gateway console. Find and delete attachments to deleted, stopped, or inactive resources.
4.  **Align AZ Subnets with Attachments:** When attaching a VPC to Transit Gateway, select only the subnets in the Availability Zones where your workloads actively run. This prevents cross-AZ data transfer routing.
5.  **Utilize VPC Lattice for Microservice Links:** For service-to-service communication, evaluate **AWS VPC Lattice**. It connects microservices across VPCs at application layer, bypassing Transit Gateway routing.
