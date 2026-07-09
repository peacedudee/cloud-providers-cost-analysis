# AWS Service Cost Research: AWS PrivateLink

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS PrivateLink is a highly secure, private network technology that connects your VPCs to AWS services, partner services, and customer services without exposing traffic to the public internet. It establishes a private connection using **Interface VPC Endpoints** (which deploy Elastic Network Interfaces inside your subnets). PrivateLink is a primary cost containment tool when replacing high-cost NAT Gateways, but it generates its own hourly and data processing fees that must be managed.

---

## 2. Billing Mechanics
PrivateLink is billed on a monthly cycle based on two dimensions:
1.  **VPC Endpoint Hourly Base Fee:** Billed per hour for each Availability Zone in which an endpoint is active.
2.  **Data Processing Fee:** Billed per GB of data processed through the endpoint.

---

## 3. Key Cost Dimensions

### A. Interface VPC Endpoints (us-east-1 Tiers)
*   **Hourly Base Fee:** Billed per endpoint per Availability Zone.
    *   *First 200 hours/month:* **$0.01 per hour** (~$7.30/month per AZ).
    *   *Next 800 hours/month:* **$0.007 per hour**.
    *   *Beyond 1,000 hours/month:* **$0.005 per hour**.
*   **Data Processing Fee (Tiered):**
    *   *First 1 Petabyte (PB)/month:* **$0.01 per GB** (compared to $0.045/GB for NAT Gateway, representing a **78% direct processing discount**).
    *   *Next 4 PB/month:* **$0.005 per GB**.
    *   *Beyond 5 PB/month:* **$0.003 per GB**.

### B. Gateway Endpoints (The Free Exception)
*   Gateway Endpoints are VPC endpoints configured strictly for **Amazon S3** and **Amazon DynamoDB**.
*   **The Cost:** **100% Free** (no hourly endpoint fees, no data processing fees).
*   *Optimization Note:* Always use Gateway Endpoints for S3 and DynamoDB instead of Interface Endpoints unless routing from on-premises over VPN/Direct Connect (which requires Interface Endpoints).

---

## 4. Detailed Pricing Rates (us-east-1 Baseline)

| Endpoint Type | Hourly Base Rate (per AZ) | Data Processing Rate (/GB) | Notes / Exceptions |
|---------------|---------------------------|----------------------------|--------------------|
| **Interface Endpoint**| **$0.0100** | **$0.0100** | Most AWS services & partners |
| **Gateway Endpoint** | **Free ($0.00)** | **Free ($0.00)** | Only S3 and DynamoDB |

---

## 5. AWS Free Tier Coverage
*   **AWS PrivateLink:** No free tier is available. Interface endpoint provisioning generates standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Deploying Interface Endpoints in Every Subnet/AZ:** Provisioning an Interface Endpoint across 3 Availability Zones for a low-traffic service. Having 3 endpoint instances generates:
    $$3\text{ AZs} \times 730\text{ hours} \times \$0.01 = \$21.90\text{ / month base fee}$$
    Doing this for 10 separate AWS services adds **$219.00/month** in base charges before any data is sent.
*   **Using Interface Endpoints for S3 instead of Gateway Endpoints:** Deploying an Interface Endpoint for S3 (`com.amazonaws.us-east-1.s3`) to handle local VPC S3 traffic, unnecessarily paying the $0.01/hr and $0.01/GB processing charges.
*   **PrivateLink Data Ingestion Surcharges:** Deploying PrivateLink to ingest massive external logs or telemetry (e.g. 100 TB/month) from client VPCs, generating $1,000/month in processing fees.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enforce S3 & DynamoDB Gateway Endpoints Globally:** Make sure all VPC route tables are updated with free S3 and DynamoDB **Gateway Endpoints**. Never use S3 Interface Endpoints for local VPC routing.
2.  **Consolidate Interface Endpoints (Avoid Multi-AZ Proliferation):**
    *   For development and staging environments, do not deploy Interface Endpoints in multiple AZs. Deploy them in a **single AZ** and update private DNS to point to that single interface. This cuts base hourly charges by **66%**.
3.  **Share PrivateLink Endpoints in Multi-VPC setups:** Instead of creating an endpoint for the same service (e.g., ECR or Systems Manager) in 10 different VPCs:
    *   Deploy the Interface Endpoints in a single **Services Hub VPC**.
    *   Connect the spoke VPCs to the hub using VPC Peering (which has free processing).
    *   Route DNS queries to the centralized endpoint. This cuts base hourly endpoint fees by **90%**.
4.  **Monitor Cross-Endpoint Traffic:** Verify that the amount of data processed justifies the hourly base endpoint charge. If an endpoint processes less than 730 GB per month, it may be cheaper to route traffic through a shared NAT Gateway (though security profiles must be considered).
