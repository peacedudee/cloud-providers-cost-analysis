# AWS Service Cost Research: AWS VPC (Virtual Private Cloud)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS VPC lets you provision a logically isolated section of the AWS cloud where you can launch AWS resources. While creating a VPC and its subnets is **100% free**, the networking infrastructure components you deploy inside it (like NAT Gateways, Transit Gateways, VPNs, PrivateLink, and public IP addresses) generate heavy hourly and data-processing charges. VPC costs are a primary source of hidden billing leaks.

---

## 2. Billing Mechanics
VPC infrastructure charges are broken down into five distinct categories:
1.  **Network Address Translation (NAT Gateways):** Hourly base charges and per-GB data processing fees.
2.  **VPC Endpoints (PrivateLink):** Hourly fees per endpoint per Availability Zone and per-GB processing fees.
3.  **Transit Gateway:** Hourly attachment fees and per-GB processing charges.
4.  **Virtual Private Networks (VPNs):** Hourly connection charges for Site-to-Site and Client VPNs.
5.  **Public IPv4 Addresses:** Hourly surcharges for all assigned public IPs.

---

## 3. Key Cost Dimensions

### A. NAT Gateways (The Largest Cost Driver)
NAT Gateways allow resources in private subnets to connect to the internet.
*   **Hourly Base Fee:** **$0.045 per hour** (~$32.85/month) per gateway.
*   **Data Processing Fee:** **$0.045 per GB** of data transferred.
*   *Processing Pitfall:* If your server in a private subnet downloads a 10 TB database dump or dataset from the internet, the NAT Gateway processing fee alone will cost:
    $$10,000\text{ GB} \times \$0.045 = \$450.00\text{ in processing fees}$$
    This is in addition to standard AWS Internet Egress fees.

### B. VPC Endpoints (PrivateLink)
VPC Endpoints allow you to connect your VPC to supported AWS services privately.
*   **Interface Endpoints (PrivateLink):** Billed per Availability Zone.
    *   *Base Fee:* **$0.01 per endpoint hour** (~$7.30/month per AZ).
    *   *Processing Fee:* **$0.01 per GB** of data processed.
*   **Gateway Endpoints (S3 and DynamoDB only):** **100% Free** (no hourly fee, no processing fee).

### C. Transit Gateway (Multi-VPC Routing)
Transit Gateways connect multiple VPCs and on-premises networks together.
*   **Attachment Fee:** **$0.05 per hour** (~$36.50/month) for each attachment (VPC, VPN, or Direct Connect).
*   **Processing Fee:** **$0.02 per GB** of data processed.
*   *Warning:* Routing high-volume data pipelines (e.g. log analytics or backups) across VPCs via Transit Gateway adds a $0.02/GB tax on all traffic.

### D. Virtual Private Networks (VPNs)
*   **Site-to-Site VPN:** **$0.05 per hour** (~$36.50/month) per active connection.
*   **Client VPN:** **$0.15 per endpoint hour** (~$109.50/month) plus **$0.05 per client connection hour** for active VPN sessions.

### E. Public IPv4 Addresses
*   **The Charge:** AWS charges **$0.005 per hour** (~$3.65/month) for every public IPv4 address allocated in your account (attached to running EC2 instances, ENIs, NAT Gateways, or Load Balancers).

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Component | Hourly Base Rate | Data Processing (/GB) | Monthly Base Cost (Est.) |
|-------------------|------------------|-----------------------|--------------------------|
| **NAT Gateway** | $0.045 / hour | **$0.045** | ~$32.85 / gateway |
| **Interface Endpoint**| $0.010 / hour / AZ | **$0.010** | ~$7.30 / endpoint / AZ |
| **Gateway Endpoint** | **Free ($0.00)** | **Free ($0.00)** | **$0.00** (S3 / DynamoDB) |
| **Transit Gateway** | $0.050 / attachment hour| **$0.020** | ~$36.50 / attachment |
| **Site-to-Site VPN** | $0.050 / connection hour| Standard Egress rates | ~$36.50 / connection |
| **Client VPN Endpoint**| $0.150 / hour | N/A | ~$109.50 / endpoint |
| **Public IPv4 Address**| $0.005 / hour | N/A | ~$3.65 / IP |

---

## 5. AWS Free Tier Coverage
*   **Public IPv4:** 750 free hours/month of public IPv4 usage for the first 12 months.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Routing ECR Image Pulls / S3 Uploads via NAT Gateway:** Running containerized tasks on ECS/Fargate that pull large Docker images from ECR, or run S3 data pipelines, through a NAT Gateway. This incurs the $0.045/GB processing fee on all image and data blocks.
*   **Transit Gateway High Processing Bills:** Connecting high-volume database replica nodes in different VPCs through Transit Gateway, incurring $0.02/GB on all sync traffic.
*   **Unused Client VPN Endpoints Left Active:** Keeping Client VPN endpoints provisioned in dev/staging accounts when developers are not actively using them, accumulating $0.15/hour.
*   **Unused NAT Gateways in Development Subnets:** Deploying a NAT Gateway in every Availability Zone of a dev VPC. Having 3 NAT Gateways idle adds **~$100.00/month** in base charges.

---

## 7. Actionable Cost Optimization Strategies
1.  **Deploy Free S3 & DynamoDB Gateway Endpoints:** Configure **Gateway VPC Endpoints** for S3 and DynamoDB in all routing tables. This immediately diverts S3 and DynamoDB traffic away from your NAT Gateways, routing it directly and securely for **free** (saving $0.045/GB processing fees).
2.  **Deploy Interface Endpoints for High-Volume Services (e.g. ECR):** If your private ECS/EKS tasks frequently pull images or send logs to CloudWatch:
    *   Deploy **Interface Endpoints** for ECR (`ecr.api` and `ecr.dkr`) and CloudWatch (`logs`).
    *   This routes traffic internally, dropping data processing charges from the NAT Gateway rate (**$0.045/GB**) to the PrivateLink rate (**$0.01/GB**), a **78% savings**.
3.  **Consolidate NAT Gateways in Non-Production:** Dev and test environments do not require high availability. Instead of deploying a NAT Gateway in each AZ, deploy a **single NAT Gateway** in AZ-A and route internet-bound traffic from private subnets in AZ-B and AZ-C through it. This cuts base NAT Gateway costs by **66%**.
4.  **Use VPC Peering Instead of Transit Gateway for High-Volume Links:** VPC Peering is **free to set up** and has **$0.00/GB data processing fees**. For high-bandwidth data transfers between two specific VPCs (like database replication), establish a direct VPC Peering connection instead of routing through Transit Gateway.
5.  **Shutdown Client VPN Endpoints Outside Office Hours:** Set up automation scripts or use AWS CloudFormation/CLI to delete Client VPN endpoints during weekends and recreate them on Monday morning.
6.  **Release Unattached Elastic IPs:** Review and release any unattached Elastic IPs to avoid the $0.005/hour idle IP charge.
