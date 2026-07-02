# AWS Service Cost Research: AWS HealthOmics (AWS Omics)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS HealthOmics (formerly AWS Omics) is a purpose-built AWS service designed for healthcare providers, pharmaceutical companies, population genomics initiatives, and life science research organizations. It provides managed infrastructure to store, query, and run analysis workflows on genomic, transcriptomic, and proteomic datasets (FASTQ, BAM, CRAM, VCF files) at scale. HealthOmics includes **Omics Storage** (Sequence, Variant, and Annotation Stores) and **Omics Workflows** (Nextflow, WDL, CWL pipeline execution engines). HealthOmics is billed per storage tier and compute run-hour.

---

## 2. Billing Mechanics
1.  **Omics Storage Tiers:**
    *   *Active Sequence Store:* Fast read access for active genomic analysis ($0.09 per GB-month).
    *   *Archive Sequence Store:* Long-term archival storage for raw read files ($0.004 per GB-month = **95.5% savings**).
    *   *Variant & Annotation Stores:* High-performance query storage for genomic variants ($0.005 per GB-month).
2.  **Omics Workflows (Private Workflows):** Billed per run-hour based on compute instance resources requested (e.g. `omics.c.xlarge` at $0.18 per run-hour; `omics.g.xlarge` GPU at $1.10 per run-hour). Each task includes 16 GiB of free ephemeral storage.
3.  **Ready2Run Workflows:** Billed flat fixed pricing per run.

---

## 3. Key Cost Dimensions

| Omics Component / Tier | Billed Unit | Rate (us-east-1) | Monthly Price for 10 TB Datasets |
|------------------------|-------------|------------------|----------------------------------|
| **Active Sequence Store**| Per GB-month | **$0.090 / GB-mo** | **$900.00 / month** |
| **Archive Sequence Store**| Per GB-month | **$0.004 / GB-mo** | **$40.00 / month** (95.5% off) |
| **Variant Store** | Per GB-month | **$0.005 / GB-mo** | **$50.00 / month** |
| **Workflow (`omics.c.xlarge`)**| Per run-hour | **$0.180 / hour** | $1.80 per 10-hour pipeline run |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Active Storage Rate:** $0.09 per GB-month ($90.00 per TB-month).
*   **Archive Storage Rate:** $0.004 per GB-month ($4.00 per TB-month).
*   **Variant Store Rate:** $0.005 per GB-month ($5.00 per TB-month).
*   **CPU Compute Rate:** $0.18 per run-hour (`omics.c.xlarge`).

---

## 5. AWS Free Tier Coverage
*   **AWS HealthOmics Free Tier:** None. Standard storage and workflow execution rates apply.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Retaining Terabytes of Raw FASTQ Files in Active Sequence Store ($0.09/GB):**
    *   Leaving 100 TB of raw sequencing reads in Active Sequence Storage ($9,000.00/mo) after alignment and variant calling pipelines complete.

---

## 7. Actionable Cost Optimization Strategies
1.  **Automate Archiving of Raw FASTQ & BAM Files to Archive Sequence Store:**
    *   Configure automated HealthOmics storage policies to move raw FASTQ and aligned BAM files to **Archive Sequence Store ($0.004/GB-mo)** as soon as downstream variant calling finishes.
    *   *Why:* Raw sequence reads are rarely re-analyzed active files; storing them in Archive cuts billing by 95.5%.
    *   **The Savings:** Slashes 100 TB storage billing from **$9,000.00 down to $400.00 per month ($8,600.00/mo saved!)**.
2.  **Optimize Workflow Task Instance Rightsizing:** Select exact CPU/memory instance sizes for Nextflow/WDL tasks in private workflow definitions, avoiding over-provisioning GPU instances (`omics.g.xlarge` at $1.10/hr) for simple CPU alignment steps.
