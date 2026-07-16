# Cost-Cutting Playbook: Amazon Bedrock

> **Companion File:** [bedrock.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/bedrock/bedrock.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon Bedrock is a fully managed serverless service providing unified API access to leading generative AI foundation models (Anthropic Claude, Meta Llama, Cohere, Stability AI, Amazon Titan). Billing is multi-dimensional based on **On-Demand Tokens** (Input vs Output), **Batch Inference** (50% discount), **Provisioned Throughput** (dedicated Model Units), and auxiliary services like **Bedrock Guardrails** ($1.00/1k requests) and **Knowledge Base Vector Storage** ($350.40/mo base for default OpenSearch Serverless).

Key billing traps include:
1. **Output Token Price Asymmetry:** Output tokens are **4x to 5x more expensive** than input tokens ($15.00/M vs $3.00/M for Claude 3.5 Sonnet).
2. **Model Over-Provisioning:** Routing simple classification/extraction tasks to Claude 3.5 Sonnet ($3.00/M input) instead of Claude 3.5 Haiku ($0.25/M input — a **91% price difference**).
3. **OpenSearch Vector Base Tax:** Deploying default Knowledge Base collections incurring a **$350.40/month minimum OCU fee** for tiny document repositories.

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **40–85% reduction in total Amazon Bedrock generative AI spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Implement Model Cascading (LLM Router):** Route initial prompts to **Claude 3.5 Haiku** or **Llama 3.1 8B** ($0.22–$0.25/M input); escalate ONLY complex reasoning to Claude 3.5 Sonnet ($3.00/M input) — cuts token spend by **60–80%**.
2. **Enable Batch Inference for Asynchronous Offline Tasks:** Secures an immediate **50% discount** on all input and output token rates for non-real-time jobs.
3. **Set Mandatory `max_tokens` Payload Limits:** Caps maximum generated output tokens (e.g. `max_tokens: 500`) to prevent runaway LLM text generation ($15.00/M output).

---

## Strategy Categories

### 1. Model Selection & Cascading Architecture

#### 1. Implement Model Cascading (LLM Router Architecture)
- **What:** Deploy a lightweight router function (in Lambda or application backend) that evaluates prompt complexity. Route routine summarization, extraction, and classification queries to **Claude 3.5 Haiku** ($0.25/M input) or **Llama 3.1 8B** ($0.22/M input). Route ONLY complex multi-step reasoning, coding, or math queries to **Claude 3.5 Sonnet** ($3.00/M input).
- **Why It Saves Money:** Claude 3.5 Haiku is **91.6% cheaper** on input prompts ($0.25/M vs $3.00/M) and **91.6% cheaper** on output tokens ($1.25/M vs $15.00/M) than Sonnet. If 80% of your application queries are standard tasks, model cascading slashes total token spend by **70%+**!
- **Detailed Implementation Steps:**
  1. Update Bedrock invocation client code in Python (Boto3):
     ```python
     def invoke_bedrock_model(prompt, is_complex=False):
         model_id = "anthropic.claude-3-5-sonnet-20241022-v2:0" if is_complex else "anthropic.claude-3-5-haiku-20241022-v1:0"
         response = bedrock_client.invoke_model(
             modelId=model_id,
             body=json.dumps({"anthropic_version": "bedrock-2023-05-31", "max_tokens": 500, "messages": [{"role": "user", "content": prompt}]})
         )
         return response
     ```
- **Estimated Savings:** **60–80% reduction** in total token billing.
- **Risk Level:** Low (router fallback escalates failed validations to Sonnet).
- **Implementation Scope:** Software Engineer / AI Developer
- **Prerequisites:** Prompt categorization logic.

#### 2. Utilize Open-Source Models (Llama 3.1 8B) for Standard NLP
- **What:** Replace proprietary commercial models with **Meta Llama 3.1 8B** ($0.22/M input, $0.22/M output) for sentiment analysis, entity extraction, and intent classification.
- **Why It Saves Money:** Llama 3.1 8B offers symmetrical pricing where output tokens cost the exact same as input tokens ($0.22/M), avoiding the 5x output token premium ($15.00/M) of Sonnet.
- **Detailed Implementation Steps:**
  1. Call `meta.llama3-1-8b-instruct-v1:0` via Bedrock API.
- **Estimated Savings:** 85–95% token cost reduction vs Claude 3.5 Sonnet.
- **Risk Level:** Low.
- **Implementation Scope:** AI Developer
- **Prerequisites:** Task evaluation accuracy benchmark.

---

### 2. Execution Mode & Batch Optimization

#### 3. Enable Bedrock Batch Inference for Asynchronous Jobs
- **What:** Submit offline asynchronous jobs (such as overnight document indexing, batch translation, or bulk dataset labeling) via **Bedrock Batch Inference**.
- **Why It Saves Money:** Batch Inference provides a **direct 50% flat discount** on all token pricing tiers! Processing 50M Claude 3.5 Sonnet tokens via Batch Inference drops cost from $900.00 down to **$450.00**.
- **Detailed Implementation Steps:**
  1. Upload input prompts JSONL file to S3 (`s3://bucket/batch-input.jsonl`).
  2. Create Model Invocation Job via AWS CLI:
     ```bash
     aws bedrock create-model-invocation-job \
       --job-name "overnight-doc-summarization" \
       --model-id "anthropic.claude-3-5-sonnet-20241022-v2:0" \
       --role-arn "arn:aws:iam::123456789012:role/BedrockBatchRole" \
       --input-data-config '{"s3InputDataConfig": {"s3Uri": "s3://bucket/batch-input.jsonl"}}' \
       --output-data-config '{"s3OutputDataConfig": {"s3Uri": "s3://bucket/batch-output/"}}'
     ```
- **Estimated Savings:** **50% flat discount** on token pricing.
- **Risk Level:** Zero risk (guaranteed 24-hour job execution completion window).
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** Asynchronous processing SLA tolerance.

---

### 3. Prompt & Payload Optimization

#### 4. Enforce Enforceable `max_tokens` Payload Caps
- **What:** Always pass explicit, strict `max_tokens` parameters (e.g. `max_tokens: 300` or `500`) in model invocation payloads.
- **Why It Saves Money:** Output tokens cost **4x to 5x more** than input tokens ($15.00/M vs $3.00/M). An uncapped prompt allow an LLM to generate 4,000 tokens of redundant filler text, billing **$0.06 per API call**. Restricting output to 300 tokens caps cost at **$0.0045 per call** (saving 92%).
- **Detailed Implementation Steps:**
  1. Add strict payload parameter in API client request JSON: `"max_tokens": 500`.
- **Estimated Savings:** 50–90% output token savings per invocation.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** None.

#### 5. Implement Context Pruning & Prompt Truncation (Chat History)
- **What:** Limit multi-turn conversational chat histories passed in API payloads to the **last 3 to 5 message turns** rather than appending the full historical chat transcript indefinitely.
- **Why It Saves Money:** Forwarding a 50-turn chat history on every user turn re-sends 20,000 input prompt tokens on every request ($0.06 per request). Pruning history to 3 turns keeps input tokens under 1,000 ($0.003 per request) — a **95% input token reduction**.
- **Detailed Implementation Steps:**
  1. Slice message array in backend before API call: `messages = full_history[-6:]`.
- **Estimated Savings:** 70–95% input token savings on chat applications.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Context window management testing.

#### 6. Optimize System Prompts & Remove Redundant Context
- **What:** Refactor System Prompts to be concise, removing repetitive instructions and unnecessary example boilerplate.
- **Why It Saves Money:** System prompt tokens are processed on *every single request*. Shortening a system prompt from 2,000 tokens down to 400 tokens saves 1,600 input tokens per invocation across millions of calls.
- **Detailed Implementation Steps:**
  1. Prune verbose system prompts and use XML tags (`<instructions>`, `<example>`) for token efficiency.
- **Estimated Savings:** 20–40% input token savings.
- **Risk Level:** Low.
- **Implementation Scope:** Prompt Engineer / AI Developer
- **Prerequisites:** Prompt evaluation benchmarks.

---

### 4. Vector Storage & RAG Architecture Optimization

#### 7. Replace OpenSearch Serverless with Aurora pgvector / RDS for Knowledge Bases
- **What:** Configure Bedrock Knowledge Bases to use **Amazon Aurora PostgreSQL (pgvector)** or existing RDS PostgreSQL instances as the vector database index instead of default **Amazon OpenSearch Serverless**.
- **Why It Saves Money:** Default OpenSearch Serverless vector collections enforce a minimum provisioned baseline of 2 OpenSearch Compute Units (OCUs), billing **$350.40 per month flat**, even if your Knowledge Base contains only 1 MB of vector embeddings! Using pgvector on an existing RDS instance costs **$0.00 extra**.
- **Detailed Implementation Steps:**
  1. Create pgvector extension on Aurora PostgreSQL database: `CREATE EXTENSION vector;`.
  2. Create Bedrock Knowledge Base pointing to Aurora pgvector vector store via AWS Console or Terraform.
- **Estimated Savings:** **$350.40 per month saved** (100% of OpenSearch Serverless baseline fees).
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer / DevOps
- **Prerequisites:** Aurora/RDS PostgreSQL instance deployed.

#### 8. Optimize Bedrock Embedding Model Selection
- **What:** Use **Cohere Embed English** ($0.10/M tokens) or **Titan Embeddings** ($0.02/M tokens) for generating vector embeddings during RAG document ingestion.
- **Why It Saves Money:** Cohere Embed is 30x cheaper than using general LLMs for vector generation.
- **Detailed Implementation Steps:**
  1. Set `cohere.embed-english-v3` as vector embedding model in Knowledge Base configuration.
- **Estimated Savings:** 80–90% embedding ingestion savings.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** None.

---

### 5. Guardrails & Provisioned Throughput Tuning

#### 9. Optimize Bedrock Guardrails Application (Avoid Request Surcharges)
- **What:** Apply **Bedrock Guardrails** ($1.00 per 1,000 requests evaluated) selectively on public user-facing input prompts rather than executing guardrails on internal trusted system calls.
- **Why It Saves Money:** Running Guardrails on 100,000 internal batch requests adds a **$100.00 surcharge** that can easily exceed the cost of the LLM invocation itself.
- **Detailed Implementation Steps:**
  1. Pass `guardrailIdentifier` parameter ONLY on public API routes.
- **Estimated Savings:** 100% of internal Guardrail surcharges.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / Security
- **Prerequisites:** Internal vs external route segregation.

#### 10. Audit & Terminate Idle Provisioned Throughput (Model Units)
- **What:** Monitor and delete idle Provisioned Throughput Model Units (MUs).
- **Why It Saves Money:** Provisioned Throughput runs 24/7. An idle 1 MU Claude 3.5 Sonnet allocation bills continuous hourly charges ($100s to $1,000s/month).
- **Detailed Implementation Steps:**
  1. List provisioned model throughput: `aws bedrock list-provisioned-model-throughputs`.
  2. Delete idle MUs: `aws bedrock delete-provisioned-model-throughput --provisioned-model-id xxx`.
- **Estimated Savings:** 100% of idle provisioned MU fees.
- **Risk Level:** Medium (affects guaranteed latency SLAs).
- **Implementation Scope:** DevOps / FinOps Team
- **Prerequisites:** Latency SLA audit.

---

### 6. Observability, Caching & Cost Governance

#### 11. Implement Semantic Caching (Redis / DynamoDB) for LLM Responses
- **What:** Deploy a **Semantic Cache** (using Amazon ElastiCache Redis with vector search or DynamoDB) in front of Bedrock API calls.
- **Why It Saves Money:** If an incoming user query is semantically similar (> 95% vector similarity) to a recently answered query, return the cached LLM response instantly for **$0.00 token cost**. Cuts LLM API calls by 20–40% for FAQs.
- **Detailed Implementation Steps:**
  1. Query Redis vector index with prompt embedding before calling Bedrock API.
  2. Return cached response if similarity score > 0.95.
- **Estimated Savings:** 20–40% reduction in overall Bedrock API invocation costs.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Redis vector cache deployed.

#### 12. Set Enforced CloudWatch Alarms on Daily Bedrock Token Spend
- **What:** Configure CloudWatch metric alarms on `InputTokenCount` and `OutputTokenCount` aggregated per model.
- **Why It Saves Money:** Provides immediate alerts if an infinite application loop or bot attack generates millions of tokens.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm targeting `AWS/Bedrock` namespace.
- **Estimated Savings:** Proactive billing risk mitigation.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 13. Restrict Model Access Permissions via IAM Policies
- **What:** Restrict developer and application IAM roles from calling expensive models directly (e.g. deny `bedrock:InvokeModel` on Sonnet for non-prod roles).
- **Why It Saves Money:** Enforces architectural compliance requiring non-prod testing on Claude 3.5 Haiku.
- **Detailed Implementation Steps:**
  1. Attach IAM SCP/Policy restricting model ARNs in dev accounts.
- **Estimated Savings:** Prevents unapproved high-tier model usage.
- **Risk Level:** Low.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** IAM policy setup.

#### 14. Use Local Regex for PII Masking Prior to Bedrock Ingestion
- **What:** Run local open-source regex/presidio PII masking in your application backend before calling Bedrock Guardrails.
- **Why It Saves Money:** Avoids paying $1.00/1k requests for Guardrail PII checks.
- **Detailed Implementation Steps:**
  1. Mask PII locally using Python `presidio-analyzer`.
- **Estimated Savings:** 50–90% Guardrail fee reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Presidio library installation.

#### 15. Consolidate RAG Chunk Sizes to Reduce Vector Search Ingestion
- **What:** Increase Knowledge Base chunking size from 200 tokens to 800 tokens.
- **Why It Saves Money:** Reduces the total count of vector embedding API calls and index storage size by 4x.
- **Detailed Implementation Steps:**
  1. Set max tokens per chunk = 800 in Knowledge Base parsing settings.
- **Estimated Savings:** 75% reduction in vector embedding API calls.
- **Risk Level:** Low.
- **Implementation Scope:** Data Engineer
- **Prerequisites:** RAG retrieval quality evaluation.

#### 16. Monitor Free Model Evaluation Workspaces
- **What:** Clean up temporary Bedrock Model Evaluation jobs in AWS Console.
- **Why It Saves Money:** Operational hygiene.
- **Detailed Implementation Steps:**
  1. Delete completed evaluation jobs.
- **Estimated Savings:** Administrative hygiene.
- **Risk Level:** Zero.
- **Implementation Scope:** AI Developer
- **Prerequisites:** None.

---

## Cross-Service Synergies

```
[ Generative AI Application ] 
        │
        ├──(LLM Router)────────> [ Claude 3.5 Haiku ] ($0.25/M input vs Sonnet $3.00/M - 91% cheaper)
        │
        ├──(Vector Storage)────> [ Aurora pgvector ] (Bypasses OpenSearch $350.40/mo base tax)
        │
        └──(Offline Ingestion)─> [ Batch Inference (50% Off) ] (50% flat discount on all tokens)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Bedrock-Claude3.5-Sonnet-Input-Token`, `Bedrock-Claude3.5-Sonnet-Output-Token`, `Bedrock-Claude3.5-Haiku-Input-Token`, `OpenSearchServerless-OCU`.
- `line_item_resource_id`: Bedrock Model ID / Knowledge Base ID.

### B. CloudWatch & Bedrock Metrics
- `AWS/Bedrock` Namespace: `InputTokenCount`, `OutputTokenCount`, `InvocationLatency`, `Invocations`, `InvocationErrors`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "BED-RTR-001",
  "service": "Bedrock",
  "category": "Model Selection & Cascading Architecture",
  "resource_id": "arn:aws:bedrock:us-east-1:123456789012:foundation-model/anthropic.claude-3-5-sonnet-20241022-v2:0",
  "resource_name": "customer-support-bot-api",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "model_id": "Claude 3.5 Sonnet (Direct)",
    "monthly_input_tokens_millions": 150.0,
    "monthly_output_tokens_millions": 50.0,
    "monthly_cost_usd": 1200.00
  },
  "recommended_config": {
    "model_id": "Model Cascading Router (80% Haiku / 20% Sonnet)",
    "projected_monthly_cost_usd": 315.00
  },
  "financial_impact": {
    "monthly_savings_usd": 885.00,
    "annual_savings_usd": 10620.00,
    "savings_percentage": 73.8
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "80% of customer support queries are simple status lookups handled perfectly by Claude 3.5 Haiku."
  },
  "implementation": {
    "scope": "Software Engineer / AI Developer",
    "effort_estimate": "2-4 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Model Cascading Router (Haiku)** | 8 | $14,500.00 | $10,701.00 | 73.8% | Low |
| **Aurora pgvector (OpenSearch Tax)** | 4 | $1,401.60 | $1,401.60 | 100.0% | Zero |
| **Batch Inference (50% Discount)** | 6 | $6,200.00 | $3,100.00 | 50.0% | Zero |
| **Payload Output Caps (max_tokens)**| 12 | $8,800.00 | $6,160.00 | 70.0% | Low |
| **Total** | **30** | **$30,901.60** | **$21,362.60** | **69.1%** | -- |
