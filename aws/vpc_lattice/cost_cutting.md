# Cost-Cutting Playbook: Amazon VPC Lattice
> **Companion File:** [vpc_lattice.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/vpc_lattice/vpc_lattice.md)
> **Last Updated:** July 2026

---

## Executive Summary
Amazon VPC Lattice simplifies connecting, securing, and monitoring application layer (L7) communication across VPCs and AWS accounts. While its pricing model is advantageous for microservices compared to legacy alternatives like NAT Gateway or PrivateLink, inefficient usage—such as routing heavy data streams or over-provisioning service registries—can lead to unexpected costs. This playbook provides 20 targeted strategies across seven categories to optimize VPC Lattice usage, reduce hourly service fees, and minimize data processing charges.

---

## Strategy Categories

### 1. Waste Elimination
*   **VPCL-001: Delete Inactive Services:** Implement automation to scan the VPC Lattice service directory and automatically delete services that have registered `0` active target endpoints or received zero requests in the last 14 days, saving ~$18.25/month per service.
*   **VPCL-002: Remove Stale Target Groups:** Identify and remove target groups that are not actively associated with any VPC Lattice listener rules or services.
*   **VPCL-003: Clean Up Unused Service Networks:** Delete Service Networks that have no associated services or VPCs attached to prevent administrative bloat and potential accidental provisioning.
*   **VPCL-004: Eradicate Orphaned Test Registries:** Implement robust teardown steps in CI/CD pipelines to ensure temporary dev/test service registries spun up for autoscale testing are deleted post-deployment.

### 2. Rightsizing
*   **VPCL-005: Consolidate Services via Path Routing:** Combine multiple small microservices behind a single VPC Lattice parent service (e.g., `internal-api`) using path-based routing (e.g., `/users`, `/orders`). Consolidating 10 services into 1 cuts fixed hourly charges by 90% (from $182.50/mo to $18.25/mo).
*   **VPCL-006: Optimize Health Check Intervals:** Tune health check intervals and thresholds to be less aggressive for non-critical targets, reducing unnecessary request volume and log generation overhead.
*   **VPCL-007: Review Target Registration Boundaries:** Avoid registering the same backend targets across multiple overlapping VPC Lattice services if not explicitly required, centralizing them instead to minimize the number of billed service hours.

### 3. Commitment Discounts
*   **VPCL-008: AWS Enterprise Discount Program (EDP):** Since VPC Lattice does not offer native Savings Plans or Reserved Instances, incorporate large-scale VPC Lattice data processing volume into overall AWS EDP negotiations for private pricing.
*   **VPCL-009: Maximize Free Tier Allowance:** Explicitly utilize the AWS Free Tier allowance (750 service-hours, 10 GB data processed, 100,000 requests per month) for new accounts by restricting sandbox registries to a single service.

### 4. Architecture Changes
*   **VPCL-010: Route L4 Heavy Data via VPC Peering:** Shift bulk data transfers, database replication, and large media streams to standard VPC Peering ($0.00/GB data processing) instead of VPC Lattice to avoid the $0.025/GB L7 processing tax.
*   **VPCL-011: Replace NAT Gateway for Internal App Traffic:** Migrate inter-VPC application communication from NAT Gateways to VPC Lattice, saving 50% on fixed hourly costs and 44% on per-GB data processing.
*   **VPCL-012: Substitute PrivateLink in Multi-AZ Setups:** Use VPC Lattice instead of PrivateLink endpoints for multi-VPC meshes. The fixed cost for a baseline Lattice service is 78% cheaper than deploying PrivateLink interface endpoints across two Availability Zones.
*   **VPCL-013: Replace Transit Gateway for Pure L7 Workloads:** Migrate pure HTTP/HTTPS/gRPC microservice communication from Transit Gateway to VPC Lattice to eliminate complex networking overhead and per-VPC attachment fees.

### 5. Scheduling & Auto-Scaling
*   **VPCL-014: Tear Down Non-Prod Services Off-Hours:** Use Infrastructure as Code (IaC) to delete VPC Lattice services in dev/test environments overnight and over weekends, recreating them automatically in the morning to save ~65% of service-hour costs.
*   **VPCL-015: Scale Backend Targets to Zero:** Ensure backend Auto Scaling groups or ECS/EKS tasks registered with Lattice target groups scale to zero when not in use during off-hours, significantly reducing underlying compute costs.

### 6. Pricing Model Optimization
*   **VPCL-016: Implement Client-Side Caching:** Add caching mechanisms (e.g., Redis or in-memory caches) on client applications making calls through Lattice to reduce the $0.10 per million requests fee and the associated data processed charges.
*   **VPCL-017: Batch API Requests:** Modify client applications to batch multiple small API requests into a single larger request, optimizing request volume efficiency while maintaining the same payload size.

### 7. Network & Data Transfer Optimization
*   **VPCL-018: Compress Payloads:** Implement GZIP or Brotli compression on HTTP/HTTPS responses at the application layer to minimize the raw GBs measured for VPC Lattice Data Processed charges ($0.025/GB).
*   **VPCL-019: Filter Extraneous Telemetry:** Prevent verbose debug logs, high-frequency metrics, or raw tracing data from routing across VPC Lattice, keeping heavy logging directed to local VPC endpoints instead.
*   **VPCL-020: Optimize Cross-AZ Traffic:** Ensure VPC Lattice target groups and client routing configurations prioritize resources in the same Availability Zone to avoid standard AWS cross-AZ data transfer fees ($0.01/GB).

---

## Cross-Service Synergies
*   **Amazon EC2 & ECS/EKS:** Proper auto-scaling of backend compute nodes behind VPC Lattice ensures you aren't paying for idle targets while Lattice remains active.
*   **AWS CloudWatch:** Heavy usage of VPC Lattice access logging can lead to high CloudWatch ingestion costs. Optimize log retention and filter out successful (HTTP 200) requests if not needed for compliance.
*   **AWS Transit Gateway & VPC Peering:** Use a hybrid approach—VPC Lattice for granular L7 microservices and TGW/Peering for heavy L4 data flows to achieve the optimal balance of security and cost.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
*   `lineItem/ProductCode` = `AmazonVPCLattice`
*   `lineItem/Operation` (e.g., `CreateService`, `DataProcessing`, `RequestProcessing`)
*   `lineItem/UsageType` (e.g., `DataProcessing-Bytes`, `Request-Count`, `Service-Hours`)
*   Resource IDs for services and service networks.

### B. CloudWatch Metrics
*   `RequestCount`, `BytesProcessed`, `TargetResponseTime`, and `ActiveFlowCount` to identify idle or low-usage VPC Lattice services.
*   Metrics indicating backend 5xx errors or zero healthy targets.

### C. AWS Config / Trusted Advisor
*   Configurations for VPC Lattice Service Networks, Services, and Target Groups.
*   Detection of unused or orphaned Lattice resources without attached VPCs.

### D. Company Policies
*   Security and compliance requirements dictating L7 segmentation and access logging via IAM policies in VPC Lattice.
*   Environment uptime requirements (identifying dev/test for scheduled teardowns).

### E. IaC (Optional)
*   Terraform state files or CloudFormation templates to identify how services are structured (e.g., single service vs. path-based routing).

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "VPCL-005",
  "resource_id": "svc-0abcd1234efgh5678",
  "account_id": "123456789012",
  "region": "us-east-1",
  "category": "Rightsizing",
  "strategy": "Consolidate Services via Path Routing",
  "current_state": "10 separate microservices registered as individual Lattice services.",
  "recommended_state": "Consolidate into 1 parent service using path-based routing rules.",
  "estimated_monthly_savings": 164.25,
  "effort_level": "Medium",
  "risk_level": "Low"
}
```

### Summary Report Table

| Finding ID | Strategy | Target Resource | Est. Monthly Savings | Effort | Risk |
|------------|----------|-----------------|----------------------|--------|------|
| VPCL-001 | Delete Inactive Services | svc-idle001 | $18.25 | Low | Low |
| VPCL-005 | Consolidate via Path Routing | svc-cluster-X | $164.25 | Medium | Low |
| VPCL-010 | Route L4 Data via Peering | sn-data-heavy | $350.00 | High | Med |
| VPCL-014 | Dev/Test Scheduled Teardown| svc-dev-api | $11.80 | Low | Low |
| VPCL-018 | Compress Payloads (GZIP) | tg-api-responses| $85.00 | Low | Low |
