# Cost-Cutting Playbook: AWS Verified Access
> **Companion File:** [verified_access.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/verified_access/verified_access.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Verified Access provides a managed Zero Trust alternative to traditional VPNs, but its pricing model—billed per application-hour and data processed—can lead to significant cost overruns if not managed carefully. This playbook outlines 20 actionable strategies to optimize costs by consolidating endpoints, automating off-hours disassociation, avoiding heavy data processing, and strategically evaluating alternative access methods like AWS Client VPN or ALB Authentication. By applying these strategies, organizations can maintain a strong security posture while preventing runaway infrastructure costs.

## Strategy Categories
### 1. Waste Elimination
*   **AVA-WST-001: Identify and Remove Idle Endpoints** – Terminate Verified Access endpoints that have zero traffic over a 30-day period.
*   **AVA-WST-002: Delete Unattached Applications** – Remove Verified Access application configurations that are no longer associated with active endpoints.
*   **AVA-WST-003: Clean Up Redundant Test Environments** – Ensure developer sandboxes and test environments are cleaned up after use to avoid accumulating $0.27/hour fees.
*   **AVA-WST-004: Audit Overlapping Access Methods** – Ensure applications aren't accessible via both Client VPN and AVA, eliminating duplicate infrastructure costs for the same service.

### 2. Rightsizing
*   **AVA-RGT-001: Application Necessity Evaluation** – Before onboarding an internal tool to AVA, assess if it requires the strict device posture checks AVA provides, or if network-level isolation is sufficient.

### 3. Commitment Discounts
*   **AVA-COM-001: Enterprise Discount Program (EDP)** – While Verified Access doesn't offer specific Savings Plans or Reserved Instances, usage contributes to and can be discounted by overall AWS EDP agreements.

### 4. Architecture Changes
*   **AVA-ARC-001: Consolidate Applications via Reverse Proxy** – Deploy a single consolidated proxy (e.g., NGINX) behind one AVA endpoint and use path-based routing for sub-tools, saving ~$197/month per consolidated app.
*   **AVA-ARC-002: Bypass AVA for Public Assets** – Route public or unauthenticated assets (CSS, JS, images) via a CDN or public ALB to avoid AVA's $0.02/GB processing fee.
*   **AVA-ARC-003: Revert to Client VPN for High-Endpoint Scenarios** – If protecting dozens of independent microservices, an AWS Client VPN ($146/mo base) may be vastly cheaper than paying ~$197/mo per app for AVA.
*   **AVA-ARC-004: Centralize AVA via AWS RAM** – Share Verified Access endpoints across multiple AWS accounts using Resource Access Manager to avoid duplicating endpoints in every account.
*   **AVA-ARC-005: Use VPC Endpoints for M2M Traffic** – Ensure machine-to-machine traffic uses VPC Endpoints or Peering instead of routing through AVA, which is designed for human-to-app access.
*   **AVA-ARC-006: Offload Heavy File Transfers** – Avoid routing large database dumps or backups through AVA; use S3 pre-signed URLs or dedicated transfer mechanisms to bypass data processing fees.
*   **AVA-ARC-007: Substitute with ALB Authentication** – For simple internal apps that only need identity verification (OIDC) without device posture checks, use an Application Load Balancer with built-in auth to save on hourly application fees.

### 5. Scheduling & Auto-Scaling
*   **AVA-SCH-001: Disassociate Non-Production Apps Off-Hours** – Automate the disassociation of dev/test applications from endpoints at 7 PM and re-associate them at 7 AM to save ~50% on hourly costs.
*   **AVA-SCH-002: Ephemeral Endpoint Cleanup** – Ensure CI/CD pipelines automatically tear down ephemeral AVA endpoints when preview environments are destroyed.

### 6. Pricing Model Optimization
*   **AVA-PRC-001: Monitor Application-Hour Tiers** – Track cumulative application-hours; usage beyond 9,999 hours drops to $0.23, and beyond 74,400 hours drops to $0.20. Centralizing AVA into a single payer account maximizes these tier discounts.

### 7. Network & Data Transfer Optimization
*   **AVA-NET-001: Enable Backend Compression** – Enable gzip or Brotli compression on the backend applications to reduce the volume of data processed by AVA, directly lowering the $0.02/GB fee.
*   **AVA-NET-002: Implement Client-Side Caching** – Use aggressive cache-control headers so user browsers cache assets, reducing repetitive data requests through AVA.
*   **AVA-NET-003: Co-locate AVA and Backend Targets** – Deploy AVA endpoints in the same region as the backend targets to avoid cross-region data transfer out fees on top of AVA processing fees.
*   **AVA-NET-004: Optimize Payload Sizes** – Minify API responses and optimize internal application data payloads to minimize total GB processed.

---
## Cross-Service Synergies
*   **AWS Client VPN:** Compare costs continuously; AVA is cost-effective for a few applications, but Client VPN scales better economically for a high number of endpoints.
*   **Application Load Balancer (ALB):** ALB OIDC authentication is a cheaper alternative for applications that do not require Zero Trust device posture checks.
*   **AWS Resource Access Manager (RAM):** Used to share AVA endpoints across multi-account environments to minimize duplicated application-hour fees.
*   **S3 & CloudFront:** Offload static asset delivery to these services to avoid AVA data processing fees.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
*   `lineItem/ProductCode` = `AmazonVerifiedAccess`
*   `lineItem/UsageType` filters for `AppHours` and `DataProcessed`
### B. CloudWatch Metrics
*   `DataProcessed` and `BytesOut` metrics for endpoints to identify high-bandwidth applications.
### C. AWS Config / Trusted Advisor
*   Resource configurations for active Endpoints and Application associations.
### D. Company Policies
*   Security requirements detailing which applications strictly require device posture checks versus standard identity checks.
### E. IaC (Optional)
*   Terraform state or CloudFormation templates to identify opportunities for consolidating applications behind a single endpoint.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "AVA-ARC-001",
  "resource_id": "va-app-0a1b2c3d4e5f6g7h8",
  "account_id": "123456789012",
  "region": "us-east-1",
  "strategy_category": "Architecture Changes",
  "title": "Consolidate Applications via Reverse Proxy",
  "description": "5 individual tools are configured with separate AVA applications. Consolidate them behind a single proxy to save on base hourly fees.",
  "estimated_monthly_savings_usd": 788.40,
  "effort_level": "Medium",
  "action_required": "Deploy a reverse proxy (e.g., NGINX) for sub-path routing and reconfigure AVA to target the proxy."
}
```

### Summary Report Table
| Finding ID | Resource ID | Strategy | Estimated Savings / Mo | Effort | Action |
|------------|-------------|----------|------------------------|--------|--------|
| AVA-ARC-001 | va-app-... | Consolidate Applications | $788.40 | Medium | Deploy reverse proxy |
| AVA-SCH-001 | va-app-... | Off-hours Disassociation | $98.55 | Low | Implement auto-shutdown |
| AVA-WST-001 | va-ep-...  | Remove Idle Endpoints | $197.10 | Low | Terminate endpoint |
