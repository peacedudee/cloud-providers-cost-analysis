# Cost-Cutting Playbook: Amazon Personalize
> **Companion File:** [personalize.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/personalize/personalize.md)
> **Last Updated:** July 2026
---
## Executive Summary
Amazon Personalize offers powerful, fully managed machine learning for tailored recommendations, but costs can escalate quickly if improperly managed. The primary cost drivers are idle real-time campaigns (with a mandatory minimum billing of 1 TPS, costing ~$146.40/month per campaign), excessive full-model retraining schedules, and over-provisioned TPS capacities. This playbook outlines actionable strategies to optimize Amazon Personalize expenditures by shifting to batch inference where appropriate, rigorously managing campaign lifecycles, and rightsizing training pipelines.

## Strategy Categories
### 1. Waste Elimination
1. **Delete Idle Non-Production Campaigns:** Actively terminate Real-Time Campaigns in dev, QA, and staging environments when not actively being tested to avoid the $146.40/month minimum baseline fee.
2. **Remove Unused Solutions and Datasets:** Periodically audit and delete abandoned solution versions, datasets, and dataset groups that consume storage or trigger unnecessary downstream costs.
3. **Clean Up Inactive Event Trackers:** Delete Event Trackers that are no longer actively receiving real-time interactions to simplify architecture and prevent accidental ingestions.
4. **Prune Stale Historical Data:** Filter out extremely old or irrelevant historical interaction data before uploading to S3 for ingestion. This reduces both the Data Ingestion costs ($0.05/GB) and Model Training costs ($0.24/hour).
5. **Eliminate Redundant Metadata:** Avoid ingesting user or item metadata fields that have proven to have no predictive value for your specific recipe, reducing ingestion and training bloat.

### 2. Rightsizing
6. **Minimize Provisioned TPS:** Set the minimum provisioned TPS for all Real-Time Campaigns to 1 (the absolute baseline). Rely on Amazon Personalize’s built-in auto-scaling to handle traffic spikes naturally.
7. **Optimize Dataset Time Windows:** Instead of training models on all-time interaction logs, restrict data to the most recent and relevant time window (e.g., last 6-12 months) that actually impacts user behavior.
8. **Rightsize Batch Inference Jobs:** Aggregate batch recommendation requests into larger, less frequent jobs rather than running numerous small micro-batches, which can introduce processing overhead.

### 3. Commitment Discounts
9. **Leverage AWS Enterprise Discount Program (EDP):** While Amazon Personalize does not offer native Reserved Instances or Savings Plans, its usage contributes to overall AWS EDP spend commitments. Ensure Personalize costs are factored into organizational EDP negotiations.

### 4. Architecture Changes
10. **Shift to Batch Recommendations for Async Use Cases:** For features like email newsletters, push notifications, or daily personalized feeds, replace expensive Real-Time Campaigns with Batch Recommendations ($0.086 per 1,000 recommendations). Export outputs to S3 and serve them cheaply via DynamoDB.
11. **Implement a Recommendation Cache:** Place a caching layer (e.g., Amazon ElastiCache for Redis or Amazon DynamoDB DAX) in front of your Real-Time Campaigns. Cache the results for specific users or contexts for a short TTL to dramatically reduce the TPS load on Personalize.
12. **Consolidate Campaigns:** If multiple disparate recommendation surfaces can be served by a single, contextualized model (e.g., using contextual recommendations), consolidate them into one campaign to avoid paying the minimum 1 TPS fee multiple times.
13. **API Gateway Rate Limiting:** Place Amazon API Gateway with strict rate limits in front of your Personalize campaign invocations to protect against bot traffic, scrapers, or DDoS attacks artificially inflating your TPS billing tier.

### 5. Scheduling & Auto-Scaling
14. **Automate Ephemeral Campaign Teardown:** Use AWS EventBridge and AWS Lambda to automatically delete dev/test campaigns at the end of the business day (e.g., 6 PM) and re-provision them only on-demand.
15. **Decrease Full Retraining Frequency:** Shift from daily full-model retraining to weekly or bi-weekly schedules. Rely on the real-time event tracker to influence recommendations dynamically between full retrains.
16. **Adopt Incremental Training:** Where supported by the recipe (like User-Personalization), utilize incremental training rather than full retraining to quickly update the model with new items and interactions at a fraction of the compute cost.
17. **Dynamic TPS Provisioning for Known Events:** Keep `minProvisionedTPS` at 1 generally, but use scheduled Lambda functions to pre-warm and scale up the minimum TPS just prior to massive expected traffic spikes (e.g., Black Friday, flash sales), then scale back down immediately after.

### 6. Pricing Model Optimization
18. **Monitor TPS Volume Tiers:** Real-time inference pricing drops from $0.20 to $0.10 per TPS-hour after 20,000 TPS-hours, and down to $0.05 after 200,000. Track your aggregate usage; in some edge cases with high volume, consolidating workloads into a single AWS account (if currently split) can help achieve these volume discount tiers faster.

### 7. Network & Data Transfer Optimization
19. **Co-locate S3 Buckets and Personalize:** Ensure that the S3 buckets containing your bulk training datasets are located in the exact same AWS Region as your Amazon Personalize deployment to avoid inter-region data transfer fees.
20. **Minimize Real-Time Event Payload Size:** When streaming data to the Event Tracker API from clients or application servers, send only the strictly necessary identifiers and properties to reduce outbound bandwidth/egress costs on the client side.

---
## Cross-Service Synergies
* **Amazon S3:** Use S3 Lifecycle policies to automatically expire or transition old Personalize dataset exports to Glacier.
* **Amazon DynamoDB & ElastiCache:** Use as low-cost serving layers for pre-computed Batch Recommendations, replacing the need for Real-Time Campaigns entirely for static use cases.
* **AWS Lambda & EventBridge:** Critical for orchestrating the automated startup/teardown of non-production campaigns and scheduling batch inference jobs.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
* `lineItem/ProductCode` == 'AmazonPersonalize'
* Query `lineItem/UsageType` for `DataUpload`, `Training-Hour`, and `RealTime-TPS-Hour`.
* Group by `resourceId` to identify specific campaigns driving high TPS costs.

### B. CloudWatch Metrics
* **Personalize Campaign Metrics:** `ActualTPS` to verify if a campaign is perpetually sitting at 0 or below the provisioned minimum.

### C. AWS Config / Trusted Advisor
* Track the creation and persistent lifecycle state of Personalize Campaigns, Solutions, and Datasets.

### D. Company Policies
* Data retention limits dictating how far back historical interactions should be kept and fed into training.
* Acceptable staleness for recommendations (to justify moving from Real-Time to Batch).

### E. IaC (Optional)
* Terraform/CloudFormation configurations to verify `minProvisionedTPS` settings and ensure automated teardown logic is present for dev environments.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "PERSONALIZE-001",
  "resource_id": "arn:aws:personalize:us-east-1:123456789012:campaign/dev-recommendations",
  "category": "Waste Elimination",
  "strategy": "Delete Idle Non-Production Campaigns",
  "severity": "High",
  "estimated_savings_monthly": 146.40,
  "recommendation": "Delete the 'dev-recommendations' campaign as its ActualTPS has been 0 for 7 days. Recreate only when active testing resumes."
}
```

### Summary Report Table
| Finding ID | Strategy | Resource | Environment | Estimated Savings / Mo | Effort |
|------------|----------|----------|-------------|------------------------|--------|
| PERSONALIZE-001 | Delete Idle Campaign | `dev-recommendations` | Dev | $146.40 | Low |
| PERSONALIZE-002 | Shift to Batch Inference | `email-marketing-campaign` | Prod | $350.00 | Med |
| PERSONALIZE-003 | Reduce Retraining Freq. | `user-personalization-solution`| Prod | $120.00 | Low |
