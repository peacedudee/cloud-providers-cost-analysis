# Cost-Cutting Playbook: Amazon SES
> **Companion File:** [ses.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/ses/ses.md)
> **Last Updated:** July 2026
---
## Executive Summary
Amazon Simple Email Service (SES) provides a cost-effective, high-throughput solution for inbound and outbound emails. However, costs can escalate due to over-provisioning dedicated IPs, sending bloated email attachments inline, and processing high bounce rates. This playbook details 20 actionable strategies to optimize SES architecture, data transfer, and configuration to minimize monthly bills while maintaining high deliverability.

## Strategy Categories
### 1. Waste Elimination
### 2. Rightsizing
### 3. Commitment Discounts
### 4. Architecture Changes
### 5. Scheduling & Auto-Scaling
### 6. Pricing Model Optimization
### 7. Network & Data Transfer Optimization
---
## Cross-Service Synergies
- **S3 / CloudFront:** Offload email attachments from SES payload ($0.12/GB) to S3/CloudFront ($0.085/GB or less) to save on data attachment costs.
- **SQS & Lambda:** Buffer bounce and complaint notifications using SQS and process in batches with Lambda to reduce serverless execution costs.
- **VPC Endpoints (PrivateLink):** Use VPC endpoints for SES SMTP to bypass NAT Gateway data transfer charges.
- **CloudWatch:** Implement automated bounce-rate alarms to pause compromised sending campaigns.
---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
- Query `lineItem/ProductCode` for `AmazonSES`.
- Break down by `UsageType` to separate `Data-Bytes-Out`, `Email-Sent`, and `Dedicated-IP`.
### B. CloudWatch Metrics
- `Reputation.BounceRate` and `Reputation.ComplaintRate`.
- `Send` and `Delivery` volume metrics.
### C. AWS Config / Trusted Advisor
- Identify underutilized Dedicated IPs.
- Review IAM policies granting SES access to ensure minimal privileges.
### D. Company Policies
- Marketing team rules for removing unengaged subscribers.
- Compliance rules on email retention and attachment distribution.
### E. IaC (Optional)
- Terraform/CloudFormation configurations for SES IP Pools, Configuration Sets, and Event Destinations.
---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "SES-001",
  "category": "Waste Elimination",
  "strategy": "Clean Email Lists",
  "estimated_savings_percent": "2-5%",
  "risk_level": "Low"
}
```
### Summary Report Table
| ID | Strategy | Savings | Risk | Scope |
|---|---|---|---|---|
| SES-001 | Clean Email Lists | 2-5% | Low | Engineer/DevOps |
| ... | ... | ... | ... | ... |

*(Table will be automatically generated in final reports based on the findings above).*

---

### 1. Waste Elimination

#### 1. SES-001. Clean Email Lists (Bounce/Complaint Prevention)
- **What:** Use email verification tools to scrub inactive or invalid email addresses before sending.
- **Why It Saves Money:** Avoids paying $0.10/1,000 for emails that will instantly bounce, and prevents account suspension which incurs massive business downtime costs.
- **Implementation Steps:** 
  1. Export the active mailing list.
  2. Run the list through a third-party verification API (e.g., NeverBounce, ZeroBounce).
  3. Remove hard bounces and toxic domains from your database.
  4. Integrate a real-time verification API at the user signup stage.
- **Estimated Savings:** 2-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Email verification service subscription.

#### 2. SES-002. Disable Unnecessary SNS Delivery Notifications
- **What:** Stop logging every successful SES delivery to SNS or CloudWatch if the data is not actively queried or needed.
- **Why It Saves Money:** SNS messages and CloudWatch Logs ingestion/storage have associated costs. Disabling successful delivery logs saves downstream logging expenses.
- **Implementation Steps:**
  1. Open Amazon SES Configuration Sets.
  2. Edit Event Destinations.
  3. Uncheck "Send" and "Delivery" events if unnecessary, but keep "Bounce" and "Complaint" enabled for reputation management.
- **Estimated Savings:** 5-10% (Indirect logging savings)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Confirmation that delivery receipts are not required for compliance.

#### 3. SES-003. Eliminate Unused Dedicated IPs
- **What:** Release Dedicated IPs that were provisioned but are no longer mapped to active sending domains or configurations.
- **Why It Saves Money:** Each Dedicated IP incurs a flat fee of $24.95/month, regardless of whether a single email is sent over it.
- **Implementation Steps:**
  1. Go to the SES Console -> Dedicated IPs.
  2. Identify IPs with zero sending volume over the last 30 days.
  3. Release the IPs.
- **Estimated Savings:** $299.40/year per unused IP
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ensure no active domains rely on the IP.

#### 4. SES-004. Prune Unengaged Subscribers
- **What:** Automatically remove or suppress users who haven't opened or clicked an email in 6+ months from marketing blasts.
- **Why It Saves Money:** Reduces the total outbound volume, saving directly on the $0.10/1,000 email rate while improving sender reputation and inbox placement.
- **Implementation Steps:**
  1. Query your marketing automation database for user engagement metrics.
  2. Segment out users with no activity in > 6 months.
  3. Exclude this segment from mass email campaigns.
- **Estimated Savings:** 10-20%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Marketing
- **Prerequisites:** Ability to track email opens/clicks.

### 2. Rightsizing

#### 5. SES-005. Revert to Shared IP Pools for Low Volume Senders
- **What:** Stop leasing Dedicated IPs and return to AWS Shared IP Pools if your organization sends fewer than 300,000 emails per month.
- **Why It Saves Money:** Shared IP pools cost $0.00 extra. Dedicated IPs cost $24.95/month. For low volume, dedicated IPs cannot be kept sufficiently "warm" anyway, leading to poor deliverability.
- **Implementation Steps:**
  1. Audit monthly SES outbound send volume.
  2. If volume is consistently < 300,000, migrate sending configurations to the shared IP pool.
  3. Release the Dedicated IP.
- **Estimated Savings:** $299.40/year per IP
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Monthly volume consistently below 300k.

#### 6. SES-006. Minimize Email Payload Size
- **What:** Reduce HTML bloat, unnecessary CSS, and inline base64 images in standard email templates.
- **Why It Saves Money:** SES bills data attachments and payloads at $0.12/GB. Bloated templates multiply across millions of emails, increasing GBs processed.
- **Implementation Steps:**
  1. Review standard transactional and marketing templates.
  2. Minify HTML and CSS.
  3. Host images on external CDNs rather than embedding them via base64 encoding.
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Front-end/Design team approval for template code changes.

#### 7. SES-007. Optimize Inbound Email Storage Lifecycle
- **What:** Set strict Amazon S3 lifecycle rules for inbound emails processed and saved via SES Receipt Rules.
- **Why It Saves Money:** Inbound emails saved to S3 will accumulate indefinitely, costing $0.023/GB/month.
- **Implementation Steps:**
  1. Identify the specific S3 bucket used by SES inbound receipt rules.
  2. Apply a bucket lifecycle policy to transition older emails to Infrequent Access or delete them after 30-90 days.
- **Estimated Savings:** 10-30% of SES-related S3 costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Verify legal compliance requirements for email retention.

### 3. Commitment Discounts

#### 8. SES-008. Maximize Free Tier Utilization
- **What:** Leverage the Amazon SES free tier specifically for development, staging, or new startup environments.
- **Why It Saves Money:** The SES free tier provides 3,000 free messages per month for the first 12 months for eligible accounts.
- **Implementation Steps:**
  1. Verify the AWS account's Free Tier eligibility.
  2. Configure dev/test application environments to route through the eligible account.
  3. Implement safeguards to stay under the 3,000 limit.
- **Estimated Savings:** Up to $3.60/year (small, but zero-cost testing)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Account must be under 12 months old.

#### 9. SES-009. Consolidate Volume for Enterprise Discount Program (EDP)
- **What:** Ensure all decoupled AWS accounts (e.g., acquisitions, distinct business units) using SES are consolidated under the primary AWS Organization to contribute to an EDP commit.
- **Why It Saves Money:** While SES has flat pricing and no reserved instances, contributing its spend to an EDP helps reach overall AWS discount thresholds (typically 9-15% across all services).
- **Implementation Steps:**
  1. Use AWS Organizations for consolidated billing.
  2. Audit stray AWS accounts using SES and invite them to the Org.
- **Estimated Savings:** 9-15% (Global discount)
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** High total AWS spend qualifying for EDP.

### 4. Architecture Changes

#### 10. SES-010. Host Attachments on S3 / CloudFront Instead of Inline
- **What:** Replace heavy inline email attachments (PDF invoices, reports) with S3 pre-signed URLs or CloudFront links.
- **Why It Saves Money:** SES data payload is billed at $0.12/GB. CloudFront data transfer is $0.085/GB or less. Furthermore, MIME-encoding inline attachments inflates file size by ~33%, drastically increasing SES byte-charges.
- **Implementation Steps:**
  1. Modify application code to upload the attachment file to S3.
  2. Generate a secure, time-limited pre-signed URL.
  3. Insert a "Download Attachment" button/link into the email body.
- **Estimated Savings:** 30-40% on data payload costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Existing S3 architecture.

#### 11. SES-011. Batch and Digest Notifications
- **What:** Send consolidated daily or weekly digests instead of real-time, individual alerts.
- **Why It Saves Money:** Sending 1 daily digest containing 10 updates instead of 10 individual emails saves 90% of the per-message outbound cost ($0.10/1,000).
- **Implementation Steps:**
  1. Update user notification preferences to allow a "digest mode".
  2. Buffer user events into an SQS queue or database table.
  3. Use a cron job (EventBridge) to process and send combined digests.
- **Estimated Savings:** Up to 90%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Product/UX approval for delayed notifications.

#### 12. SES-012. Process Bounces/Complaints with SQS Batching
- **What:** Route SES bounce and complaint notifications to Amazon SQS, and process them in large batches using AWS Lambda.
- **Why It Saves Money:** Triggering a Lambda function per individual bounce incurs high invocation costs. Using an SQS event source mapping to batch 10-100 events per Lambda invocation slashes computing costs.
- **Implementation Steps:**
  1. Configure SES SNS topics to forward to SQS.
  2. Configure a Lambda function to consume the SQS queue with a batch size of 100.
  3. Process the bounce updates in your database.
- **Estimated Savings:** 50-80% on Lambda execution costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** High outbound sending volume resulting in frequent bounces.

#### 13. SES-013. Drop Unwanted Inbound Emails at the SES Rule Level
- **What:** Use SES Receipt Rules to drop spam or unwanted inbound emails immediately before they are processed by downstream services (like Lambda or S3).
- **Why It Saves Money:** You pay $0.10/1k to receive emails, but dropping them early prevents expensive downstream Lambda invocations, S3 storage, and data transfer costs.
- **Implementation Steps:**
  1. Configure SES Receipt Rules in the console.
  2. Add specific condition blocks for known unwanted senders or spam patterns.
  3. Set the action to "Bounce" or "Stop Rule Set".
- **Estimated Savings:** 10-50% on inbound processing pipeline
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Identifiable spam or junk mail patterns.

### 5. Scheduling & Auto-Scaling

#### 14. SES-014. Implement an Auto-Pause Mechanism for High Bounces
- **What:** Automatically pause outbound sending campaigns if the SES bounce rate exceeds the 5% warning threshold.
- **Why It Saves Money:** Sending to dead lists wastes money ($0.10/1,000), but more importantly, exceeding a 10% bounce rate will cause AWS to suspend the SES account—resulting in catastrophic business downtime and support costs.
- **Implementation Steps:**
  1. Monitor the CloudWatch metric `Reputation.BounceRate`.
  2. Create an alarm triggering if the rate exceeds 0.05 (5%).
  3. Trigger an SNS topic to alert on-call engineers, and invoke a Lambda to automatically pause SES sending via API.
- **Estimated Savings:** Avoids unquantifiable downtime costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Active CloudWatch monitoring.

#### 15. SES-015. Use Scheduled Provisioning for Dedicated IP Warmup
- **What:** When a Dedicated IP is leased, warm it up gradually using automated scheduled scripts instead of manually blasting maximum volume.
- **Why It Saves Money:** Proper automated warm-up prevents the IP from being blacklisted by ISPs. Blacklisted IPs force you to release them and lease new ones ($24.95/mo), while wasting money on undelivered emails.
- **Implementation Steps:**
  1. Script a schedule (via EventBridge/Lambda) to ramp up daily sending volume by 20%.
  2. Monitor reputation metrics rigorously during the warmup phase.
- **Estimated Savings:** Avoids replacing leased IPs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Outbound volume sufficient for a Dedicated IP (> 300k/mo).

### 6. Pricing Model Optimization

#### 16. SES-016. Bring Your Own IP (BYOIP) vs Leasing
- **What:** If your organization already owns a /24 IP range, import it to SES instead of leasing AWS IPs.
- **Why It Saves Money:** Avoids paying AWS $24.95/month per dedicated IP. You utilize your owned IPs at no additional monthly AWS IP charge.
- **Implementation Steps:**
  1. Verify BYOIP eligibility for your target AWS region.
  2. Create a Route Origin Authorization (ROA).
  3. Provision the IP range into your AWS account and allocate it to SES.
- **Estimated Savings:** $24.95/month per IP
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** Ownership of a registered /24 IPv4 block.

#### 17. SES-017. Segregate Standard vs Dedicated Workloads
- **What:** Split email workloads based on criticality. Send bulk/promotional emails via the $0.00 Shared Pool, and sensitive transactional emails (password resets) via a single Dedicated IP.
- **Why It Saves Money:** Prevents over-provisioning multiple Dedicated IPs. You don't need a $24.95/mo IP for every domain/application—only for workloads that demand absolute strict reputation isolation.
- **Implementation Steps:**
  1. Create separate SES Configuration Sets.
  2. Route traffic programmatically based on email type (marketing vs transactional).
- **Estimated Savings:** Reduces total Dedicated IP lease count
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Categorized internal email streams.

### 7. Network & Data Transfer Optimization

#### 18. SES-018. Route Traffic via VPC Interface Endpoints (PrivateLink)
- **What:** Connect applications in private subnets to the SES API using a VPC Interface Endpoint instead of traversing a NAT Gateway.
- **Why It Saves Money:** NAT Gateway data processing costs $0.045/GB. VPC Endpoints cost approximately $0.01/GB, slashing network data transfer costs.
- **Implementation Steps:**
  1. Go to VPC Console -> Endpoints.
  2. Create a VPC Interface Endpoint for `email-smtp.[region].amazonaws.com`.
  3. Ensure application security groups route SMTP/API traffic locally.
- **Estimated Savings:** ~75% on VPC data transfer to SES
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workloads deployed in private VPC subnets.

#### 19. SES-019. Compress Payloads Locally Before Sending
- **What:** If you absolutely must send raw attachments through SES, gzip compress them before attaching to the email payload.
- **Why It Saves Money:** Significantly reduces the size of the payload billed at the $0.12/GB data attachment rate.
- **Implementation Steps:**
  1. Add compression logic to the backend attachment generator.
  2. Attach the resulting `.zip` or `.gz` file to the outbound email instead of `.csv` or `.json`.
- **Estimated Savings:** 50-80% on data payload costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** End-users are capable of extracting zipped files.

#### 20. SES-020. Use SES SMTP within the Same Region
- **What:** Ensure that EC2, Fargate, or Lambda instances sending emails are located in the exact same AWS region as the SES endpoint they are calling.
- **Why It Saves Money:** Cross-region data transfer out (e.g., an EC2 instance in us-west-2 sending data to SES in us-east-1) costs $0.02/GB. Keeping traffic in-region eliminates this hidden fee.
- **Implementation Steps:**
  1. Audit the region of your compute workloads.
  2. Configure the SMTP client to use the SES endpoint specific to that region.
- **Estimated Savings:** Eliminates cross-region transit fees
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SES is supported in your active compute region.
