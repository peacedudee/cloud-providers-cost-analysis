# AWS Service Cost Research: Amazon QuickSight

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon QuickSight is a fully managed, serverless business intelligence (BI) service that allows you to create and publish interactive dashboards, run machine learning-powered insights, and embed analytics into your applications. QuickSight is serverless, meaning you do not manage servers or cluster infrastructure. Instead, QuickSight is billed on a licensing model based on user roles (Authors vs. Readers) and SPICE in-memory storage capacity.

---

## 2. Billing Mechanics
QuickSight is billed monthly based on the following dimensions:
1.  **Author Licensing:** A flat monthly fee per user who can create, edit, and publish dashboards.
2.  **Reader Licensing:** A usage-based fee per user who only views dashboards, calculated per session.
3.  **Reader Billing Cap:** A customer-friendly monthly cap that limits the maximum charge per reader.
4.  **SPICE In-Memory Storage:** Billed per GB-month for data cached in the SPICE calculation engine.
5.  **Paginated Reports (Optional):** Billed per report page generated.

---

## 3. Key Cost Dimensions

### A. Author Licensing (Enterprise Edition - us-east-1)
*   **Annual Commitment:** **$18.00 per user per month** (billed annually at $216.00/year).
*   **Monthly Subscription:** **$24.00 per user per month** (no commitment, pay-as-you-go).
*   *Note: Standard Edition is cheaper ($9.00 annual / $12.00 monthly) but lacks Active Directory integration, SPICE auto-refresh, and dashboard embedding.*

### B. Reader Licensing (The Session-Based Model)
*   **The Session Fee:** **$0.30 per session** for dashboard access.
*   **Definition of a Session:** A session begins when a reader logs in and starts viewing dashboards, lasting for up to **30 minutes**. If the user stays active beyond 30 minutes, a new 30-minute session ($0.30) automatically starts.
*   **The Monthly Cap (Cost Shield):** Billed session fees are capped at a maximum of **$5.00 per reader per month**. Once a reader reaches $5.00 in a calendar month (approx. 17 sessions), all subsequent sessions for that reader are **free ($0.00)**.
*   *Embedded Capacity:* For large-scale external portals where user-based licensing is not possible, you can purchase a **Session Capacity Plan** starting at **$250.00/month** for a bundle of 500 sessions.

### C. SPICE Storage Capacity (In-Memory Engine)
SPICE (Super-fast, Parallel, In-memory Calculation Engine) is used to cache data from databases for instant dashboard rendering.
*   **Included Capacity:** Each Author license includes **10 GB** of free SPICE capacity.
*   **Overage Charge:** Extra SPICE capacity is billed at **$0.38 per GB-month** in `us-east-1`.
*   *Note: SPICE capacity is shared across the entire QuickSight account.*

---

## 4. Detailed Pricing Rates (us-east-1 Enterprise Edition)

| User Role / Component | Annual Commitment Rate | Monthly Rate | Details / Caps |
|-----------------------|------------------------|--------------|----------------|
| **Author License** | **$18.00 / month** | **$24.00 / month** | For dashboard creators |
| **Reader License** | N/A | **$0.30 / session**| **Capped at $5.00 / month** |
| **SPICE Storage** | N/A | **$0.38 / GB-month** | First 10 GB free per Author |
| **Paginated Reports** | N/A | **$0.50 / page** | For PDF generation jobs |

---

## 5. AWS Free Tier Coverage
*   **Authors:** One free Author license with 10 GB of SPICE capacity for 30 days (new QuickSight accounts).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Assigning Author Licenses to Dashboard Viewers:** Creating Author accounts for managers or stakeholders who only need to view reports. Authors cost $24.00/month, whereas Readers cost a maximum of $5.00/month (a **79% billing variance**).
*   **SPICE Overage Accumulation:** Caching massive database tables in SPICE that are not actively used by dashboards, incurring $0.38/GB-month.
*   **Frequent Dashboard Auto-Refresh Queries:** Scheduling SPICE datasets to refresh every 15 minutes, driving up database queries, network transfer, and compute charges on target databases (like RDS or Redshift).

---

## 7. Actionable Cost Optimization Strategies
1.  **Downgrade Viewers to Reader Licenses:** Audit your QuickSight users under the Admin console. Downgrade any users who have not edited or created dashboards in the last 30 days from **Author** to **Reader**. This cuts their cost from $18–$24/month to a maximum of $5.00/month.
2.  **Utilize Annual Commitments for Long-Term Authors:** For core business intelligence developers who require Author rights permanently, purchase the **Annual Author commitment** to save **25%** compared to monthly pricing.
3.  **Manage and Prune SPICE Capacity:**
    *   Review SPICE capacity usage in the QuickSight settings.
    *   Delete unused SPICE datasets or apply filters/row limits in your SQL queries to import only the data required.
    *   This prevents paying the **$0.38/GB-month** overage tax.
4.  **Optimize Dataset Refresh Intervals:** Do not schedule hourly SPICE dataset refreshes for dashboards that are only reviewed weekly. Set schedules to daily or weekly.
5.  **Use Reader Cap to Budget BI Costs:** Because reader sessions cap at $5.00/month, you can calculate the absolute maximum monthly bill for internal reporting:
    $$\text{Max Cost } = (\text{Authors} \times \$24.00) + (\text{Readers} \times \$5.00)$$
    Use this formula to budget BI infrastructure without run-away usage worries.
