# Cost-Cutting Playbook: Amazon Forecast
> **Companion File:** [forecast.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/forecast/forecast.md)
> **Last Updated:** July 2026

---

## Executive Summary
Amazon Forecast is a legacy, fully managed time-series forecasting service. As of 2024, AWS deprecated it for new customers, maintaining it strictly for existing legacy workloads. Because of this deprecated status, the primary cost-optimization recommendation is migrating off the service to Amazon SageMaker Canvas or SageMaker-hosted open-source algorithms. For workloads still operating on Amazon Forecast, costs are driven primarily by three dimensions: Forecast Points Generated ($0.60 per 1,000 forecasts), Model Training Compute ($0.24/hour), and Data Ingestion ($0.088/GB). Optimizing legacy pipelines involves drastically reducing forecast horizons, lowering retraining frequencies, and purging inactive models and datasets.

## Strategy Categories

### 1. Waste Elimination
### 2. Rightsizing
### 3. Commitment Discounts
*(Not applicable to Amazon Forecast; AWS does not offer Savings Plans or Reserved Instances for this service.)*
### 4. Architecture Changes
### 5. Scheduling & Auto-Scaling
### 6. Pricing Model Optimization
### 7. Network & Data Transfer Optimization

---

## Cross-Service Synergies
- **Amazon SageMaker Canvas:** The recommended migration destination for no-code Amazon Forecast users. 
- **Amazon SageMaker:** The recommended destination for data science teams to deploy DeepAR or Prophet models on Spot instances.
- **Amazon S3:** Used for importing historical datasets and exporting forecast predictions. Storing redundant CSVs in S3 inflates storage costs.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Look for usage types related to `AmazonForecast-Points`, `AmazonForecast-TrainingCompute`, and `AmazonForecast-IngestionStorage`.
### B. CloudWatch Metrics
- Not heavily integrated for cost-tracking natively, but useful for monitoring Lambda triggers if you have event-driven pipelines wrapping Forecast.
### C. AWS Config / Trusted Advisor
- AWS Config can track dataset and predictor configuration state. 
### D. Company Policies
- Data retention policies for historical forecasting datasets.
- Required business horizons (e.g., does the business act on 7-day or 30-day horizons?).
### E. IaC (Optional)
- Review CloudFormation or Terraform configurations to identify automated retraining pipelines.

---

## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "FORECAST-001",
  "strategy_name": "Migrate from Amazon Forecast to SageMaker Canvas",
  "category": "Architecture Changes",
  "estimated_savings_percentage": 50,
  "risk_level": "Medium",
  "scope": "Engineer/DevOps"
}
```

### Summary Report Table

#### 1. FORECAST-001: Delete Inactive Legacy Predictors
- **What:** Identify and delete machine learning predictor models that were trained but are no longer actively used to generate forecasts.
- **Why It Saves Money:** Storing inactive datasets and historical models continues to incur the $0.088 per GB data ingestion storage fee.
- **Implementation Steps:** 
  1. Use the AWS CLI (`aws forecast list-predictors`) to inventory all predictors.
  2. Cross-reference predictors with active forecasting jobs.
  3. Delete predictors that haven't generated forecasts in the last 30 days.
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Visibility into which predictors are actively queried.

#### 2. FORECAST-002: Purge Stale Ingestion Datasets
- **What:** Remove historical time-series datasets (CSVs) that have already been used to train predictors and are no longer needed.
- **Why It Saves Money:** Prevents paying the $0.088/GB ingestion fee for old data.
- **Implementation Steps:**
  1. Identify datasets in Amazon Forecast older than your retention policy.
  2. Confirm the data is safely backed up in S3 standard or Glacier.
  3. Delete the dataset from Amazon Forecast (`aws forecast delete-dataset`).
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** S3 backup of original data.

#### 3. FORECAST-003: Turn Off Test/Dev Forecast Environments
- **What:** Shut down and delete predictors, datasets, and forecast pipelines in non-production environments that were used for POCs.
- **Why It Saves Money:** Eliminates 100% of the cost for dev/test forecasting environments which are no longer needed since the service is deprecated.
- **Implementation Steps:**
  1. Identify AWS accounts or tags corresponding to Dev/Test environments.
  2. Run a cleanup script to delete all Forecast resources in those environments.
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Approval to tear down Dev/Test infrastructure.

#### 4. FORECAST-004: Delete Failed Predictor Trainings
- **What:** Clean up artifacts and datasets associated with failed predictor training runs.
- **Why It Saves Money:** Failed training attempts can still leave residual datasets consuming storage at $0.088/GB.
- **Implementation Steps:**
  1. Filter predictors by status `CREATE_FAILED`.
  2. Delete the failed predictors and any associated temporary datasets.
- **Estimated Savings:** <1%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None

#### 5. FORECAST-005: Remove Redundant Forecast Exports
- **What:** Stop exporting the exact same forecast results to multiple S3 buckets.
- **Why It Saves Money:** Reduces S3 storage costs and cross-region transfer fees if exported to remote buckets.
- **Implementation Steps:**
  1. Review the destination paths of `CreateForecastExportJob`.
  2. Consolidate exports to a single centralized S3 bucket.
- **Estimated Savings:** <1%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Centralized data lake architecture.

#### 6. FORECAST-006: Reduce Forecast Horizon
- **What:** Decrease the `ForecastHorizon` parameter to only predict the exact number of future time steps required by the business.
- **Why It Saves Money:** Forecast generation is billed at $0.60 per 1,000 forecast points. Reducing a 30-day hourly horizon to a 7-day horizon reduces the forecast points generated by ~76%, drastically lowering the execution bill.
- **Implementation Steps:**
  1. Audit how the business uses the forecast data.
  2. Adjust the `ForecastHorizon` parameter in your `CreateForecast` API call.
- **Estimated Savings:** 20-80%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Alignment with business stakeholders on required planning horizons.

#### 7. FORECAST-007: Decrease Forecast Granularity
- **What:** Change the frequency of the forecast from hourly or minute-by-minute to daily or weekly.
- **Why It Saves Money:** Generating a daily forecast over 7 days produces 7 points. An hourly forecast produces 168 points. Moving to daily reduces the forecast points bill by 95%.
- **Implementation Steps:**
  1. Verify if downstream systems require hourly data or if daily data suffices.
  2. Change the dataset frequency and predictor frequency to `D` (Daily).
- **Estimated Savings:** 50-95%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Downstream systems must support aggregated daily metrics.

#### 8. FORECAST-008: Filter Input Item Time Series
- **What:** Stop generating forecasts for deprecated, out-of-stock, or low-value items.
- **Why It Saves Money:** Every item processed multiplies the forecast points generated. Removing 20% of your product catalog from the input dataset saves 20% on forecast generation costs.
- **Implementation Steps:**
  1. Pre-process your target time-series CSV in S3 before importing it to Forecast.
  2. Filter out SKUs or entities that no longer require active forecasting.
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Data engineering pipeline to filter S3 CSVs.

#### 9. FORECAST-009: Limit the Number of Quantiles Generated
- **What:** Generate only the specific prediction quantiles your business uses (e.g., only p50, instead of p10, p50, p90).
- **Why It Saves Money:** While Forecast charges per forecast *point*, generating excessive custom quantiles in legacy configurations can multiply export sizes and processing time.
- **Implementation Steps:**
  1. Review the `ForecastTypes` parameter in the `CreateForecast` API.
  2. Restrict to `["0.50"]` if P10 and P90 confidence intervals are ignored by business users.
- **Estimated Savings:** 5-15%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Business validation that risk intervals are unused.

#### 10. FORECAST-010: Reduce Predictor Retraining Frequency
- **What:** Stop retraining the machine learning model daily; shift to weekly or monthly retraining.
- **Why It Saves Money:** Model training costs $0.24 per compute hour, and AutoPredictor tuning can take many hours. Retraining weekly instead of daily cuts compute costs by ~85%.
- **Implementation Steps:**
  1. Modify your scheduling pipeline (e.g., EventBridge or Step Functions) to trigger `CreatePredictor` less frequently.
  2. Continue triggering `CreateForecast` at your regular cadence using the existing predictor.
- **Estimated Savings:** 50-85% (on Training costs)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Verification that model accuracy doesn't significantly drift over a week.

#### 11. FORECAST-011: Downsample Historical Data for Training
- **What:** Limit the lookback window of historical data used to train the predictor (e.g., 2 years instead of 5 years).
- **Why It Saves Money:** Reduces the hours of training compute ($0.24/hour) required to optimize the model, and lowers ingestion storage fees ($0.088/GB).
- **Implementation Steps:**
  1. Truncate the historical CSV dataset in S3 to only include the last 18-24 months of data.
  2. Import the smaller dataset into Forecast.
- **Estimated Savings:** 20-50% (on Training costs)
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Backtesting to ensure accuracy remains acceptable with less data.

#### 12. FORECAST-012: Migrate to SageMaker Canvas (Primary Recommendation)
- **What:** Transition legacy Forecast workloads to Amazon SageMaker Canvas.
- **Why It Saves Money:** Forecast is a deprecated service. SageMaker Canvas offers modern, supported no-code ML forecasting, often with more transparent and optimized ML hosting fees.
- **Implementation Steps:**
  1. Export historical data to S3.
  2. Import data into SageMaker Canvas.
  3. Build and train time-series models within the Canvas UI.
  4. Decommission the Amazon Forecast pipeline.
- **Estimated Savings:** Variable (20-60%)
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** SageMaker Canvas onboarding.

#### 13. FORECAST-013: Migrate to SageMaker with Open Source Models
- **What:** Rebuild the forecasting pipeline using Amazon SageMaker Studio/Notebooks and open-source models like DeepAR or Prophet.
- **Why It Saves Money:** Allows the use of Spot Instances for model training, cutting compute costs by up to 90%. Avoids the premium managed-service markup of Amazon Forecast.
- **Implementation Steps:**
  1. Have a data science team build a custom SageMaker pipeline using DeepAR.
  2. Configure SageMaker Training Jobs to utilize Managed Spot Training.
- **Estimated Savings:** 50-90%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | Data Science
- **Prerequisites:** Data science expertise.

#### 14. FORECAST-014: Consolidate Forecast Pipelines
- **What:** Combine multiple disparate forecasting jobs (e.g., forecasting East Coast sales and West Coast sales separately) into a single unified dataset with a location dimension.
- **Why It Saves Money:** Reduces redundant training compute overhead. A single predictor trained on all data can often learn shared patterns faster and cheaper than training 5 separate predictors.
- **Implementation Steps:**
  1. Merge target time-series datasets, adding a dimension column (e.g., `Region`).
  2. Train a single AutoPredictor.
- **Estimated Savings:** 20-40%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compatible schemas across datasets.

#### 15. FORECAST-015: Use Built-in Weather Index (If Applicable)
- **What:** Utilize Amazon Forecast's built-in weather index instead of purchasing, processing, and ingesting massive 3rd-party weather datasets as related time-series data.
- **Why It Saves Money:** Reduces the GBs of data ingested ($0.088/GB) and the associated training compute required to process custom external datasets.
- **Implementation Steps:**
  1. Enable the Weather Index feature during Predictor creation.
  2. Remove custom weather data from your Related Time Series inputs.
- **Estimated Savings:** 10-20%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** US or European locations supported by the built-in index.

#### 16. FORECAST-016: Implement Event-Driven Retraining
- **What:** Instead of retraining models on a fixed cron schedule, trigger retraining only when incoming data volume exceeds a certain drift threshold.
- **Why It Saves Money:** Prevents paying for training compute hours when the underlying data hasn't changed enough to warrant a new model.
- **Implementation Steps:**
  1. Implement a Lambda function to check data drift or volume changes in S3.
  2. Only trigger `CreatePredictor` if the threshold is met.
- **Estimated Savings:** 30-50%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Data drift monitoring logic.

#### 17. FORECAST-017: Batch Inference Generation
- **What:** Generate forecasts in large, infrequent batches rather than small, frequent drips, if the business can consume it that way.
- **Why It Saves Money:** While points cost the same, reducing the frequency of the surrounding pipeline (Lambda triggers, Step Functions, S3 reads) reduces auxiliary infrastructure costs.
- **Implementation Steps:**
  1. Shift from daily forecast generation jobs to a single weekly batch job.
- **Estimated Savings:** 1-5% (Auxiliary costs)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Weekly consumption model.

#### 18. FORECAST-018: Monitor Free Tier Exhaustion for Legacy Accounts
- **What:** Set up AWS Budgets to alert when the 10,000 free forecast points/month limit is reached in the first 12 months.
- **Why It Saves Money:** Prevents unexpected pay-as-you-go charges for new/legacy POC accounts crossing the free tier threshold.
- **Implementation Steps:**
  1. Go to AWS Budgets.
  2. Create a usage budget specifically for `AmazonForecast-Points`.
- **Estimated Savings:** 100% of unintended overages.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Account is still within its first 12 months.

#### 19. FORECAST-019: Co-locate S3 Buckets for Import/Export
- **What:** Ensure that the S3 buckets used for importing datasets and exporting forecasts are in the exact same AWS Region as the Amazon Forecast service instance.
- **Why It Saves Money:** Eliminates cross-region data transfer fees ($0.01 - $0.02 per GB) when reading/writing large forecast datasets.
- **Implementation Steps:**
  1. Verify the region of your Amazon Forecast pipeline (e.g., `us-east-1`).
  2. Ensure the source and destination S3 buckets are also in `us-east-1`.
- **Estimated Savings:** <1%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 20. FORECAST-020: Enforce Resource Tagging for Forecast Assets
- **What:** Apply strict tagging policies to all Datasets, Predictors, and Forecasts to track costs by project or department.
- **Why It Saves Money:** Doesn't save money directly, but enables chargebacks and identifies exactly which department is generating excessive forecast points so FinOps can intervene.
- **Implementation Steps:**
  1. Implement Tag Policies in AWS Organizations.
  2. Require `CostCenter` or `Project` tags on all Forecast API creation calls.
- **Estimated Savings:** Variable
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** AWS Organizations.
