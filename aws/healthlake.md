# AWS Service Cost Research: AWS HealthLake

> **Status:** ✅ Research Complete (181/181 AWS Services Completed!)
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS HealthLake is a HIPAA-eligible, managed data store service designed for healthcare providers, health plans, and life sciences companies. It ingests, stores, searches, queries, and transforms healthcare data (electronic health records - EHR, clinical notes, lab reports) into standard Fast Healthcare Interoperability Resources (FHIR R4) formats. HealthLake uses integrated natural language processing (NLP via Text Analytics for Health) to extract clinical entities (medical conditions, medications, dosages) from unstructured clinical notes. HealthLake is billed per data store-hour, storage volume, and API request.

---

## 2. Billing Mechanics
1.  **Data Store Provisioning Hours:** Billed per hour per active HealthLake data store instance ($0.28 per data store-hour = ~$204.40 per month per data store).
2.  **Structured FHIR Data Storage:** Billed per GB of structured FHIR resource data stored ($0.378 per GB-month).
3.  **API Requests (CRUD & Search):** Billed per 10,000 API calls ($0.715 per 10,000 API requests).
4.  **Text Analytics for Health (NLP Extractions):** Billed per 1,000 clinical records processed ($25.00 per 1,000 records).

---

## 3. Key Cost Dimensions

| HealthLake Component | Billed Unit | Rate (us-east-1) | Monthly Price (1 Datastore / 100 GB) |
|----------------------|-------------|------------------|---------------------------------------|
| **Data Store Instance**| Per datastore-hr| **$0.280 / hour** | **$204.40 / month** |
| **FHIR Data Storage**| Per GB-month | **$0.378 / GB-mo** | **$37.80 / month** |
| **API Requests** | Per 10k calls | **$0.715 / 10k** | **$7.15** (100k API calls) |
| **NLP Clinical Extraction**| Per 1,000 records| **$25.00 / 1k**| $250.00 (10k clinical notes) |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Data Store Rate:** $0.28 per hour ($204.40 per month).
*   **FHIR Storage Rate:** $0.378 per GB-month.
*   **API Request Rate:** $0.715 per 10,000 requests.
*   **NLP Extraction Rate:** $25.00 per 1,000 clinical records.

---

## 5. AWS Free Tier Coverage
*   **AWS HealthLake Free Tier:** Includes a **30-day free trial** of 1 data store instance, 10 GB of storage, and 10,000 API requests.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Provisioning Multiple Isolated Data Stores for Small Sub-Departments:**
    *   Creating separate HealthLake data stores ($204.40/mo per datastore) for different clinic departments or small pilot projects when a single shared datastore would suffice.

---

## 7. Actionable Cost Optimization Strategies
1.  **Consolidate Medical Departments into Shared HealthLake Data Stores:**
    *   Consolidate multiple internal clinical departments onto a single shared HealthLake data store using FHIR Compartment definitions and RBAC tagging.
    *   *Why:* Eliminates redundant $204.40/mo base data store provisioning charges.
    *   **The Savings:** Saves **$2,452.80 per year** for every redundant data store eliminated.
2.  **Batch NLP Extractions and Avoid Re-Processing Historical Notes:** Run Text Analytics for Health NLP extractions ONCE upon ingestion of unstructured clinical notes, saving extracted FHIR extension attributes in the datastore to avoid triggering repeat $25.00/1k record NLP extractions.
