# AWS Service Cost Research: AWS NAT Gateway

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS NAT Gateway is a fully managed Network Address Translation service that enables instances in a private subnet to connect to the internet or other AWS services, while preventing the internet from initiating connections with those instances. NAT Gateway is widely recognized as one of the most common sources of high networking bills in AWS due to its dual billing structure (hourly base fee + per-GB data processing fee) and the fact that it is often deployed redundantly across multiple Availability Zones.

---

## 2. Billing Mechanics
NAT Gateway billing is calculated on a monthly cycle based on four dimensions:
1. **NAT Gateway Hourly Base Fee:** A flat fee charged per hour or partial hour that the gateway is active.
2. **Public IPv4 / Elastic IP Surcharge:** Billed per hour for the mandatory public Elastic IP attached to the NAT Gateway ($0.005 per hour).
3. **Data Processing Fee:** Billed per GB of data processed (sent OR received) through the gateway.
4. **Network Egress Fees:** Standard AWS Internet Data Transfer Out rates apply for outbound traffic to the internet.

---

## 3. Key Cost Dimensions

### A. Hourly Flat Rates & Elastic IP (us-east-1)
* **Hourly Base Fee:** **$0.045 per hour** (~$32.85/month) per NAT Gateway.
* **Public IP Fee:** **$0.005 per hour** (~$3.65/month) for the mandatory Elastic IP attached to the gateway.
* **Baseline Idle Cost:** A single running NAT Gateway costs:
  $$\$0.045\text{ base} + \$0.005\text{ IP} = \$0.050\text{ / hour } (\sim\$36.50\text{ / month flat})$$
* **Multi-AZ Multiplier:** Deploying NAT Gateways across 3 AZs for high availability creates a flat baseline fee of **$109.50/month**, before processing any data.

### B. Data Processing Fee (The Cost Hotspot)
* **The Surcharge:** AWS bills **$0.045 per GB** for all data processed (upload and download) through the NAT Gateway.
* *Warning:* Unlike standard S3 or EC2 data transfer, where data *ingress* (upload) is free, NAT Gateway charges **$0.045/GB for both uploads and downloads**.
* *Example:* If a server behind a NAT Gateway downloads a 10 TB backup from S3, the NAT Gateway processing fee is:
  $$10,000\text{ GB} \times \$0.045 = \mathbf{\$450.00\text{ in processing fees}}$$
  This charge applies even if the destination service (e.g. S3) is in the same AWS region.

### C. Cross-AZ Routing Fees
* If an EC2 instance in AZ-A routes internet-bound traffic through a NAT Gateway in AZ-B, you are billed standard inter-AZ data transfer fees (**$0.01 per GB in each direction**) on top of the NAT Gateway processing fee ($0.045/GB) and standard internet egress ($0.09/GB).

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Component | Hourly Base Rate | Elastic IP Rate | Data Processing (/GB) | Monthly Base Cost (per unit) |
|-------------------|------------------|-----------------|-----------------------|------------------------------|
| **NAT Gateway** | **$0.045 / hour** | **$0.005 / hour**| **$0.045** | ~$36.50 / month |
| **HA (3 AZs)** | **$0.135 / hour** | **$0.015 / hour**| **$0.045** | ~$109.50 / month |

---

## 5. AWS Free Tier Coverage
* **AWS NAT Gateway:** No free tier is available. All running gateways and processed gigabytes generate standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
* **S3 and DynamoDB Data Pipelines:** Running data processing jobs (e.g. EMR, Glue, or python scripts) in private subnets that read/write terabytes of data directly from S3 or DynamoDB through a NAT Gateway. This adds a $0.045/GB tax on all database writes and reads.
* **ECR Image Pulls:** Spawning hundreds of Kubernetes pods or Fargate tasks daily that pull large container images from Amazon ECR through a NAT Gateway, billing $0.045/GB for the image transfer.
* **Dev/Test HA Redundancy:** Deploying separate NAT Gateways in every Availability Zone of a development or QA VPC. This is completely unnecessary, as dev systems rarely require multi-AZ high availability.
* **Continuous Log Uploads:** Streaming raw application logs to CloudWatch or third-party log providers through a NAT Gateway.

---

## 7. Actionable Cost Optimization Strategies
1. **Deploy Free VPC Gateway Endpoints for S3 & DynamoDB:**
   * Add Gateway Endpoints for S3 and DynamoDB in your VPC route tables.
   * This diverts all S3 and DynamoDB traffic away from the NAT Gateway, routing it internally for **free ($0.00/GB)**, instantly eliminating $0.045/GB processing fees.
2. **Use Interface VPC Endpoints for ECR, CloudWatch, and SSM:**
   * If private containers pull ECR images or send logs to CloudWatch, deploy **Interface VPC Endpoints** for ECR (`ecr.api` and `ecr.dkr`) and CloudWatch (`logs`).
   * This routes traffic internally, dropping data processing charges from the NAT Gateway rate (**$0.045/GB**) to the PrivateLink rate (**$0.01/GB**), a **78% direct savings**.
3. **Consolidate NAT Gateways in Non-Production:**
   * Dev/test VPCs do not require multi-AZ network failover.
   * Deploy a **single NAT Gateway** in AZ-A. Configure private subnet route tables in AZ-B and AZ-C to route their internet-bound traffic through the single NAT Gateway in AZ-A.
   * This cuts your monthly base charges by **66%** (saving ~$73/month per VPC).
4. **Use Public IPs (with IGW) for Non-Critical Servers:**
   * For instances that do not hold sensitive data (e.g., jump boxes or web proxies), deploy them in a public subnet with a public IP and route traffic directly through an Internet Gateway (IGW is free).
   * This avoids NAT Gateway compute and processing fees entirely (you only pay the $3.65/month public IP charge).
5. **Audit Egress Traffic with VPC Flow Logs:** Enable VPC Flow Logs and use Athena or CloudWatch Log Insights to identify the top IP endpoints generating data transfer through your NAT Gateways. Refactor those applications to use endpoints or private peering.
