# Cost-Cutting Playbook: AWS Global Accelerator
> **Companion File:** [global_accelerator.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/global_accelerator/global_accelerator.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Global Accelerator provides performance benefits by routing traffic over the AWS global network backbone. However, at $18.00 per month per accelerator plus Data Transfer-Premium (DT-Premium) fees per GB, costs can escalate rapidly. This playbook provides strategies to optimize Global Accelerator usage, focusing on waste elimination, architectural pivots, and network optimizations to minimize both the hourly fixed fees and data transfer premiums.

## Strategy Categories
### 1. Waste Elimination
### 2. Rightsizing
### 3. Commitment Discounts
### 4. Architecture Changes
### 5. Scheduling & Auto-Scaling
### 6. Pricing Model Optimization
### 7. Network & Data Transfer Optimization
---
## Cross-Service Synergies
- **Amazon CloudFront:** Used in tandem with GA to offload static traffic, minimizing GA's DT-Premium.
- **Route 53:** Can act as a cheaper, DNS-level routing alternative to GA's anycast IP routing.
- **AWS Direct Connect / Site-to-Site VPN:** Alternative network ingestion paths for enterprise traffic.
---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
Identify hourly charges and DT-Premium bytes transferred across regions.
### B. CloudWatch Metrics
Analyze `ProcessedBytesIn` and `ProcessedBytesOut` to determine dominant direction and utilization.
### C. AWS Config / Trusted Advisor
Identify provisioned, disabled, or idle accelerators.
### D. Company Policies
Acceptable latency SLAs for global users (determines if GA is strictly necessary).
### E. IaC (Optional)
Terraform or CloudFormation scripts to verify accelerator and endpoint group configurations.
---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "GA-001",
  "strategy_category": "Waste Elimination",
  "resource_id": "arn:aws:globalaccelerator::123456789012:accelerator/xxx",
  "monthly_savings": 18.00,
  "recommendation_action": "Delete disabled accelerator"
}
```
### Summary Report Table
| Strategy | Potential Savings | Effort | Risk |
|---|---|---|---|
| Delete Idle Accelerators | Low | Low | Low |
| CloudFront Offloading | High | Med | Med |

#### GA-01. Delete Disabled or Idle Accelerators
- **What:** Identify and delete accelerators that are disabled or show zero processed bytes in CloudWatch.
- **Why It Saves Money:** Disabled or idle accelerators continue to bill the flat hourly fee of $0.025/hour (~$18/month).
- **Implementation Steps:** 
  1. Review AWS Config or GA Console for disabled accelerators.
  2. Check CloudWatch `ProcessedBytes` metrics for the last 30 days.
  3. Delete accelerators with no traffic or that are explicitly disabled.
- **Estimated Savings:** 1-5% (up to $18/month per accelerator)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch metrics access.

#### GA-02. Consolidate Multiple Accelerators Using Custom Routing
- **What:** Use a single Custom Routing Accelerator to route traffic to thousands of EC2 instances instead of provisioning multiple Standard Accelerators for different workloads.
- **Why It Saves Money:** Avoids paying multiple $18/month base fees by sharing a single accelerator across applications using custom port mapping.
- **Implementation Steps:**
  1. Identify workloads using standard accelerators pointing to internal EC2 fleets.
  2. Deploy a single Custom Routing Accelerator.
  3. Map specific ports to specific VPC subnets/instances.
  4. Decommission the redundant standard accelerators.
- **Estimated Savings:** 5-15% of fixed fees
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Workloads compatible with port-based custom routing.

#### GA-03. Disable Accelerated Site-to-Site VPN if Unneeded
- **What:** Turn off the "Acceleration" feature on AWS Site-to-Site VPN connections if latency over the standard internet is acceptable.
- **Why It Saves Money:** Accelerated VPN uses Global Accelerator under the hood, incurring the $18/month fixed fee plus data transfer premiums.
- **Implementation Steps:**
  1. Review VPN performance and latency metrics.
  2. If the user base/office is close to the AWS region, disable acceleration.
  3. Monitor for any dropped packets or latency spikes.
- **Estimated Savings:** 10-20% of VPN costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ability to tolerate standard internet routing for VPN traffic.

#### GA-04. Route Only Dynamic Traffic via Global Accelerator
- **What:** Separate application traffic by type, routing dynamic APIs through GA and static assets through CloudFront.
- **Why It Saves Money:** Static assets don't benefit from GA's TCP/UDP protocol optimizations. Routing them via CloudFront avoids the GA DT-Premium ($0.015 - $0.050/GB).
- **Implementation Steps:**
  1. Analyze web traffic paths.
  2. Update DNS/Client configurations to pull `/static/`, `/images/`, etc., from a CloudFront domain.
  3. Route only transactional API traffic (e.g., `/api/`) to the GA Anycast IP.
- **Estimated Savings:** 40-70% on DT-Premium
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application code allows separating asset domains.

#### GA-05. Offload HTTP/HTTPS Traffic to CloudFront with Origin Shield
- **What:** Replace Global Accelerator completely with Amazon CloudFront for standard web traffic.
- **Why It Saves Money:** CloudFront is generally cheaper for standard HTTP/HTTPS traffic, caches content at the edge, and doesn't charge a flat $18/month fee per distribution.
- **Implementation Steps:**
  1. Verify if the workload relies strictly on HTTP/HTTPS.
  2. Provision a CloudFront distribution pointing to the regional ALB.
  3. Enable Origin Shield if request collapsing is needed.
  4. Route DNS to CloudFront and decommission GA.
- **Estimated Savings:** 30-50%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload must be HTTP/HTTPS (not generic TCP/UDP like gaming/IoT).

#### GA-06. Remove Unused Endpoint Groups for Low-Traffic Regions
- **What:** Remove GA endpoint groups in regions that receive minimal traffic and let those users route to the primary region over standard internet.
- **Why It Saves Money:** Reduces the infrastructure footprint and regional ALB/EC2 costs that were deployed purely to support a secondary GA endpoint group.
- **Implementation Steps:**
  1. Analyze traffic volume per endpoint group.
  2. Identify groups with negligible traffic.
  3. Remove the endpoint group from GA and decommission the underlying regional resources.
- **Estimated Savings:** 5-10% 
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Acceptable latency for users in the removed region.

#### GA-07. Leverage AWS Enterprise Discount Program (EDP)
- **What:** Commit to a total AWS spend to receive a blanket discount across services, including Global Accelerator.
- **Why It Saves Money:** EDP provides a flat percentage discount (typically 5-20%) on all eligible AWS usage, including GA fixed fees and DT-Premium.
- **Implementation Steps:**
  1. Forecast total annual AWS spend.
  2. Negotiate an EDP contract with your AWS Account Manager.
  3. Discounts apply automatically at the billing layer.
- **Estimated Savings:** 5-20%
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** Minimum annual AWS spend threshold (usually $1M+).

#### GA-08. Replace GA with Route 53 Latency-Based Routing
- **What:** Use DNS-based routing (Route 53) to send users to the closest regional endpoint instead of using GA's Anycast IPs.
- **Why It Saves Money:** Route 53 latency routing costs a few dollars per million queries, eliminating the GA $18/month fee and the per-GB DT-Premium entirely.
- **Implementation Steps:**
  1. Create Route 53 Latency records for your regional ALBs.
  2. Update client DNS.
  3. Decommission Global Accelerator.
- **Estimated Savings:** 80-100% of GA costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** DNS propagation delays are acceptable during failover events.

#### GA-09. Implement API Gateway Edge-Optimized Endpoints
- **What:** Use Edge-Optimized API Gateway instead of placing a Global Accelerator in front of a Regional API Gateway.
- **Why It Saves Money:** Edge-Optimized APIs natively use CloudFront under the hood, providing edge routing without the standalone GA fixed and premium fees.
- **Implementation Steps:**
  1. Convert Regional API Gateway to Edge-Optimized.
  2. Update DNS to point to the API Gateway edge domain.
  3. Delete the Global Accelerator.
- **Estimated Savings:** 20-40%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload is REST APIs hosted on API Gateway.

#### GA-10. Evaluate AWS Direct Connect for High-Volume Known Locations
- **What:** Use AWS Direct Connect for massive, predictable traffic from known corporate offices instead of routing them over the public internet via GA.
- **Why It Saves Money:** Direct Connect data transfer rates are significantly cheaper than standard internet egress + GA DT-Premium.
- **Implementation Steps:**
  1. Identify heavy source IP ranges in VPC Flow Logs.
  2. If traffic originates from a corporate office, provision a Direct Connect circuit.
  3. Route office traffic through the private connection.
- **Estimated Savings:** 30-60%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | Procurement/Leadership
- **Prerequisites:** High volume of traffic from static, owned physical locations.

#### GA-11. Avoid Inter-Region Hairpinning
- **What:** Ensure internal AWS resources do not communicate with each other using the public Global Accelerator Anycast IPs.
- **Why It Saves Money:** Internal traffic routed through GA incurs internet egress charges plus GA DT-Premium, whereas VPC Peering or Transit Gateway is much cheaper.
- **Implementation Steps:**
  1. Audit internal application configuration (e.g., microservices).
  2. Change target URLs to internal ALB endpoints or Route 53 private hosted zones.
  3. Ensure internal security groups allow private IP traffic.
- **Estimated Savings:** 50-80% on internal traffic
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC Peering or Transit Gateway established.

#### GA-12. Teardown Non-Production Accelerators After Hours
- **What:** Use Infrastructure as Code (IaC) to delete dev/staging Global Accelerators outside of business hours.
- **Why It Saves Money:** Avoids the $0.025/hour fee (~$12/month per accelerator) for environments nobody uses on nights and weekends.
- **Implementation Steps:**
  1. Write Terraform/CloudFormation to provision/destroy the GA resource.
  2. Schedule a CI/CD pipeline job to destroy dev GA instances at 6 PM.
  3. Schedule a job to recreate them at 8 AM.
- **Estimated Savings:** ~65% of fixed fees for non-prod
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ephemeral infrastructure practices and IaC adoption.

#### GA-13. Provision GA Just-In-Time for Global Events
- **What:** Only deploy Global Accelerator during specific high-traffic events (e.g., a massive game launch or global webinar) and remove it afterward.
- **Why It Saves Money:** Limits GA fixed fees and DT-Premium exclusively to windows where performance is absolutely critical and ROI is highest.
- **Implementation Steps:**
  1. Define event windows.
  2. Update Route 53 to point to GA Anycast IPs 24 hours before the event.
  3. Revert DNS to regional ALBs and delete GA after the event.
- **Estimated Savings:** 80-95%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** DNS TTLs managed properly to ensure smooth cutover.

#### GA-14. Capitalize on the Dominant Direction Rule
- **What:** Structure application architecture to ensure data transfer is heavily asymmetric, as GA only bills the dominant direction.
- **Why It Saves Money:** If 95% of traffic is outbound and 5% is inbound, the 5% inbound traffic incurs absolutely no GA DT-Premium charge.
- **Implementation Steps:**
  1. Analyze inbound vs outbound payloads.
  2. Group heavy uploads and heavy downloads onto the same accelerator if they peak at different times, effectively masking smaller traffic flows.
  3. Alternatively, don't split asymmetric workloads across different accelerators.
- **Estimated Savings:** 1-5%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Deep understanding of hourly traffic flow shapes.

#### GA-15. Implement Payload Compression (Gzip/Brotli)
- **What:** Compress all API responses and data payloads at the origin (EC2, ALB, or application layer) before sending them over GA.
- **Why It Saves Money:** GA DT-Premium is billed per GB transferred. Compressing text/JSON payloads reduces the number of GBs sent over the AWS backbone.
- **Implementation Steps:**
  1. Enable gzip or brotli compression on web servers (Nginx/Apache) or ALBs.
  2. Ensure clients send `Accept-Encoding` headers.
  3. Validate payloads are compressed.
- **Estimated Savings:** 20-50% on DT-Premium
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application clients support compressed responses.

#### GA-16. Enforce Client-Side Caching (Cache-Control)
- **What:** Set aggressive `Cache-Control` headers so clients do not re-request unchanged dynamic data over the Global Accelerator.
- **Why It Saves Money:** Prevents repeated API calls from traversing the GA network, eliminating redundant DT-Premium charges.
- **Implementation Steps:**
  1. Identify API endpoints returning static or slow-changing JSON.
  2. Configure HTTP headers: `Cache-Control: public, max-age=3600`.
  3. Test client behavior.
- **Estimated Savings:** 10-30% on DT-Premium
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application logic tolerates cached data.

#### GA-17. Optimize MTU Size and TCP Windowing
- **What:** Tune Maximum Transmission Unit (MTU) and TCP settings on origin servers to prevent packet fragmentation.
- **Why It Saves Money:** Packet fragmentation and TCP retransmissions mean the same data is sent multiple times, artificially inflating the GBs billed for DT-Premium.
- **Implementation Steps:**
  1. Ensure path MTU discovery is enabled on EC2 instances.
  2. Optimize TCP keepalives and window scaling parameters.
  3. Monitor `TCP_Retransmit` metrics.
- **Estimated Savings:** 1-2% on DT-Premium
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Advanced networking expertise.
