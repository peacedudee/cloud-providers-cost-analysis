# Cost-Cutting Playbook: AWS AppSync
> **Companion File:** [appsync.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/appsync/appsync.md)
> **Last Updated:** July 2026

---

## Executive Summary
AWS AppSync is a fully managed GraphQL service that simplifies application development by letting you create a flexible API to securely access, manipulate, and combine data from one or more data sources. Since AppSync is a serverless, request-based service, costs scale linearly with API requests ($4.00 per 1M operations), real-time connection minutes ($0.08 per 1M minutes), and subscription messages ($2.00 per 1M messages). The key to optimizing AppSync costs lies in minimizing unnecessary API calls through robust client and server-side caching, filtering real-time subscription payloads, consolidating multiple backend calls into single GraphQL operations, and efficiently managing optional server-side cache instances.

## Strategy Categories
### 1. Waste Elimination
Eliminating idle resources, unnecessary logging, and abandoned APIs to prevent baseline cost creep.
### 2. Rightsizing
Optimizing cache instances, payload sizes, and resolver logic to match actual workload requirements.
### 3. Commitment Discounts
While AppSync itself is purely on-demand, organization-wide EDPs (Enterprise Discount Programs) apply to total spend.
### 4. Architecture Changes
Redesigning API interactions, implementing enhanced filtering, and leveraging local resolvers to fundamentally reduce request volumes and message deliveries.
### 5. Scheduling & Auto-Scaling
Managing optional provisioned resources (like Server-Side caches) to align with usage hours.
### 6. Pricing Model Optimization
Using the right service for the right job, such as substituting HTTP APIs for simple endpoints.
### 7. Network & Data Transfer Optimization
Reducing outbound payload sizes and preventing cross-region communication to eliminate data transfer fees.

---

## Cross-Service Synergies
- **Amazon DynamoDB:** AppSync directly integrates with DynamoDB. Batching operations in AppSync reduces DynamoDB RCUs/WCUs, compounding savings across both services.
- **AWS Lambda:** Offloading simple data transformations from Lambda to AppSync VTL/JS resolvers removes Lambda execution costs entirely.
- **AWS WAF:** Fronting AppSync with WAF rate limiting prevents abusive bot traffic from generating massive GraphQL operation and backend processing fees.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
Identify spend by usage type: `AppSync-Request`, `AppSync-ConnectMinutes`, `AppSync-UpdateMessages`, and `AppSync-Cache`.
### B. CloudWatch Metrics
Analyze `4XXError`, `5XXError`, `Latency`, `CacheHitCount`, `CacheMissCount`, `ConnectClientCount`, and `PublishDataMessageCount`.
### C. AWS Config / Trusted Advisor
Identify unattached/unused AppSync APIs or misconfigured caching instances.
### D. Company Policies
Security requirements for WAF and logging retention periods.
### E. IaC (Optional)
Terraform or AWS CDK code to verify resolver types, caching configurations, and subscription filtering setups.

---

## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "APPSYNC-001",
  "resource_id": "arn:aws:appsync:us-east-1:123456789012:apis/api-id",
  "strategy_used": "Enforce Enhanced Subscription Filtering",
  "monthly_savings": 50.00,
  "effort": "Medium"
}
```

### Summary Report Table
| Finding ID | API Name | Strategy | Est. Savings/Mo | Risk Level |
|------------|----------|----------|-----------------|------------|
| APPSYNC-001 | UserAPI | Enforce Enhanced Subscription Filtering | $250.00 | Medium |

---

## Strategies

### 1. Waste Elimination

#### 1. Disable Server-Side Caching in Non-Production
- **What:** Turn off provisioned server-side caching (e.g., `cache.t3.small`) in development, testing, and staging environments.
- **Why It Saves Money:** Caching is billed hourly per instance (e.g., $0.05/hour = ~$36/month). Disabling it saves 100% of the provisioned cost in idle environments.
- **Implementation Steps:** 
  1. Audit AppSync APIs using AWS CLI to check for active cache instances.
  2. Modify IaC (Terraform/CDK) to conditionally provision caching only for `prod` environments.
  3. Flush and delete the cache instances in non-prod environments.
- **Estimated Savings:** 100% of cache costs for non-prod ($36-$2,000+ per month per API).
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Reliance on client-side caching for dev testing.

#### 2. Reduce CloudWatch Logging Verbosity
- **What:** Disable `ALL` or detailed field-level CloudWatch logging for AppSync in production unless actively debugging.
- **Why It Saves Money:** High-throughput APIs generate massive log volumes. CloudWatch charges $0.50 per GB ingested. Detailed logging logs every resolver and request.
- **Implementation Steps:**
  1. Go to AppSync API Settings.
  2. Change CloudWatch logging level from `ALL` to `ERROR` or `NONE`.
  3. Turn off `Include verbose content`.
- **Estimated Savings:** 60-90% of CloudWatch Logs ingestion costs associated with AppSync.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Proper error handling mechanisms in place.

#### 3. Deprecate and Delete Unused GraphQL APIs
- **What:** Identify and tear down AppSync APIs that have zero traffic over the last 30 days.
- **Why It Saves Money:** Eliminates associated baseline costs like unused server-side caches, WAF ACLs, and lingering backend resources.
- **Implementation Steps:**
  1. Check CloudWatch metrics (`ConnectClientCount`, `Latency`) for zero activity over 30 days.
  2. Snapshot/export backend data if necessary.
  3. Delete the AppSync API and associated backend resources.
- **Estimated Savings:** 100% of baseline infrastructure costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Usage validation with product teams.

#### 4. Clean Up Inactive WebSocket Connections
- **What:** Implement idle timeouts and explicitly disconnect idle clients instead of leaving WebSockets hanging open.
- **Why It Saves Money:** Connections cost $0.08 per 1 Million minutes. Zombie connections accumulate minutes indefinitely.
- **Implementation Steps:**
  1. Configure client applications to disconnect explicitly when backgrounded or idle.
  2. Set connection heartbeat limits and close stale connections from the client side.
- **Estimated Savings:** 10-30% of Real-Time Connection Minute costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Control over client-side application code.

### 2. Rightsizing

#### 5. Rightsize Server-Side Cache Instances
- **What:** Scale down oversized Server-Side Cache instances based on actual cache hit rates and memory utilization.
- **Why It Saves Money:** Moving from `cache.r4.xlarge` ($0.68/hr) to `cache.t3.medium` ($0.10/hr) saves $417/month. 
- **Implementation Steps:**
  1. Review `CacheHitCount`, `CacheMissCount`, and memory metrics for the cache cluster.
  2. If hit rates are stable but memory is vastly underutilized, modify the cache instance class to a smaller tier.
  3. Apply changes during a maintenance window.
- **Estimated Savings:** 40-80% of AppSync caching costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudWatch metrics baseline.

#### 6. Optimize AppSync Resolvers
- **What:** Write efficient VTL (Velocity Template Language) or JavaScript resolvers to prevent unnecessary data transformation overhead or excess payload sizes.
- **Why It Saves Money:** While it doesn't directly reduce the AppSync $4.00/M request fee, it drastically reduces backend costs (e.g., Lambda execution durations, DynamoDB RCUs) and Data Transfer out.
- **Implementation Steps:**
  1. Analyze resolver code.
  2. Remove bloated mappings and only return requested fields.
  3. Use built-in utility functions (`$util`) instead of delegating to Lambda.
- **Estimated Savings:** 10-25% of backend and Data Transfer costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Familiarity with VTL or JS resolvers.

#### 7. Prevent Over-Fetching (Enforce GraphQL Limits)
- **What:** Restrict the maximum depth of GraphQL queries and the maximum number of items returned in a single list query.
- **Why It Saves Money:** Massive queries trigger excessive backend read operations (e.g., thousands of DynamoDB RCUs).
- **Implementation Steps:**
  1. Set up query depth limiting in AppSync settings (if using AWS WAF) or at the application level.
  2. Enforce pagination limits (e.g., `limit: 50`) on all list resolvers.
- **Estimated Savings:** Avoids cost spikes; saves 20-50% on backend database read costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Client apps must support pagination.

#### 8. Consolidate Multiple Backend Calls into 1 GraphQL Query
- **What:** Utilize AppSync to fetch nested data from multiple backend sources in a single request rather than making multiple separate client REST/GraphQL calls.
- **Why It Saves Money:** AppSync bills $4.00 per 1M operations. 1 combined GraphQL request costs $0.000004. Making 5 separate API requests costs 5x more.
- **Implementation Steps:**
  1. Identify client patterns making sequential requests.
  2. Restructure the GraphQL schema to support nested relational queries.
  3. Update client code to fetch all required data in one unified query.
- **Estimated Savings:** Up to 80% of AppSync Operations costs.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Schema refactoring capabilities.

### 3. Commitment Discounts

#### 9. Monitor for AWS Enterprise Discount Program (EDP) Thresholds
- **What:** Roll AppSync spend into a broader AWS EDP commitment if overall organization spend is high enough ($1M+/year).
- **Why It Saves Money:** EDPs provide blanket discounts (usually 9-15%) across all AWS services, including AppSync.
- **Implementation Steps:**
  1. Consolidate billing via AWS Organizations.
  2. Engage AWS Account Manager for EDP negotiations.
- **Estimated Savings:** 9-15% across all services.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team / Procurement
- **Prerequisites:** Total AWS annual spend exceeding $1M.

### 4. Architecture Changes

#### 10. Enforce Enhanced Subscription Filtering
- **What:** Implement AppSync Enhanced Subscriptions with argument filters (e.g., `owner: $userId`).
- **Why It Saves Money:** Subscription messages cost $2.00 per 1M messages. Broadcasting changes globally results in massive unwanted message delivery. Filtering ensures clients only receive pertinent data.
- **Implementation Steps:**
  1. Update GraphQL schema to include filter arguments on subscriptions.
  2. Apply the `@aws_subscribe` directive with specific mutations.
  3. Update backend publish operations to include filter context.
- **Estimated Savings:** Up to 95% of Subscription Message costs.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Client apps must pass filter arguments upon subscribing.

#### 11. Implement Client-Side Caching (Apollo/Amplify)
- **What:** Rely heavily on in-memory or local storage caching on the client applications (using Apollo Client or AWS Amplify).
- **Why It Saves Money:** Avoids sending the query over the network entirely. Saves the $4.00/M operations fee and associated backend costs.
- **Implementation Steps:**
  1. Configure `cache-first` or `stale-while-revalidate` fetch policies in the frontend client.
  2. Ensure proper cache invalidation logic is applied.
- **Estimated Savings:** 30-60% of Operations fees.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Frontend code access.

#### 12. Substitute AppSync with API Gateway HTTP APIs for Simple Routes
- **What:** Migrate simple, flat RESTful endpoints that do not require GraphQL capabilities to API Gateway HTTP APIs.
- **Why It Saves Money:** HTTP APIs cost $1.00 per 1M requests, whereas AppSync costs $4.00 per 1M requests (a 75% savings for basic operations).
- **Implementation Steps:**
  1. Identify simple CRUD operations in the AppSync schema.
  2. Deploy an HTTP API mapping to the same Lambda/DynamoDB backends.
  3. Point the client to the new HTTP API for those specific tasks.
- **Estimated Savings:** 75% on request fees for migrated endpoints.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Hybrid API architecture support in clients.

#### 13. Leverage Local Resolvers (None Data Source) for Pub/Sub
- **What:** Use the `None` data source in AppSync to build purely real-time Pub/Sub channels that do not trigger a backend data store (like DynamoDB or Lambda).
- **Why It Saves Money:** Bypasses the cost of invoking a backend Lambda ($0.20/M + duration) or DynamoDB write ($1.25/M WCU) if the data only needs to be pushed to connected clients immediately.
- **Implementation Steps:**
  1. Create a data source of type `None`.
  2. Attach it to a mutation designed solely to trigger subscriptions.
  3. Map the payload directly to the subscription channel.
- **Estimated Savings:** 100% of backend processing costs for Pub/Sub events.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Use cases that do not require data persistence (e.g., chat typing indicators).

#### 14. Use Delta Sync for Offline Clients
- **What:** Implement AWS AppSync Delta Sync for clients that reconnect after being offline.
- **Why It Saves Money:** Instead of fetching the entire dataset upon reconnection (triggering massive AppSync operations and Data Transfer), clients only fetch the changes (deltas) since their last sync.
- **Implementation Steps:**
  1. Enable Delta Sync in AppSync settings.
  2. Configure an underlying Base table and a Delta table (TTL-enabled) in DynamoDB.
  3. Configure the client SDK to utilize Delta Sync queries.
- **Estimated Savings:** 50-90% reduction in query operations and Data Transfer out for mobile/offline applications.
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** AWS Amplify SDK or equivalent.

#### 15. Shift Static Read Workloads to Amazon CloudFront
- **What:** Put an Amazon CloudFront distribution in front of static or highly cacheable AppSync GraphQL GET requests.
- **Why It Saves Money:** CloudFront serving cached responses costs ~$0.085/GB and zero per-request compute fees, vastly undercutting the $4.00/M AppSync request fee and associated backend database reads.
- **Implementation Steps:**
  1. Configure AppSync to support HTTP GET requests.
  2. Create a CloudFront distribution pointing to the AppSync endpoint.
  3. Configure Cache Behaviors based on query strings.
- **Estimated Savings:** 60-90% on heavy read-heavy, static GraphQL queries.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Queries must be executable via HTTP GET and highly cacheable.

### 5. Scheduling & Auto-Scaling

#### 16. Schedule Server-Side Cache Provisioning
- **What:** Automate the starting and stopping (or creating/deleting) of AppSync Server-Side caches to align with business hours for internal-facing applications.
- **Why It Saves Money:** Running a cache instance 24/7 costs 168 hours/week. Running it 10 hours a day, 5 days a week costs 50 hours/week.
- **Implementation Steps:**
  1. Create an EventBridge Scheduler rule.
  2. Trigger a Lambda function that makes an `UpdateApiCache` or `DeleteApiCache`/`CreateApiCache` API call.
- **Estimated Savings:** Up to 70% of Server-Side cache costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Predictable usage hours (e.g., B2B or internal tools).

#### 17. Implement WAF Rate Limiting
- **What:** Attach AWS WAF to the AppSync API and implement rate-based rules.
- **Why It Saves Money:** Prevents malicious scrapers, bots, or DDoS attacks from racking up millions of unexpected AppSync API operations ($4.00/M) and backend execution costs.
- **Implementation Steps:**
  1. Create a Web ACL in AWS WAF.
  2. Add a rate-based rule (e.g., block IPs exceeding 1,000 requests per 5 minutes).
  3. Associate the WAF Web ACL with the AppSync API.
- **Estimated Savings:** Preventative measure; can save thousands of dollars during a targeted attack.
- **Risk Level:** Low
- **Implementation Scope:** Security / Engineer
- **Prerequisites:** Budget for AWS WAF base costs ($5/month/ACL + $0.60/M requests).

### 6. Pricing Model Optimization

#### 18. Migrate Lambda Data Loaders to VTL/JavaScript Resolvers
- **What:** Rewrite simple Lambda data loader functions directly into AppSync VTL or JavaScript resolvers.
- **Why It Saves Money:** Bypasses Lambda execution costs ($0.20 per 1M requests + memory/duration) completely. AppSync resolvers execute within the AppSync pricing tier at no additional compute cost.
- **Implementation Steps:**
  1. Identify Lambda resolvers performing simple CRUD mapping.
  2. Rewrite the logic using AppSync JavaScript (APPSYNC_JS) functions.
  3. Point the pipeline resolver directly to the target data source (e.g., DynamoDB).
- **Estimated Savings:** 100% of Lambda costs for the migrated resolvers.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Business logic must be achievable within VTL/JS runtime limits.

### 7. Network & Data Transfer Optimization

#### 19. Ensure Same-Region Backend Hosting
- **What:** Ensure that AppSync and its backend resources (DynamoDB, RDS, Lambda) are deployed in the exact same AWS Region.
- **Why It Saves Money:** Cross-region data transfer incurs charges of $0.01 to $0.02 per GB. Same-region data transfer between AppSync and DynamoDB/Lambda is generally free.
- **Implementation Steps:**
  1. Audit region configurations for the AppSync API and data sources.
  2. If mismatched, migrate the API or data sources to unify the region.
- **Estimated Savings:** 100% of cross-region Data Transfer fees.
- **Risk Level:** High (if migration is required)
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region compliance is not strictly required.

#### 20. Implement GraphQL Payload Compression
- **What:** Ensure clients pass the `Accept-Encoding: gzip` header when making AppSync queries.
- **Why It Saves Money:** Reduces the size of the HTTP response body over the internet, saving on Data Transfer Out to Internet costs ($0.09/GB).
- **Implementation Steps:**
  1. Inspect client HTTP requests to AppSync.
  2. Configure Apollo Client, Amplify, or generic `fetch` calls to accept gzip compression.
- **Estimated Savings:** 30-70% reduction in AppSync Data Transfer Out costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Clients must support gzip decompression.
