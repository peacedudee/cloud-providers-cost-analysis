# Cost-Cutting Playbook: Amazon API Gateway

> **Companion File:** [api_gateway.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/api_gateway/api_gateway.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon API Gateway is a fully managed serverless API proxy supporting **HTTP APIs**, **REST APIs**, and **WebSocket APIs**. While request rates appear modest ($1.00/M for HTTP APIs vs $3.50/M for REST APIs), architectural missteps — such as defaulting to REST APIs when lightweight HTTP APIs suffice (3.5x price multiplier), leaving flat hourly API cache instances running in dev/staging environments ($14.60 to $2,774/month flat), and passing multi-megabyte payloads directly through responses ($0.09/GB egress) — create major cost leaks.

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **35–75% reduction in total API Gateway spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Migrate Simple Lambda Proxies from REST APIs to HTTP APIs:** Instant **71% request cost reduction** ($1.00/M vs $3.50/M).
2. **Disable API Gateway Caching in Non-Production Stages:** Eliminates flat hourly cache charges ($14.60 to $2,774/mo) on dev/test environments.
3. **Offload Large Response Payloads (> 1 MB) to S3 Presigned URLs:** Avoids API Gateway payload egress processing taxes ($0.09/GB).

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Disable API Gateway Stage Caching on Dev/Test/Staging Stages
- **What:** Disable fixed API stage caching on non-production deployment stages.
- **Why It Saves Money:** API Gateway caching on REST APIs bills a flat hourly fee based on memory size regardless of request volume or cache hit ratio (0.5 GB costs **$14.60/mo**, 6.1 GB costs **$182.50/mo**, 118 GB costs **$2,774.00/mo**). Leaving cache active on 10 idle dev stages wastes $1,825/month for zero traffic.
- **Detailed Implementation Steps:**
  1. Audit active caches across stages via CLI:
     ```bash
     aws apigateway get-stages \
       --rest-api-id api-123456 \
       --query "item[*].{Stage:stageName,CacheEnabled:methodSettings.'*/*'.cachingEnabled,CacheSize:methodSettings.'*/*'.cacheClusterSize}"
     ```
  2. Disable caching on dev/test stages:
     ```bash
     aws apigateway update-stage \
       --rest-api-id api-123456 \
       --stage-name dev \
       --patch-operations op=replace,path=/cacheClusterEnabled,value=false
     ```
- **Estimated Savings:** $14.60 to $2,774.00 per stage per month (100% of non-prod cache fees).
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Stage audit.

#### 2. Decommission Unused or Orphaned API Gateway Routes & Deployments
- **What:** Identify and delete API Gateway instances with zero request volume over 30 days.
- **Why It Saves Money:** Cleans up API key associations, CloudWatch log groups, and WAF Web ACL attachments.
- **Detailed Implementation Steps:**
  1. Query CloudWatch metric `Count` with `Sum = 0` over 30 days.
  2. Delete API Gateway:
     ```bash
     aws apigatewayv2 delete-api --api-id http-api-id
     ```
- **Estimated Savings:** Administrative hygiene and log group cleanup.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** 30-day request metric audit.

---

### 2. Architecture & API Type Optimization

#### 3. Migrate Simple Lambda Proxies & HTTP Endpoints to HTTP APIs
- **What:** Convert REST APIs used primarily for proxying requests to AWS Lambda or internal VPC endpoints to **HTTP APIs**.
- **Why It Saves Money:** HTTP APIs cost **$1.00 per million requests** vs REST APIs at **$3.50 per million requests** — an immediate **71% direct request cost reduction**. Processing 100M requests/month drops from $350.00 to $100.00 ($3,000/year saved).
- **Detailed Implementation Steps:**
  1. Audit REST APIs for complex enterprise features (XML mapping templates, API Keys, inline request validation).
  2. For standard proxy APIs, recreate using API Gateway v2 (HTTP API) in Terraform:
     ```hcl
     resource "aws_apigatewayv2_api" "http_api" {
       name          = "orders-http-api"
       protocol_type = "HTTP"
       target        = aws_lambda_function.orders_lambda.arn
     }
     ```
  3. Update DNS CNAME to point to new HTTP API custom domain endpoint.
- **Estimated Savings:** **71% direct request fee reduction**.
- **Risk Level:** Low to Medium (requires API spec verification).
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** API does not require REST-only features (WAF native integration, client certificates, XML mapping).

#### 4. Front API Gateway with Amazon CloudFront CDN
- **What:** Place an Amazon CloudFront distribution in front of regional API Gateway custom domains.
- **Why It Saves Money:**
  1. CloudFront data egress rates ($0.085/GB) are cheaper than direct API Gateway egress ($0.090/GB).
  2. CloudFront includes **1 TB/month of free internet egress**.
  3. CloudFront provides edge caching without fixed hourly API Gateway cache instance fees.
- **Detailed Implementation Steps:**
  1. Create CloudFront distribution pointing to API Gateway regional domain as origin.
  2. Set Cache Policy to forward `Authorization` and query parameters.
- **Estimated Savings:** 15–40% on data egress & caching fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudFront distribution setup.

#### 5. Offload Large Payloads (> 1 MB) via S3 Presigned URLs
- **What:** Refactor API endpoints returning large JSON reports, image binaries, or CSV exports (> 1 MB) to generate an **S3 Presigned URL** instead of returning the binary payload directly in the HTTP response.
- **Why It Saves Money:** Returning a 10 MB payload through API Gateway processes 10 MB of data transfer ($0.09/GB egress) and consumes Lambda memory/duration. Serving via S3 Presigned URL uses S3 storage/egress rates directly and bypasses API Gateway payload taxes.
- **Detailed Implementation Steps:**
  1. Update Lambda function code:
     ```python
     import boto3

     s3_client = boto3.client('s3')

     def lambda_handler(event, context):
         url = s3_client.generate_presigned_url(
             'get_object',
             Params={'Bucket': 'reports-bucket', 'Key': 'monthly-report.pdf'},
             ExpiresIn=300
         )
         return {'statusCode': 200, 'body': json.dumps({'download_url': url})}
     ```
- **Estimated Savings:** 80–95% cost reduction for large asset responses.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Application client support for 2-step download redirect.

---

### 3. Pricing Model & Network Optimization

#### 6. Implement Response Compression (Gzip / Brotli)
- **What:** Enable minimum payload compression on API Gateway REST and HTTP APIs.
- **Why It Saves Money:** Automatically compresses JSON response payloads larger than 1 KB, reducing bandwidth egress volume ($0.09/GB) by 60–80%.
- **Detailed Implementation Steps:**
  1. Enable minimum compression size (1024 bytes) on REST API:
     ```bash
     aws apigateway update-rest-api \
       --rest-api-id api-123456 \
       --patch-operations op=replace,path=/minimumCompressionSize,value=1024
     ```
- **Estimated Savings:** 60–80% payload bandwidth egress savings.
- **Risk Level:** Zero risk (clients send `Accept-Encoding: gzip`).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 7. Enforce Idle Disconnects on WebSocket APIs
- **What:** Configure client-side or server-side idle timeout handlers on WebSocket APIs to disconnect idle client connections after 10–15 minutes of inactivity.
- **Why It Saves Money:** WebSocket connection time costs **$0.25 per million connection minutes**. Leaving 10,000 idle client connections open 24/7 generates **4.32 billion connection minutes/month**, billing **$1,080.00/month** for zero active traffic!
- **Detailed Implementation Steps:**
  1. Add heartbeat ping/pong timer in client WebSocket SDK (disconnect if no pong after 5 minutes).
  2. Implement backend connection cleanup Lambda triggered by EventBridge.
- **Estimated Savings:** 70–90% reduction in WebSocket connection minute charges.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Client-side reconnect handling.

---

### 4. Security & WAF Integration

#### 8. Use AWS WAF Rate Limiting at CloudFront Edge (Protect API Gateway)
- **What:** Attach AWS WAF rate-limiting rules to CloudFront edge distributions fronting API Gateway rather than invoking API Gateway directly.
- **Why It Saves Money:** Blocks malicious scraper bots and HTTP flood attacks at the CloudFront edge before they hit API Gateway, avoiding the **$3.50/M REST API request charge** for blocked requests.
- **Detailed Implementation Steps:**
  1. Create AWS WAF Web ACL with Rate-Based Rule (e.g. max 1,000 requests per 5 min per IP).
  2. Attach Web ACL to CloudFront distribution.
- **Estimated Savings:** Protects against $1,000s in sudden DDoS request fee spikes.
- **Risk Level:** Low.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** WAF Web ACL configuration.

#### 9. Optimize API Gateway Custom Authorizers (Enable Token Caching)
- **What:** Enable Authorization Caching (`authorizerResultTtlInSeconds`) on Lambda Custom Authorizers.
- **Why It Saves Money:** Prevents API Gateway from executing your Custom Authorizer Lambda function on *every single incoming API request*. Setting a 300-second (5-minute) TTL caches the IAM policy result, cutting Lambda authorizer invocations by **up to 95%**.
- **Detailed Implementation Steps:**
  1. Update Authorizer TTL via AWS CLI:
     ```bash
     aws apigateway update-authorizer \
       --rest-api-id api-123456 \
       --authorizer-id auth-123456 \
       --patch-operations op=replace,path=/authorizerResultTtlInSeconds,value=300
     ```
- **Estimated Savings:** 80–95% reduction in custom authorizer Lambda invocation duration fees.
- **Risk Level:** Low (authorizer tokens must contain standard expiration claims).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** JWT bearer token usage.

---

### 5. Logging & Observability Tuning

#### 10. Throttle & Filter API Gateway Execution Logs in CloudWatch
- **What:** Disable `FULL_REQUEST_DATA_TRACE` logging on production REST API stages and restrict CloudWatch execution logging levels to `ERROR` only.
- **Why It Saves Money:** Detailed execution logging outputs verbose multi-line headers and body payloads for every request, generating gigabytes of CloudWatch log ingestion ($0.50/GB) and storage ($0.03/GB-mo).
- **Detailed Implementation Steps:**
  1. Set logging level to `ERROR` and disable data trace on stage:
     ```bash
     aws apigateway update-stage \
       --rest-api-id api-123456 \
       --stage-name prod \
       --patch-operations \
         op=replace,path=/*/*/logging/loglevel,value=ERROR \
         op=replace,path=/*/*/logging/dataTrace,value=false
     ```
- **Estimated Savings:** 70–90% reduction in API Gateway CloudWatch logging fees.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Stage log configuration access.

#### 11. Use Single-Line Access Logging (JSON Format)
- **What:** Configure API Gateway Access Logging to output compact, single-line JSON format containing only essential fields (`requestId`, `ip`, `status`, `latency`).
- **Why It Saves Money:** Reduces log byte payload size by 60–80% compared to verbose multi-line default formatting.
- **Detailed Implementation Steps:**
  1. Set custom access log format JSON in stage settings:
     ```json
     {"requestId":"$context.requestId","ip":"$context.identity.sourceIp","status":"$context.status","latency":"$context.responseLatency"}
     ```
- **Estimated Savings:** 60–80% log volume reduction.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Access log destination log group configured.

---

### 6. Operational & Client Batching Optimization

#### 12. Consolidate Multiple Single-Metric Endpoint Calls via Batch APIs
- **What:** Refactor frontend mobile/web client code to send batched HTTP POST requests (e.g. `/api/v1/metrics/batch`) containing arrays of events instead of issuing separate GET/POST requests per event.
- **Why It Saves Money:** Sending 100 client events in 1 batched API request costs 1 API Gateway request. Sending 100 individual requests costs 100 requests.
- **Detailed Implementation Steps:**
  1. Implement batch request endpoint handler in Lambda.
  2. Update client SDK to buffer events locally for 5 seconds before dispatching.
- **Estimated Savings:** 90–99% reduction in request volume for telemetry workloads.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Client SDK batching logic.

#### 13. Implement API Gateway Usage Plans & Quotas to Cap Malicious Clients
- **What:** Configure API Gateway Usage Plans with hard monthly request quotas for third-party API consumers.
- **Why It Saves Money:** Prevents buggy external client scripts from executing runaway loops that generate millions of billable requests.
- **Detailed Implementation Steps:**
  1. Create Usage Plan with quota:
     ```bash
     aws apigateway create-usage-plan \
       --name "Standard-Partner-Plan" \
       --quota limit=100000,offset=0,period=MONTH \
       --throttle burstLimit=100,rateLimit=50
     ```
- **Estimated Savings:** Protects against un-capped third-party billing overages.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** API Key usage plans configured.

#### 14. Use Private API Endpoints inside VPCs
- **What:** For internal microservices communicating across subnets, configure API Gateway endpoints as `PRIVATE` (accessed via VPC Endpoints).
- **Why It Saves Money:** Eliminates internet egress data transfer fees ($0.09/GB) for internal service-to-service communication.
- **Detailed Implementation Steps:**
  1. Set endpoint configuration type to `PRIVATE` and attach VPC Endpoint ID.
- **Estimated Savings:** Saves $0.09/GB on internal traffic.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC Interface Endpoint deployed.

#### 15. Enforce Maximum Request Throttling Limits on API Stages
- **What:** Configure stage-level throttling limits (`rateLimit` and `burstLimit`) on all public API stages.
- **Why It Saves Money:** Protects downstream Lambda and database resources from being overwhelmed by unexpected traffic spikes.
- **Detailed Implementation Steps:**
  1. Update stage throttling settings: `rateLimit = 500`, `burstLimit = 1000`.
- **Estimated Savings:** Downstream compute risk protection.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Peak traffic assessment.

#### 16. Utilize Free Tier Allowances Across Multiple Accounts
- **What:** Structure non-production testing across developer accounts to fully utilize the 1 million free API requests/month 12-month free tier allowance.
- **Why It Saves Money:** Delivers $0.00 API Gateway bills for small development projects.
- **Detailed Implementation Steps:**
  1. Track free tier usage in AWS Billing Console.
- **Estimated Savings:** Baseline free testing.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Account management setup.

---

## Cross-Service Synergies

```
[ Amazon API Gateway ] 
        │
        ├──(API Type Migration)─> [ HTTP APIs (v2) ] (71% direct request price discount)
        │
        ├──(Fronting CDN)───────> [ CloudFront ] (Lower egress rates $0.085/GB + 1TB Free)
        │
        └──(Authorizer Caching)─> [ Lambda Custom Authorizer ] (Caches IAM tokens, cuts Lambda calls 95%)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `ApiGatewayRequest`, `HTTPAPI-Request-Count`, `WebSocket-ConnectionMinutes`, `CacheUsage:0.5GB`.
- `line_item_resource_id`: API Gateway ID (`api-xxxx`).

### B. CloudWatch Metrics
- `AWS/ApiGateway` Namespace: `Count`, `4XXError`, `5XXError`, `Latency`, `IntegrationLatency`, `CacheHitCount`, `CacheMissCount`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "APIGW-TYP-001",
  "service": "API Gateway",
  "category": "Architecture & API Type Optimization",
  "resource_id": "arn:aws:apigateway:us-east-1::/restapis/a1b2c3d4e5",
  "resource_name": "mobile-backend-proxy-api",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "api_type": "REST",
    "monthly_requests_millions": 85.0,
    "caching_enabled": false,
    "monthly_cost_usd": 297.50
  },
  "recommended_config": {
    "api_type": "HTTP",
    "projected_monthly_cost_usd": 85.00
  },
  "financial_impact": {
    "monthly_savings_usd": 212.50,
    "annual_savings_usd": 2550.00,
    "savings_percentage": 71.4
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "API is a standard Lambda proxy; does not use XML mapping or REST-only enterprise features."
  },
  "implementation": {
    "scope": "Software Engineer / DevOps",
    "effort_estimate": "2-4 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **API Type Migration (REST -> HTTP)**| 18 | $8,500.00 | $6,069.00 | 71.4% | Low |
| **Waste Elimination (Dev Caches)** | 8 | $3,200.00 | $3,200.00 | 100.0% | Zero |
| **Authorizer Caching (Lambda)** | 12 | $4,100.00 | $3,485.00 | 85.0% | Low |
| **Payload Egress (S3 Presigned URLs)**| 6 | $2,900.00 | $2,320.00 | 80.0% | Low |
| **Total** | **44** | **$18,700.00** | **$15,074.00** | **80.6%** | -- |
