# AWS Service Cost Research: Amazon Aurora

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Aurora is a fully managed relational database engine compatible with MySQL and PostgreSQL. Built on a cloud-native, distributed storage system that replicates 6 copies of data across 3 Availability Zones, Aurora delivers high availability and performance. However, its multi-dimensional billing structure (compute, storage tiering, I/O requests, and replication) requires careful cost management.

---

## 2. Billing Mechanics
Aurora billing is split across four main dimensions:
1. **Database Compute:** Offered in two models:
   * *Provisioned Instances:* Billed hourly per instance (e.g., `db.r6g.large`).
   * *Serverless v2:* Billed per second based on **Aurora Capacity Units (ACUs)** consumed.
2. **Storage & I/O Configurations:** Configured at the cluster level:
   * *Aurora Standard:* Lower base storage cost ($0.10/GB-mo), but charges separately for all read/write I/O requests ($0.20 per million requests).
   * *Aurora I/O-Optimized:* Higher base storage ($0.225/GB-mo) and a ~30% compute premium, but provides **$0.00** unlimited I/O requests.
3. **Backup Storage:** Charged per GB-month for automated backups and manual snapshots exceeding 100% of active database volume ($0.095/GB-mo).
4. **Global Database Replication:** Charged for cross-region data transfer egress ($0.01–$0.02/GB) plus secondary region compute and storage.

---

## 3. Key Cost Dimensions

### A. Compute: Provisioned vs. Serverless v2 (us-east-1)
* **Provisioned Instances:** Flat hourly rate based on instance tier (e.g., `db.r6g.large` at $0.29/hour). Convertible and Reserved Instances offer up to 55% savings.
* **Serverless v2:** Dynamically scales database CPU and RAM based on workload demand.
  * *ACU Definition:* 1 ACU = 2 GB RAM with proportional CPU and networking.
  * *Pricing:* **$0.12 per ACU-hour** (in `us-east-1` under Standard configuration).
  * *Scaling Limits:* Scales in 0.5 ACU increments from a minimum of 0.5 ACUs to a maximum of 128 ACUs.
  * *Idle Compute Baseline:* Serverless v2 cannot scale down to 0 ACUs while active. A minimum of 0.5 ACUs generates a fixed idle charge:
    $$\text{Idle Cost} = 0.5\text{ ACU} \times 730\text{ hours} \times \$0.12 = \$43.80\text{ / month per instance}$$

### B. Standard vs. I/O-Optimized Storage Configurations

| Billing Metric | Aurora Standard Rate | Aurora I/O-Optimized Rate | Notes |
|----------------|----------------------|---------------------------|-------|
| **Serverless v2 Compute** | **$0.1200 / ACU-hr** | **$0.1560 / ACU-hr** | 30% compute premium |
| **`db.r6g.large` Compute** | **$0.2900 / hr** | **$0.3770 / hr** | 30% compute premium |
| **Storage Capacity** | **$0.1000 / GB-mo** | **$0.2250 / GB-mo** | 125% storage premium |
| **I/O Requests** | **$0.2000 / Million** | **Free ($0.00)** | Unlimited free I/Os |

* **Conversion Decision Rule:** Switch from Standard to **I/O-Optimized** if I/O request charges represent **more than 25%** of your total monthly Aurora bill.

### C. Backup Storage & Extended Support
* Backup storage up to **100% of active DB cluster size** is **100% Free ($0.00)**.
* Excess backup storage: **$0.095 per GB-month**.
* **RDS Extended Support Surcharge:** Running legacy engine versions (e.g., MySQL 5.7 or PostgreSQL 11) past EOL incurs an additional **$0.10 to $0.20 per vCPU-hour** charge.

---

## 4. Detailed Pricing Rates (us-east-1 MySQL Example)

| Billing Component | Aurora Standard Rate | Aurora I/O-Optimized Rate | Unit |
|-------------------|----------------------|---------------------------|------|
| **Serverless v2 Compute** | $0.1200 | $0.1560 | Per ACU-hour |
| **db.r6g.large Compute** | $0.2900 | $0.3770 | Per instance-hour |
| **Storage Capacity** | $0.1000 | $0.2250 | Per GB-month |
| **I/O Requests** | $0.2000 | **Free ($0.00)** | Per million requests |
| **Backup Storage** | $0.0950 | $0.0950 | Per GB-month |

---

## 5. AWS Free Tier Coverage
* **Amazon Aurora:** No free tier compute available. All active clusters generate standard billing immediately.

---

## 6. Common Cost Hotspots & Pitfalls
* **Standard I/O Cost Spikes on Write-Heavy Workloads:** Running intensive batch ETLs, unindexed queries, or frequent full-table scans on Aurora Standard, generating hundreds of millions of I/O charges ($0.20/M requests).
* **Idle Serverless v2 Clusters Left Uncapped:** Leaving Serverless v2 enabled on non-production clusters without setting strict `MaxACU` caps, allowing scaling spikes up to 128 ACUs ($15.36/hr).
* **Duplicating Storage with Manual Physical Snapshots:** Creating full database copies for testing rather than leveraging fast storage-cloning features.

---

## 7. Actionable Cost Optimization Strategies
1. **Evaluate I/O-Optimized vs. Standard Monthly:** Review AWS Cost Explorer. Convert clusters to **Aurora I/O-Optimized** if I/O charges exceed 25% of total cluster costs.
2. **Set Strict Max ACU Caps on Serverless v2:** In cluster auto-scaling settings, set `MaxACU` limits (e.g., max 4 to 8 ACUs for dev/staging, max 16 to 32 ACUs for medium production apps) to prevent query spikes from inflating compute fees.
3. **Use Aurora Fast Database Cloning for Testing:** Use **Aurora Fast Database Cloning** instead of creating manual snapshot copies for dev/test environments. Clones share underlying storage blocks, charging **$0.00 for storage** on all unmodified data.
4. **Auto-Stop Non-Production Clusters:** Schedule automated stop/start scripts (via Systems Manager or Lambda) to stop non-production Aurora clusters during nights and weekends to save compute fees.
