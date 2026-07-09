# Cost-Cutting Playbook: AWS Shield
> **Companion File:** [shield.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/shield/shield.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Shield provides essential DDoS protection for your infrastructure. Shield Standard is 100% free and active on all AWS accounts, mitigating common Layer 3/4 attacks. Shield Advanced costs a flat $3,000/month per organization, plus Data Transfer Out (DTO) fees, providing enterprise L7 mitigation, free AWS WAF, and DDoS cost reimbursement. Cost optimization for Shield primarily focuses on avoiding unnecessary Advanced subscriptions, maximizing the financial benefits (like free WAF) if subscribed, and shifting security perimeters to edge services like CloudFront to reduce origin scaling costs. This playbook outlines 17 strategies to optimize your AWS Shield and DDoS mitigation costs.

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
- **Amazon CloudFront & Route 53:** Placing these edge services in front of regional resources natively absorbs massive volumetric DDoS attacks at no extra compute cost, leveraging Shield Standard natively.
- **AWS WAF:** Shield Advanced includes AWS WAF usage at no additional cost for protected resources. WAF rate-limiting is often a cheaper alternative to Shield Advanced for L7 protection.
- **AWS Firewall Manager:** Included for free with Shield Advanced, enabling centralized management of WAF and Shield policies across the organization.
- **AWS Organizations:** Essential for consolidating the $3,000/month Shield Advanced fee across multiple accounts.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query `lineItem/ProductCode` for `AWSShield` and `AWSDDoSProtection`.
- Analyze `lineItem/UsageType` for `Shield-Adv-Request` and DTO metrics.
### B. CloudWatch Metrics
- Monitor `DDoSDetected` metrics in the `AWS/DDoSProtection` namespace.
- Review WAF `BlockedRequests` vs `AllowedRequests`.
### C. AWS Config / Trusted Advisor
- Check for "AWS Shield Advanced Protection on Resources".
- Review WAF configuration and rule deployments.
### D. Company Policies
- Determine strict compliance requirements (e.g., PCI-DSS, SOC2) that may mandate Shield Advanced or 24/7 SRT access.
### E. IaC (Optional)
- Terraform/CloudFormation states to map currently protected resources and WAF ACL associations.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "SHIELD-001",
  "strategy_category": "Waste Elimination",
  "resource_id": "arn:aws:shield::123456789012:protection/abc123xx",
  "recommended_action": "Downgrade to Shield Standard",
  "estimated_monthly_savings_usd": 3000.00,
  "effort_level": "Low"
}
```

### Summary Report Table
| Finding ID | Category | Resource / Target | Recommended Action | Est. Savings ($) | Risk Level |
|------------|----------|-------------------|--------------------|------------------|------------|
| SHIELD-001 | Waste Elimination | Org Accounts | Downgrade to Shield Standard | $3,000/mo | High |
| SHIELD-002 | Waste Elimination | Standalone Accounts | Consolidate under AWS Organizations | $3,000/mo per account | Low |
| SHIELD-013 | Pricing Model | AWS WAF | Utilize Free WAF via Shield Adv. | Variable | Low |

---
## 1. Waste Elimination

#### SHIELD-01. Downgrade from Shield Advanced to Shield Standard
- **What:** Cancel your AWS Shield Advanced subscription if your workloads do not require 24/7 AWS Shield Response Team (SRT) access, Layer 7 proactive mitigation, or financial cost reimbursement.
- **Why It Saves Money:** Saves the flat $3,000/month subscription fee and eliminates the Shield Advanced Data Transfer Out (DTO) premiums.
- **Implementation Steps:** 
  1. Audit current Shield Advanced utilization and historical DDoS attacks.
  2. Verify that compliance or enterprise policies do not mandate Shield Advanced.
  3. Ensure the mandatory 1-year commitment period has expired.
  4. Unsubscribe from Shield Advanced via the AWS console or API.
- **Estimated Savings:** $3,000+ per month.
- **Risk Level:** High (Reduces access to immediate AWS SRT assistance during attacks).
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Expiration of 1-year commitment term.

#### SHIELD-02. Consolidate Multiple Accounts Under AWS Organizations
- **What:** If you must use Shield Advanced across multiple AWS accounts, ensure they are consolidated under a single AWS Organizations master billing account.
- **Why It Saves Money:** Shield Advanced charges a single $3,000/month fee for the entire AWS Organization. Standalone accounts without Organizations enabled are charged $3,000/month each.
- **Implementation Steps:** 
  1. Identify all standalone AWS accounts subscribing to Shield Advanced.
  2. Invite these accounts to join a central AWS Organization.
  3. Associate the Shield Advanced subscription at the Organization management account level.
- **Estimated Savings:** $3,000/month per eliminated duplicate subscription.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** AWS Organizations enabled.

#### SHIELD-03. Remove Shield Advanced Protection from Idle Resources
- **What:** Detach Shield Advanced protections from development, staging, or idle resources (e.g., non-prod ALBs or EIPs).
- **Why It Saves Money:** While the base fee is fixed, Shield Advanced charges a premium for Data Transfer Out (DTO) on *protected* resources. Unprotecting them removes this DTO premium.
- **Implementation Steps:** 
  1. Go to AWS Shield > Protected resources.
  2. Identify non-production resources currently protected.
  3. Remove the protection configuration for these resources.
- **Estimated Savings:** 1-5% (Depending on DTO volume of non-prod resources).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Resource tagging to identify non-prod vs prod.

#### SHIELD-04. Clean Up Unattached Elastic IPs
- **What:** Release Elastic IPs (EIPs) that are no longer attached to active EC2 instances or NAT Gateways, even if they were previously protected by Shield.
- **Why It Saves Money:** AWS charges for unattached EIPs (and charges more under recent public IPv4 pricing changes). Eliminating them reduces underlying network costs.
- **Implementation Steps:** 
  1. List all EIPs in the VPC console.
  2. Identify EIPs with no associated Instance ID or Network Interface.
  3. Release the EIPs back to AWS.
- **Estimated Savings:** ~$3.60/month per EIP.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ensure EIPs are not hardcoded in external DNS or third-party allow-lists.

---
## 2. Rightsizing

#### SHIELD-05. Replace Shield Advanced with WAF Rate-Based Rules
- **What:** Use AWS WAF rate-based rules coupled with Shield Standard instead of paying for Shield Advanced.
- **Why It Saves Money:** A WAF rule costs $1.00/month + $0.60 per 1M requests. For most small to medium web apps, a rate limit rule perfectly mitigates Layer 7 floods (HTTP floods) for <$50/month, avoiding the $3,000/mo Shield Advanced fee.
- **Implementation Steps:** 
  1. Deploy AWS WAF on your CloudFront distribution or ALB.
  2. Create a rate-based rule (e.g., block IPs exceeding 2,000 requests per 5 minutes).
  3. Monitor false positives and adjust thresholds.
- **Estimated Savings:** $2,950+/month (vs Shield Advanced).
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workloads that only require standard HTTP flood protection.

#### SHIELD-06. Consolidate Regional Endpoints Behind CloudFront
- **What:** Instead of enabling Shield protection on dozens of regional Application Load Balancers (ALBs) across multiple VPCs, place them all behind a consolidated set of CloudFront distributions.
- **Why It Saves Money:** Reduces the architectural footprint and centralizes DDoS protection, making it easier to manage and potentially reducing WAF processing and Shield DTO charges by heavily relying on CloudFront's native edge absorption.
- **Implementation Steps:** 
  1. Audit existing public-facing ALBs.
  2. Route traffic through CloudFront.
  3. Restrict ALB access so it only accepts traffic from CloudFront IPs.
- **Estimated Savings:** 5-15% (Through caching and consolidated WAF rules).
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Applications must support being routed through CloudFront.

---
## 3. Commitment Discounts

#### SHIELD-07. Review Shield Advanced 1-Year Commitment Renewal
- **What:** Shield Advanced requires a 1-year commitment. Turn off auto-renewal if the business no longer requires enterprise DDoS protection.
- **Why It Saves Money:** Prevents getting locked into another $36,000/year contract if the company's risk profile or architecture has changed (e.g., moving to a different CDN/security vendor).
- **Implementation Steps:** 
  1. Check the subscription anniversary date in the AWS Billing console.
  2. Set calendar alerts 60 days prior to renewal.
  3. If downgrading, disable auto-renewal before the deadline.
- **Estimated Savings:** $36,000/year (Cost avoidance).
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Visibility into contract dates.

---
## 4. Architecture Changes

#### SHIELD-08. Move Protection to the Edge (CloudFront vs. ALB)
- **What:** Expose only edge services (CloudFront, Route 53, API Gateway) to the public internet, and keep regional resources (EC2, ALB) completely private.
- **Why It Saves Money:** Edge services scale natively to absorb massive volumetric attacks using AWS's massive global network capacity without directly impacting your bill. If attacks hit an ALB directly, the ALB auto-scales, incurring significant regional AWS charges.
- **Implementation Steps:** 
  1. Deploy CloudFront in front of the web application.
  2. Use AWS Managed Prefix Lists to lock down ALBs to only allow CloudFront IPs.
  3. Rely on Shield Standard at the edge.
- **Estimated Savings:** 10-100x cost avoidance during a massive volumetric attack.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Edge-compatible web workloads.

#### SHIELD-09. Use Edge-Optimized API Gateway over Regional ALBs
- **What:** For API workloads, use Edge-optimized Amazon API Gateways (which utilize CloudFront implicitly) rather than directly exposing ALBs.
- **Why It Saves Money:** Shifts the DDoS attack surface to the AWS edge. Edge-optimized API Gateways benefit automatically from Shield Standard's massive L3/L4 mitigation capacity without custom infrastructure scaling costs.
- **Implementation Steps:** 
  1. Migrate API endpoints from ALB to API Gateway.
  2. Select "Edge-optimized" endpoint type.
- **Estimated Savings:** Variable (Cost avoidance during attacks).
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** API-driven workloads.

#### SHIELD-10. Implement Static Error Pages on S3/CloudFront during Attacks
- **What:** Configure CloudFront to serve static error pages directly from an S3 bucket if the origin becomes overwhelmed or during an active DDoS attack.
- **Why It Saves Money:** Prevents an overloaded origin from spinning up expensive EC2/Fargate instances to serve 503/504 error pages to attackers, completely offloading the traffic to low-cost S3/CloudFront.
- **Implementation Steps:** 
  1. Create a static "Site under maintenance" HTML page in S3.
  2. Configure CloudFront Custom Error Responses for 5xx codes to point to the S3 bucket path.
- **Estimated Savings:** High cost avoidance during downtime/attacks.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudFront distribution deployed.

---
## 5. Scheduling & Auto-Scaling

#### SHIELD-11. Strict WAF Limiting to Prevent Malicious Auto-Scaling
- **What:** Use AWS WAF rules to drop malicious traffic *before* it reaches the application origin and triggers EC2/ECS auto-scaling.
- **Why It Saves Money:** Auto-scaling groups will happily scale up to serve millions of malicious bot requests, resulting in huge EC2 compute bills. Blocking them at the WAF edge prevents the compute scale-out.
- **Implementation Steps:** 
  1. Analyze access logs for typical user request rates.
  2. Implement strict rate-based rules in WAF.
  3. Implement bot control rules (AWS Managed Rules).
- **Estimated Savings:** Potentially thousands of dollars during an active bot/DDoS campaign.
- **Risk Level:** Medium (Requires careful tuning to avoid blocking real users).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS WAF enabled.

#### SHIELD-12. Tune Auto-Scaling Thresholds for DDoS Resilience
- **What:** Adjust EC2 Auto Scaling Groups (ASG) to not scale purely on generic network traffic or CPU spikes, which attackers easily manipulate.
- **Why It Saves Money:** Prevents attackers from intentionally inflating your compute bill by driving up generic metrics.
- **Implementation Steps:** 
  1. Change scaling policies to rely on custom application-level metrics (e.g., successful checkout completions, queued background jobs) rather than raw CPU or NetworkIn.
  2. Set strict maximum capacity limits on the ASG.
- **Estimated Savings:** Variable (Cost avoidance).
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch custom metrics.

---
## 6. Pricing Model Optimization

#### SHIELD-13. Maximize Shield Advanced "Free WAF" Benefits
- **What:** If you are paying the $3,000/month Shield Advanced fee, ensure you are not separately paying for AWS WAF on those protected resources.
- **Why It Saves Money:** Shield Advanced includes AWS WAF Web ACLs, Rules, and request processing fees for **free** on all protected resources. If you have a heavy WAF footprint ($1,000+/mo in WAF fees), Shield Advanced subsidizes this cost.
- **Implementation Steps:** 
  1. Identify all WAF deployments across the organization.
  2. Ensure the resources underlying these WAFs (ALBs, CloudFront) are added as "Protected Resources" under Shield Advanced.
  3. Verify WAF charges drop to $0.00 on the billing console for these resources.
- **Estimated Savings:** Up to 100% of your AWS WAF bill.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Active Shield Advanced subscription.

#### SHIELD-14. Utilize Free AWS Firewall Manager (FMS) with Shield Advanced
- **What:** AWS Firewall Manager costs $100 per policy per region. However, if you subscribe to Shield Advanced, FMS is completely **free** for managing Shield and WAF policies.
- **Why It Saves Money:** Eliminates the $100/policy/region FMS management fees while allowing centralized security governance across all AWS accounts.
- **Implementation Steps:** 
  1. Ensure AWS Organizations is enabled.
  2. Designate an FMS admin account.
  3. Deploy WAF and Shield policies globally using FMS without incurring the per-policy fees.
- **Estimated Savings:** $100 - $1,000+ per month depending on the number of policies and regions.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Active Shield Advanced subscription and AWS Organizations.

#### SHIELD-15. Claim DDoS Cost Protection Reimbursement Credits
- **What:** Request service credits from AWS if a DDoS attack causes your protected resources (EC2, ALB, CloudFront) to scale up and incur unexpected charges.
- **Why It Saves Money:** This is a built-in financial safeguard of Shield Advanced. You are entitled to refunds for the compute and bandwidth surge caused by the attack.
- **Implementation Steps:** 
  1. Identify the time window of the DDoS attack via Shield events.
  2. Calculate the exact spike in AWS costs (CloudFront DTO, EC2 instances, ALB LCUs).
  3. Open an AWS Support case (Billing/Shield Advanced) and request "DDoS Cost Protection" credits.
- **Estimated Savings:** 100% of the attack-induced cost spike.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Active Shield Advanced subscription at the time of the attack.

---
## 7. Network & Data Transfer Optimization

#### SHIELD-16. Optimize CloudFront Caching to Reduce Shield Advanced DTO
- **What:** Increase cache hit ratios on CloudFront distributions protected by Shield Advanced.
- **Why It Saves Money:** Shield Advanced charges a premium on Data Transfer Out ($0.050/GB for the first 100TB). However, caching static assets heavily at the edge reduces the payload that requires deep inspection and processing, streamlining overall DTO and origin fetch costs.
- **Implementation Steps:** 
  1. Analyze CloudFront cache hit ratios.
  2. Optimize Cache-Control headers on static assets (images, CSS, JS) to maximize TTLs.
  3. Remove unnecessary query strings from cache keys.
- **Estimated Savings:** 5-20% of DTO and origin compute costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudFront deployed.

#### SHIELD-17. Implement Geo-Blocking at the Edge
- **What:** Use WAF Geo-Match conditions to block traffic from countries where you have no customers.
- **Why It Saves Money:** Dropping traffic at the edge immediately prevents attackers from foreign countries from downloading assets, which avoids incurring Data Transfer Out (DTO) fees and downstream ALB/EC2 processing fees. 
- **Implementation Steps:** 
  1. Identify target customer demographics and operational regions.
  2. Deploy an AWS WAF rule blocking high-risk or non-operational country codes.
  3. Apply WAF to the CloudFront edge.
- **Estimated Savings:** 2-10% of total bandwidth and compute costs (by eliminating background bot noise).
- **Risk Level:** Medium (Risk of blocking legitimate users traveling abroad).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS WAF enabled.
