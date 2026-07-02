# AWS Service Cost Research: AWS Application Migration Service (MGN)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Application Migration Service (MGN) is the primary AWS service recommended for lift-and-shift migrations of physical, virtual, or cloud-hosted servers to AWS. MGN performs continuous, non-disruptive block-level replication of source server disks into a lightweight staging area in your AWS account. Once synchronized, you can launch non-disruptive test instances and perform final production cutovers with minimal downtime.

---

## 2. Billing Mechanics
1.  **Replication Free Period:** **90 days (2,160 hours) FREE per source server** registered in MGN.
2.  **Post-Free Period Rate:** Billed at **$0.042 per server per hour** (~$30.24 per month per server) if replication continues past the 90-day free window.
3.  **Underlying Staging Resources:** Billed for standard AWS resources provisioned during continuous replication:
    *   *Staging EC2 Instances:* Lightweight replication servers (e.g. `t3.small` / `c7a.medium`).
    *   *Staging EBS Storage:* EBS volumes (`gp3` / `st1`) storing continuous disk replicas.
    *   *Data Transfer:* Standard data transfer rates into AWS (inbound data transfer to AWS is free).

---

## 3. Key Cost Dimensions

| Feature / Timeline | Allowance | Rate (us-east-1) | Price for 100 Replicated Servers |
|--------------------|-----------|------------------|----------------------------------|
| **First 90 Days Replication**| 2,160 hrs / server| **Free ($0.00)** | **$0.00** |
| **Replication After 90 Days** | Per server-hour | **$0.042 / hr** | **$3,024.00 / month** |
| **Staging EC2 (`t3.small`)** | Per instance-hr | $0.0208 / hr | ~$1,500.00 / month |
| **Staging EBS Volume** | Per GB-month | $0.08 / GB-mo | Driven by source disk size |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Free Window:** 90 days free per source server.
*   **Post 90-Day Rate:** $0.042 per server per hour ($1.008 per server per day).

---

## 5. AWS Free Tier Coverage
*   **AWS MGN Free Tier:** Includes **90 days of free replication** for each server you add to AWS MGN.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Replication Stagnation (Exceeding the 90-Day Free Window):**
    *   Installing MGN replication agents on hundreds of source servers but delaying target testing and production cutover past 90 days.
    *   *Math:* Replicating 200 idle servers past 90 days costs **$6,048.00/month** in server fees alone, plus thousands in staging EBS volumes!

---

## 7. Actionable Cost Optimization Strategies
1.  **Complete Server Cutover Within the 90-Day Free Window:**
    *   Establish tight migration waves and complete target testing and final cutover within 90 days of installing the MGN agent.
    *   **The Savings:** Avoids all $0.042/hr MGN service charges ($0.00 total service bill).
2.  **Use Low-Cost Staging EC2 Instance Types:**
    *   Configure MGN staging area settings to use shared `t3.small` or `t3.medium` instances to handle block replication.
    *   **The Savings:** Slashes staging compute costs by **60%** compared to default larger instance types.
3.  **Delete Disconnected Source Servers:** Once production cutover is verified, disconnect and remove source servers from the MGN console to terminate staging EBS volumes and EC2 replication servers immediately.
