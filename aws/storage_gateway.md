# AWS Service Cost Research: AWS Storage Gateway

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Storage Gateway is a hybrid cloud storage service that gives on-premises applications access to virtually unlimited AWS cloud storage. It deploys a local virtual machine (or hardware appliance) on-premises that caches files locally while syncing them back to AWS. It is a common billing pitfall because charges are split across local compute, data transfer, data write processing, and target AWS storage classes.

---

## 2. Billing Mechanics
Storage Gateway bills on a pay-as-you-go model with three primary cost dimensions:
1.  **Data Written to AWS (Processing Fee):** Billed per GB of data written by the gateway to AWS.
2.  **Target Cloud Storage:** Billed standard rates for the underlying AWS storage service used (S3, EBS, or Glacier).
3.  **Virtual Machine Hosting:** Billed standard EC2 rates if you host the gateway VM in AWS (though local on-premises VM software is free).
4.  **Network Data Transfer:** Standard AWS data transfer egress fees apply when retrieving data from AWS back to the on-premises gateway.

---

## 3. Key Cost Dimensions

### A. Data Write Processing Fees (us-east-1)
*   **The Charge:** AWS charges **$0.01 per GB** of data uploaded to AWS through the gateway.
*   **Monthly Cap:** This write processing fee is capped at **$125.00 per gateway per month**.
*   **Free Tier:** The first 100 GB written to AWS per account is free.
*   *Note: In the past, AWS charged a flat $125.00/month virtual appliance software fee. That flat fee has been eliminated and replaced entirely by this processing cap.*

### B. Target Cloud Storage Rates
Storage Gateway stores data in different AWS services based on the gateway type:
*   **S3 File Gateway:** Stores files as standard S3 objects. You pay standard S3 storage rates (Standard, IA, or Intelligent-Tiering) plus standard S3 request fees (Class A/B).
*   **FSx File Gateway:** Links to FSx for Windows File Server. You pay standard FSx for Windows storage and throughput charges.
*   **Volume Gateway (Cached/Stored):** Stores block volumes in S3. Storage is billed at a flat rate of **$0.023 per GB-month** in `us-east-1`. Snapshots are billed at standard EBS snapshot rates (**$0.05/GB-month**).
*   **Tape Gateway (Virtual Tape Library):** 
    *   *Active Virtual Tapes:* Billed at **$0.023 per GB-month**.
    *   *Archived Tapes (Glacier Flexible Retrieval):* Billed at **$0.0036 per GB-month**.
    *   *Archived Tapes (Glacier Deep Archive):* Billed at **$0.00099 per GB-month**.

### C. Retrieval and Egress Charges (Network Egress)
*   **Inbound Data:** Free (on-premises to AWS).
*   **Outbound Data (Internet Egress):** Retrieving files from AWS back to the on-premises gateway cache incurs standard AWS Internet Egress fees:
    *   First 100 GB/month: **Free**.
    *   Up to 10 TB/month: **$0.09 per GB**.
*   **Glacier Archive Retrieval Fees (Tape Gateway):** Retrieving archived virtual tapes from Glacier back to the active VTL costs **$0.01 per GB** (Glacier Flexible) or **$0.03 per GB** (Glacier Deep Archive) plus S3 request charges.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Dimension | Billing Basis | Rate (us-east-1) | Monthly Cap / Notes |
|-------------------|---------------|------------------|---------------------|
| **Data Written to AWS** | Per GB uploaded | **$0.01 / GB** | Capped at **$125.00** per gateway |
| **Volume Gateway Storage** | Per GB-month stored | **$0.023 / GB-month** | Backed by S3 |
| **Tape Gateway (Active)** | Per GB-month stored | **$0.023 / GB-month** | Virtual Tape Library |
| **Tape Gateway (Glacier Archive)** | Per GB-month stored | **$0.0036 / GB-month** | Glacier Flexible Retrieval |
| **Tape Gateway (Deep Archive)** | Per GB-month stored | **$0.00099 / GB-month**| Glacier Deep Archive |
| **Tape Gateway Retrieval** | Per GB retrieved | **$0.01 – $0.03 / GB**| Varies by Archive class |

---

## 5. AWS Free Tier Coverage
*   **Data Written:** 100 GB written to AWS per billing account for free (applies to new accounts).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Unnecessary Gateway VM Sprawl:** Running multiple separate file gateways for different departments. Each gateway that uploads more than 12.5 TB of data will hit the **$125/month write cap** separately.
*   **High Internet Egress Charges (Cache Misses):** If the local on-premises gateway VM is provisioned with a tiny local cache disk, files will be constantly evicted. On-premises clients requesting these files will cause a "cache miss," forcing the gateway to download the file from AWS over the internet, generating heavy Internet Egress fees ($0.09/GB).
*   **Virtual Tape Retrieval Storms:** Retrieving tens of terabytes of archived virtual tapes from Glacier Deep Archive to perform a legacy audit. The retrieval processing fee of $0.03/GB plus transition requests can generate thousands in unexpected bills.
*   **E2 Instance Fees for Cloud Deployment:** Running the Storage Gateway appliance on an EC2 instance (e.g. `m5.xlarge`) inside AWS to bridge VPCs. You pay standard EC2 hourly rates for the host VM.

---

## 7. Actionable Cost Optimization Strategies
1.  **Consolidate Gateways where Possible:** Instead of deploying a separate gateway VM for every team or office, consolidate file shares onto a single large Storage Gateway. This helps share the local cache and aggregates uploads under a single **$125/month data write cap**.
2.  **Right-Size the Local Cache Disk (Avoid Cache Misses):** Check the `CacheHitPercent` metric in CloudWatch. If it is below 90%, expand the local SSD cache storage allocated to the gateway VM. A larger local cache reduces cache misses, preventing expensive AWS internet egress charges.
3.  **Transition Virtual Tapes to Deep Archive:** Configure your on-premises backup software (like Veeam or Backup Exec) to write virtual tapes and immediately eject them to the tape vault. Move long-term compliance tapes to **Glacier Deep Archive** to drop tape storage costs from $0.023/GB to **$0.00099/GB-month**.
4.  **Use S3 Lifecycle Rules for File Gateways:** S3 File Gateways write directly to S3. Apply S3 Lifecycle rules on the target bucket to transition old files to S3 Standard-IA or S3 Glacier to lower storage fees.
5.  **Utilize On-Premises VMs (Free Hosting):** Host the Storage Gateway VM on local VMware ESXi or Microsoft Hyper-V infrastructure. This is completely free, avoiding the need to pay hourly EC2 instance charges.
