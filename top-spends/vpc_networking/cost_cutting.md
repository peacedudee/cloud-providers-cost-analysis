# Cost-Cutting Playbook: AWS VPC Networking & Transit Gateway

> **Companion Files:** [vpc.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/vpc_networking/vpc.md) | [transit_gateway.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/vpc_networking/transit_gateway.md)  
> **Last Updated:** July 2026

---

## Executive Summary

AWS VPC infrastructure components — **NAT Gateways** ($0.045/hr + $0.045/GB), **Transit Gateways** ($0.05/att-hr + $0.02/GB), **VPC Endpoints (PrivateLink)** ($0.01/hr/AZ + $0.01/GB), and **Public IPv4 Addresses** ($0.005/hr) — represent one of the largest sources of hidden infrastructure billing leaks in AWS.

Key billing traps include:
1. **NAT Gateway Processing Surcharge for AWS Services:** Routing container image pulls from ECR, CloudWatch logs, or S3 data dumps through a NAT Gateway ($0.045/GB processing tax).
2. **Transit Gateway Data Tax on High-Volume Traffic:** Routing database replication or big data pipelines across VPCs via Transit Gateway ($0.02/GB processing fee) instead of using **VPC Peering ($0.00/GB processing fee)**.
3. **Public IPv4 Surcharge Leakage:** Paying $3.65/month per assigned public IPv4 address across running EC2, EKS nodes, and idle Elastic IPs.

This playbook provides **20 actionable strategies** across six operational categories, delivering an estimated **35–80% reduction in total VPC networking spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Deploy Free Gateway VPC Endpoints for S3 and DynamoDB:** Instantly diverts S3 and DynamoDB traffic away from NAT Gateways, routing it directly for **100% FREE ($0.00/GB)**.
2. **Replace Transit Gateway with Free VPC Peering for High-Bandwidth Pairs:** Eliminates the $0.02/GB Transit Gateway data tax on heavy inter-VPC traffic (saves $20.00 per 1 TB).
3. **Consolidate Multi-AZ NAT Gateways to a Single NAT Gateway in Non-Prod:** Reduces base NAT Gateway hourly charges by **66%** ($32.85/mo vs $98.55/mo per 3-AZ dev VPC).

---

## Strategy Categories

### 1. NAT Gateway Optimization (The Largest Cost Driver)

#### 1. Deploy Free Gateway VPC Endpoints for S3 & DynamoDB in All Subnet Route Tables
- **What:** Create **Gateway VPC Endpoints** for Amazon S3 and DynamoDB in all VPC routing tables.
- **Why It Saves Money:** Gateway VPC Endpoints have **$0.00 hourly fees and $0.00 data processing fees**. Without them, private subnets downloading container images, backup files, or DynamoDB tables route data through NAT Gateways, paying **$0.045 per GB** in processing fees. Downloading a 10 TB dataset via NAT Gateway costs **$450.00**; via Gateway Endpoint it costs **$0.00**!
- **Detailed Implementation Steps:**
  1. Create Gateway Endpoints in Terraform:
     ```hcl
     resource "aws_vpc_endpoint" "s3" {
       vpc_id       = module.vpc.vpc_id
       service_name = "com.amazonaws.us-east-1.s3"
       vpc_endpoint_type = "Gateway"
       route_table_ids   = module.vpc.private_route_table_ids
     }

     resource "aws_vpc_endpoint" "dynamodb" {
       vpc_id       = module.vpc.vpc_id
       service_name = "com.amazonaws.us-east-1.dynamodb"
       vpc_endpoint_type = "Gateway"
       route_table_ids   = module.vpc.private_route_table_ids
     }
     ```
- **Estimated Savings:** **100% savings** on S3 and DynamoDB NAT Gateway processing fees ($45.00/TB saved).
- **Risk Level:** Zero risk (seamless, non-disruptive routing table update).
- **Implementation Scope:** Network Engineer / DevOps
- **Prerequisites:** VPC private route table access.

#### 2. Deploy Interface VPC Endpoints (PrivateLink) for ECR, CloudWatch Logs, and SSM
- **What:** Deploy **Interface VPC Endpoints (PrivateLink)** for high-volume internal AWS services (ECR `.api` and `.dkr`, CloudWatch `logs`, Systems Manager `ssm`).
- **Why It Saves Money:** NAT Gateway processes traffic at **$0.045 per GB**. Interface Endpoints process traffic at **$0.01 per GB** (after a small $7.30/mo base fee per AZ). For EKS/ECS clusters pulling gigabytes of container images daily, Interface Endpoints deliver a **77.8% direct processing fee discount**!
- **Detailed Implementation Steps:**
  1. Create Interface Endpoints for ECR in Terraform:
     ```hcl
     resource "aws_vpc_endpoint" "ecr_dkr" {
       vpc_id            = module.vpc.vpc_id
       service_name      = "com.amazonaws.us-east-1.ecr.dkr"
       vpc_endpoint_type = "Interface"
       subnet_ids        = module.vpc.private_subnets
       security_group_ids = [aws_security_group.vpce_sg.id]
       private_dns_enabled = true
     }
     ```
- **Estimated Savings:** **77.8% reduction** in data processing fees for supported services ($35.00/TB saved).
- **Risk Level:** Zero risk.
- **Implementation Scope:** Network Engineer / DevOps
- **Prerequisites:** Private DNS enabled on VPC.

#### 3. Consolidate NAT Gateways in Non-Production VPCs (Single-AZ NAT)
- **What:** In non-production (dev, test, staging) VPCs, deploy a **Single NAT Gateway** in AZ-A instead of provisioning 1 NAT Gateway per Availability Zone.
- **Why It Saves Money:** Each NAT Gateway costs $0.045/hour (**$32.85/month**). Deploying 3 NAT Gateways across 3 AZs in a dev VPC costs **$98.55/month** in flat base fees. Consolidating to 1 NAT Gateway in AZ-A cuts base costs by **66.6% (saving $65.70/month per VPC)**.
- **Detailed Implementation Steps:**
  1. Update VPC module parameter: `single_nat_gateway = true` (Terraform AWS VPC Module).
  2. Route internet-bound traffic (`0.0000/0`) from private subnets in AZ-B and AZ-C to the NAT Gateway in AZ-A.
- **Estimated Savings:** **66.6% reduction** in NAT Gateway base hourly charges ($65.70/mo per dev VPC).
- **Risk Level:** Low (dev environment does not require multi-AZ NAT failover HA).
- **Implementation Scope:** DevOps / Network Engineer
- **Prerequisites:** Non-prod SLA alignment.

#### 4. Decommission Unused NAT Gateways in Dev Subnets with Zero Traffic
- **What:** Delete NAT Gateways in developer sandbox VPCs where private instances have no active outbound internet requirements.
- **Why It Saves Money:** Reclaims $32.85/month base fee per NAT Gateway.
- **Detailed Implementation Steps:**
  1. Delete NAT Gateway: `aws ec2 delete-nat-gateway --nat-gateway-id nat-123456`.
  2. Release attached Elastic IP address.
- **Estimated Savings:** $32.85/mo per deleted NAT Gateway.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Outbound internet traffic audit.

---

### 2. Multi-VPC & Transit Gateway Optimization

#### 5. Replace Transit Gateway with Free VPC Peering for High-Bandwidth Links
- **What:** Establish direct **VPC Peering** connections for high-volume data transfers between specific VPC pairs (e.g. cross-VPC database replication or data lake ETL).
- **Why It Saves Money:**
  - **Transit Gateway:** Charges $0.05/attachment-hour ($36.50/mo) PLUS **$0.02 per GB** data processing fee. Replicating 100 TB/month across VPCs via Transit Gateway costs **$2,000.00/month** in data processing taxes!
  - **VPC Peering:** **$0.00 setup fee and $0.00 data processing fee** (100% FREE; you pay only standard intra-region AZ data transfer).
  - Switching high-volume links to VPC Peering saves **$20.00 per 1 TB transferred**!
- **Detailed Implementation Steps:**
  1. Create VPC Peering connection in Terraform:
     ```hcl
     resource "aws_vpc_peering_connection" "analytics_to_db" {
       peer_vpc_id = module.db_vpc.vpc_id
       vpc_id      = module.analytics_vpc.vpc_id
       auto_accept = true
     }
     ```
  2. Add specific CIDR route table entries pointing high-volume traffic to `pcx-123456` instead of `tgw-123456`.
- **Estimated Savings:** **100% savings** on Transit Gateway data processing fees ($20.00/TB saved).
- **Risk Level:** Low.
- **Implementation Scope:** Network Architect / DevOps
- **Prerequisites:** Non-overlapping CIDR blocks between VPCs.

#### 6. Audit & Prune Unused Transit Gateway Attachments
- **What:** Identify and delete orphaned Transit Gateway attachments (`aws ec2 describe-transit-gateway-attachments`) pointing to deleted or inactive VPCs.
- **Why It Saves Money:** Reclaims **$36.50/month per attachment** ($0.05/hour).
- **Detailed Implementation Steps:**
  1. Delete attachment: `aws ec2 delete-transit-gateway-vpc-attachment --transit-gateway-attachment-id tgw-attach-123`.
- **Estimated Savings:** $36.50/mo per deleted attachment.
- **Risk Level:** Low.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Verification that attachment traffic count = 0.

#### 7. Consolidate Transit Gateway Route Tables
- **What:** Consolidate sprawling Transit Gateway route tables across accounts to streamline routing and reduce administrative overhead.
- **Why It Saves Money:** Reclaims unused attachments and simplifies security group auditing.
- **Detailed Implementation Steps:**
  1. Use AWS Network Manager to map Transit Gateway topology.
- **Estimated Savings:** Governance efficiency.
- **Risk Level:** Medium.
- **Implementation Scope:** Network Architect
- **Prerequisites:** Network topology audit.

---

### 3. Public IPv4 Address Cost Elimination

#### 8. Release Unattached Elastic IP Addresses (EIPs)
- **What:** Identify and release all unattached Elastic IPv4 addresses sitting idle in your AWS accounts.
- **Why It Saves Money:** AWS charges **$0.005 per hour ($3.65/month)** for every allocated public IPv4 address that is NOT attached to a running EC2 instance. 50 idle EIPs waste **$182.50/month**.
- **Detailed Implementation Steps:**
  1. List unattached Elastic IPs via CLI:
     ```bash
     aws ec2 describe-addresses \
       --query "Addresses[?AssociationId==null].{AllocationId:AllocationId,PublicIp:PublicIp}" \
       --output table
     ```
  2. Release unattached EIPs:
     ```bash
     aws ec2 release-address --allocation-id eipalloc-1234567890
     ```
- **Estimated Savings:** **$3.65 per month saved** per released Elastic IP.
- **Risk Level:** Zero risk (unattached = zero active traffic).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Confirm EIP is not reserved for whitelisted third-party firewall rules.

#### 9. Remove Public IPv4 Addresses from Private Worker Nodes (EKS / EC2)
- **What:** Configure auto-assign public IP = `false` on subnets used for backend EC2 instances, EKS worker nodes, and internal database servers.
- **Why It Saves Money:** Since February 2024, AWS charges **$0.005/hour ($3.65/month)** for EVERY public IPv4 address assigned to running resources. Assigning public IPs to 100 private worker nodes wastes **$365.00/month**!
- **Detailed Implementation Steps:**
  1. Disable auto-assign public IP on private subnets:
     ```bash
     aws ec2 modify-subnet-attribute --subnet-id subnet-123456 --no-map-public-ip-on-launch
     ```
- **Estimated Savings:** **$3.65 per node per month saved**.
- **Risk Level:** Zero risk (private nodes access internet via NAT Gateway or VPC Endpoints).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Subnet architecture check.

---

### 4. VPN & Client Remote Access Optimization

#### 10. Auto-Shutdown Client VPN Endpoints Outside Office Hours
- **What:** Delete or disable AWS Client VPN Endpoints in non-production environments during nights and weekends (7 PM to 7 AM M-F, all weekend).
- **Why It Saves Money:** Client VPN endpoints bill **$0.15 per hour ($109.50/month)** flat base fee plus $0.05/hr per connection. Running non-prod Client VPN endpoints 24/7 wastes 118 idle hours/week ($77.60/month per endpoint).
- **Detailed Implementation Steps:**
  1. Automate Client VPN endpoint recreation via Terraform or CloudFormation scripts triggered by EventBridge.
- **Estimated Savings:** **70% reduction** in Client VPN base hourly fees.
- **Risk Level:** Low.
- **Implementation Scope:** DevOps Engineer
- **Prerequisites:** Automated VPN provisioning pipeline.

#### 11. Decommission Stale Site-to-Site VPN Connections
- **What:** Terminate inactive Site-to-Site VPN connections (`$0.05/hour = $36.50/month per connection`).
- **Why It Saves Money:** Reclaims $36.50/mo per idle IPSec tunnel.
- **Detailed Implementation Steps:**
  1. List Site-to-Site VPNs with 0 active tunnel status: `aws ec2 describe-vpn-connections`.
  2. Delete stale connections.
- **Estimated Savings:** $36.50/mo saved per VPN connection.
- **Risk Level:** Low.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** On-premises datacenter connectivity audit.

---

### 5. Flow Logs & Traffic Observability Tuning

#### 12. Compress & Filter VPC Flow Logs in CloudWatch / S3
- **What:** Filter VPC Flow Logs to record ONLY `REJECT` traffic (or restrict logging to specific ENIs) and publish directly to S3 in **Parquet format** instead of CloudWatch Logs.
- **Why It Saves Money:** Logging all `ACCEPT` traffic to CloudWatch Logs generates gigabytes of ingestion fees ($0.50/GB) and storage ($0.03/GB-mo). Publishing Parquet to S3 cuts ingestion and storage fees by 90%+.
- **Detailed Implementation Steps:**
  1. Create VPC Flow Log targeting S3 with Parquet format and 1-minute aggregation:
     ```bash
     aws ec2 create-flow-logs \
       --resource-type VPC --resource-ids vpc-123456 \
       --traffic-type REJECT \
       --log-destination-type s3 \
       --log-destination arn:aws:s3:::company-vpc-flow-logs \
       --destination-options FileFormat=parquet,HiveCompatiblePartitions=true,PerHourPartition=true
     ```
- **Estimated Savings:** 70–95% reduction in VPC Flow Logging spend.
- **Risk Level:** Low.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** Security compliance alignment.

---

### 6. IPAM & Network Governance

#### 13. Optimize AWS IPAM (IP Address Manager) Tiering
- **What:** Downgrade AWS IPAM from `Advanced Tier` ($0.00027/IP-hr) to `Free Tier` in accounts that do not require multi-region IP management.
- **Why It Saves Money:** Advanced IPAM bills $0.20/month per monitored IP address across your active VPC footprint.
- **Detailed Implementation Steps:**
  1. Update IPAM tier settings in AWS Console to Free Tier.
- **Estimated Savings:** Eliminates IPAM monitoring surcharges.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Network Architect
- **Prerequisites:** IPAM feature evaluation.

#### 14. Eliminate Intra-Region Cross-AZ Traffic Overhead
- **What:** Configure application load balancing and service discovery (AWS Cloud Map) to keep request routing within the **same Availability Zone**.
- **Why It Saves Money:** Cross-AZ traffic between EC2 instances costs **$0.01/GB in each direction ($0.02/GB roundtrip)**. Intra-AZ traffic is **100% FREE ($0.00/GB)**.
- **Detailed Implementation Steps:**
  1. Enable topology-aware routing in Kubernetes (EKS) or ECS service discovery.
- **Estimated Savings:** 100% of inter-AZ network egress fees ($20.00/TB saved).
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Multi-AZ deployment symmetry.

#### 15. Enforce CloudWatch Alarms for NAT Gateway Data Processing Surges
- **What:** Put CloudWatch metric alarm on NAT Gateway `BytesOutToDestination` (> 50 GB/hour).
- **Why It Saves Money:** Provides instant alert if an un-optimized backup script or container image loop starts dumping terabytes through NAT Gateway.
- **Detailed Implementation Steps:**
  1. Put CloudWatch alarm targeting `AWS/NATGateway`.
- **Estimated Savings:** Proactive billing risk protection.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 16. Restrict Inter-Region Data Egress (Use Local S3 Buckets)
- **What:** Ensure private workloads download assets from local regional S3 buckets rather than cross-region buckets ($0.02/GB cross-region transfer).
- **Why It Saves Money:** Eliminates cross-region data transfer fees.
- **Detailed Implementation Steps:**
  1. Replicate shared asset buckets to local regions using S3 Same-Region or Cross-Region Replication with local lifecycle.
- **Estimated Savings:** Saves $0.02/GB on cross-region file access.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** S3 bucket replication setup.

#### 17. Use Egress-Only Internet Gateways for IPv6 Subnets
- **What:** Deploy **Egress-Only Internet Gateways (EIGW)** for IPv6 subnets requiring outbound internet access.
- **Why It Saves Money:** Egress-Only Internet Gateways are **100% FREE ($0.00/hr, $0.00/GB)**, replacing expensive NAT Gateways for IPv6 workloads.
- **Detailed Implementation Steps:**
  1. Create Egress-Only IGW: `aws ec2 create-egress-only-internet-gateway --vpc-id vpc-123456`.
- **Estimated Savings:** 100% of NAT Gateway processing fees for IPv6.
- **Risk Level:** Low.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** IPv6 CIDR enabled.

#### 18. Consolidate Public IP Allocations behind Shared Ingress Proxies
- **What:** Front public applications with a single shared ingress proxy or CloudFront distribution.
- **Why It Saves Money:** Minimizes total public IPv4 allocations ($3.65/mo per IP).
- **Detailed Implementation Steps:**
  1. Centralize ingress routing behind shared load balancer.
- **Estimated Savings:** Reclaims public IPv4 address surcharges.
- **Risk Level:** Low.
- **Implementation Scope:** Infrastructure Engineer
- **Prerequisites:** Ingress architecture review.

#### 19. Utilize AWS Network Firewall Rule Capacity Optimization
- **What:** Right-size Network Firewall stateful rule group capacity units.
- **Why It Saves Money:** Prevents firewall endpoint over-provisioning.
- **Detailed Implementation Steps:**
  1. Optimize Network Firewall policy rules.
- **Estimated Savings:** Firewall capacity savings.
- **Risk Level:** Low.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** Network Firewall usage.

#### 20. Audit Reachability Analyzer & Network Access Analyzer Runs
- **What:** Delete completed Reachability Analyzer analysis runs.
- **Why It Saves Money:** Prevents diagnostic charge accumulation ($0.10 per analysis).
- **Detailed Implementation Steps:**
  1. Delete analysis runs via CLI.
- **Estimated Savings:** Diagnostic fee savings.
- **Risk Level:** Zero.
- **Implementation Scope:** Network Engineer
- **Prerequisites:** None.

---

## Cross-Service Synergies

```
[ VPC Private Subnet ] 
        │
        ├──(S3 & DynamoDB Traffic)─> [ Gateway VPC Endpoints ] (100% FREE - Bypasses NAT $0.045/GB)
        │
        ├──(AWS Service Traffic)───> [ Interface VPC Endpoints ] (78% discount vs NAT processing)
        │
        └──(Inter-VPC Traffic)─────> [ Free VPC Peering ] (100% FREE - Bypasses TGW $0.02/GB tax)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `NatGateway-Hours`, `NatGateway-Bytes`, `TransitGateway-Host-Hours`, `TransitGateway-Bytes`, `VpcEndpoint-Hours`, `VpcEndpoint-Bytes`, `PublicIPv4:InUseAddress`, `PublicIPv4:UnassignedAddress`.
- `line_item_resource_id`: NAT Gateway ID (`nat-xxxx`) / Transit Gateway Attachment ID (`tgw-attach-xxxx`).

### B. VPC Flow Logs & CloudWatch Metrics
- `AWS/NATGateway` Namespace: `BytesOutToDestination`, `BytesInFromSource`, `ActiveConnectionCount`.
- `AWS/TransitGateway` Namespace: `BytesIn`, `BytesOut`, `PacketDropCount`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "VPC-GW-001",
  "service": "VPC Networking",
  "category": "NAT Gateway Optimization",
  "resource_id": "arn:aws:ec2:us-east-1:123456789012:natgateway/nat-0a1b2c3d4e5f6g7h8",
  "resource_name": "prod-nat-az1",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "gateway_endpoints_deployed": false,
    "monthly_s3_data_processed_gb": 45000,
    "monthly_nat_processing_cost_usd": 2025.00
  },
  "recommended_config": {
    "action": "Deploy S3 & DynamoDB Gateway VPC Endpoints",
    "projected_monthly_nat_processing_cost_usd": 0.00
  },
  "financial_impact": {
    "monthly_savings_usd": 2025.00,
    "annual_savings_usd": 24300.00,
    "savings_percentage": 100.0
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Gateway Endpoints update route tables non-disruptively; 100% free native AWS routing feature."
  },
  "implementation": {
    "scope": "Network Engineer / DevOps",
    "effort_estimate": "15 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Gateway Endpoints (S3/DynamoDB)** | 18 | $24,500.00 | $24,500.00 | 100.0% | Zero |
| **VPC Peering vs Transit Gateway** | 8 | $18,200.00 | $18,200.00 | 100.0% | Low |
| **Single-AZ NAT in Non-Prod** | 12 | $4,730.00 | $3,153.60 | 66.6% | Low |
| **Unattached EIP & Public IPv4** | 25 | $2,400.00 | $2,400.00 | 100.0% | Zero |
| **Total** | **63** | **$49,830.00** | **$48,253.60** | **96.8%** | -- |
