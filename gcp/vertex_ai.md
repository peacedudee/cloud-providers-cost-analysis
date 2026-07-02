# Vertex AI Cost Optimization & Research

Google Cloud Vertex AI is an unified machine learning platform that orchestrates custom model training, model deployment (online and batch prediction), vector search, and GenAI (Gemini) APIs. Because model serving and vector databases require continuous compute and memory (often backed by expensive GPUs), Vertex AI spend can rapidly balloon. This file covers optimization practices for Vertex AI.

---

## 1. Vertex AI Billing Components

Vertex AI billing is separated across four primary areas:
1. **Custom Model Training:** Billed per hour for compute instances, GPUs, and storage utilized during training jobs.
2. **Online Prediction (Endpoints):** Billed per hour for instances deployed to host models for live API calls. **Endpoints do not scale to zero by default.**
3. **Vector Search (Matching Engine):** Billed per node-hour for hosting indexes to run high-speed vector similarity searches.
4. **Generative AI APIs (Gemini & Model Garden):** Billed per token (input and output) or per execution (image/video generation).

---

## 2. Core Cost-Reduction Tactics

### A. Undeploy Idle Online Prediction Endpoints
* **The Cost Trap:** Deployed models run on active VMs (often G2 or N1 shapes with attached GPUs) to guarantee immediate response times. If an endpoint receives 0 requests for days, you continue to pay the full node hourly rate.
* **Action:**
  1. Audit active endpoints in the Vertex AI console.
  2. Implement an automated script (e.g., using Cloud Functions and Cloud Asset Inventory) to scan for deployed models with no request traffic over the last 24-48 hours and automatically **undeploy** them.
  3. **Alternative for Light Models:** For smaller scikit-learn, XGBoost, or TensorFlow models that receive low or spiky traffic, wrap them in a container and deploy to **Cloud Run (with scale-to-zero)** rather than a continuous Vertex AI endpoint.

### B. Implement LLM Model Routing (The Waterfall Strategy)
Using flagship models (like Gemini 1.5 Pro) for basic tasks is highly inefficient.
* **Action:** Implement a routing layer in your application:
  * **Level 1 (Gemini Flash-Lite / Flash):** Route simple tasks like classification, JSON formatting, sentiment detection, and basic summarization here. This handles 80% of traffic at a **70-90% cost reduction**.
  * **Level 2 (Gemini Pro):** Route complex reasoning, multi-turn coding assistance, and deep analysis here.
  * **Level 3 (Gemini Ultra):** Reserve exclusively for high-end reasoning tasks that fail on Pro.

### C. Leverage Context/Prompt Caching
For GenAI applications that append large system prompts, user context databases, or source documents (like PDFs, textbooks, code repos) to every API request:
* **The Solution:** Enable **Vertex AI Context Caching**.
* **How it works:** Google caches the parsed tokens of your initial long context on its servers. Subsequent requests referencing that cache ID only pay for the dynamic new query tokens and a small cache read fee, bypassing the full input token charge.
* **The Benefit:** Saves up to **90%** on input token costs for conversational or document-heavy workflows (or 75% for legacy Gemini 1.0 models).

### D. Use Batch Predictions for Offline Workloads
* **The Waste:** Keeping an online endpoint deployed to run predictions on databases in bulk once a day or once a week.
* **Action:** Submit a **Vertex Batch Prediction Job**.
* **The Benefit:** Dataproc/Compute Engine workers are provisioned, run the inference in parallel, write the outputs to BigQuery/GCS, and are immediately deleted. You pay only for the processing duration.

### E. Spot VMs for Custom Training
* **Action:** When configuring custom training pipelines (e.g. PyTorch training jobs), configure the job to run on **Spot VMs** (e.g., by setting the `preemptible` or `enableSpotVm` flag in your worker pool configuration, depending on the SDK/API version).
* **Mitigation:** Ensure your training code implements checkpointing (e.g. writing weights to a GCS bucket every epoch) so that if a Spot VM is pre-empted, the training job can resume from the last saved state without loss of progress.

---

## 3. Vertex AI Audit Checklist

1. [ ] **Idle Endpoint Sweep:** Sweep all projects and undeploy Vertex AI endpoints showing 0 traffic over the last 7 days.
2. [ ] **Model Routing Audit:** Review application LLM integrations. Ensure Gemini Flash is used for basic classifications and formatting.
3. [ ] **Context Caching Integration:** Implement Context Caching for all workflows with system prompts or attachments exceeding 32k tokens.
4. [ ] **Batch Inference Migration:** Convert daily analytical inference schedules from online endpoints to Batch Prediction jobs.
5. [ ] **Training Spot VMs:** Check custom training configurations to ensure Spot VMs are enabled where possible.
6. [ ] **Vector Search Index Sizing:** Audit deployed Vector Search indexes. Downsize node configurations (e.g. from `e2-standard-16` to `e2-standard-2`) if index size is small.
