# AWS Service Cost Research: Amazon Forecast

> **Status:** ⚠️ Deprecated / Legacy Support only
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Forecast is a legacy fully managed service that uses machine learning to deliver highly accurate time-series forecasts. It was built on the same technology used for time-series forecasting at Amazon.com.
*   **Critical Cost Notice:** As of **2024**, AWS has deprecated Amazon Forecast for new customers. The service remains active strictly for existing customer workloads. AWS recommends migrating forecasting pipelines to **Amazon SageMaker Canvas** or using SageMaker native algorithms (like DeepAR).

---

## 2. Billing Mechanics
Amazon Forecast billing for legacy workloads is pay-as-you-go based on three primary dimensions:
1.  **Forecast Points Generated:** Billed per 1,000 forecasts generated (1 forecast point represents one prediction at a specific time step).
2.  **Training Compute Time:** Billed per hour of compute consumed during model training (Auto-predictor tuning).
3.  **Data Ingestion & Storage:** Billed per GB of data ingested and stored in the service database.

---

## 3. Key Cost Dimensions

### A. Generated Forecasts (us-east-1 Rates)
*   **The Rate:** **$0.60 per 1,000 forecasts** ($0.0006 per forecast point).
*   *Math Example:* If you forecast hourly demand for 100 products over a 24-hour horizon:
    $$100\text{ products} \times 24\text{ hours} = 2,400\text{ forecast points}$$
    $$\text{Cost: } 2,400 \times \$0.0006 = \$1.44\text{ per run}$$

### B. Model Training Compute
*   **The Rate:** **$0.24 per hour** of training compute.
*   *Compute Surcharges:* The training duration depends on the volume of historical data and selected prediction algorithms. AWS automatically runs training in parallel, multiplying the effective hour metric.

### C. Data Ingestion
*   **The Rate:** **$0.088 per GB** of time-series data ingested into Amazon Forecast storage.

---

## 4. Detailed Pricing Rates (us-east-1 Legacy Workloads)

| Forecast Component | Billing Metric | Rate (us-east-1) | Details |
|--------------------|----------------|------------------|---------|
| **Forecast Generation**| Per 1,000 predictions | **$0.600** | Billed per forecast point |
| **Model Training** | Per training compute-hour| **$0.240** | Auto-predictor optimization |
| **Data Ingestion** | Per GB | **$0.088** | Target csv data ingested |

---

## 5. AWS Free Tier Coverage
*   **Amazon Forecast:** Legacy free tier included 10,000 free forecast points, 10 GB data storage, and up to 10 training hours per month for the first 12 months.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Generating Excessive Forecast Horizons:** Configuring a pipeline to forecast hourly demand for 90 days out when the business only acts on the next 7 days. This generates 12x the forecast points unnecessarily.
*   **Inefficient Data Training Schedules:** Training a new predictor model daily on historical data when monthly retraining is sufficient, driving up training hours.
*   **Leaving Historical Data in Forecast Datasets:** Retaining massive, historical transactional CSV files in Forecast ingestion storage when they have already been trained.

---

## 7. Actionable Cost Optimization Strategies
1.  **Migrate to SageMaker Canvas or DeepAR:**
    *   Since Amazon Forecast is deprecated, transition forecasting pipelines.
    *   For no-code users, migrate to **Amazon SageMaker Canvas** (offering modern ML forecasting models with standard ML hosting fees).
    *   For developers, migrate to **SageMaker Notebooks** running the open-source **DeepAR** or **Prophet** algorithms on Spot instances, which cuts training compute costs by up to **90%**.
2.  **Optimize Forecast Horizons:** Review your forecast configurations. Reduce the `ForecastHorizon` parameter to match your actual business planning cycle (e.g. only predict 7 days instead of 30 days).
3.  **Reduce predictor retraining frequencies:** Do not retrain models on every forecast run. Retrain predictors only when new historical trends are introduced (e.g. monthly) and use existing predictors for daily inferences.
4.  **Delete Inactive Predictors:** Go to the Amazon Forecast console, clean up unused datasets, and delete historical predictors that are no longer referenced by automated reporting systems to stop data ingestion storage charges.
