# Cost-Cutting Playbook: AWS Client VPN
> **Companion File:** [client_vpn.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/client_vpn/client_vpn.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Client VPN is a fully managed client-to-site VPN service that offers secure remote access. However, its pricing model—charging a flat continuous hourly fee for subnet associations ($0.10/hour per subnet) plus a variable hourly fee for active client connections ($0.05/hour per user)—often leads to unexpected and unoptimized costs. The primary drivers of waste include idle 24/7 endpoints in non-production environments, unnecessary multi-AZ deployments, forgotten weekend connections, and missing split tunneling resulting in high data egress costs. This playbook provides a structured set of strategies to optimize, rightsize, and architecturally refine Client VPN deployments.

---
## Strategy Categories

### 1. Waste Elimination

#### 1. Delete Unused Client VPN Endpoints
- **What:** Identify and delete Client VPN endpoints that have zero active connections over the last 30-60 days.
- **Why It Saves Money:** A completely idle endpoint with one associated subnet still costs ~$73.00/month. Eliminating it saves 100% of this fixed cost.
- **Implementation Steps:**
  1. Query CloudWatch metrics (`ActiveConnectionsCount`) for all Client VPN endpoints.
  2. Identify endpoints with `ActiveConnectionsCount = 0` for the past 30 days.
  3. Verify with stakeholders if the endpoint is still needed (e.g., for disaster recovery).
  4. Delete the endpoint and its target network associations.
- **Estimated Savings:** 100% of endpoint fixed costs ($73.00 - $146.00 per month per unused endpoint).
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch metrics access.

#### 2. Enforce Connection Max Session Duration
- **What:** Configure the maximum session duration (Connection Timeout) to forcefully disconnect idle or forgotten user sessions.
- **Why It Saves Money:** Users often leave VPNs connected over the weekend (64 hours). 64 hours * $0.05/hour = $3.20 per user per weekend. For 50 users, this is $160/weekend or $640/month wasted in idle connection fees.
- **Implementation Steps:**
  1. Modify the Client VPN endpoint settings in the AWS Console or via IaC.
  2. Set the `Session timeout hours` to a typical workday limit (e.g., 8, 10, or 12 hours).
  3. Communicate the change to end-users so they expect to re-authenticate daily.
- **Estimated Savings:** 10-20% of total Client Connection fees.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 3. Clean Up Unused Target Network Associations
- **What:** Remove extra subnet associations from endpoints if the subnet is not strictly required for routing or high availability.
- **Why It Saves Money:** Every associated subnet adds a flat $0.10/hour ($73/month). Removing an unnecessary second or third subnet immediately cuts the fixed cost overhead.
- **Implementation Steps:**
  1. Review all Target Network Associations for each Client VPN endpoint.
  2. Determine if the subnets provide necessary AZ redundancy or if a single AZ is sufficient.
  3. Disassociate the unneeded subnets.
- **Estimated Savings:** $73.00/month per removed subnet.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Network architecture review.

#### 4. Monitor and Alert on High Connection Usage
- **What:** Set up AWS Budgets or CloudWatch Alarms to trigger when active connection counts or connection hours spike unexpectedly.
- **Why It Saves Money:** Prevents bill shock from automated scripts stuck in reconnection loops, unapproved usage, or teams sharing VPN profiles inappropriately.
- **Implementation Steps:**
  1. Create a CloudWatch alarm on the `ActiveConnectionsCount` metric.
  2. Set a threshold slightly above the peak expected concurrent users.
  3. Configure SNS notifications to the FinOps/DevOps team.
- **Estimated Savings:** 5-10% (Cost Avoidance).
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Baseline usage understanding.

#### 5. Offboard Terminated Employees Promptly
- **What:** Ensure SSO (IAM Identity Center) or Active Directory integration is properly synchronized to immediately revoke VPN access for departing employees.
- **Why It Saves Money:** Prevents unauthorized users from consuming connection hours and maintains strict zero-trust security.
- **Implementation Steps:**
  1. Integrate Client VPN with SAML-based IdP or AWS Managed Microsoft AD.
  2. Ensure automated HR offboarding workflows disable IdP accounts.
  3. Terminate any active VPN sessions for the offboarded user via AWS CLI (`aws ec2 terminate-client-vpn-connections`).
- **Estimated Savings:** Variable (Cost Avoidance).
- **Risk Level:** Low
- **Implementation Scope:** IT/Security
- **Prerequisites:** SAML IdP or Active Directory integration.

### 2. Rightsizing

#### 6. Reduce High Availability (HA) to Single AZ for Non-Production
- **What:** Use only one Target Network Association (one subnet in one AZ) for development, staging, or testing Client VPN endpoints.
- **Why It Saves Money:** HA (2 subnets) costs $146/month. Single AZ costs $73/month. Non-production environments rarely require strict 99.9% uptime for remote access.
- **Implementation Steps:**
  1. Identify Client VPN endpoints serving non-production VPCs.
  2. Check if they have subnets associated in multiple AZs.
  3. Disassociate one of the subnets.
- **Estimated Savings:** 50% of the endpoint base cost ($73.00/month per downgraded endpoint).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Acceptable downtime policy for non-prod.

#### 7. Limit the Number of Associated Subnets Per Endpoint
- **What:** Avoid mapping every subnet in a VPC to the Client VPN endpoint. Only map the minimum required for network routing.
- **Why It Saves Money:** Routing in AWS relies on route tables. You only need to associate a subnet to provide an ENI for the VPN to enter the VPC. Once in the VPC, it can route to any other subnet. Associating 4 subnets costs $292/month unnecessarily.
- **Implementation Steps:**
  1. Audit endpoints with > 2 associated subnets.
  2. Consolidate associations to 1 (for non-prod) or 2 (for prod HA).
  3. Ensure VPC route tables allow traffic from the VPN ENIs to the target resources.
- **Estimated Savings:** $73.00 - $219.00/month per over-provisioned endpoint.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC routing knowledge.

### 3. Commitment Discounts

#### 8. Evaluate Compute Savings Plans for Self-Managed VPNs
- **What:** While AWS Client VPN natively does not support Savings Plans or RIs, if you migrate to an EC2-based VPN (see Strategy 12), you can cover the underlying EC2 compute with Savings Plans.
- **Why It Saves Money:** EC2 Compute Savings Plans can reduce the compute cost of a self-managed VPN instance by up to 72%.
- **Implementation Steps:**
  1. Assess the feasibility of migrating to a self-managed EC2 VPN.
  2. Determine the steady-state EC2 instance type required.
  3. Purchase Compute Savings Plans to cover the instance commitment.
- **Estimated Savings:** Up to 72% on the compute portion of the VPN alternative.
- **Risk Level:** Low (for the SP purchase)
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Migration to EC2-based VPN architecture.

### 4. Architecture Changes

#### 9. Consolidate VPN Endpoints via Centralized Management VPC
- **What:** Instead of deploying a separate Client VPN endpoint in every VPC, deploy a single centralized Client VPN endpoint in a Transit/Management VPC and route traffic to spoke VPCs via Transit Gateway or VPC Peering.
- **Why It Saves Money:** Avoids paying the $73-$146/month subnet association flat fee multiple times. 10 VPCs with 10 endpoints = $1,460/month. 1 Central Endpoint = $146/month.
- **Implementation Steps:**
  1. Deploy a Client VPN endpoint in a central Shared Services / Network VPC.
  2. Configure Transit Gateway (TGW) or VPC Peering between the central VPC and all target spoke VPCs.
  3. Update Client VPN route table to route spoke VPC CIDRs through the TGW/Peering connection.
  4. Decommission the old spoke VPC VPN endpoints.
- **Estimated Savings:** 50-90% of subnet association fees.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Transit Gateway or VPC Peering architecture.

#### 10. Replace Client VPN with AWS Systems Manager (SSM) Session Manager
- **What:** Use SSM Session Manager to securely connect (SSH/RDP) to EC2 instances without requiring a VPN or bastion host.
- **Why It Saves Money:** SSM Session Manager is free. It completely eliminates the Client VPN subnet association and connection hourly fees for infrastructure administration.
- **Implementation Steps:**
  1. Install the SSM Agent on target EC2 instances.
  2. Attach the `AmazonSSMManagedInstanceCore` IAM policy to the instance profiles.
  3. Connect via the AWS Console or AWS CLI using the Session Manager plugin.
  4. Decommission Client VPN if its only purpose was EC2 server access.
- **Estimated Savings:** 100% of Client VPN costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** EC2 instances only (does not provide generalized network access).

#### 11. Replace Client VPN with EC2 Instance Connect (EIC) Endpoint
- **What:** Use EIC Endpoints to establish secure SSH/RDP tunnels to instances in private subnets without a VPN or public IPs.
- **Why It Saves Money:** EIC Endpoints have NO hourly cost and NO data transfer cost. You only pay for standard VPC routing.
- **Implementation Steps:**
  1. Create an EC2 Instance Connect Endpoint in the target VPC.
  2. Configure Security Groups to allow traffic from the EIC Endpoint to target instances.
  3. Use the AWS CLI to tunnel connections (`aws ec2-instance-connect open-tunnel`).
  4. Retire Client VPN for engineering access.
- **Estimated Savings:** 100% of Client VPN costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Users comfortable with AWS CLI for tunneling.

#### 12. Migrate to a Self-Managed EC2 VPN (WireGuard/OpenVPN)
- **What:** For high-volume, 24/7 VPN usage, replace the fully managed Client VPN with a self-managed VPN server running on an EC2 instance (e.g., WireGuard, OpenVPN Access Server, Pritunl).
- **Why It Saves Money:** A `t3.small` EC2 instance costs ~$15/month and can support dozens of users. 50 users on AWS Client VPN for 160 hours/month = $400 + $146 base = $546/month. The EC2 approach is a flat ~$15/month regardless of user connection hours.
- **Implementation Steps:**
  1. Provision an EC2 instance in a public subnet.
  2. Install and configure the open-source VPN software.
  3. Distribute new client configuration files to users.
  4. Decommission the AWS Client VPN endpoint.
- **Estimated Savings:** 70-95% for high-usage scenarios.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Willingness to manage OS patching, VPN software updates, and high availability manually.

#### 13. Replace Client VPN with AWS Verified Access
- **What:** For internal web applications, replace Client VPN with AWS Verified Access to provide secure, zero-trust access based on user identity and device posture.
- **Why It Saves Money:** Verified Access charges $0.27/hour per application + $0.02 per GB. For a single heavily used application, this might be cheaper or more secure than wide VPN access.
- **Implementation Steps:**
  1. Set up Verified Access Trust Providers (IdP, Device management).
  2. Create a Verified Access Instance and Group.
  3. Create Verified Access Endpoints for internal load balancers or ENIs.
  4. Retire the VPN.
- **Estimated Savings:** Varies (requires breakeven analysis against VPN connection hours).
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Web-based HTTP/HTTPS applications only.

### 5. Scheduling & Auto-Scaling

#### 14. Automate Subnet Disassociation Outside Work Hours
- **What:** Programmatically remove the Target Network Associations from Client VPN endpoints during nights and weekends.
- **Why It Saves Money:** The $0.10/hour fee is continuous. By disassociating subnets from 7 PM to 7 AM and over weekends, the endpoint is only billed for ~40 hours a week instead of 168 hours.
- **Implementation Steps:**
  1. Create an AWS Lambda function with `ec2:DisassociateClientVpnTargetNetwork` and `ec2:AssociateClientVpnTargetNetwork` permissions.
  2. Set up Amazon EventBridge rules to trigger the Lambda at start and end of business hours.
  3. Note: The endpoint remains, but it stops billing the association fee while disconnected.
- **Estimated Savings:** 60-76% of subnet association fees.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-production environments or strict business-hours-only access policies.

### 6. Pricing Model Optimization

#### 15. Shift to a Cheaper AWS Region for VPN Endpoints
- **What:** Deploy the centralized Client VPN endpoint in a region with lower baseline pricing if latency is not a critical factor.
- **Why It Saves Money:** Some regions charge a premium on base services. If your workforce is globally distributed and connecting to global resources, centralizing the VPN in a cheaper tier region (e.g., us-east-2) avoids regional pricing premiums.
- **Implementation Steps:**
  1. Compare AWS Client VPN pricing across active AWS regions.
  2. Deploy the endpoint in the most cost-effective region.
  3. Connect to cross-region VPCs via inter-region VPC Peering or Transit Gateway.
- **Estimated Savings:** 0-20% depending on region migration.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Latency tolerance for remote users.

### 7. Network & Data Transfer Optimization

#### 16. Enable Split Tunneling to Bypass VPN for Public Internet Traffic
- **What:** Enable the "Split-tunnel" configuration on the Client VPN endpoint so only traffic destined for private VPC subnets travels over the VPN.
- **Why It Saves Money:** Without split tunneling (full tunnel), a remote worker watching streaming video or downloading OS updates routes all traffic through the AWS VPN, incurring standard AWS Data Transfer Out to Internet charges ($0.09/GB). Split tunneling sends only VPC-destined traffic to AWS, keeping personal internet traffic local.
- **Implementation Steps:**
  1. Edit the Client VPN endpoint settings in the AWS Console.
  2. Enable "Split-tunnel".
  3. Ensure the VPN route table only advertises the necessary private AWS CIDRs.
- **Estimated Savings:** 50-90% on NAT/Data Transfer Out costs generated by VPN users.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Security compliance approval (some enterprises mandate full-tunnel for DLP/inspection).

#### 17. Use VPC Endpoints for AWS Services Instead of VPN NAT Egress
- **What:** If remote users connect to the VPN specifically to securely access public AWS APIs (like S3 or DynamoDB), route that traffic through VPC Interface/Gateway Endpoints instead of a NAT Gateway.
- **Why It Saves Money:** NAT Gateway processing charges ($0.045/GB) and standard Data Transfer Out rates apply if VPN users go out to the internet to hit AWS public endpoints. VPC Endpoints keep the traffic on the AWS backbone for cheaper ($0.01/GB for PrivateLink, Free for Gateway Endpoints).
- **Implementation Steps:**
  1. Deploy Gateway Endpoints for S3 and DynamoDB in the VPC.
  2. Deploy Interface Endpoints (PrivateLink) for other required AWS services.
  3. Ensure the Client VPN DNS resolution points to the private endpoint IPs.
- **Estimated Savings:** 60-100% of NAT data processing charges for AWS API traffic.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC Endpoints architecture setup.

#### 18. Restrict Access via Security Groups to Prevent Unnecessary Data Egress
- **What:** Apply strict Security Groups to the Target Network Association ENIs used by the Client VPN, allowing access only to specific internal resources.
- **Why It Saves Money:** Prevents compromised or misconfigured VPN clients from scanning the VPC, transferring massive amounts of internal data, or using the VPC as an outbound proxy to the internet, all of which incur data transfer charges.
- **Implementation Steps:**
  1. Create a dedicated Security Group for the Client VPN endpoint.
  2. Define explicit egress rules to only the required internal subnets/IPs (e.g., Port 443 to internal apps, Port 22/3389 to jump boxes).
  3. Remove the default `0.0.0.0/0` outbound rule.
- **Estimated Savings:** Variable (Cost Avoidance).
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Clear understanding of user access requirements.

---
## Cross-Service Synergies
- **VPC / Transit Gateway:** Centralizing Client VPN endpoints heavily leverages Transit Gateway and VPC Peering to map network topologies efficiently.
- **AWS Directory Service (Managed AD) / IAM Identity Center:** Accurate billing relies on robust IdP integration to terminate sessions the moment a user's status changes.
- **EC2 / SSM:** Leveraging Systems Manager or EC2 Instance Connect Endpoints can completely eliminate the need for Client VPN for infrastructure administration.
- **Amazon EventBridge / Lambda:** Critical for building custom scheduling logic to disassociate subnets during off-hours, maximizing waste elimination.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query for `line_item_product_code` = `AmazonVPC` and `line_item_operation` = `ClientVPNEndpoint`.
- Identify the split between `ClientVPN-ConnectionHours` and `ClientVPN-EndpointHours`.
- Analyze `line_item_usage_account_id` to map costs to specific environments (Prod vs. Non-Prod).
### B. CloudWatch Metrics
- **ActiveConnectionsCount:** To find completely idle endpoints (0 connections) or to calculate concurrent usage peaks.
- **IngressBytes / EgressBytes:** To identify endpoints processing unusually high traffic, suggesting full-tunnel misuse.
### C. AWS Config / Trusted Advisor
- Use AWS Config to inventory all Client VPN endpoints and inspect their `splitTunnel` configuration status.
### D. Company Policies
- Determine if the security team mandates full-tunnel routing for DLP (Data Loss Prevention) or if split tunneling is permitted.
- Ascertain the acceptable timeout policies for active user sessions.
### E. IaC (Optional)
- Review Terraform (`aws_ec2_client_vpn_endpoint`) or CloudFormation to ensure `split_tunnel` is explicitly set to `true` and `session_timeout_hours` is configured.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "CLIENTVPN-001",
  "service": "AWS Client VPN",
  "strategy_category": "Waste Elimination",
  "strategy_name": "Delete Unused Client VPN Endpoints",
  "resource_id": "cvpn-endpoint-0abc1234def567890",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_monthly_cost": 146.00,
  "estimated_monthly_savings": 146.00,
  "recommendation_details": "Endpoint has 0 ActiveConnectionsCount over the last 30 days.",
  "action_script": "aws ec2 delete-client-vpn-endpoint --client-vpn-endpoint-id cvpn-endpoint-0abc1234def567890"
}
```

### Summary Report Table

| Finding ID | Strategy Name | Resource / Account | Current Cost | Estimated Savings | Implementation Effort |
|------------|---------------|--------------------|--------------|-------------------|-----------------------|
| CLIENTVPN-001 | Delete Unused Endpoints | cvpn-endpoint-0abc... | $146.00/mo | $146.00/mo (100%) | Low |
| CLIENTVPN-002 | Enable Split Tunneling | cvpn-endpoint-1xyz... | $500.00/mo (Data) | $400.00/mo (80%) | Low |
| CLIENTVPN-003 | Automate Subnet Disassociation | cvpn-endpoint-dev... | $146.00/mo | $93.00/mo (64%) | Medium |
| CLIENTVPN-004 | Consolidate Endpoints via TGW | Multiple Accounts | $1,460.00/mo | $1,314.00/mo (90%) | High |
