# AutoML Cost Optimization & Research

Vertex AI AutoML allows developers to train high-quality custom machine learning models (for tabular, image, text, and video datasets) without writing training code. While it democratizes ML, AutoML training carries high hourly node rates (up to $20.00/hour for tabular models), and running persistent online prediction endpoints can generate major idle charges.

---

## 1. AutoML Billing Components

AutoML costs are divided into training and serving phases:
1. **Training:** Billed per node-hour consumed during model training:
   * **Image / Video Models:** Approx. $3.00 per node-hour.
   * **Text / NLP Models:** Approx. $3.00 per node-hour.
   * **Tabular Models:** Approx. $20.00 per node-hour (highly compute-intensive).
2. **Online Prediction (Serving):** Billed hourly per node for the duration the model is deployed to an endpoint (e.g. $0.20 - $1.00+/hr depending on machine size and replicas). Billed 24/7.
3. **Batch Prediction:** Billed per node-hour of active processing time. No persistent endpoint required.

---

## 2. Core Cost-Optimization Levers

### A. Strict Endpoint Undeployment (Scale to Zero Alternative)
* **The Cost Trap:** Deploying an AutoML model to a Vertex AI endpoint to test a feature, and leaving the endpoint active for months. Since online endpoints do not support automatic scaling to 0 nodes, you are billed 24/7.
* **Action:**
  1. For testing and development, always **undeploy** the model from the endpoint when you are finished testing.
  2. For low-volume or async inference, use **Batch Prediction** instead of Online Prediction. Batch prediction spins up resources, processes the inputs, and shuts down, costing $0 during idle hours.

### B. Enforce Training Budgets
* When launching an AutoML training job, Google allows you to specify a maximum training budget (node-hours limit).
* **Action:** Set a conservative budget cap on training runs (e.g., 2–8 hours for tabular or text models). Google Cloud automatically stops training when the budget is reached or when the model stops improving, preventing runaway training costs.

### C. Downscale Web Console Experimentation
* **Action:** Do not allow junior engineers to run multi-node AutoML tabular training runs on massive datasets for exploratory purposes. Run initial experiments on smaller, downsampled datasets to validate model viability before committing to a full-scale AutoML training budget.

---

## 3. AutoML Audit Checklist

1. [ ] **Active Endpoint Review:** Run a CLI sweep to identify all active Vertex AI endpoints. Undeploy models from idle or unused test endpoints.
  ```bash
  # List all endpoints in a region
  gcloud ai endpoints list --region=us-central1
  ```
2. [ ] **Batch Prediction Migration:** Migrate non-real-time prediction flows from persistent online endpoints to batch prediction jobs.
3. [ ] **Training Budget Policies:** Enforce an organizational guideline requiring a maximum node-hours cap on all AutoML training tasks.
4. [ ] **Dataset Downsampling:** Require dataset filtering/downsampling during the model design and experimentation phase to reduce training time.
