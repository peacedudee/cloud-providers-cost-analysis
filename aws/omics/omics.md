# AWS Service Cost Research: AWS HealthOmics (AWS Omics)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS HealthOmics (formerly AWS Omics) is a purpose-built AWS service designed for healthcare providers, pharmaceutical companies, population genomics initiatives, and life science research organizations. It provides managed infrastructure to store, query, and run analysis workflows on genomic, transcriptomic, and proteomic datasets (FASTQ, BAM, CRAM, VCF files) at scale. HealthOmics includes **Omics Storage** (Sequence, Variant, and Annotation Stores) and **Omics Workflows** (Private Nextflow/WDL/CWL pipelines and Ready2Run pre-built pipelines). HealthOmics is billed per storage tier and compute run-hour or fixed per-run rates.

---

## 2. Billing Mechanics
1. **Omics Storage Tiers (Automated Tiering):**
   * *Active Sequence Store:* Fast read access for active genomic analysis ($0.09 per GB-month).
   * *Archive Sequence Store:* Automatic archival storage for raw read files inactive for 30+ days ($0.004 per GB-month = **95.5% savings**). Free reactivation.
   * *Variant & Annotation Stores:* High-performance query storage for genomic variants ($0.005 per GB-month).
2. **Private Workflows:** Billed per run-hour based on compute instance resources requested (e.g. `omics.c.xlarge` at $0.18 per run-hour; `omics.g.xlarge` GPU at $1.10 per run-hour). Each task includes 16 GiB of free ephemeral storage (expandable to 3,072 GiB).
3. **Ready2Run Workflows:** Pre-built pipelines (e.g., AlphaFold, GATK-BP, NVIDIA Parabricks) billed at a predictable **fixed cost per run**.

---

## 3. Key Cost Dimensions

| Omics Component / Tier | Billed Unit | Rate (us-east-1) | Monthly Price for 10 TB Datasets |
|------------------------|-------------|------------------|----------------------------------|
| **Active Sequence Store**| Per GB-month | **$0.090 / GB-mo** | **$900.00 / month** |
| **Archive Sequence Store**| Per GB-month | **$0.004 / GB-mo** | **$40.00 / month** (95.5% off) |
| **Variant Store** | Per GB-month | **$0.005 / GB-mo** | **$50.00 / month** |
| **Workflow (`omics.c.xlarge`)**| Per run-hour | **$0.180 / hour** | $1.80 per 10-hour pipeline run |
| **Ready2Run Workflow** | Per run | Fixed Rate | Predictable per-sample cost |

---

## 4. Detailed Pricing Rates (us-east-1)

* **Active Storage Rate:** $0.09 per GB-month ($90.00 per TB-month).
* **Archive Storage Rate:** $0.004 per GB-month ($4.00 per TB-month).
* **Variant Store Rate:** $0.005 per GB-month ($5.00 per TB-month).
* **CPU Compute Rate:** $0.18 per run-hour (`omics.c.xlarge`).

---

## 5. AWS Free Tier Coverage
* **AWS HealthOmics Free Tier:** Includes **275 instance-hours** of `omics.m.xlarge` compute, **49,000 GB-hours** of run storage for private workflows, and **1,500 gigabase-months** of active and archive storage.

---

## 6. Common Cost Hotspots & Pitfalls
* **Retaining Terabytes of Raw FASTQ Files in Active Sequence Store ($0.09/GB):**
  * Leaving 100 TB of raw sequencing reads in Active Sequence Storage ($9,000.00/mo) after alignment and variant calling pipelines complete.

---

## 7. Actionable Cost Optimization Strategies
1. **Automate Archiving of Raw FASTQ & BAM Files to Archive Sequence Store:**
   * Configure automated HealthOmics storage policies to move raw FASTQ and aligned BAM files to **Archive Sequence Store ($0.004/GB-mo)** as soon as downstream variant calling finishes (or let the 30-day auto-archive policy run).
   * *Why:* Raw sequence reads are rarely re-analyzed active files; storing them in Archive cuts billing by 95.5%. Reactivation is free.
   * **The Savings:** Slashes 100 TB storage billing from **$9,000.00 down to $400.00 per month ($8,600.00/mo saved!)**.
2. **Use Ready2Run Workflows for Predictable Sample Costs:** For standard pipelines like GATK or NVIDIA Parabricks, use Ready2Run workflows to secure flat, predictable per-sample pricing without tuning compute instance parameters.
3. **Optimize Private Workflow Task Instance Rightsizing:** Select exact CPU/memory instance sizes for Nextflow/WDL tasks in private workflow definitions, avoiding over-provisioning GPU instances (`omics.g.xlarge` at $1.10/hr) for simple CPU alignment steps.
