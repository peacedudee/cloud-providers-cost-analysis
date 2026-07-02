# Recommendations AI Cost Optimization & Research

Google Cloud Recommendations AI is a managed service that delivers personalized product and content recommendations at scale, powered by Google's recommendation algorithms. Because Recommendations AI charges for data ingestion, prediction requests, and active model training/tuning, keeping training loops structured and event data filtered is essential.

---

## 1. Recommendations AI Billing Components

Billing is calculated across three primary dimensions:
1. **Data Ingestion:** Billed per GB of catalog items and user event data uploaded (e.g. clicks, add-to-carts, views).
2. **Recommendation Requests:** Billed per 1,000 predictions served (tiered pricing: costs decrease as volume scales, but starts around $0.27 per 1,000 queries).
3. **Model Training & Tuning:** Billed per node-hour consumed to train and continuously tune recommendation models (e.g., $1.00 - $2.50 per node-hour depending on model type).

---

## 2. Core Cost-Optimization Levers

### A. Control Model Tuning Frequency
To keep recommendations fresh, the service continuously tunes the models.
* **The Cost Trap:** Leaving high-frequency automated tuning active on small catalogs. Tuning runs consume substantial node-hour compute fees.
* **Action:** Adjust the tuning interval in the serving configurations. Limit deep tuning runs to **weekly** or **bi-weekly** cycles rather than daily, especially if your product catalog is relatively static.

### B. Filter and Batch Ingested User Events
* **The Waste:** Ingesting highly verbose, low-value telemetry events (e.g., mouse hovering, scroll percentages, asset loading events) which balloons ingestion storage and processing fees.
* **Action:** Limit event tracking strictly to high-signal conversion steps (views, search queries, detail-page clicks, add-to-carts, purchase completions). Batch user events and ingest them daily using JSON files rather than streaming every minor interaction in real-time.

### C. Undeploy Inactive Serving Configs
* **Action:** If you are running A/B tests or have set up custom recommendation models (e.g., "Others You May Like", "Frequently Bought Together") for temporary seasonal campaigns, explicitly **deactivate and delete the serving configurations** when the campaigns end. Active serving configs can continue triggering background polling and training cycles.

---

## 3. Recommendations AI Audit Checklist

1. [ ] **Tuning Schedule Optimization:** Review model configuration settings and set tuning runs to a weekly or bi-weekly cadence.
2. [ ] **Event Filtering Audit:** Inspect client-side tags to ensure low-signal interactions (hovers, scrolls) are excluded from Recommendations AI ingestion.
3. [ ] **Stale Serving Config Decommission:** Audit the console and delete retired recommendation placement IDs and serving configs.
