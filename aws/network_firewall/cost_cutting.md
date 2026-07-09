# Cost-Cutting Playbook: AWS Network Firewall
> **Companion File:** [network_firewall.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/network_firewall/network_firewall.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Network Firewall provides robust, stateful network inspection, but its pricing model can lead to exorbitant costs if deployed inefficiently. The service charges a high flat hourly rate for provisioned endpoints in every Availability Zone ($0.395/hr per endpoint), plus a data processing fee ($0.065/GB). The most critical cost-optimization strategy is moving from a distributed deployment model (endpoints in every VPC) to a Centralized Inspection VPC architecture using AWS Transit Gateway. This playbook outlines 20 actionable strategies to eliminate waste, optimize architectures, and reduce unnecessary data processing fees for AWS Network Firewall.

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
- **AWS Transit Gateway:** Essential for consolidating network traffic and reducing the number of required firewall endpoints.
- **NAT Gateway:** Taking advantage of the 1-to-1 fee waiver when NAT Gateways are chained with Network Firewall.
- **AWS WAF:** Offloading Layer 7 web inspection to WAF to reduce data processing volume on Network Firewall.
- **VPC Endpoints (PrivateLink & Gateway Endpoints):** Bypassing Network Firewall for AWS service API traffic to eliminate data processing fees.
---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query `lineItem/ProductCode` for `AWSNetworkFirewall`.
- Filter by `lineItem/UsageType` to identify `EndpointHour` vs `DataProcessing-Bytes`.
### B. CloudWatch Metrics
- `DataProcessedBytes` (monitor traffic volume going through the firewall).
- `ReceivedPackets` and `DroppedPackets` (to assess the effectiveness of rule groups).
### C. AWS Config / Trusted Advisor
- Check for VPCs with dedicated Network Firewall endpoints.
- Verify presence of S3 and DynamoDB Gateway Endpoints in VPC route tables.
### D. Company Policies
- Compliance requirements governing which traffic *must* be inspected (e.g., all egress, all east-west).
### E. IaC (Optional)
- Terraform/CloudFormation state files to identify how Network Firewalls are deployed across environments.
---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "NETFW-001",
  "strategy_name": "Adopt a Centralized Inspection VPC Architecture",
  "resource_id": "arn:aws:network-firewall:us-east-1:123456789012:firewall/spoke-vpc-fw",
  "account_id": "123456789012",
  "estimated_monthly_savings": 865.05,
  "effort_level": "High"
}
```
### Summary Report Table
| Finding ID | Strategy Name | Estimated Savings | Risk Level |
|------------|---------------|-------------------|------------|
| NETFW-001 | Centralized Inspection VPC | $8,000+ | High |
| NETFW-002 | NAT Gateway Fee Waiver | $500+ | Low |
| NETFW-003 | S3/DynamoDB Gateway Endpoints | $1,000+ | Low |

---

### 1. Waste Elimination

#### NETFW-001. Tear Down Idle Non-Production Firewalls
- **What:** Identify and delete Network Firewall endpoints deployed in staging, development, or sandbox VPCs that do not strictly require stateful inspection.
- **Why It Saves Money:** Each endpoint costs $0.395/hr ($288.35/month). Removing 3 endpoints in a dev VPC saves $865.05/month.
- **Implementation Steps:**
  1. Identify non-production Network Firewalls using AWS Tagging or Config.
  2. Modify route tables in dev VPCs to route directly to NAT Gateway or IGW instead of the firewall endpoint.
  3. Delete the Network Firewall resource.
- **Estimated Savings:** 20-40% of total Network Firewall endpoint costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Approval from security to use standard Security Groups for dev environments.

#### NETFW-002. Use Security Groups for Internal East-West Traffic
- **What:** Rely on VPC Security Groups and NACLs for internal microservice communication instead of routing intra-VPC traffic through Network Firewall.
- **Why It Saves Money:** Security Groups are free. Routing internal traffic through Network Firewall incurs a $0.065/GB data processing fee.
- **Implementation Steps:**
  1. Analyze VPC route tables to ensure the local route (`10.x.x.x/16 -> local`) is respected.
  2. Remove any more specific routes that force internal subnet traffic to the firewall endpoint.
- **Estimated Savings:** 10-30% on data processing costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Well-defined Security Group rules.

#### NETFW-003. Reduce AZ Endpoints in Non-Production
- **What:** Deploy Network Firewall endpoints in only 1 Availability Zone for development or QA environments, rather than a highly available 3-AZ setup.
- **Why It Saves Money:** Reduces the baseline hourly endpoint fee from $865.05/month (3 AZs) to $288.35/month (1 AZ).
- **Implementation Steps:**
  1. Reconfigure non-prod Network Firewalls to provision in a single AZ.
  2. Route all outbound traffic in dev through that single AZ endpoint.
- **Estimated Savings:** 66% reduction in base endpoint fees for affected environments.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Tolerance for potential AZ failure in non-prod.

### 2. Rightsizing

#### NETFW-004. Rightsize Inspection Scope
- **What:** Configure routing so that only traffic explicitly requiring inspection (e.g., untrusted egress) passes through the firewall.
- **Why It Saves Money:** Bypassing the firewall for trusted traffic avoids the $0.065/GB data processing fee.
- **Implementation Steps:**
  1. Identify high-volume, low-risk traffic (e.g., trusted internal cross-account backups).
  2. Update Transit Gateway (TGW) or VPC route tables to bypass the Inspection VPC for this specific traffic.
- **Estimated Savings:** 10-40% on data processing costs.
- **Risk Level:** Medium
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Clear definition of trusted vs. untrusted traffic flows.

#### NETFW-005. Optimize VPC Flow Logs Around the Firewall
- **What:** Configure VPC Flow Logs on the firewall subnets to only capture `REJECT` traffic rather than `ALL` traffic.
- **Why It Saves Money:** High-throughput firewalls generate massive amounts of VPC Flow Log data, costing $0.50/GB for CloudWatch Logs ingestion.
- **Implementation Steps:**
  1. Locate the VPC Flow Logs attached to the Network Firewall subnets.
  2. Change the filter from `ALL` to `REJECT`.
- **Estimated Savings:** 50-80% reduction in CloudWatch Logs ingestion costs for those subnets.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

### 3. Commitment Discounts

#### NETFW-006. Note on Lack of Commitment Discounts
- **What:** AWS Network Firewall does not currently offer Reserved Instances or Compute Savings Plans for its endpoints or data processing.
- **Why It Saves Money:** Prevents FinOps teams from wasting time looking for commitment discount options for this specific service.
- **Implementation Steps:**
  1. Focus FinOps efforts entirely on architectural optimization (Transit Gateway) and data volume reduction.
- **Estimated Savings:** 0% (Guidance only)
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

### 4. Architecture Changes

#### NETFW-007. Adopt a Centralized Inspection VPC Architecture
- **What:** Use AWS Transit Gateway to route traffic from dozens of spoke VPCs to a single Centralized Inspection VPC containing one highly available Network Firewall cluster.
- **Why It Saves Money:** Eliminates the need to deploy dedicated endpoints ($865.05/mo each) in every single VPC. Consolidating 10 VPCs into 1 Inspection VPC saves over $7,700/month.
- **Implementation Steps:**
  1. Deploy a Centralized Inspection VPC with Network Firewall across 3 AZs.
  2. Deploy AWS Transit Gateway and attach all spoke VPCs and the Inspection VPC.
  3. Configure TGW route tables to funnel egress/east-west traffic through the Inspection VPC.
- **Estimated Savings:** 50-90% of total endpoint fees.
- **Risk Level:** High
- **Implementation Scope:** Network Engineer / Architect
- **Prerequisites:** AWS Transit Gateway deployed.

#### NETFW-008. Leverage the NAT Gateway 1-to-1 Fee Waiver
- **What:** Chain NAT Gateways directly with Network Firewall endpoints in the exact same VPC routing path.
- **Why It Saves Money:** AWS waives the NAT Gateway hourly fee ($0.045/hr) and data processing fee ($0.045/GB) on a 1-to-1 basis when used directly with Network Firewall.
- **Implementation Steps:**
  1. Ensure the routing path is: Private Subnet -> NAT Gateway -> Network Firewall Endpoint -> IGW (or vice versa).
  2. Verify billing statements to ensure the "NAT Gateway Fee Waiver" line item appears.
- **Estimated Savings:** Effectively makes NAT Gateway free for traffic passing through the firewall.
- **Risk Level:** Low
- **Implementation Scope:** Network Engineer
- **Prerequisites:** NAT GW and Network Firewall in the same VPC.

#### NETFW-009. Use S3 and DynamoDB Gateway VPC Endpoints
- **What:** Ensure every VPC uses Gateway Endpoints for S3 and DynamoDB to route traffic directly to these AWS services over the AWS backbone.
- **Why It Saves Money:** Gateway Endpoints are 100% free. Routing S3 traffic through Network Firewall costs $0.065/GB. For a 10TB backup, this saves $650 per month.
- **Implementation Steps:**
  1. Create Gateway VPC Endpoints for S3 and DynamoDB in all spoke VPCs.
  2. Ensure local route tables prioritize the `pl-xxxxxx` prefix list for these services over the default route (`0.0.0.0/0`) pointing to the firewall.
- **Estimated Savings:** 20-60% on data processing costs depending on AWS service usage.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### NETFW-010. Use Interface VPC Endpoints (PrivateLink)
- **What:** Deploy Interface Endpoints for high-traffic AWS services (e.g., CloudWatch, Kinesis, ECR, SQS) inside spoke VPCs.
- **Why It Saves Money:** Interface Endpoints cost ~$0.01/GB for processing. Routing this AWS API traffic out to the internet through Network Firewall costs $0.065/GB (a savings of $0.055/GB).
- **Implementation Steps:**
  1. Identify high-bandwidth AWS service APIs being accessed over public endpoints.
  2. Deploy Interface VPC endpoints for these services in the spoke VPCs.
- **Estimated Savings:** 10-20% on data processing costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### NETFW-011. Use Multi-VPC Secondary Endpoint Associations
- **What:** Instead of Transit Gateway, associate secondary VPCs directly to an existing primary Network Firewall endpoint.
- **Why It Saves Money:** Secondary endpoints have a reduced hourly rate and incur **$0.00** for data processing.
- **Implementation Steps:**
  1. Configure multi-VPC associations in Network Firewall.
  2. Attach secondary VPCs to the primary firewall.
- **Estimated Savings:** 100% savings on data processing for the secondary VPCs.
- **Risk Level:** Medium
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Network topology supports secondary endpoint association constraints.

#### NETFW-012. Bypass Firewall for High-Volume Video/Media Egress
- **What:** For specific subnets or VPCs serving massive amounts of encrypted media/video out to the internet, route directly to the Internet Gateway.
- **Why It Saves Money:** Inspecting terabytes of streaming video is often unnecessary (as it's encrypted outbound data) and incurs heavy $0.065/GB fees.
- **Implementation Steps:**
  1. Isolate high-volume media workloads into specific subnets.
  2. Route these subnets directly to IGW or NAT GW, bypassing the TGW/Inspection VPC.
- **Estimated Savings:** High (e.g., $65 saved per TB of media).
- **Risk Level:** Medium
- **Implementation Scope:** Network Engineer / Security
- **Prerequisites:** Security approval to bypass egress inspection for specific payload types.

#### NETFW-013. Deploy AWS WAF for Web Workloads
- **What:** Use AWS WAF on Application Load Balancers for inbound Layer 7 web traffic instead of routing inbound public traffic through Network Firewall.
- **Why It Saves Money:** WAF charges per request/ACL rather than a steep per-GB bandwidth fee. It is generally more cost-effective for HTTP/HTTPS web traffic.
- **Implementation Steps:**
  1. Attach AWS WAF to public-facing ALBs.
  2. Remove inbound internet routing through Network Firewall for these applications.
- **Estimated Savings:** Varies; highly cost-effective for bandwidth-heavy web apps.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workloads must be HTTP/HTTPS.

### 5. Scheduling & Auto-Scaling

#### NETFW-014. Schedule Firewall Deletion in Ephemeral CI/CD
- **What:** Ensure automated CI/CD pipelines create Network Firewall endpoints dynamically for integration testing and destroy them immediately after.
- **Why It Saves Money:** Leaving an endpoint running 24/7 costs $288/month per AZ. Running it only 10 hours a week costs ~$16/month.
- **Implementation Steps:**
  1. Update Terraform/CloudFormation scripts in CI/CD pipelines to tear down `aws_networkfirewall_firewall` resources on pipeline completion.
- **Estimated Savings:** 90%+ endpoint cost reduction for ephemeral environments.
- **Risk Level:** Low
- **Implementation Scope:** DevOps Team
- **Prerequisites:** Fully automated infrastructure as code.

### 6. Pricing Model Optimization

#### NETFW-015. Consolidate AWS Accounts via AWS Organizations
- **What:** Mandate that all teams use the shared Centralized Inspection VPC located in the central networking account.
- **Why It Saves Money:** Prevents shadow IT or isolated teams from spinning up their own $865/month firewalls in their standalone AWS accounts.
- **Implementation Steps:**
  1. Share the Transit Gateway to all organizational accounts using AWS Resource Access Manager (RAM).
  2. Implement Service Control Policies (SCPs) denying the creation of Network Firewall endpoints outside the networking account.
- **Estimated Savings:** Prevents unpredictable billing spikes across the organization.
- **Risk Level:** Low
- **Implementation Scope:** Cloud Center of Excellence (CCoE) / FinOps
- **Prerequisites:** AWS Organizations enabled.

#### NETFW-016. Monitor and Alert on Data Processing Spikes
- **What:** Set up AWS Budgets and CloudWatch alarms based on Network Firewall data processing metrics.
- **Why It Saves Money:** Instantly detects misconfigurations (e.g., routing 100TB of database backups through the firewall) before a massive bill is generated.
- **Implementation Steps:**
  1. Create a CloudWatch Alarm for `DataProcessedBytes` exceeding baseline thresholds.
  2. Send alerts to an SNS topic for the network team.
- **Estimated Savings:** Cost avoidance of potentially thousands of dollars.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

### 7. Network & Data Transfer Optimization

#### NETFW-017. Use VPC NACLs to Drop Traffic Before the Firewall
- **What:** Block known bad IPs or irrelevant subnets at the Network Access Control List (NACL) level on the spoke VPCs or firewall subnets.
- **Why It Saves Money:** Traffic dropped by a NACL never reaches the Network Firewall endpoint, avoiding the $0.065/GB data processing fee entirely.
- **Implementation Steps:**
  1. Identify high-volume noisy IP ranges.
  2. Add DENY rules to the VPC NACLs for those ranges.
- **Estimated Savings:** 1-5% of data processing costs.
- **Risk Level:** Low
- **Implementation Scope:** Security / Network Engineer
- **Prerequisites:** IP lists to block.

#### NETFW-018. Optimize Stateless Rules for Early Drops
- **What:** Configure Network Firewall stateless rule groups to DROP unwanted traffic immediately.
- **Why It Saves Money:** While it doesn't directly reduce the per-GB processing fee (as bytes hitting the endpoint are billed), it dramatically reduces the processing overhead on the stateful engine, improving overall throughput limits and preventing bottleneck upgrades.
- **Implementation Steps:**
  1. Analyze firewall logs for high-frequency blocked traffic.
  2. Move these blocks from stateful rules to stateless rules.
- **Estimated Savings:** Indirect performance optimization.
- **Risk Level:** Low
- **Implementation Scope:** Security Engineer
- **Prerequisites:** None.

#### NETFW-019. Compress Payload Data Before Transit
- **What:** Ensure that applications compress data (e.g., GZIP) before sending it across the network to external destinations.
- **Why It Saves Money:** Network Firewall charges per gigabyte of traffic processed. Compressing JSON or text payloads can reduce data volume by 70%+, directly reducing the $0.065/GB fee.
- **Implementation Steps:**
  1. Enable compression on ALBs, web servers, or application payloads.
- **Estimated Savings:** Up to 70% on data processing for text-based traffic.
- **Risk Level:** Low
- **Implementation Scope:** Application Developers
- **Prerequisites:** Application support for compression.

#### NETFW-020. Review TLS Inspection Necessity
- **What:** AWS removed the per-GB surcharge for TLS inspection, but performing TLS inspection requires heavier endpoint sizing and complex certificate management. Evaluate if full deep packet inspection is necessary versus domain-based filtering.
- **Why It Saves Money:** Offloading TLS termination to ALBs/WAF and relying on SNI (Server Name Indication) domain filtering on the firewall can reduce complexity and potential throughput caps.
- **Implementation Steps:**
  1. Use SNI-based domain filtering (Server Name Indication) in Network Firewall instead of full TLS decryption if you only need to block specific domains.
- **Estimated Savings:** Operational overhead reduction and performance gain.
- **Risk Level:** Low
- **Implementation Scope:** Security Engineer
- **Prerequisites:** None.
