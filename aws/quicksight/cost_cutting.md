# Cost-Cutting Playbook: Amazon QuickSight

> **Companion File:** [quicksight.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/quicksight/quicksight.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon QuickSight is a serverless business intelligence (BI) platform. Billing includes **Author Licensing** ($24/mo pay-as-you-go vs $18/mo annual commitment), **Reader Licensing** ($0.30 per 30-min session, capped at **$5.00/reader-month**), **SPICE In-Memory Storage** ($0.38/GB-mo beyond 10 GB free per Author), and **Paginated Reports** ($0.50/page).

Key billing traps include:
1. **Assigning Author Licenses to Dashboard Viewers:** Provisioning Author accounts ($24.00/mo) for managers or stakeholders who only view dashboards. Viewers assigned **Reader accounts cost a maximum of $5.00/month (a 79% billing reduction)**!
2. **SPICE In-Memory Storage Overages:** Caching raw database tables in SPICE without filtering, accumulating hundreds of gigabytes of unneeded in-memory cache ($0.38/GB-mo).
3. **Frequent High-Volume SPICE Refreshes:** Scheduling 15-minute SPICE dataset auto-refreshes for dashboards that are only reviewed weekly.

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **35–80% reduction in total Amazon QuickSight spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Downgrade Inactive / Viewer Authors to Reader Licenses:** Cuts per-user BI licensing costs from $24.00/mo to a maximum of **$5.00/month (79% savings)**.
2. **Purchase Annual Commitments for Core BI Authors:** Saves **25%** on Author licenses ($18/mo vs $24/mo).
3. **Prune SPICE Storage Overages:** Filter SQL queries to import ONLY required dashboard columns/rows, eliminating $0.38/GB-mo SPICE overage fees.

---

## Strategy Categories

### 1. User Licensing & Role Right-Sizing

#### 1. Downgrade Non-Authoring Users to Reader Licenses ($5.00 Monthly Cap)
- **What:** Audit QuickSight user roles in the Admin Console. Downgrade any users who have not published or edited a dashboard in the last 30 days from **Author** to **Reader**.
- **Why It Saves Money:**
  - **Author License:** **$24.00 per user per month** (monthly pay-as-you-go).
  - **Reader License:** **$0.30 per session**, capped at **$5.00 per reader per month**.
  - Downgrading 100 dashboard viewers from Author to Reader drops monthly licensing fees from $2,400.00/mo to a maximum of **$500.00/month (saving $1,900.00/month = 79.1% savings)**!
- **Detailed Implementation Steps:**
  1. Audit user activity via AWS CLI / QuickSight API:
     ```bash
     aws quicksight list-users --aws-account-id 123456789012 --namespace default \
       --query "UserList[?Role=='AUTHOR'].{UserName:UserName,Email:Email}"
     ```
  2. Update user role to Reader via CLI:
     ```bash
     aws quicksight update-user \
       --aws-account-id 123456789012 --namespace default \
       --user-name "john.doe" \
       --email "john.doe@company.com" \
       --role "READER"
     ```
- **Estimated Savings:** **79.1% reduction** in per-user BI licensing spend ($19.00/mo saved per downgraded user).
- **Risk Level:** Zero risk (Readers retain full interactive dashboard viewing and filtering capabilities).
- **Implementation Scope:** BI Administrator / IAM Admin
- **Prerequisites:** 30-day dashboard authoring activity audit.

---

### 2. Commitment Discounts & Plan Selection

#### 2. Purchase Annual Author Commitments for Permanent Developers
- **What:** Convert core business intelligence developers and data engineers from monthly pay-as-you-go Author subscriptions ($24/mo) to **Annual Author Commitments ($18/mo)**.
- **Why It Saves Money:** Saves **25%** directly on Author licensing ($18.00 vs $24.00/month). Converting 20 permanent Authors saves **$120.00/month ($1,440.00/year)**.
- **Detailed Implementation Steps:**
  1. Update subscription plan to Annual in QuickSight Account Management console settings.
- **Estimated Savings:** **25% discount** on core Author licenses.
- **Risk Level:** Zero risk (for permanent BI developers).
- **Implementation Scope:** FinOps Team / Procurement
- **Prerequisites:** 12-month Author role stability commitment.

#### 3. Evaluate Embedded Session Capacity Pricing for External Portals
- **What:** Evaluate switching from per-user Reader licenses to **Session Capacity Pricing** ($250.00/month for 500 sessions) for embedded external customer portals with high user counts but low per-user frequency.
- **Why It Saves Money:** Provides a flat-rate predictable monthly budget for large external customer populations.
- **Detailed Implementation Steps:**
  1. Purchase Session Capacity Plan in QuickSight console.
- **Estimated Savings:** 40–70% savings for large-scale external user embedding.
- **Risk Level:** Low.
- **Implementation Scope:** BI Architect / FinOps
- **Prerequisites:** External embedded portal deployment.

---

### 3. SPICE In-Memory Storage Optimization

#### 4. Audit & Prune SPICE Capacity Overages ($0.38/GB-Mo)
- **What:** Review SPICE capacity usage in QuickSight settings. Delete unused SPICE datasets and filter underlying SQL queries to select ONLY the specific columns and date ranges required for active dashboards.
- **Why It Saves Money:**
  - Each Author license includes **10 GB of free SPICE capacity**.
  - Unused SPICE capacity overage bills at **$0.38 per GB-month** ($380.00/TB-mo).
  - Deleting 500 GB of stale or unneeded SPICE dataset caches saves **$190.00/month ($2,280.00/year)**!
- **Detailed Implementation Steps:**
  1. List SPICE datasets via AWS CLI:
     ```bash
     aws quicksight list-data-sets --aws-account-id 123456789012 \
       --query "DataSetSummaries[*].{ID:DataSetId,Name:Name,ImportMode:ImportMode}"
     ```
  2. Delete stale dataset: `aws quicksight delete-data-set --aws-account-id 123456789012 --data-set-id DATASET_ID`.
- **Estimated Savings:** **100% savings** on unneeded SPICE storage overages.
- **Risk Level:** Zero risk.
- **Implementation Scope:** BI Developer / Data Engineer
- **Prerequisites:** Dashboard-to-dataset mapping audit.

---

### 4. Refresh Schedule & Data Source Tuning

#### 5. Right-Size SPICE Dataset Refresh Schedules
- **What:** Adjust SPICE dataset refresh schedules from aggressive hourly/15-minute intervals to **daily or weekly** for executive dashboards that are only reviewed periodically.
- **Why It Saves Money:**
  1. Reduces database compute, memory, and query processing fees on target databases (Athena, Redshift, RDS, Snowflake).
  2. Reduces S3 data transfer and API charges.
- **Detailed Implementation Steps:**
  1. Update dataset refresh schedule to `DAILY` at off-peak hours in QuickSight dataset settings.
- **Estimated Savings:** 70–90% reduction in target database query scanning costs.
- **Risk Level:** Low.
- **Implementation Scope:** BI Developer
- **Prerequisites:** Business freshness SLA agreement.

---

### 5. Standard Edition vs Enterprise Alignment

#### 6. Evaluate Standard Edition for Internal Team Usage (Where Compliant)
- **What:** Evaluate deploying **Standard Edition** ($9.00/mo annual Author) for internal teams that do not require Active Directory / Single Sign-On (SSO), SPICE auto-refresh, or embedded analytics.
- **Why It Saves Money:** Standard Edition Author licenses cost **$9.00/month** compared to Enterprise Edition ($18.00–$24.00/month) — a **50% savings**.
- **Detailed Implementation Steps:**
  1. Review Enterprise feature requirements (AD/SSO, row-level security).
- **Estimated Savings:** **50% reduction** in Author licensing fees.
- **Risk Level:** Medium (compliance and SSO dependent).
- **Implementation Scope:** BI Administrator
- **Prerequisites:** Enterprise feature audit.

---

### 6. Observability & Governance

#### 7. Implement Automatic User De-Provisioning (Identity Center Sync)
- **What:** Configure IAM Identity Center (AWS SSO) or Active Directory auto-provisioning rules to revoke QuickSight licenses automatically when employees leave the company or change teams.
- **Why It Saves Money:** Stops paying $24.00/mo for inactive user accounts left provisioned.
- **Detailed Implementation Steps:**
  1. Sync QuickSight groups with SCIM / Active Directory groups.
- **Estimated Savings:** Eliminates ghost user licensing waste.
- **Risk Level:** Zero.
- **Implementation Scope:** IAM / IT Admin
- **Prerequisites:** AWS SSO / Okta integration.

#### 8. Use Paginated Reports Optimally (Avoid Batch PDF Bloat)
- **What:** Restrict Paginated Report PDF generation jobs ($0.50/page) to critical financial statements rather than mass-emailing 100-page reports to large distribution lists.
- **Why It Saves Money:** Avoids $0.50/page generation surcharges on multi-page PDF exports.
- **Detailed Implementation Steps:**
  1. Audit Paginated Report schedules and recipient lists.
- **Estimated Savings:** 60–80% report generation savings.
- **Risk Level:** Zero.
- **Implementation Scope:** BI Developer
- **Prerequisites:** None.

#### 9. Enforce CloudWatch Alarms for SPICE Overage Capacity
- **What:** Put CloudWatch alarm on SPICE storage consumption metrics.
- **Why It Saves Money:** Instant alert before SPICE overage charges accumulate.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm for SPICE consumption.
- **Estimated Savings:** Proactive billing risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch alarm setup.

#### 10. Direct Query vs SPICE Selection Analysis
- **What:** Use **Direct Query** mode (instead of SPICE) for real-time dashboards backed by high-performance columnar databases like Amazon Redshift or Athena with result caching.
- **Why It Saves Money:** Bypasses SPICE storage fees ($0.38/GB-mo) entirely.
- **Detailed Implementation Steps:**
  1. Set dataset import mode to `DIRECT_QUERY` in QuickSight dataset settings.
- **Estimated Savings:** 100% SPICE storage savings.
- **Risk Level:** Low.
- **Implementation Scope:** BI Developer
- **Prerequisites:** Target database query performance SLA met.

#### 11. Implement Row-Level Security (RLS) to Consolidate Dashboards
- **What:** Apply **Row-Level Security (RLS)** policies on datasets so multiple user groups share a single dashboard instead of duplicating dashboards and SPICE datasets per department.
- **Why It Saves Money:** Consolidates SPICE dataset storage.
- **Detailed Implementation Steps:**
  1. Attach RLS dataset rules in QuickSight console.
- **Estimated Savings:** 50–70% SPICE storage consolidation.
- **Risk Level:** Zero.
- **Implementation Scope:** BI Security Developer
- **Prerequisites:** RLS policy design.

#### 12. Decommission Abandoned Dashboards & Analyses
- **What:** Identify and delete analyses (`aws quicksight list-analyses`) with zero views over 90 days.
- **Why It Saves Money:** Reclaims associated SPICE dataset storage.
- **Detailed Implementation Steps:**
  1. Delete abandoned analyses via CLI.
- **Estimated Savings:** Administrative cleanliness.
- **Risk Level:** Low.
- **Implementation Scope:** BI Administrator
- **Prerequisites:** 90-day dashboard access audit.

#### 13. Audit Embedded Dashboard Session Lifetimes
- **What:** Set embedded dashboard session timeout to **30 minutes** exactly in application iframe code.
- **Why It Saves Money:** Prevents an idle open browser tab from triggering a second 30-minute reader session ($0.30 fee).
- **Detailed Implementation Steps:**
  1. Configure `sessionLifetimeInMinutes = 30` in GetDashboardEmbedUrl API call.
- **Estimated Savings:** 20–30% reader session fee optimization.
- **Risk Level:** Zero.
- **Implementation Scope:** Web Developer
- **Prerequisites:** Application embedding code update.

#### 14. Standardize QuickSight Account Billing Formula
- **What:** Apply the QuickSight monthly cost formula for accurate budgeting:
  $$\text{Max Cost} = (\text{Authors} \times \$24.00) + (\text{Readers} \times \$5.00) + (\text{SPICE GB over 10G/Author} \times \$0.38)$$
- **Why It Saves Money:** Provides a hard ceiling on BI reporting budget overruns.
- **Detailed Implementation Steps:**
  1. Embed formula into monthly FinOps forecasting models.
- **Estimated Savings:** Budget predictability.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

#### 15. Enforce IAM Admin Least Privilege Access
- **What:** Restrict `quicksight:CreateUser` and `quicksight:Subscribe` permissions to authorized IT admins.
- **Why It Saves Money:** Prevents unapproved user provisioning.
- **Detailed Implementation Steps:**
  1. Restrict IAM policies for QuickSight management.
- **Estimated Savings:** Security risk mitigation.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** IAM policy review.

#### 16. Monitor SPICE Import Error Logs
- **What:** Audit SPICE dataset ingestion failure logs to fix broken query joins.
- **Why It Saves Money:** Prevents failed SPICE refresh jobs from retrying endlessly.
- **Detailed Implementation Steps:**
  1. Review SPICE ingestion history in QuickSight console.
- **Estimated Savings:** Operational reliability.
- **Risk Level:** Zero.
- **Implementation Scope:** BI Developer
- **Prerequisites:** None.

---

## Cross-Service Synergies

```
[ QuickSight BI Environment ] 
        │
        ├──(User Roles)─────────> [ Downgrade Viewers to Readers ($5.00 Cap) ] (Saves 79.1% vs $24/mo Author)
        │
        ├──(Author Commitments)─> [ Annual Author Subscriptions ($18/mo) ] (Saves 25% on core BI devs)
        │
        └──(SPICE Storage)──────> [ Prune Unused Datasets & Filter SQL ] (Eliminates $0.38/GB-mo overages)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Author-Month`, `Reader-Session`, `Reader-Month-Cap`, `SPICE-GB-Month`, `PaginatedReport-Page`.
- `line_item_resource_id`: QuickSight User ARN / Dataset ID (`arn:aws:quicksight:us-east-1:123456789012:user/default/john.doe`).

### B. QuickSight Admin Console & CloudWatch Metrics
- QuickSight User Audit Logs: User Role, Last Login Date, Dashboard Edit Count.
- `AWS/QuickSight` Namespace: `SPICEStorageCapacity`, `ReaderSessions`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "QS-ROL-001",
  "service": "QuickSight",
  "category": "User Licensing & Role Right-Sizing",
  "resource_id": "arn:aws:quicksight:us-east-1:123456789012:user/default/exec-viewer-group",
  "resource_name": "exec-viewer-group",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "current_role": "AUTHOR",
    "user_count": 85,
    "monthly_author_cost_usd": 2040.00
  },
  "recommended_config": {
    "target_role": "READER",
    "monthly_cap_per_reader_usd": 5.00,
    "projected_monthly_cost_usd": 425.00
  },
  "financial_impact": {
    "monthly_savings_usd": 1615.00,
    "annual_savings_usd": 19380.00,
    "savings_percentage": 79.1
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Users are dashboard viewers who have not authored dashboards in 30+ days; Reader role retains 100% interactive viewing features."
  },
  "implementation": {
    "scope": "BI Administrator / IAM Admin",
    "effort_estimate": "30 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Author-to-Reader Role Downgrades** | 85 | $2,040.00 | $1,615.00 | 79.1% | Zero |
| **Annual Author Commitments**        | 20 | $480.00 | $120.00 | 25.0% | Zero |
| **SPICE Overage Storage Pruning**    | 12 | $1,520.00 | $1,368.00 | 90.0% | Zero |
| **SPICE Refresh Schedule Tuning**    | 15 | $4,200.00 | $2,940.00 | 70.0% | Low |
| **Total** | **132** | **$8,240.00** | **$6,043.00** | **73.3%** | -- |
