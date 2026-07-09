# Cost-Cutting Playbook: Amazon WorkSpaces Secure Browser
> **Companion File:** [workspaces_secure_browser.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/workspaces_secure_browser/workspaces_secure_browser.md)
> **Last Updated:** July 2026

---

## Executive Summary
Amazon WorkSpaces Secure Browser (formerly WorkSpaces Web) provides a highly cost-effective, fully managed solution for securing access to internal corporate web applications from unmanaged devices. Billed primarily on a Monthly Active User (MAU) basis at just $7.00/month (including 200 streaming hours), it represents a significant savings opportunity when used to replace full Windows VDI desktops for task workers. This playbook outlines strategies to maximize the ROI of WorkSpaces Secure Browser by migrating appropriate workloads, minimizing overage streaming hours, consolidating portals to avoid duplicate billing, and optimizing network routing to prevent hidden data transfer costs.

---

## Strategy Categories

### 1. Waste Elimination
Eliminating waste in WorkSpaces Secure Browser revolves around preventing users from accidentally leaving streaming sessions open, which can burn through the 200 included hours and incur overage fees ($0.035/hour).
*   **Set Strict Idle Disconnects:** Configure the portal to automatically disconnect sessions after a short period of inactivity (e.g., 15-30 minutes).
*   **Enforce Maximum Session Durations:** Force a hard disconnect after a typical workday (e.g., 9 hours) to ensure sessions left open overnight are terminated.
*   **Delete Unused Portals:** Remove stale or abandoned web portals, as active users logging into multiple test portals may incur duplicate MAU charges.
*   **Block High-Bandwidth Non-Business Sites:** Use URL filtering policies to prevent streaming Netflix or YouTube, which inflates session times and VPC data egress.

### 2. Rightsizing
Rightsizing involves ensuring the service is used by the right personas instead of more expensive alternatives.
*   **Replace Full VDI Desktops:** Migrate contractors, BPO agents, and web-only task workers from Amazon WorkSpaces ($33/mo) to Secure Browser ($7/mo), saving 78% per user.
*   **Replace AppStream 2.0 for Intranet:** If AppStream 2.0 is only being used to deliver an internal web browser, Secure Browser is typically more cost-effective and easier to manage.
*   **Restrict Audio/Video Features:** If users only need to access text-based internal CRMs or wikis, disable audio/video redirection in the browser policies to reduce resource overhead and network transfer.

### 3. Commitment Discounts
Since Secure Browser is billed per MAU, it does not support EC2 Savings Plans or Reserved Instances, but other discount mechanisms apply.
*   **Utilize Free Tier for PoCs:** Maximize the 30-day free trial ($7 credit per user for up to 30 users) to validate the service before committing to enterprise rollout.
*   **Negotiate Private Pricing (EDP):** For organizations deploying thousands of MAUs, engage AWS account teams to include Secure Browser in an Enterprise Discount Program (EDP) for volume discounts.

### 4. Architecture Changes
Architectural changes help reduce the hidden network costs associated with Secure Browser instances accessing resources inside your VPC.
*   **Consolidate Web Portals:** Ensure the entire organization uses a single identity-integrated portal where possible to prevent a single human from generating multiple MAU charges across different portals.
*   **Use VPC Endpoints (PrivateLink):** Ensure the VPC subnets associated with Secure Browser use VPC Endpoints to access AWS services (like S3 or DynamoDB web interfaces) to avoid NAT Gateway data processing charges.
*   **Optimal Region Deployment:** Deploy portals in regions geographically closest to the end users. This minimizes latency and potential cross-region data transfer costs if users are routed inefficiently.

### 5. Scheduling & Auto-Scaling
While the service is fully managed and auto-scales instances under the hood, access scheduling can limit overages.
*   **IdP Time-of-Day Controls:** Use your Identity Provider (e.g., Okta, Entra ID) to enforce conditional access policies that only allow Secure Browser logins during authorized business hours, inherently preventing late-night overage streaming.

### 6. Pricing Model Optimization
Optimizing the MAU pricing model.
*   **Monitor Overage Trends:** Continuously track the "Streaming Hours" CloudWatch metrics. If a specific user consistently exceeds 200 hours and approaches 740 hours (costing ~$25.90/mo), evaluate if a fixed-price desktop is more suitable, though Secure Browser usually remains cheaper than full VDI.

### 7. Network & Data Transfer Optimization
Data egress from the Secure Browser VPC to the internet or internal networks can accumulate.
*   **Avoid NAT Gateway for Internal Traffic:** Route internal corporate web traffic over AWS Transit Gateway or Direct Connect rather than pushing it through a NAT Gateway.
*   **Localize DNS Resolution:** Use Amazon Route 53 Resolver endpoints efficiently to prevent excessive external DNS query costs generated by Secure Browser sessions.

---

## Cross-Service Synergies
*   **Amazon WorkSpaces:** Use Secure Browser for task workers and reserve full Amazon WorkSpaces solely for power users or developers requiring thick clients.
*   **AWS IAM Identity Center:** Centralize access to Secure Browser to ensure offboarded employees instantly lose access, preventing trailing MAU charges.
*   **AWS VPC / NAT Gateway:** Optimize Secure Browser subnets to prevent unexpected NAT Gateway data processing spikes caused by heavy web browsing.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
*   `lineItem/ProductCode` = `AmazonWorkSpacesWeb` (Note: Product code may still reference 'Web').
*   `lineItem/UsageType` = `*-MAU` or `*-Overage-Streaming-Hours`.
*   `lineItem/UnblendedCost` to calculate total spend.

### B. CloudWatch Metrics
*   **Namespace:** `AWS/WorkSpacesWeb`
*   **Metrics:** `StreamingHours`, `ActiveUsers` to correlate usage with overage billing.

### C. AWS Config / Trusted Advisor
*   Check portal configuration details (Idle Disconnect Timeout, Max Session Duration).
*   Check associated VPC subnets and routing tables for NAT Gateway reliance.

### D. Company Policies
*   Acceptable Use Policy (to determine if non-business video streaming can be aggressively blocked).
*   Remote Worker Device Policy (BYOD vs Corporate Managed).

### E. IaC (Optional)
*   Terraform (`aws_workspacesweb_portal`) to audit configured session policies and user access controls.

---

## Output Schema

### Finding Record (JSON)
```json
[
  {
    "finding_id": "WKSWEB-WST-001",
    "category": "Waste Elimination",
    "description": "Enforce strict idle session disconnect timeouts to prevent users from consuming overage streaming hours ($0.035/hr).",
    "estimated_savings_percentage": 5,
    "effort": "Low"
  },
  {
    "finding_id": "WKSWEB-WST-002",
    "category": "Waste Elimination",
    "description": "Set maximum session durations to ensure sessions are forcefully closed overnight.",
    "estimated_savings_percentage": 5,
    "effort": "Low"
  },
  {
    "finding_id": "WKSWEB-WST-003",
    "category": "Waste Elimination",
    "description": "Delete abandoned or unused Secure Browser portals to prevent accidental duplicate MAU charges.",
    "estimated_savings_percentage": 2,
    "effort": "Low"
  },
  {
    "finding_id": "WKSWEB-WST-004",
    "category": "Waste Elimination",
    "description": "Block high-bandwidth non-business streaming sites via URL filtering to reduce session time and VPC egress costs.",
    "estimated_savings_percentage": 8,
    "effort": "Medium"
  },
  {
    "finding_id": "WKSWEB-RIGHT-001",
    "category": "Rightsizing",
    "description": "Migrate web-only workers from full Amazon WorkSpaces ($33/mo) to Secure Browser ($7/mo).",
    "estimated_savings_percentage": 78,
    "effort": "High"
  },
  {
    "finding_id": "WKSWEB-RIGHT-002",
    "category": "Rightsizing",
    "description": "Replace expensive AppStream 2.0 instances used merely for intranet browsing with Secure Browser.",
    "estimated_savings_percentage": 60,
    "effort": "Medium"
  },
  {
    "finding_id": "WKSWEB-RIGHT-003",
    "category": "Rightsizing",
    "description": "Restrict audio/video features in browser policies for text-only task workers to lower resource overhead.",
    "estimated_savings_percentage": 3,
    "effort": "Low"
  },
  {
    "finding_id": "WKSWEB-RIGHT-004",
    "category": "Rightsizing",
    "description": "Analyze users consistently generating overage hours to ensure Secure Browser remains the most cost-effective option.",
    "estimated_savings_percentage": 5,
    "effort": "Medium"
  },
  {
    "finding_id": "WKSWEB-COMM-001",
    "category": "Commitment Discounts",
    "description": "Utilize the 30-day free trial ($7 credit per user for up to 30 users) for initial PoC deployments.",
    "estimated_savings_percentage": 100,
    "effort": "Low"
  },
  {
    "finding_id": "WKSWEB-COMM-002",
    "category": "Commitment Discounts",
    "description": "Negotiate an Enterprise Discount Program (EDP) with AWS for high-volume Secure Browser deployments.",
    "estimated_savings_percentage": 10,
    "effort": "High"
  },
  {
    "finding_id": "WKSWEB-ARCH-001",
    "category": "Architecture Changes",
    "description": "Consolidate multiple portals to prevent a single user from incurring multiple $7 MAU charges.",
    "estimated_savings_percentage": 15,
    "effort": "Medium"
  },
  {
    "finding_id": "WKSWEB-ARCH-002",
    "category": "Architecture Changes",
    "description": "Deploy portals in regions geographically closest to users to minimize potential cross-region data transfer costs.",
    "estimated_savings_percentage": 4,
    "effort": "Medium"
  },
  {
    "finding_id": "WKSWEB-SCH-001",
    "category": "Scheduling & Auto-Scaling",
    "description": "Implement Time-of-Day access controls in the IdP to prevent streaming sessions outside of business hours.",
    "estimated_savings_percentage": 12,
    "effort": "Medium"
  },
  {
    "finding_id": "WKSWEB-PRICE-001",
    "category": "Pricing Model Optimization",
    "description": "Offboard terminated employees from the IdP immediately to prevent trailing MAU billing.",
    "estimated_savings_percentage": 5,
    "effort": "Low"
  },
  {
    "finding_id": "WKSWEB-NET-001",
    "category": "Network & Data Transfer Optimization",
    "description": "Implement VPC Endpoints (PrivateLink) in Secure Browser subnets to access AWS services without NAT Gateway fees.",
    "estimated_savings_percentage": 10,
    "effort": "Medium"
  },
  {
    "finding_id": "WKSWEB-NET-002",
    "category": "Network & Data Transfer Optimization",
    "description": "Route intranet traffic via Direct Connect or Transit Gateway instead of NAT Gateways.",
    "estimated_savings_percentage": 8,
    "effort": "Medium"
  },
  {
    "finding_id": "WKSWEB-NET-003",
    "category": "Network & Data Transfer Optimization",
    "description": "Configure local Route 53 Resolver endpoints to optimize internal DNS resolution costs for Secure Browser sessions.",
    "estimated_savings_percentage": 2,
    "effort": "Low"
  }
]
```

### Summary Report Table

| Category | Strategy | Effort | Estimated Savings |
|---|---|---|---|
| Waste Elimination | Enforce strict idle disconnects | Low | ~5% |
| Waste Elimination | Set maximum session durations | Low | ~5% |
| Waste Elimination | Delete unused portals | Low | ~2% |
| Waste Elimination | Block non-business streaming | Medium | ~8% |
| Rightsizing | Replace full VDI ($33) with Secure Browser ($7) | High | ~78% / user |
| Rightsizing | Replace AppStream 2.0 | Medium | ~60% |
| Rightsizing | Restrict audio/video features | Low | ~3% |
| Rightsizing | Analyze overage hours for better fit | Medium | ~5% |
| Commitment | Utilize Free Tier for PoCs | Low | 100% of PoC |
| Commitment | Negotiate EDP for high volume | High | ~10% |
| Architecture | Consolidate web portals | Medium | ~15% |
| Architecture | Deploy in optimal regions | Medium | ~4% |
| Scheduling | Implement IdP Time-of-Day controls | Medium | ~12% |
| Pricing | Immediate IdP offboarding | Low | ~5% |
| Network | Use VPC Endpoints to avoid NAT charges | Medium | ~10% |
| Network | Route internal traffic efficiently | Medium | ~8% |
| Network | Optimize internal DNS resolution | Low | ~2% |
