# Cost-Cutting Playbook: AWS Lambda

> **Companion File:** [lambda.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/lambda/lambda.md)  
> **Last Updated:** July 2026

---

## Executive Summary

AWS Lambda provides serverless event-driven execution billed strictly on invocation count, memory allocation, and runtime duration (rounded up to 1 ms). While highly cost-effective for variable or low-traffic workloads, high-volume serverless applications can experience massive billing leaks from over-allocated memory, idle wait times, unoptimized cold starts, and runaway recursive triggers.

This playbook outlines **20 actionable strategies** across seven operational categories, delivering an estimated **25–60% reduction in total AWS Lambda spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Migrate Functions to ARM64 (AWS Graviton):** Instant 20% price-per-GB-second discount with zero code changes for most runtimes.
2. **Optimize Memory Sizing with AWS Lambda Power Tuning:** Align memory to CPU scaling, often reducing runtime duration so significantly that total cost drops.
3. **Set Aggressive Function Timeouts (5–10s):** Prevents runaway external API locks from billing up to the 15-minute execution maximum.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Eliminate Unused or Orphaned Lambda Functions
- **What:** Identify and delete Lambda functions that have recorded zero invocations over the last 90 days.
- **Why It Saves Money:** Prevents accidental triggers, reduces security surface area, and cleans up associated Provisioned Concurrency, IAM roles, and CloudWatch Log Groups.
- **Implementation Steps:**
  1. Query CloudWatch metric `Invocations` with `Sum = 0` over 90 days across all functions.
  2. Export function list and verify with development leads.
  3. Delete functions: `aws lambda delete-function --function-name fn-name`.
- **Estimated Savings:** 100% of idle function maintenance overhead and associated log storage fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch invocation metrics active over 90 days.

#### 2. Guard Against Recursive Invocation Loops
- **What:** Implement safety switches and architectural guardrails to prevent recursive loops (e.g. Lambda writing an object to S3, which triggers the same Lambda function infinitely).
- **Why It Saves Money:** Recursive loops can generate millions of requests in hours, exhausting account concurrency limits and billing thousands of dollars in minutes.
- **Implementation Steps:**
  1. Enforce strict input/output path segregation (e.g. S3 trigger on `/incoming/`, output written to `/processed/`).
  2. Set `ReservedConcurrentExecutions` low (e.g. max 10) on new event-driven functions until verified.
  3. Enable `AWS_LAMBDA_RECURSIVE_LOOP_DETECTION` (enabled by default for SQS and S3).
- **Estimated Savings:** Avoids $1000s in catastrophic loop billing spikes.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Event source mapping review.

#### 3. Audit & Right-Size Over-Provisioned Concurrency
- **What:** Review functions using Provisioned Concurrency (PC) to ensure allocated warm instances match actual concurrent traffic spikes.
- **Why It Saves Money:** Provisioned Concurrency costs **$0.015 per GB-hour** continuously whether invoked or not. Allocating 100 warm GB for a month costs **$1,095/month** in flat capacity fees!
- **Implementation Steps:**
  1. Compare `ProvisionedConcurrencyUtilization` vs `ProvisionedConcurrencySpilloverInvocations` in CloudWatch.
  2. Reduce PC allocation to match true baseline concurrency needs during off-peak hours.
  3. Configure Application Auto Scaling for Provisioned Concurrency.
- **Estimated Savings:** 40–80% on Provisioned Concurrency fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application Auto Scaling setup.

---

### 2. Rightsizing

#### 4. Migrate Functions to ARM64 (AWS Graviton) Architecture
- **What:** Change function execution architecture setting from `x86_64` to `arm64` across Python, Node.js, Java, Go, and Ruby runtimes.
- **Why It Saves Money:** ARM duration pricing is **20% cheaper** ($0.0000133334/GB-s vs $0.0000166667/GB-s) and often runs faster on Graviton processors.
- **Implementation Steps:**
  1. Update function architecture: `aws lambda update-function-configuration --function-name fn-name --architectures arm64`.
  2. Recompile native C/C++ dependencies (if any) for Linux `arm64`.
  3. Deploy to staging, run unit tests, and promote to production.
- **Estimated Savings:** 20% direct price reduction + performance gains.
- **Risk Level:** Low (for interpreted runtimes like Node/Python/Java).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-arch container image or interpreted code dependencies.

#### 5. Tune Memory Allocation using AWS Lambda Power Tuning
- **What:** Run the open-source AWS Lambda Power Tuning state machine to benchmark function performance and cost across various memory allocations (from 128 MB to 10,240 MB).
- **Why It Saves Money:** CPU scales proportionally with memory (1,769 MB = 1 full vCPU). Increasing memory from 512 MB to 1024 MB often speeds up execution by 3x, lowering total GB-second duration cost!
- **Implementation Steps:**
  1. Deploy Lambda Power Tuning SAM template.
  2. Execute tuning state machine against target functions with sample payloads.
  3. Set memory configuration to the optimal "cost-sweet-spot" identified by the tool.
- **Estimated Savings:** 15–40% reduction in total execution cost.
- **Risk Level:** Zero risk (data-backed optimization).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Representative test payload.

#### 6. Tune Function Execution Timeouts Aggressively
- **What:** Reduce default function timeouts from 15 minutes (or 30s) down to 2–3 seconds above average peak execution time.
- **Why It Saves Money:** If an external HTTP call or database lock hangs, a function with a 15-minute timeout will run for 900 seconds before failing, billing 900x its normal execution cost.
- **Implementation Steps:**
  1. Analyze 99th percentile execution duration (`p99`) in CloudWatch.
  2. Set function timeout to `p99 + 2 seconds`.
  3. Ensure client-side retries handle occasional timeout events.
- **Estimated Savings:** Prevents 900x cost spikes on runaway executions.
- **Risk Level:** Low to Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch duration metrics analysis.

#### 7. Right-Size Ephemeral Storage (`/tmp` Space)
- **What:** Keep ephemeral storage at the default **512 MB free allocation** unless larger disk space is strictly required.
- **Why It Saves Money:** Allocating `/tmp` storage above 512 MB costs **$0.0000000309 per GB-second**. Allocating 10 GB when only 500 MB is used adds unnecessary storage fees.
- **Implementation Steps:**
  1. Review function configuration for Ephemeral Storage setting.
  2. Reset `/tmp` storage allocation to 512 MB where possible.
- **Estimated Savings:** 100% of additional ephemeral storage charges.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Disk usage audit.

---

### 3. Commitment Discounts

#### 8. Cover Steady-State Lambda Spend with Compute Savings Plans
- **What:** Include Lambda duration spend in organization-wide 1-year or 3-year Compute Savings Plans.
- **Why It Saves Money:** Compute Savings Plans apply automatically to Lambda duration usage, yielding discounts of **up to 17% (1-year) or 19% (3-year)** off standard GB-second rates.
- **Implementation Steps:**
  1. Combine EC2, Fargate, and Lambda baseline spend into Cost Explorer Savings Plans evaluation.
  2. Purchase Compute Savings Plan commitment matching 70–80% of minimum hourly compute.
- **Estimated Savings:** 12–19% discount on Lambda duration spend.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Multi-month baseline spend stability.

---

### 4. Architecture Changes

#### 9. Replace Multi-Lambda Orchestration with AWS Step Functions
- **What:** Refactor functions that call other Lambda functions synchronously (chained Lambda calls) to use AWS Step Functions Express/Standard workflows.
- **Why It Saves Money:** When Lambda A calls Lambda B synchronously and waits for a response, you pay double: Lambda A bills for idle wait time while Lambda B executes.
- **Implementation Steps:**
  1. Map multi-step execution flows into a Step Functions state machine.
  2. Replace synchronous `lambda:Invoke` API calls with task states.
- **Estimated Savings:** 30–50% elimination of double-billed idle waiting duration.
- **Risk Level:** Medium (architectural refactoring required).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workflow mapping.

#### 10. Implement SQS Batching to Minimize Function Invocations
- **What:** Configure SQS event source mappings to process messages in batches (`BatchSize = 10` to `1000`) with a `MaximumBatchingWindowInSeconds` (e.g. 10–30s).
- **Why It Saves Money:** Lambda charges $0.20 per 1 million requests. Processing 100 messages individually costs 100 requests. Batching 100 messages into 1 invocation reduces request charges by **99%**.
- **Implementation Steps:**
  1. Update SQS Event Source Mapping in Lambda console or IaC.
  2. Set `BatchSize = 100` and `MaximumBatchingWindowInSeconds = 10`.
  3. Ensure code uses `ReportBatchItemFailures` for partial batch error handling.
- **Estimated Savings:** 90–99% reduction in request charges for high-velocity queues.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Code support for iterating payload arrays.

#### 11. Replace Polling Lambdas with Event-Driven EventBridge / S3 Triggers
- **What:** Replace cron-triggered Lambdas that poll databases, S3, or external APIs every minute checking for new data with event-driven push triggers.
- **Why It Saves Money:** A Lambda running every 1 minute executes 43,200 times/month. If 95% of executions find no new data, 41,040 executions are completely wasted.
- **Implementation Steps:**
  1. Configure S3 Event Notifications or EventBridge Rules to trigger Lambda ONLY when new data arrives.
  2. Remove 1-minute Scheduled Expressions.
- **Estimated Savings:** 80–95% reduction in unnecessary polling invocations.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Event source availability.

---

### 5. Scheduling & Auto-Scaling

#### 12. Autoscale Provisioned Concurrency with Application Auto Scaling
- **What:** Attach Application Auto Scaling target tracking policies to Lambda Provisioned Concurrency, dynamically scaling warm environments down during nights/weekends.
- **Why It Saves Money:** Keeps warm instances active during business hours (e.g. 8 AM – 6 PM) while dropping PC to 0 off-hours, cutting PC flat fees by **65%**.
- **Implementation Steps:**
  1. Register Lambda Provisioned Concurrency alias as scalable target.
  2. Apply `ScheduledAction` policies for business hour scaling.
- **Estimated Savings:** 60–70% PC cost savings.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Function alias and published version.

---

### 6. Pricing Model Optimization

#### 13. Optimize Function Initialization (Cold Start) Code
- **What:** Move database connection pooling, SDK client initialization, and heavy imports outside the Lambda handler function body (into global scope).
- **Why It Saves Money:** Code outside the handler executes once during container cold start and stays warm across invocations, reducing every subsequent invocation duration.
- **Implementation Steps:**
  1. Move `new AWS.DynamoDB.DocumentClient()` outside `exports.handler`.
  2. Implement database connection reuse patterns (e.g. RDS Proxy or global pool).
- **Estimated Savings:** 10–30% reduction in average invocation duration.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Code refactoring.

#### 14. Minimize Lambda Deployment Package Size
- **What:** Strip unused dependencies, documentation, and devDependencies from Lambda zip deployment packages, or use lightweight container layers.
- **Why It Saves Money:** Smaller deployment packages load significantly faster during cold starts, reducing unbilled cold start lag and overall memory footprint.
- **Implementation Steps:**
  1. Use Webpack/Esbuild to bundle and tree-shake Node.js/Python code.
  2. Remove heavy AWS SDK v2 libraries (use built-in SDK or v3 modular packages).
- **Estimated Savings:** 5–15% duration savings on cold-start-heavy functions.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Build pipeline integration.

---

### 7. Network & Data Transfer Optimization

#### 15. Avoid NAT Gateway Data Processing Fees with Interface VPC Endpoints
- **What:** Configure VPC Interface Endpoints for AWS services (DynamoDB, S3, Secrets Manager, SQS) accessed by Lambda functions attached to a VPC.
- **Why It Saves Money:** Lambda functions running inside a private VPC route external traffic through a NAT Gateway ($0.045/GB processing tax). VPC endpoints drop data transfer costs to $0.01/GB (PrivateLink) or $0.00/GB (S3/DynamoDB Gateway).
- **Implementation Steps:**
  1. Create VPC Endpoints for required services in Lambda VPC.
  2. Ensure Security Groups allow inbound traffic from Lambda security group.
- **Estimated Savings:** $45.00 saved per 1 TB of data accessed.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC-attached Lambda functions.

#### 16. Remove Unnecessary VPC Configurations from Lambda Functions
- **What:** Detach Lambda functions from VPCs if they only access public internet APIs, DynamoDB, or S3, and do not access internal RDS/ElastiCache resources.
- **Why It Saves Money:** Eliminates cold start network interface attachment overhead and avoids NAT Gateway processing charges entirely.
- **Implementation Steps:**
  1. Audit VPC-attached functions: `aws lambda list-functions`.
  2. Remove VPC configuration (`VpcConfig = {}`) for non-VPC dependent functions.
- **Estimated Savings:** Eliminates NAT Gateway fees ($0.045/GB) and reduces execution time.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Verification that function does not access private VPC resources.

#### 17. Configure CloudWatch Logs Retention on Lambda Log Groups
- **What:** Enforce explicit log retention limits (e.g. 7 days or 14 days) on all `/aws/lambda/*` CloudWatch Log Groups.
- **Why It Saves Money:** Lambda automatically creates a CloudWatch Log Group for every function set to "Never Expire". Over time, log storage ($0.03/GB-mo) builds up continuously.
- **Implementation Steps:**
  1. Run AWS CLI script to update all log groups: `aws logs put-retention-policy --log-group-name /aws/lambda/fn-name --retention-in-days 14`.
  2. Deploy an AWS Systems Manager Quick Setup to enforce default log retention.
- **Estimated Savings:** 70–90% reduction in long-term log storage fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Log archiving policy review.

#### 18. Use Infrequent Access Log Class for High-Volume Lambda Logs
- **What:** Switch high-volume Lambda log groups from `Standard` log class ($0.50/GB) to `Infrequent Access` log class ($0.25/GB).
- **Why It Saves Money:** Saves **50% directly** on log ingestion charges for verbose debug or transaction logs.
- **Implementation Steps:**
  1. Modify log group class via AWS CLI or Terraform: `aws logs create-log-group --log-group-class INFREQUENT_ACCESS`.
- **Estimated Savings:** 50% on CloudWatch log ingestion costs.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Confirm advanced features like metric filters are not required for these log groups.

#### 19. Implement Response Compression on API Gateway / Lambda Integration
- **What:** Enable Gzip/Brotli compression on API Gateway endpoints backed by Lambda.
- **Why It Saves Money:** Compresses JSON payloads sent over the network, reducing network egress bandwidth and Lambda runtime execution duration.
- **Implementation Steps:**
  1. Enable `MinimumCompressionSize = 1024` on API Gateway REST/HTTP APIs.
- **Estimated Savings:** 50–80% reduction in network payload egress fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** API Gateway configuration permissions.

#### 20. Filter Event Source Triggers at the Source (EventBridge / Kinesis Filtering)
- **What:** Configure Event Source Mapping filter criteria so Lambda is ONLY invoked when specific payload fields match (e.g. `status == "FAILED"`).
- **Why It Saves Money:** Drops unwanted messages at the event bus layer before invoking Lambda, saving 100% of invocation and duration charges for ignored messages.
- **Implementation Steps:**
  1. Add `FilterCriteria` JSON pattern to Lambda Event Source Mapping.
- **Estimated Savings:** 50–90% reduction in invocations for filtered streams.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Event filtering rule definition.

---

## Cross-Service Synergies

```
[ Lambda Function ] ──(VPC Endpoint)──> [ S3 / DynamoDB ] (Avoids NAT $0.045/GB)
        │
        ├──(Batching)───────────> [ SQS Queue ] (BatchSize=100 saves 99% request fees)
        │
        ├──(Orchestration)──────> [ Step Functions ] (Eliminates double-billed idle waits)
        │
        └──(Log Tuning)─────────> [ CloudWatch Logs ] (IA Class saves 50% log ingestion)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `Lambda-GB-Second`, `Lambda-Provisioned-GB-Second`, `Request`.
- `line_item_resource_id`: Lambda Function ARN (`arn:aws:lambda:us-east-1:123456789012:function:my-fn`).
- `product_group`: `AWS-Lambda-Duration`, `AWS-Lambda-Requests`.

### B. CloudWatch Metrics
- `AWS/Lambda` Namespace: `Invocations`, `Duration`, `Errors`, `Throttles`, `ConcurrentExecutions`, `ProvisionedConcurrencyUtilization`, `ProvisionedConcurrencySpilloverInvocations`.
- Resolution: 1-hour metrics over 30 days.

### C. Diagnostics & Optimization Tools
- **AWS Lambda Power Tuning:** State machine execution outputs.
- **AWS Config Rules:** `lambda-function-public-access-prohibited`, `lambda-inside-vpc`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "LAM-RS-001",
  "service": "Lambda",
  "category": "Rightsizing",
  "resource_id": "arn:aws:lambda:us-east-1:123456789012:function:payment-webhook-handler",
  "resource_name": "payment-webhook-handler",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "architecture": "x86_64",
    "memory_mb": 2048,
    "timeout_seconds": 300,
    "avg_duration_ms": 450,
    "monthly_invocations": 15000000,
    "monthly_cost_usd": 384.50
  },
  "recommended_config": {
    "architecture": "arm64",
    "memory_mb": 1024,
    "timeout_seconds": 10,
    "projected_monthly_cost_usd": 153.80
  },
  "financial_impact": {
    "monthly_savings_usd": 230.70,
    "annual_savings_usd": 2768.40,
    "savings_percentage": 60.0
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "Node.js runtime natively supported on ARM64; Power Tuning confirmed 1024MB maintains equal 450ms performance."
  },
  "implementation": {
    "scope": "Engineer/DevOps",
    "effort_estimate": "1 hour",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Waste Elimination** | 12 | $1,800.00 | $1,620.00 | 90.0% | Low |
| **Rightsizing (Graviton/Memory)** | 35 | $9,400.00 | $3,290.00 | 35.0% | Low |
| **Architecture (Batching/StepFn)** | 14 | $4,500.00 | $2,700.00 | 60.0% | Medium |
| **Network & Logs Optimization** | 28 | $3,100.00 | $1,860.00 | 60.0% | Low |
| **Total** | **89** | **$18,800.00** | **$9,470.00** | **50.4%** | -- |
