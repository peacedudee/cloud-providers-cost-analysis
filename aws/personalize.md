# AWS Service Cost Research: Amazon Personalize

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Personalize is a fully managed machine learning service that allows developers to build and deploy real-time curated recommendations (such as product recommendations, personalized search, and customized email marketing) at scale. It abstracts ML model building, training, and deployment. Since running personalized recommendation engines requires persistent endpoints, idle real-time campaigns represent a significant source of high baseline costs.

---

## 2. Billing Mechanics
Amazon Personalize billing is structured around three main dimensions:
1.  **Data Ingestion:** Charged per GB of data uploaded to the service (via S3 batch uploads or real-time event streaming).
2.  **Model Training:** Charged based on the compute training hours consumed to train custom models.
3.  **Inference (Recommendations):**
    *   *Real-Time Campaigns:* Billed hourly based on Transactions Per Second (TPS) capacity.
    *   *Batch Recommendations:* Billed per 1,000 recommendations generated.

---

## 3. Key Cost Dimensions

### A. Data Ingestion (us-east-1)
*   **The Rate:** **$0.05 per GB** of data uploaded.
*   *Applies to:* Bulk S3 uploads (users, items, and historical interaction datasets) and real-time interaction logs streamed via the Event Tracker API.

### B. Model Training
*   **The Rate:** **$0.24 per training hour**.
*   *Definition:* A training hour represents 1 hour of compute capacity utilizing 4 vCPUs and 8 GiB memory. Training time scales automatically with the size of your datasets and selected recipes.

### C. Inference Options (Real-Time vs. Batch)
*   **Real-Time Campaigns (TPS-hour Billing):** Billed based on the maximum of your minimum provisioned TPS or actual TPS (5-minute average).
    *   *0 to 20,000 TPS-hours/month:* **$0.20 per TPS-hour**.
    *   *20,000 to 200,000 TPS-hours/month:* **$0.10 per TPS-hour**.
    *   *Over 200,000 TPS-hours/month:* **$0.05 per TPS-hour**.
    *   *The Idle Campaign Hazard:* Amazon Personalize campaigns enforce a **minimum billing threshold of 1 TPS** per active campaign. Keeping a real-time campaign active with 0 traffic generates:
        $$1\text{ TPS} \times 24\text{ hours} \times \$0.20 = \$4.80\text{ / day}$$
        $$\$4.80 \times 30.5\text{ days} = \$146.40\text{ / month flat baseline fee per campaign}$$
*   **Batch Recommendations:** Used for offline batch jobs (e.g., generating recommendations for email marketing campaigns).
    *   *The Rate:* **$0.086 per 1,000 recommendations** generated.

---

## 4. Detailed Pricing Rates (us-east-1)

| Cost Component | Billed Metric | Rate (us-east-1) | Details |
|----------------|---------------|------------------|---------|
| **Data Ingestion**| Per GB | **$0.050** | S3 imports & real-time streams |
| **Model Training**| Per compute-hour | **$0.240** | Scales based on dataset size |
| **Real-Time Inference**| Per TPS-hour (0-20K)| **$0.200** | Capped at minimum 1 TPS ($146.40/mo) |
| **Batch Inference**| Per 1,000 recommendations| **$0.086** | For offline batch runs |

---

## 5. AWS Free Tier Coverage
*   **Data Ingestion:** First 20 GB free per month (first 2 months).
*   **Model Training:** First 100 training hours free per month (first 2 months).
*   **Real-Time Inference:** First 185,000 recommendation requests (or equivalent TPS capacity) free per month (first 2 months).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Orphaned Real-Time Campaigns:** Leaving experimental or non-production recommendation campaigns running in development or QA environments. Each active campaign drains **$146.40/month** even if 0 requests are processed.
*   **Over-Provisioning Minimum TPS:** Setting a high minimum provisioned TPS (e.g., 20 TPS) to handle peak traffic without setting up dynamic scaling, inflating idle capacity bills (20 TPS-hours = $4.00/hour = **$2,920.00/month**).
*   **Frequent Retraining on Large Datasets:** Scheduling full model retraining (solutions) daily for large historical datasets where incremental updates or weekly retraining would suffice.

---

## 7. Actionable Cost Optimization Strategies
1.  **Delete Campaigns in Non-Production Environments:**
    *   Do not leave recommendation campaigns active in development, staging, or test environments.
    *   Automate deletion of campaigns when tests are complete. Re-create them via the SDK only when new testing sessions begin.
    *   **The Savings:** Saves the **$146.40/month** idle minimum fee per campaign.
2.  **Prefer Batch Recommendations for Async Use Cases:**
    *   If recommendations are delivered via email newsletters, scheduled reports, or static user profile pages, use **Batch Recommendations**.
    *   Run batch recommendations once a day or week, export outputs to S3/DynamoDB, and serve them locally.
    *   **The Savings:** Bypasses continuous hourly TPS campaigns, paying only a fraction of the cost ($0.086/1,000).
3.  **Optimize Minimum TPS Configurations:**
    *   For real-time campaigns, set the minimum provisioned TPS to **1** (the absolute baseline).
    *   Rely on Amazon Personalize's automatic scaling to handle spikes, and let it scale back down to the baseline minimum when traffic drops.
4.  **Decrease Solution Retraining Frequencies:**
    *   Avoid daily retraining of the entire dataset.
    *   Use **Solution Version Creation** with incremental training or update solutions weekly or bi-weekly.
    *   Keep user-interaction logs pruned to exclude ancient history that no longer represents current behavior.
