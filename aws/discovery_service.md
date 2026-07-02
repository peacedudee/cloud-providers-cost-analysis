# AWS Service Cost Research: AWS Application Discovery Service

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Application Discovery Service helps enterprise customers plan cloud migration projects by automatically gathering information about on-premises data centers. It offers two data collection options: the **Agentless Collector** (an OVA virtual appliance deployed on VMware vCenter) and the **Application Discovery Agent** (installed directly on physical or virtual Windows/Linux servers). It records server hardware specifications, CPU/RAM/disk performance metrics, running processes, and active network connections. Collected data feeds directly into AWS Migration Hub.

---

## 2. Billing Mechanics
1.  **Discovery Appliance & Agent Software:** **100% Free ($0.00)**. No licensing or per-discovered-server charges.
2.  **AWS Migration Hub Integration:** **100% Free ($0.00)**.
3.  **Athena & S3 Data Export (Optional):** If you export discovery data to your own S3 bucket for custom SQL analysis, standard S3 storage ($0.023/GB) and Amazon Athena query fees ($5.00/TB scanned) apply.

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

---

## 5. AWS Free Tier Coverage
*   **AWS Application Discovery Service:** Always 100% free for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Split-Migration Latency & Data Transfer Costs:** Migrating web servers to AWS while leaving dependent database servers on-premises without discovering network dependencies first, creating heavy hybrid cloud data egress charges ($0.09/GB) and slow application performance.

---

## 7. Actionable Cost Optimization Strategies
1.  **Map Network Dependencies to Prevent Cross-Premises Data Egress:**
    *   Review Application Discovery Service network connection maps in AWS Migration Hub.
    *   *Why:* Group tightly coupled servers (e.g., application server and database) into the **same migration wave**.
    *   **The Savings:** Prevents continuous cross-premises AWS data egress charges ($0.09/GB) and high network latency during phased migrations.
2.  **Export Discovery Data to Athena for Custom Right-Sizing Analysis:** Export raw server performance metrics to Amazon Athena ($5.00/TB scanned) to identify underutilized servers and size target EC2 instances based on peak 95th-percentile utilization metrics.
