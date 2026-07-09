# AWS Service Cost Research: AWS Application Discovery Service

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Application Discovery Service helps enterprise customers plan cloud migration projects by automatically gathering information about on-premises data centers. It offers two data collection options: the **Agentless Collector** (an OVA virtual appliance deployed on VMware vCenter) and the **Application Discovery Agent** (installed directly on physical or virtual Windows/Linux servers). It records server hardware specifications, CPU/RAM/disk performance metrics, running processes, and active network connections. Collected data feeds directly into AWS Migration Hub.

---

## 2. Billing Mechanics
1.  **Discovery Appliance & Agent Software:** **100% Free ($0.00)**. No licensing, subscription, or per-discovered-server charges.
2.  **AWS Migration Hub Integration:** **100% Free ($0.00)**.
3.  **Athena & S3 Data Export (Optional):** If you export discovery data to your own Amazon S3 bucket for custom SQL analysis, standard S3 storage ($0.023/GB-mo) and Amazon Athena query fees ($5.00/TB scanned) apply.

---

## 3. Key Cost Dimensions

| Feature / Appliance | Billed Unit | Rate (us-east-1) | Price |
|---------------------|-------------|------------------|-------|
| **Agentless Collector OVA**| Per appliance | **Free ($0.00)** | **$0.00** |
| **Discovery Agent** | Per server | **Free ($0.00)** | **$0.00** |
| **AWS Migration Hub Store**| Per project | **Free ($0.00)** | **$0.00** |

---

## 4. Detailed Pricing Rates (us-east-1)
*   **Discovery Service Rate:** $0.00 per month.
*   **Discovery Data Export Storage (Amazon S3):** Standard S3 pricing applies (typically $0.023 per GB-month).
*   **Discovery Data Queries (Amazon Athena):** Standard Athena pricing applies ($5.00 per TB of data scanned).

---

## 5. AWS Free Tier Coverage
*   **AWS Application Discovery Service:** Always 100% free for all AWS accounts, with no time or scale limitations on discovered assets.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Split-Migration Latency & Data Transfer Costs:** Migrating web servers to AWS while leaving dependent database servers on-premises without discovering network dependencies first. This creates heavy hybrid cloud data egress charges ($0.09/GB) and slow application performance.
*   **Orphaned S3 Discovery Exports:** Leaving historical data exports in S3 buckets long after the migration is complete, generating ongoing S3 storage fees.

---

## 7. Actionable Cost Optimization Strategies
1.  **Map Network Dependencies to Prevent Cross-Premises Data Egress:**
    *   Review Application Discovery Service network connection maps in AWS Migration Hub.
    *   Group tightly coupled servers (e.g., application server and database) into the **same migration wave**.
    *   **The Savings:** Prevents continuous cross-premises AWS data egress charges ($0.09/GB) and high network latency during phased migrations.
2.  **Export Discovery Data to Athena for Custom Right-Sizing Analysis:**
    *   Export raw server performance metrics to Amazon Athena ($5.00/TB scanned) to identify underutilized servers.
    *   Size target EC2 instances based on peak 95th-percentile utilization metrics rather than matching on-premises provisioned capacity.
    *   **The Savings:** Slashes target cloud compute costs by **30–50%** by avoiding over-provisioning.
3.  **Implement S3 Lifecycle Rules on Export Buckets:**
    *   Set S3 lifecycle rules to automatically delete or archive discovery CSV/JSON exports 90 days after completing migration analysis.
