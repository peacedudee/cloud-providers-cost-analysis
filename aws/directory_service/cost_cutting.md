# Cost-Cutting Playbook: AWS Directory Service
> **Companion File:** [directory_service.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/directory_service/directory_service.md)
> **Last Updated:** July 2026

---

## Executive Summary
AWS Directory Service provides managed active directory capabilities but can quickly become a cost center due to its hourly billing, multi-AZ requirements, and directory sharing fees. The most common drivers of excess cost include over-provisioning (Enterprise vs. Standard), unmonitored directory sharing across numerous child accounts, and using Managed AD where a cheaper AD Connector or IAM Identity Center would suffice. This playbook outlines actionable strategies to optimize Directory Service spend through waste elimination, rightsizing, architectural shifts, and network optimizations.

## Strategy Categories

### 1. Waste Elimination

#### 1. DIRECTORY-Delete-Unused-Managed-AD
- **What:** Identify and delete AWS Managed Microsoft AD directories that are no longer associated with active workloads or user authentication flows.
- **Why It Saves Money:** Standard costs $175.20/month and Enterprise costs $584.00/month. Deleting an abandoned directory instantly stops these hourly charges.
- **Implementation Steps:** 
  1. Identify directories in the AWS Console.
  2. Audit CloudWatch metrics and AD logs for LDAP query activity and authentication requests.
  3. Confirm no EC2 instances, WorkSpaces, or RDS instances are relying on the directory.
  4. Delete the directory.
- **Estimated Savings:** 100% per deleted directory ($175.20 - $584.00/month)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Full audit of dependency services (WorkSpaces, RDS, EC2).

#### 2. DIRECTORY-Remove-Unnecessary-Directory-Shares
- **What:** Unshare directories from AWS accounts that no longer require domain-join capabilities.
- **Why It Saves Money:** Directory sharing incurs a fee of $0.075/hr (Standard) or $0.25/hr (Enterprise) per shared account. Unsharing removes this recurring cost.
- **Implementation Steps:** 
  1. Navigate to the Directory Service console.
  2. Select the directory and view the "Shared accounts" tab.
  3. Identify accounts with no active AD-dependent resources.
  4. Revoke the share for those accounts.
- **Estimated Savings:** $54.75 to $182.50/month per unshared account
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Cross-account visibility to verify resource AD dependencies.

#### 3. DIRECTORY-Terminate-Unused-AD-Connectors
- **What:** Delete AD Connectors that were set up for legacy workloads or proof-of-concepts but are no longer routing traffic to on-premise ADs.
- **Why It Saves Money:** Small AD Connectors cost $73.00/month; Large costs $219.00/month.
- **Implementation Steps:** 
  1. Check metrics for authentication traffic through the AD Connector.
  2. Verify if the target on-premises AD is still reachable/active.
  3. Delete inactive AD Connectors.
- **Estimated Savings:** 100% per connector ($73.00 - $219.00/month)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Network/traffic analysis to ensure zero usage.

#### 4. DIRECTORY-Remove-Abandoned-Trust-Relationships
- **What:** Delete trust relationships between AWS Managed AD and on-premises AD that are no longer utilized.
- **Why It Saves Money:** While trusts themselves don't have a direct hourly fee, they often keep directories active artificially and cause unnecessary cross-region/cross-VPN data transfer.
- **Implementation Steps:** 
  1. Audit existing trust relationships in Directory Service.
  2. Confirm with identity teams if the trust is still needed.
  3. Delete the trust relationship.
- **Estimated Savings:** Indirect savings on data transfer and potential directory decommissioning.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Coordination with enterprise IAM/Security teams.

#### 5. DIRECTORY-Clean-Up-Orphaned-Simple-ADs
- **What:** Identify and remove legacy Simple AD deployments (no longer available to new customers as of July 2026) that are no longer in use.
- **Why It Saves Money:** Simple ADs cost $73.00 to $219.00/month.
- **Implementation Steps:** 
  1. Audit for remaining Simple AD directories.
  2. Migrate remaining workloads to AD Connector or IAM Identity Center.
  3. Delete the Simple AD.
- **Estimated Savings:** $73.00 to $219.00/month per directory
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Identification of legacy resources.

### 2. Rightsizing

#### 6. DIRECTORY-Downsize-Enterprise-to-Standard
- **What:** Migrate from AWS Managed Microsoft AD Enterprise Edition to Standard Edition for directories with fewer than 5,000 users or 30,000 objects.
- **Why It Saves Money:** Drops the baseline cost from $584.00/month to $175.20/month, and the sharing fee from $0.25/hr to $0.075/hr per account.
- **Implementation Steps:** 
  1. Audit the current AD object count.
  2. Provision a new Standard directory.
  3. Establish a trust and migrate objects/users, or recreate them.
  4. Update dependent services to the new directory.
  5. Delete the Enterprise directory.
- **Estimated Savings:** 70% ($408.80/month base + sharing savings)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Active Directory migration tools and downtime window.

#### 7. DIRECTORY-Downsize-AD-Connector
- **What:** Replace Large AD Connectors with Small AD Connectors for workloads that authenticate fewer than 10,000 users.
- **Why It Saves Money:** Small Connectors cost $73.00/month compared to Large Connectors at $219.00/month.
- **Implementation Steps:** 
  1. Verify total user count and authentication volume.
  2. Deploy a new Small AD Connector pointing to the same on-premises AD.
  3. Shift workloads to the new connector.
  4. Delete the Large AD Connector.
- **Estimated Savings:** 66% ($146.00/month per connector)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ability to update directory IDs in dependent services.

#### 8. DIRECTORY-Migrate-Managed-AD-to-AD-Connector
- **What:** Replace a standalone AWS Managed AD with an AD Connector pointing to an existing on-premises or EC2-hosted Active Directory.
- **Why It Saves Money:** A Small AD Connector ($73.00/month) is significantly cheaper than Managed AD Standard ($175.20/month) and avoids data replication costs.
- **Implementation Steps:** 
  1. Confirm connectivity to existing on-premises/EC2 AD via VPN/Direct Connect.
  2. Deploy AD Connector.
  3. Migrate workloads to the AD Connector.
  4. Decommission Managed AD.
- **Estimated Savings:** 58% to 87% ($102.20 to $511.00/month)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Reliable network path to self-managed AD.

#### 9. DIRECTORY-Optimize-AD-Object-Counts
- **What:** Purge stale users, computers, and groups from AD to stay under the 30,000 object limit of Standard Edition.
- **Why It Saves Money:** Prevents forced upgrades to Enterprise Edition ($584/month).
- **Implementation Steps:** 
  1. Run PowerShell scripts to find inactive computers/users.
  2. Archive and delete stale objects.
  3. Monitor object counts regularly.
- **Estimated Savings:** Cost avoidance of $408.80/month.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AD Administrative credentials.

### 3. Commitment Discounts

#### 10. DIRECTORY-Migrate-to-Self-Managed-AD-with-Savings-Plans
- **What:** For massive deployments requiring numerous DCs, migrate from Managed AD to self-managed Active Directory on EC2 instances covered by Compute Savings Plans.
- **Why It Saves Money:** Managed AD pricing is fixed. EC2 instances can be discounted by up to 72% via SPs/RIs. Windows Server licensing costs apply but can be optimized.
- **Implementation Steps:** 
  1. Size appropriate EC2 instances for Domain Controllers.
  2. Deploy Windows Server and promote to DCs.
  3. Purchase Compute Savings Plans covering the instance families.
  4. Migrate resources off Managed AD.
- **Estimated Savings:** 30-50% for large-scale enterprise deployments.
- **Risk Level:** High
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Capability to manage Windows OS patching, backups, and DC replication manually.

### 4. Architecture Changes

#### 11. DIRECTORY-Centralize-Managed-AD-via-Sharing
- **What:** Share a single central Managed AD directory across child AWS accounts instead of provisioning a new directory in every account.
- **Why It Saves Money:** A dedicated Standard directory costs $175.20/month. Sharing a directory costs $54.75/month ($0.075/hr).
- **Implementation Steps:** 
  1. Identify decentralized Managed ADs in child accounts.
  2. Use AWS RAM / Directory sharing from the central account.
  3. Migrate resources in child accounts to the shared directory.
  4. Delete the child account directories.
- **Estimated Savings:** $120.45/month per child account (68% savings)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Network connectivity (VPC peering/TGW) between shared accounts.

#### 12. DIRECTORY-Replace-Managed-AD-with-IAM-Identity-Center
- **What:** Use IAM Identity Center (formerly AWS SSO) instead of AWS Directory Service if the only requirement is single sign-on for AWS Console and enterprise apps.
- **Why It Saves Money:** IAM Identity Center is 100% free, whereas Managed AD costs $175.20+ per month.
- **Implementation Steps:** 
  1. Verify no workloads require LDAP, Kerberos, or EC2 domain-joining.
  2. Enable IAM Identity Center in AWS Organizations.
  3. Configure an external IdP (Entra ID, Okta) or use the internal identity store.
  4. Decommission AWS Directory Service.
- **Estimated Savings:** 100% of Directory Service costs ($175.20 - $584.00/month)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Migration plan for existing user access.

#### 13. DIRECTORY-Share-Directories-Across-VPCs-in-Same-Account
- **What:** Utilize a single directory across multiple VPCs within the *same* AWS account rather than sharing across accounts.
- **Why It Saves Money:** Directory sharing across VPCs within the *same* AWS account incurs **no sharing fee**, unlike cross-account sharing.
- **Implementation Steps:** 
  1. Consolidate AD-dependent workloads into a single AWS account where feasible.
  2. Connect the VPCs via Transit Gateway or VPC Peering.
  3. Use the directory across VPCs.
- **Estimated Savings:** $54.75 to $182.50/month per eliminated shared account.
- **Risk Level:** Medium
- **Implementation Scope:** Architecture/DevOps
- **Prerequisites:** Network routing and security group configuration.

#### 14. DIRECTORY-Consolidate-Multiple-Small-Directories
- **What:** Merge multiple departmental Standard Managed ADs into a single Standard Managed AD using Organizational Units (OUs) for separation.
- **Why It Saves Money:** Each standalone directory incurs a minimum $175.20/month base cost. 
- **Implementation Steps:** 
  1. Design an OU structure in a central directory.
  2. Migrate users, groups, and computer objects to the centralized directory.
  3. Delete departmental directories.
- **Estimated Savings:** $175.20/month per eliminated directory.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AD migration tooling and careful permission planning.

### 5. Scheduling & Auto-Scaling

#### 15. DIRECTORY-Ephemeral-Dev-Test-Directories
- **What:** Use Infrastructure as Code (IaC) to deploy and tear down AWS Managed AD or AD Connectors in non-production environments based on testing schedules.
- **Why It Saves Money:** Directory Service is billed hourly. Destroying directories during nights and weekends saves ~70% of the cost.
- **Implementation Steps:** 
  1. Define the directory setup in Terraform or CloudFormation.
  2. Script the domain join processes for test instances.
  3. Implement CI/CD pipelines to spin up the directory for testing, and destroy it afterward.
- **Estimated Savings:** ~70% of non-prod directory costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Fully automated domain join and testing pipelines.

#### 16. DIRECTORY-Suspend-AD-Connector-during-Off-Hours
- **What:** Delete and recreate AD Connectors dynamically for transient workspaces or developer environments during non-working hours.
- **Why It Saves Money:** Saves $0.10 to $0.30 per hour when the connector is not running.
- **Implementation Steps:** 
  1. Track usage patterns of dependent services (e.g., WorkSpaces).
  2. Automate the removal of the AD Connector at night.
  3. Re-provision before business hours.
- **Estimated Savings:** ~$50.00/month per AD connector.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Scripted creation/deletion and dependent service tolerance for offline directories.

### 6. Pricing Model Optimization

#### 17. DIRECTORY-Strategic-Use-of-Free-Trial
- **What:** Maximize the 30-day (1,500 hours) free trial for new AWS accounts to conduct proof-of-concepts without incurring costs.
- **Why It Saves Money:** Absorbs the first month of Managed AD or AD Connector costs.
- **Implementation Steps:** 
  1. Provision the directory in a net-new account.
  2. **Do not share the directory** with other accounts (sharing fees are not covered by the free trial).
  3. Complete testing within 30 days.
- **Estimated Savings:** Up to $584.00 for the first month.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** New AWS account eligible for the Free Tier.

### 7. Network & Data Transfer Optimization

#### 18. DIRECTORY-Intra-Region-Directory-Lookups
- **What:** Ensure that EC2 instances and AWS services query a directory located in the same AWS Region.
- **Why It Saves Money:** Cross-region data transfer costs $0.02/GB. Heavy LDAP traffic or AD replication across regions can generate high bandwidth bills.
- **Implementation Steps:** 
  1. Review VPC flow logs for cross-region port 389/636 traffic.
  2. Deploy local AD Connectors or secondary AD regions (if using Enterprise Edition multi-region feature) locally.
- **Estimated Savings:** Variable, dependent on data transfer volume.
- **Risk Level:** Low
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Multi-region deployment footprint.

#### 19. DIRECTORY-Centralize-DNS-Proxying-via-TGW
- **What:** Route AD DNS resolution centrally through a Transit Gateway rather than provisioning Route 53 Outbound Endpoints in every VPC.
- **Why It Saves Money:** Reduces the architectural bloat of supporting services needed to resolve the Directory Service endpoints.
- **Implementation Steps:** 
  1. Configure central Route 53 Resolver endpoints in the directory's VPC.
  2. Share resolving rules via RAM to other VPCs connected by TGW.
- **Estimated Savings:** ~$90.00/month per eliminated Route 53 endpoint pair.
- **Risk Level:** Medium
- **Implementation Scope:** Network Engineer
- **Prerequisites:** Transit Gateway architecture.

---

## Cross-Service Synergies
- **Amazon EC2 & FSx for Windows:** Proper directory sharing drastically reduces the cost of joining multiple Windows fleets and file systems across accounts.
- **Amazon WorkSpaces:** Use AD Connector linked to on-prem AD to provision WorkSpaces cheaply without paying for a full Managed AD.
- **AWS IAM Identity Center:** Serves as a free alternative to Managed AD for SSO, integrating seamlessly with AWS Organizations.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
- `lineItem/ProductCode`: `AWSDirectoryService`
- `lineItem/UsageType`: Look for `DirectoryUsage`, `SharedDirectoryUsage`, `ADConnectorUsage`.
- `lineItem/UnblendedCost`: The exact spend per directory/share.

### B. CloudWatch Metrics
- `DirectoryService` Namespace: `DirectoryDataSize`, `FreeStorageSpace`. (Helps in identifying if Enterprise is over-provisioned).
- Network In/Out on AD DCs to track utilization.

### C. AWS Config / Trusted Advisor
- Identify untagged or long-running Directory Service instances.
- Check AWS RAM for the number of active directory shares.

### D. Company Policies
- Identity and Access Management policies (SSO vs. LDAP requirements).
- Compliance requirements for isolated domains vs. shared domains.

### E. IaC (Optional)
- Terraform/CloudFormation templates to identify hardcoded directory deployments that should be converted to shared resources.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "DIRECTORY-Rightsizing-001",
  "strategy_name": "DIRECTORY-Downsize-Enterprise-to-Standard",
  "resource_id": "d-1234567890",
  "account_id": "111122223333",
  "region": "us-east-1",
  "current_monthly_cost": 584.00,
  "projected_monthly_cost": 175.20,
  "estimated_monthly_savings": 408.80,
  "risk_level": "High",
  "action_required": "Migrate objects to new Standard directory and decommission Enterprise directory."
}
```

### Summary Report Table

| Finding ID | Strategy | Resource | Current Cost | Projected Cost | Monthly Savings | Risk |
|------------|----------|----------|--------------|----------------|-----------------|------|
| DIRECTORY-Rightsizing-001 | Downsize Enterprise to Standard | d-1234567890 | $584.00 | $175.20 | $408.80 | High |
| DIRECTORY-Waste-001 | Remove Unnecessary Directory Share | shr-0987654 | $54.75 | $0.00 | $54.75 | Medium |
| DIRECTORY-Arch-001 | Replace with IAM Identity Center | d-abcdef123 | $175.20 | $0.00 | $175.20 | Medium |
