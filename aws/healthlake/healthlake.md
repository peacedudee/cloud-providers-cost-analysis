# AWS Service Cost Research: AWS HealthLake

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS HealthLake is a HIPAA-eligible, managed data store service designed for healthcare providers, health plans, and life sciences companies. It ingests, stores, searches, queries, and transforms healthcare data (electronic health records - EHR, clinical notes, lab reports) into standard Fast Healthcare Interoperability Resources (FHIR R4) formats. HealthLake uses integrated natural language processing (NLP via Text Analytics for Health) to extract clinical entities (medical conditions, medications, dosages) from unstructured clinical notes. HealthLake is billed per data store-hour, query capacity, and storage volume.

---

## 2. Billing Mechanics
1.  **Data Store Provisioning Hours:** Billed per hour per active HealthLake data store instance ($0.27 per data store-hour = ~$197.10 per month per data store). All new data stores are provisioned on the **Advanced Tier** (the legacy Standard Tier was discontinued in September 2023).
2.  **Free Query Allowance:** Each Data Store hour includes up to **3,500 queries per hour** at no additional charge.
3.  **Additional Queries:** Queries exceeding the 3,500/hour threshold are charged at **$0.048 per 10,000 queries**.
4.  **Structured FHIR Data Storage:** Billed per GB of structured FHIR resource data stored ($0.378 per GB-month).
5.  **FHIR Data Export:** Exporting data from HealthLake to Amazon S3 is billed at **$0.19 per GB** exported.
6.  **Integrated Medical NLP:** Analyzing unstructured text with built-in medical NLP is billed at **$0.0010 per 100 characters** processed (rounded up to the nearest 100 characters per request).

---

## 3. Key Cost Dimensions

| HealthLake Component | Billed Unit | Rate (us-east-1) | Monthly Price (1 Datastore / 100 GB) |
|----------------------|-------------|------------------|---------------------------------------|
| **Data Store Instance**| Per datastore-hr| **$0.270 / hour** | **$197.10 / month** |
| **FHIR Data Storage**| Per GB-month | **$0.378 / GB-mo** | **$37.80 / month** |
| **Additional Queries**| Per 10k queries | **$0.048 / 10k** | $0.00 (within 3,500/hr baseline) |
| **FHIR Data Export** | Per GB exported | **$0.190 / GB** | $19.00 (100 GB export) |
| **NLP Clinical Extraction**| Per 100 characters| **$0.0010** | $10.00 (1 million characters) |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Data Store Rate:** $0.27 per hour ($197.10 per month).
*   **FHIR Storage Rate:** $0.378 per GB-month.
*   **Additional Query Rate:** $0.048 per 10,000 queries.
*   **FHIR Export Rate:** $0.19 per GB.
*   **NLP Extraction Rate:** $0.0010 per 100 characters.

### Example Monthly Cost Calculation (Query Surcharge and NLP Usage)
*Workload: A clinic runs a single HealthLake Advanced data store. Over the month, they store 200 GB of structured FHIR data. During a peak data sync phase, they execute 3.6 million queries in a single hour. Additionally, they run medical NLP on 5 million characters of unstructured clinician notes.*

*   **Data Store Provisioning Cost:**
    $$\text{Data Store Cost} = 730\text{ hours} \times \$0.27 = \$197.10$$
*   **FHIR Storage Cost:**
    $$\text{Storage Cost} = 200\text{ GB} \times \$0.378 = \$75.60$$
*   **Additional Queries Surcharge (3,500 queries are free in that hour; 3,596,500 queries are billed):**
    $$\text{Queries Billed} = 3,600,000 - 3,500 = 3,596,500\text{ queries}$$
    $$\text{Query Surcharge} = \frac{3,596,500}{10,000} \times \$0.048 = \$17.26$$
*   **Medical NLP Cost:**
    $$\text{NLP Units Billed} = \frac{5,000,000\text{ characters}}{100\text{ characters}} = 50,000\text{ units}$$
    $$\text{NLP Cost} = 50,000 \times \$0.0010 = \$50.00$$
*   **Total Monthly Cost:** **$339.96/month**

---

## 5. AWS Free Tier Coverage
*   **AWS HealthLake Free Trial:** Includes a **30-day free trial** of 1 data store instance, 10 GB of storage, and 10,000 API requests for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Provisioning Multiple Isolated Data Stores for Small Sub-Departments:** Creating separate HealthLake data stores ($197.10/mo per datastore) for different clinic departments when a single shared datastore would suffice.
*   **Continuous Over-Limit Queries:** Running inefficient search scripts that continuously exceed the 3,500 queries/hour baseline, leading to unnecessary query surcharges.

---

## 7. Actionable Cost Optimization Strategies
1.  **Consolidate Medical Departments into Shared HealthLake Data Stores:**
    *   Consolidate multiple internal clinical departments onto a single shared HealthLake data store using FHIR Compartment definitions and RBAC tagging.
    *   **The Savings:** Eliminates redundant $197.10/mo base data store provisioning charges, saving **$2,365.20 per year** per redundant data store.
2.  **Batch NLP Extractions and Avoid Re-Processing Historical Notes:**
    *   Run Text Analytics for Health NLP extractions ONCE upon ingestion of unstructured clinical notes, saving extracted FHIR extension attributes in the datastore to avoid triggering repeat NLP extractions ($0.0010 per 100 characters).
3.  **Optimize Search Queries to Stay Within Baseline:**
    *   Distribute high-volume query tasks over time rather than executing them in massive bursts, preventing spikes above the **3,500 queries/hour** limit.
