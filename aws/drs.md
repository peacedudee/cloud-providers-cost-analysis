# AWS Service Cost Research: AWS Elastic Disaster Recovery (DRS)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Elastic Disaster Recovery (DRS) is the recommended service for disaster recovery to AWS. It minimizes downtime and data loss by providing fast, reliable recovery of physical, virtual, and cloud servers into AWS. DRS continuously replicates your source machines (disks, OS, and databases) into a low-cost staging area in your AWS account. Because it uses a staging area to hold replicated data, your active recovery instances are only booted (and billed) during drills or actual failover events.

---

## 2. Billing Mechanics
DRS costs are split into replication charges and staging resource charges:
1.  **DRS Replication Fee:** A flat hourly fee charged per actively replicating server.
2.  **Staging Area Resource Fees:** You pay standard rates for the compute (EC2) and storage (EBS) resources used to maintain the replication staging area.
3.  **Failover/Drill Instance Fees:** Standard EC2, EBS, and OS licensing charges apply *only* when you launch recovery or drill instances.
4.  **Network Data Transfer:** Standard AWS data transfer rates apply if copying data across regions within AWS.

---

## 3. Key Cost Dimensions

### A. Flat Replication Surcharge (us-east-1)
*   **The Fee:** AWS charges **$0.028 per hour** (approximately **$20.44 per server per month**) for every source server actively replicating to AWS.
*   **Characteristics:** This is a flat rate per server. It does not scale based on the storage size of the server or the volume of disk changes.

### B. Staging Area Resources (The Hidden Overhead)
To keep data in sync, DRS automatically provisions a staging area in your target VPC:
*   **Replication Servers (EC2):** DRS provisions lightweight EC2 instances (usually `t3.small` or `t3.micro` instances) to write replicating data blocks. A single replication server can handle multiple source disks. You are billed standard EC2 hourly rates for these servers.
*   **Staging Disks (EBS):** DRS provisions EBS volumes that mirror the size of your source server disks. These are typically provisioned as cheap **sc1 (Cold HDD)** or **gp3** volumes. You pay standard EBS GB-month rates for these staging disks.
*   *Example:* Replicating a 1 TB server using a shared `t3.small` staging server and 1 TB of `gp3` storage adds ~$80/month in staging storage + ~$15/month compute + $20.44 replication fee.

### C. Failover / Drill Launches
When a disaster occurs or you execute a DR drill:
*   DRS launches target recovery EC2 instances using the replicated EBS volumes.
*   **Compute Charges:** You are billed standard hourly rates for these target EC2 instances (e.g. `m6i.xlarge` or `c6i.large`) only for the duration they run.
*   **Storage Charges:** Replicated EBS volumes are hydrated into standard high-speed SSD volumes (gp3) during launch, and you pay standard gp3 rates while they exist.
*   **OS Licensing:** Premium OS licenses (e.g., Windows Server, Red Hat Enterprise Linux) are billed hourly during the drill/failover period.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Component | Rate (us-east-1) | Monthly Cost (Est.) | Billing Type |
|-------------------|------------------|---------------------|--------------|
| **DRS Replication** | **$0.028 / hour** | ~$20.44 / server | Flat per-server rate |
| **Staging EC2 Host** | *Standard EC2 rates* | ~$15.00 / host | Shared replication VM |
| **Staging Storage** | *Standard EBS rates* | Varies by volume size | gp3 ($0.08/GB) or sc1 ($0.015/GB) |
| **Recovery/Drill compute** | *Standard EC2 rates* | Billed only when launched | Varies by target instance type |

---

## 5. AWS Free Tier Coverage
*   **30-Day Free Trial:** AWS DRS offers a **free trial of 30 days** per replicating server. Replication fees ($0.028/hr) are waived for the first 720 hours of replication per server. 
    *   *Warning:* Staging area compute (EC2) and storage (EBS) resources used during the free trial are **not free** and will generate standard billing.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Drill Instances Left Running:** Launching DR drills to test failover readiness, and forgetting to execute the "Archive" or "Delete Recovery Instance" command in the DRS console. If you drill 20 enterprise servers (e.g. m5.2xlarge) and leave them running for a month, it will add **thousands of dollars** to your bill.
*   **Over-Provisioning Staging Volumes:** Provisioning fast SSD (`gp3` or `io2`) storage in the staging area for source servers that have very low write volumes.
*   **Replication of Temp/Scratch Disks:** Including swap space, temp directories, or database staging disks in the replication scope. This causes continuous block changes, generating heavy disk writes and inter-AZ data transfer fees in AWS.

---

## 7. Actionable Cost Optimization Strategies
1.  **Exclude Temp/Scratch Volumes from Replication:** Configure the AWS Replication Agent on the source server to exclude swap files, temp volumes, and pagefiles from replication. This reduces staging storage sizes and eliminates unnecessary network block writes.
2.  **Clean up Recovery/Drill Instances Immediately:** Set up a rule or scheduler to automatically alert or terminate recovery drill instances after a set test period (e.g. 24 hours). Use the "Delete Recovery Instances" action in the DRS console to tear down the compute and storage resources cleanly.
3.  **Use Cold HDD (`sc1`) for Staging Storage:** For source servers with low transactional write rates (e.g. web servers, app servers, file servers), configure the replication template to use **sc1 HDD storage** instead of gp3. This reduces staging storage costs by **80%** ($0.015/GB vs $0.08/GB).
4.  **Consolidate Replication Servers:** Configure the replication template to allow multiple source servers to replicate through a single shared replication server (multi-tenant staging), reducing the count of hourly running EC2 instances.
5.  **Utilize the 30-Day Trial for Migrations:** If using DRS to migrate servers to AWS (rather than for DR), complete the migration and cutover within the 30-day free trial period to pay $0 in DRS replication fees.
