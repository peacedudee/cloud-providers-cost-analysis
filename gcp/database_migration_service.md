# Database Migration Service (DMS) Cost Research

Google Cloud Database Migration Service (DMS) provides serverless, guided migrations to MySQL, PostgreSQL, and SQL Server databases on Cloud SQL and AlloyDB. While DMS is a high-value tool, managing network bandwidth and cleanup practices is key to preventing unexpected billing.

---

## 1. DMS Billing Mechanics

DMS operates under a very user-friendly pricing structure:
1. **Migration Service Fee:** **DMS is 100% free** for migrating databases to Cloud SQL or AlloyDB. There is no charge for the migration orchestrator or serverless workers.
2. **Target Database Billing:** You are charged standard rates for the target database instances (Cloud SQL, AlloyDB) that you create to receive the data.
3. **Data Egress & Network Connectivity:**
   * Ingress (data entering GCP) is free.
   * If your source database is inside another cloud provider (e.g. AWS RDS) or another GCP region, transferring the database dump and continuous change data capture (CDC) logs will incur **network egress fees** from the source environment.
4. **Temporary Connection Resources:** Billed for Cloud VPN, Interconnect, or Reverse SSH tunnel instances used to establish secure connectivity between the source and target databases during the migration window.

---

## 2. Core Cost-Reduction Tactics

### A. Egress Cost Management (Cross-Cloud / Cross-Region)
* Migrating a multi-terabyte database across cloud providers generates substantial egress.
* **Action:**
  * If migrating from AWS RDS or Azure SQL, establish the connection over **Cloud VPN** or **Cloud Interconnect** rather than public internet transit, as VPN/Interconnect routes egress at a discounted rate compared to standard internet egress.
  * Keep your destination Cloud SQL/AlloyDB instances in the **same region** as the migration pipeline endpoints to avoid inter-region GCP network charges.

### B. ephemerality: Decommission Migration Connections
* During migration, engineers provision Reverse SSH tunnels, Cloud VPNs, or proxy VMs (Compute Engine) to bridge the network gap between databases.
* **Action:** Immediately destroy, stop, or delete these connectivity resources (VPN tunnels, route entries, proxy VMs) as soon as the database migration is complete and the application cutover is finalized.

### C. Right-Size Target DB During Sync (Scale Up for Load, Scale Down for Live)
* **Tactic:**
  1. During the initial full data dump phase, provision the target Cloud SQL database as a large instance (high vCPU and memory) with HA disabled and read replicas deleted to maximize write throughput.
  2. Once the migration finishes and the databases enter continuous replication (CDC phase), scale down the target database size to the correct production shape.
  3. Enable HA and add read replicas only prior to the final application cutover.
* **The Benefit:** Decreases total compute instance billing during the days or weeks of migration sync testing.

---

## 3. DMS Audit Checklist

1. [ ] **Target Instance Rightsizing:** Confirm the target database size is set to a cost-effective shape during testing.
2. [ ] **Connection Decommissioning:** Ensure all VPN tunnels, static IPs, and SSH proxy VMs are deleted after migration completion.
3. [ ] **Egress Routing Check:** Verify that data replication routes over private tunnels or dedicated connections rather than public internet to leverage discounted egress rates.
4. [ ] **Clean Up DMS Console:** Complete or delete migration jobs that have successfully cut over to prevent stale connections or monitoring checks.
