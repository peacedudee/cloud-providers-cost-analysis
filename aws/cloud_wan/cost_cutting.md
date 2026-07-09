# Cost-Cutting Playbook: AWS Cloud WAN
> **Companion File:** [cloud_wan.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/cloud_wan/cloud_wan.md)
> **Last Updated:** July 2026
---
## Executive Summary
This playbook outlines comprehensive strategies to optimize costs associated with AWS Cloud WAN. By targeting fixed Core Network Edge (CNE) hourly charges, network attachment fees, and data processing costs, organizations can significantly reduce their global networking expenditure while maintaining high availability and performance.

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
- **AWS Transit Gateway:** The primary cost-effective alternative for single-region or less complex multi-region setups.
- **Amazon VPC:** Utilizing VPC Peering reduces Cloud WAN attachment dependencies and data processing fees.
- **AWS Direct Connect & VPN:** Hybrid connectivity optimization to manage attachment and variable data transfer costs.
- **Amazon CloudFront:** Offloading data transfer to edge caching bypasses core network data processing completely.
---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- `lineItem/ProductCode`: `AmazonVPC` (Cloud WAN billing often surfaces here under specific usage types).
- `lineItem/UsageType`: Look for `CloudWAN-CNE-Hour`, `CloudWAN-VPC-AttachHour`, `CloudWAN-DataProcessing-Bytes`.
### B. CloudWatch Metrics
- `DataProcessed` on Cloud WAN attachments.
- `InBytes` and `OutBytes` for Core Network Edges to identify idle regions.
### C. AWS Config / Trusted Advisor
- Inventory of Core Network Edges, route policies, and active attachments.
- Identification of idle or underutilized VPC attachments.
### D. Company Policies
- Global network architecture standards, isolation policies, and multi-region compliance requirements.
- Acceptable failover and redundancy routing models.
### E. IaC (Optional)
- Terraform/CloudFormation defining the AWS Cloud WAN core network, segments, and attachments.
---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "CLOUDWAN-001",
  "strategy_name": "Migrate Single-Region Architectures to AWS Transit Gateway",
  "resource_id": "core-network-123",
  "account_id": "123456789012",
  "region": "us-east-1",
  "monthly_savings": 365.00,
  "confidence_score": 0.95
}
```

### Summary Report Table
| Finding ID | Strategy | Resource | Estimated Savings | Risk |
|------------|----------|----------|-------------------|------|
| CLOUDWAN-001 | Migrate Single-Region to TGW | core-network-123 | $365.00/mo | Low |

---
### 1. Waste Elimination

#### 1. Remove Core Network Edges (CNE) in Low-Traffic Regions
- **What:** Identify and delete CNEs deployed in regions with minimal or no network traffic.
- **Why It Saves Money:** Each regional CNE incurs a flat $365.00/month charge regardless of utilization.
- **Implementation Steps:**
  1. Review CloudWatch metrics for `InBytes`/`OutBytes` across all CNEs.
  2. Identify regions with zero or negligible traffic.
  3. Update the Cloud WAN core network policy to remove the CNE from those specific regions.
- **Estimated Savings:** 10-30%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Network traffic analysis confirming lack of utilization in the target region.

#### 2. Prune Inactive VPC Attachments
- **What:** Remove Cloud WAN attachments to VPCs that are no longer active, deprecated, or belong to decommissioned projects.
- **Why It Saves Money:** Each VPC attachment costs $0.065/hour (~$47.45/month). 
- **Implementation Steps:**
  1. Audit all Cloud WAN VPC attachments.
  2. Cross-reference attachment IDs with active VPCs and instance activity.
  3. Disassociate and delete attachments for orphaned or inactive VPCs.
- **Estimated Savings:** 5-15%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Clear asset inventory mapping VPCs to active applications.

#### 3. Prune Inactive VPN and Direct Connect Attachments
- **What:** Identify and remove unused Site-to-Site VPN or Direct Connect (DX) attachments to Cloud WAN.
- **Why It Saves Money:** Saves $0.065/hour per attachment (~$47.45/month), plus associated VPN/DX hourly costs.
- **Implementation Steps:**
  1. Monitor VPN and DX attachment metrics for lack of throughput over a 30-day period.
  2. Verify with network operations if the remote site has been decommissioned.
  3. Delete the attachment from the Cloud WAN core network.
- **Estimated Savings:** 5-10%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Network operations approval.

### 2. Rightsizing

#### 4. Consolidate Spoke VPCs to Reduce Attachment Count
- **What:** Merge micro-VPCs or loosely coupled environments into fewer, larger VPCs to reduce the number of required Cloud WAN attachments.
- **Why It Saves Money:** Reduces the number of $0.065/hr (~$47.45/month) attachment fees.
- **Implementation Steps:**
  1. Identify organizational units or projects with multiple small VPCs in the same region.
  2. Architect a consolidated VPC using subnet isolation and Security Groups.
  3. Migrate workloads to the consolidated VPC and attach only the single VPC to Cloud WAN.
- **Estimated Savings:** 10-25%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Complex migration planning and network redesign.

#### 5. Rightsize Site-to-Site VPN Connections
- **What:** Evaluate if multiple redundant VPN connections to the same on-premises locations are necessary or if they can be consolidated.
- **Why It Saves Money:** Eliminates redundant attachment hourly fees ($47.45/mo) and AWS VPN connection fees.
- **Implementation Steps:**
  1. Audit Site-to-Site VPNs attached to Cloud WAN.
  2. Consolidate redundant tunnels where high availability requirements safely allow.
  3. Remove unneeded VPN attachments.
- **Estimated Savings:** 5-10%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Acceptance of adjusted redundancy SLA.

### 3. Commitment Discounts

#### 6. Negotiate Private Pricing (EDP) for Cloud WAN Data Processing
- **What:** Leverage an Enterprise Discount Program (EDP) or Private Pricing Agreement (PPA) to lower the per-GB data processing fee.
- **Why It Saves Money:** The $0.02/GB processing fee can become the largest cost component at scale; volume discounts directly reduce this variable cost.
- **Implementation Steps:**
  1. Forecast projected Cloud WAN data processing volume.
  2. Engage AWS Account Team to negotiate a PPA covering Cloud WAN data processing and inter-region data transfer.
  3. Sign the committed spend agreement.
- **Estimated Savings:** 10-20%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Large scale AWS spend and predictable data volumes.

### 4. Architecture Changes

#### 7. Migrate Single-Region Architectures to AWS Transit Gateway
- **What:** Replace Cloud WAN with AWS Transit Gateway for networks confined to a single AWS region.
- **Why It Saves Money:** Eliminates the $365.00/month CNE charge entirely and reduces attachment fees from $0.065/hr to $0.05/hr (a 30% discount). Saves at least $4,380.00/year in baseline costs.
- **Implementation Steps:**
  1. Confirm that network topology is strictly single-region.
  2. Provision a Transit Gateway and configure route tables.
  3. Migrate attachments from Cloud WAN to the Transit Gateway.
  4. Decommission the Cloud WAN core network.
- **Estimated Savings:** 40-80%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** No immediate plans for multi-region network expansion.

#### 8. Connect Minor Spoke Regions via Inter-Region VPC Peering
- **What:** For regions with minor footprints, use Inter-Region VPC Peering directly to a main hub rather than provisioning a local CNE.
- **Why It Saves Money:** Avoids the $365.00/month CNE fee and $47.45/month attachment fees for minor regions.
- **Implementation Steps:**
  1. Identify regions with very few VPCs (e.g., 1-2 VPCs).
  2. Establish VPC Peering connections from these spoke VPCs to the primary hub VPC.
  3. Update route tables and delete the local CNE.
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-overlapping IP CIDR blocks and simple routing requirements.

#### 9. Implement Intra-Region VPC Peering for High-Volume Traffic
- **What:** Route heavy VPC-to-VPC traffic within the same region over direct VPC Peering instead of through the Cloud WAN backbone.
- **Why It Saves Money:** Bypasses the $0.02/GB data processing fee for traffic routed via Cloud WAN, as intra-region VPC Peering data transfer is generally cheaper or free (within the same AZ).
- **Implementation Steps:**
  1. Analyze flow logs to identify high-bandwidth communication between two specific VPCs in the same region.
  2. Create a VPC Peering connection between them.
  3. Update VPC route tables to prefer the peering connection for the target CIDR over the Cloud WAN attachment.
- **Estimated Savings:** 20-50%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-overlapping CIDR blocks.

#### 10. Localize Internet Egress to Avoid Backbone Backhaul
- **What:** Deploy NAT Gateways or Internet Gateways directly in local regional VPCs instead of backhauling internet-bound traffic over Cloud WAN to a central egress VPC.
- **Why It Saves Money:** Avoids the $0.02/GB Cloud WAN data processing fee and inter-region data transfer fees for outbound internet traffic.
- **Implementation Steps:**
  1. Analyze routing policies to see if default routes (0.0.0.0/0) point to the Cloud WAN core network.
  2. Provision local Internet egress resources (NAT/IGW) in the regional VPCs.
  3. Update route tables to route internet traffic locally.
- **Estimated Savings:** 15-30%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Decentralized security and egress inspection policies are acceptable.

#### 11. Implement Gateway VPC Endpoints for S3 and DynamoDB
- **What:** Use Gateway VPC Endpoints to route traffic to S3 and DynamoDB within the AWS network rather than over the Cloud WAN backbone.
- **Why It Saves Money:** Gateway endpoints are free and avoid the $0.02/GB Cloud WAN data processing fee for accessing S3 or DynamoDB.
- **Implementation Steps:**
  1. Create Gateway VPC Endpoints for S3 and DynamoDB in each VPC.
  2. Route tables are automatically updated.
  3. Traffic natively flows to S3/DynamoDB bypassing the Cloud WAN attachment.
- **Estimated Savings:** 10-40%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 12. Implement Interface VPC Endpoints for AWS Services
- **What:** Use AWS PrivateLink (Interface Endpoints) within local VPCs to access AWS services instead of routing via a centralized VPC over Cloud WAN.
- **Why It Saves Money:** Prevents AWS service API traffic (e.g., KMS, SNS, SQS) from incurring Cloud WAN $0.02/GB processing fees.
- **Implementation Steps:**
  1. Provision Interface VPC endpoints for heavily used AWS services in the local VPC.
  2. Ensure DNS resolution is enabled for the endpoint.
- **Estimated Savings:** 5-15%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Cost analysis ensuring PrivateLink hourly costs are lower than the Cloud WAN data processing savings.

### 5. Scheduling & Auto-Scaling

#### 13. Automate Attachment Lifecycle for Ephemeral Environments
- **What:** Use Infrastructure as Code (IaC) or AWS Step Functions to create and destroy Cloud WAN attachments dynamically alongside temporary CI/CD or testing VPCs.
- **Why It Saves Money:** Ensures you only pay the $0.065/hour attachment fee when the ephemeral environment actually exists.
- **Implementation Steps:**
  1. Integrate attachment creation into the deployment pipeline of ephemeral VPCs.
  2. Ensure the teardown pipeline systematically un-attaches and deletes the Cloud WAN attachment before VPC destruction.
- **Estimated Savings:** 5-10%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CI/CD pipeline and IaC maturity.

### 6. Pricing Model Optimization

#### 14. Evaluate Direct Connect vs. VPN for High Bandwidth
- **What:** For connecting on-premises data centers, analyze whether replacing multiple Site-to-Site VPNs with a dedicated Direct Connect (DX) is more cost-effective.
- **Why It Saves Money:** While both incur the $0.065/hr Cloud WAN attachment fee, DX offers lower data transfer out rates compared to standard VPN internet egress, saving significantly at high volumes.
- **Implementation Steps:**
  1. Calculate current data transfer out costs via VPN.
  2. Compare against the fixed port costs and reduced data transfer rates of Direct Connect.
  3. Migrate connectivity to DX if the breakeven point is met.
- **Estimated Savings:** 10-25%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Physical facility presence and colocation requirements.

### 7. Network & Data Transfer Optimization

#### 15. Offload Static Content Delivery to Amazon CloudFront
- **What:** Serve static assets and cached dynamic content via Amazon CloudFront (CDN) rather than fetching them from origin servers through the Cloud WAN backbone.
- **Why It Saves Money:** Bypasses the $0.02/GB Cloud WAN processing fee and inter-region data transfer fees, while providing cheaper and faster data transfer rates at the edge.
- **Implementation Steps:**
  1. Configure CloudFront distribution in front of user-facing endpoints.
  2. Ensure aggressive caching policies for static objects.
- **Estimated Savings:** 20-40%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Content is cacheable.

#### 16. Filter Traffic at the Source using Security Groups
- **What:** Apply strict Security Groups and Network ACLs at the source ENIs to drop unwanted traffic before it enters the Cloud WAN attachment.
- **Why It Saves Money:** You are billed $0.02/GB for data *processed* by the Cloud WAN attachment. Dropping illegitimate or broadcast traffic at the source prevents it from being billed.
- **Implementation Steps:**
  1. Audit current open internal ports.
  2. Implement least-privilege Security Group rules on all instances.
  3. Deny unused protocols and broadcast traffic.
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Comprehensive understanding of application communication requirements.

#### 17. Compress Data Transmitted Across the Core Network
- **What:** Implement payload compression (e.g., GZIP, Brotli) for application communication traversing the Cloud WAN backbone.
- **Why It Saves Money:** Directly reduces the volume of bytes transmitted, lowering the $0.02/GB data processing and inter-region data transfer costs.
- **Implementation Steps:**
  1. Enable HTTP compression on internal web servers/microservices.
  2. Use compressed serialization formats (like Parquet or compressed Protocol Buffers) for internal data pipelines.
- **Estimated Savings:** 10-20%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application support for compression.
