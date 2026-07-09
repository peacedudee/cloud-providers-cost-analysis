# Cost-Cutting Playbook: AWS WAF (Web Application Firewall)

> **Companion File:** [waf.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/waf/waf.md)  
> **Last Updated:** July 2026

---

## Executive Summary

AWS WAF protects web applications and APIs attached to CloudFront, Application Load Balancers (ALB), API Gateway, App Runner, and Cognito. Billing includes **Web ACL base fees** ($5.00/mo per ACL), **Rule base fees** ($1.00/mo per rule), **Request inspection fees** ($0.60/M requests), and **Managed Rule Group Add-ons** (Bot Control at $10/mo sub + $1.00–$10.00/M requests; Account Takeover Prevention - ATP).

Key cost traps include:
1. **Unscoped Bot Control Evaluation:** Applying Bot Control ($1.00–$10.00/M requests) or Fraud Control to an entire Web ACL attached to a CloudFront distribution serving static assets (images, CSS, JS). Inspecting 50M image requests/mo with targeted Bot Control adds **$500.00/month in Bot Control fees alone**!
2. **Duplicate Web ACL Proliferation:** Deploying individual Web ACLs ($5.00/mo + $1.00/rule) per regional ALB instead of sharing 1 central Web ACL across multiple ALBs.
3. **Over-Inspecting Body Payloads (64 KB Inspection):** Enabling 64 KB body inspection across high-volume upload endpoints ($0.30/M surcharge per 16 KB block).

This playbook provides **16 actionable strategies** across six operational categories, delivering an estimated **35–85% reduction in total AWS WAF spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Add Scope-Down Statements to Bot & Fraud Control Rules:** Restrict Bot Control evaluation strictly to POST requests on sensitive URIs (`/api/v1/login`, `/checkout`), skipping static image/CSS traffic (**95%+ fee reduction**).
2. **Attach Web ACLs at CloudFront (Edge Absorption):** Blocks malicious traffic at the CloudFront edge, saving downstream ALB, EC2, and bandwidth egress costs.
3. **Share Regional Web ACLs Across Multiple ALBs / API Gateways:** Consolidates Web ACLs to eliminate $5.00/mo ACL fees and $1.00/rule fees per endpoint.

---

## Strategy Categories

### 1. Managed Rule & Scope-Down Optimization

#### 1. Add Scope-Down Statements to Bot Control & Fraud Control Managed Rules
- **What:** Configure **Scope-Down Statements** on AWS WAF Managed Rule Groups (Bot Control, Account Takeover Prevention - ATP, Account Creation Fraud - ACF).
- **Why It Saves Money:**
  - Without a Scope-Down Statement, WAF inspects *every single incoming HTTP request* against the managed rule group.
  - **Bot Control (Targeted):** Costs **$10.00 per 1 Million requests** inspected. Inspecting 50M image/CSS requests costs **$500.00/month**.
  - Adding a Scope-Down Statement (`HTTP Method = POST AND URI Path STARTS_WITH /api/auth`) restricts inspection ONLY to login requests (e.g. 500k requests/mo = **$5.00/month**), delivering a **99% cost reduction**!
- **Detailed Implementation Steps:**
  1. Add Scope-Down Statement in WAF Rule Group configuration in Terraform:
     ```hcl
     rule {
       name     = "AWSManagedRulesBotControlRuleSet"
       priority = 1
       override_action { count {} }
       statement {
         managed_rule_group_statement {
           name        = "AWSManagedRulesBotControlRuleSet"
           vendor_name = "AWS"
           scope_down_statement {
             byte_match_statement {
               search_string         = "/api/v1/login"
               field_to_match { uri_path {} }
               positional_constraint = "EXACTLY"
               transformation_type   = "LOWERCASE"
             }
           }
         }
       }
       visibility_config { ... }
     }
     ```
- **Estimated Savings:** **95–99% reduction** in Bot Control and ATP request inspection fees ($100s to $1,000s/mo saved).
- **Risk Level:** Zero risk (guarantees authentication routes are protected while skipping static assets).
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** Sensitive URI route identification.

#### 2. Prefer AWS Managed Rules (Core Rule Set) Over Expensive Third-Party Rules
- **What:** Use **AWS Managed Rules** (Core Rule Set - CRS, Known Bad Inputs, SQLi, Linux OS) instead of third-party Marketplace rule groups that charge extra monthly subscription fees.
- **Why It Saves Money:** AWS Managed Rules have **$0.00 subscription fees** ($1.00/mo rule wrapper + $0.60/M standard request fee). Third-party marketplace rules charge $20 to $100/month subscriptions + $1.00–$3.00/M request fees.
- **Detailed Implementation Steps:**
  1. Add `AWSManagedRulesCommonRuleSet` in WAF Web ACL definition.
- **Estimated Savings:** 50–80% savings on managed rule group fees.
- **Risk Level:** Low.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** Rule evaluation testing in COUNT mode.

---

### 2. Architecture & Web ACL Consolidation

#### 3. Share Regional Web ACLs Across Multiple ALBs and API Gateways
- **What:** Associate a single regional Web ACL across multiple Application Load Balancers and API Gateways within the same region.
- **Why It Saves Money:** Creating separate Web ACLs for 10 regional ALBs costs 10 × $5.00 = **$50.00/month** in base ACL fees, plus 10 × (Rule Count × $1.00/mo). Sharing 1 regional Web ACL costs **$5.00/month** base (saving 90% in base fees).
- **Detailed Implementation Steps:**
  1. Associate central Web ACL ARN with secondary ALB ARNs via AWS CLI:
     ```bash
     aws wafv2 associate-web-acl \
       --web-acl-arn arn:aws:wafv2:us-east-1:123456789012:regional/webacl/shared-prod-acl/xxx \
       --resource-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/alb-2/xxx
     ```
- **Estimated Savings:** **90% reduction** in Web ACL base storage fees.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Common security policy requirements across ALBs.

#### 4. Attach Web ACLs at CloudFront CDN Layer (Edge Absorption)
- **What:** Attach global WAF Web ACLs to Amazon CloudFront edge distributions rather than directly on backend regional Application Load Balancers.
- **Why It Saves Money:**
  1. Blocks malicious scrapers, SQL injection, and HTTP flood attacks at the edge before they reach regional ALBs, NAT Gateways, or EC2 instances.
  2. Saves downstream EC2 compute, ALB LCU capacity, and bandwidth egress charges.
- **Detailed Implementation Steps:**
  1. Create Global (`CLOUDFRONT`) Web ACL in `us-east-1` and associate with CloudFront distribution ID.
- **Estimated Savings:** 20–40% downstream infrastructure spend protection.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** CloudFront distribution fronting application.

---

### 3. Rule Customization & Payload Sizing

#### 5. Use Free Custom Rate-Based Rules for DDoS Mitigation
- **What:** Configure custom **Rate-Based Rules** (e.g. limit clients to 2,000 requests per 5 minutes per IP) for basic HTTP flood protection.
- **Why It Saves Money:** Rate-based rules cost **$1.00 per month** (standard custom rule price) and block volumetric flood attacks automatically without requiring expensive Bot Control subscriptions ($10/mo + $1.00/M requests).
- **Detailed Implementation Steps:**
  1. Create Rate-Based Rule in ASL / Terraform:
     ```hcl
     rule {
       name     = "IPRateLimit"
       priority = 2
       action { block {} }
       statement {
         rate_based_statement {
           limit              = 2000
           aggregate_key_type = "IP"
         }
       }
       visibility_config { ... }
     }
     ```
- **Estimated Savings:** $100s/mo saved vs commercial bot protection rules.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** Rate limit threshold evaluation.

#### 6. Restrict Body Inspection Size to 16 KB Baseline
- **What:** Keep WAF body inspection size at the default **16 KB** limit unless 64 KB payload inspection is strictly required for application payload verification.
- **Why It Saves Money:** Inspecting up to 64 KB incurs an extra surcharge of **$0.30 per 1 Million requests** per 16 KB block increment ($0.90/M surcharge for 64 KB).
- **Detailed Implementation Steps:**
  1. Ensure body inspection size settings remain at 16 KB default.
- **Estimated Savings:** Saves $0.30 to $0.90 per 1M requests.
- **Risk Level:** Low.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** Body inspection requirements review.

---

### 4. Logging & Observability Optimization

#### 7. Filter & Sample WAF Logs (Send Only Blocked Traffic to S3/CloudWatch)
- **What:** Configure WAF Logging Filters to record ONLY `DROP` / `BLOCK` actions, or sample 10% of `ALLOW` traffic, and stream directly to S3 via Kinesis Firehose in Parquet format.
- **Why It Saves Money:** Logging 100% of allowed HTTP request traffic to CloudWatch Logs generates gigabytes of ingestion ($0.50/GB) and storage ($0.03/GB-mo). Filtering logs to `BLOCK` actions cuts logging costs by **95%+**.
- **Detailed Implementation Steps:**
  1. Set WAF logging filter via AWS CLI:
     ```bash
     aws wafv2 put-logging-configuration --logging-configuration '{
       "ResourceArn": "arn:aws:wafv2:us-east-1:123456789012:regional/webacl/shared-acl/xxx",
       "LogDestinationConfigs": ["arn:aws:kinesis-firehose:us-east-1:123456789012:deliverystream/waf-logs"],
       "LoggingFilter": {
         "Filters": [{
           "Behavior": "KEEP",
           "Requirement": "MEETS_ANY",
           "Conditions": [{"ActionCondition": {"Action": "BLOCK"}}]
         }],
         "DefaultBehavior": "DROP"
       }
     }'
     ```
- **Estimated Savings:** **95% reduction** in WAF CloudWatch/S3 logging spend.
- **Risk Level:** Low.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** Kinesis Firehose / S3 destination configured.

---

### 5. Shield Advanced Alignment

#### 8. Utilize Free WAF Web ACLs Included with AWS Shield Advanced
- **What:** If your enterprise subscribes to **AWS Shield Advanced** ($3,000/month flat), ensure all protected resources (ALBs, CloudFront distributions) use AWS WAF.
- **Why It Saves Money:** AWS Shield Advanced includes **AWS WAF Web ACLs, AWS Managed Rules, and WAF request inspection fees at 100% NO EXTRA CHARGE ($0.00)** for all protected resources!
- **Detailed Implementation Steps:**
  1. Verify Shield Advanced protection covers attached WAF Web ACLs.
- **Estimated Savings:** 100% of WAF Web ACL, rule, and request inspection fees on protected resources.
- **Risk Level:** Zero risk.
- **Implementation Scope:** FinOps Team / Security Architect
- **Prerequisites:** AWS Shield Advanced subscription active.

---

### 6. Cleanup & Governance

#### 9. Decommission Orphaned WAF Web ACLs
- **What:** Identify and delete Web ACLs (`aws wafv2 list-web-acls`) with 0 associated AWS resources over 14 days.
- **Why It Saves Money:** Reclaims $5.00/mo base ACL fee plus $1.00/mo per configured rule.
- **Detailed Implementation Steps:**
  1. List associated resources: `aws wafv2 list-resources-for-web-acl --web-acl-arn ARN`.
  2. Delete unassociated Web ACL: `aws wafv2 delete-web-acl --arn ARN`.
- **Estimated Savings:** $5.00 to $20.00/mo saved per orphaned Web ACL.
- **Risk Level:** Zero risk (0 associated resources = zero impact).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Resource association audit.

#### 10. Audit Rule Group Rule Counts (Keep WCU < 1500 Limit)
- **What:** Keep total Web ACL Web Request Capacity Units (WCU) under the 1,500 default threshold to avoid WCU overage surcharges.
- **Why It Saves Money:** Avoids premium capacity tiering surcharges.
- **Detailed Implementation Steps:**
  1. Check WCU count in Web ACL details console.
- **Estimated Savings:** Prevents WCU overage fees.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** WCU calculation.

#### 11. Run New Custom Rules in COUNT Mode First
- **What:** Deploy new custom WAF rules with action = `COUNT` for 48 hours before switching to `BLOCK`.
- **Why It Saves Money:** Prevents accidental blocking of legitimate user traffic that causes business downtime and emergency rollback costs.
- **Detailed Implementation Steps:**
  1. Set rule action to `count` in initial deployment.
- **Estimated Savings:** Operational business continuity protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** WAF CloudWatch metrics monitoring.

#### 12. Consolidate Duplicate IP Set Match Rules
- **What:** Group individual IP blocking statements into a single **IP Set Match Statement** (`AWS::WAFv2::IPSet`).
- **Why It Saves Money:** Storing 500 IP addresses in 1 IP Set counts as 1 rule ($1.00/mo), compared to 500 individual rules ($500.00/mo — a **99.8% savings**!).
- **Detailed Implementation Steps:**
  1. Add IPs to central WAF IPSet object via CLI.
- **Estimated Savings:** 99.8% rule storage fee reduction for IP blocklists.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** WAF IPSet usage.

#### 13. Enable CloudWatch Alarms for WAF Request Inspection Volume
- **What:** Put CloudWatch alarm on `AllowedRequests` and `BlockedRequests` metrics.
- **Why It Saves Money:** Instant alert on traffic spikes driving up inspection fees.
- **Detailed Implementation Steps:**
  1. Create CloudWatch alarm targeting `AWS/WAFV2`.
- **Estimated Savings:** Proactive billing risk protection.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 14. Standardize IAM WAF Management Policies
- **What:** Restrict `wafv2:CreateWebACL` and `wafv2:PutLoggingConfiguration` permissions.
- **Why It Saves Money:** Prevents unapproved Web ACL proliferation.
- **Detailed Implementation Steps:**
  1. Restrict IAM policies for WAF administration.
- **Estimated Savings:** Governance enforcement.
- **Risk Level:** Zero.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** IAM policy review.

#### 15. Consolidate Regex Pattern Sets
- **What:** Package multiple regex string matches into a single **Regex Pattern Set** object.
- **Why It Saves Money:** Reduces custom rule count and WCU consumption.
- **Detailed Implementation Steps:**
  1. Create WAF RegexPatternSet resource in Terraform.
- **Estimated Savings:** Rule fee optimization.
- **Risk Level:** Zero.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** Regex pattern set setup.

#### 16. Audit Custom Header Inspection Statements
- **What:** Simplify complex nested header inspection rules in custom rules.
- **Why It Saves Money:** Improves evaluation speed and WCU efficiency.
- **Detailed Implementation Steps:**
  1. Simplify nested logic statements.
- **Estimated Savings:** Performance optimization.
- **Risk Level:** Low.
- **Implementation Scope:** Security Engineer
- **Prerequisites:** ASL rule review.

---

## Cross-Service Synergies

```
[ Application Traffic ] 
        │
        ├──(Bot Control Rule)───> [ Scope-Down Statement (/api/v1/login) ] (Cuts Bot Control fees by 95%+)
        │
        ├──(Edge Protection)────> [ CloudFront Edge Attachment ] (Blocks bad traffic before ALB/EC2)
        │
        └──(Logging Filter)─────> [ S3 via Firehose (BLOCK Only) ] (Saves 95% on WAF CloudWatch log ingestion)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `WebACL`, `Rule`, `Request`, `BotControl-Request`, `ATP-Request`.
- `line_item_resource_id`: WAF Web ACL ARN (`arn:aws:wafv2:us-east-1:123456789012:regional/webacl/shared-acl/xxx`).

### B. CloudWatch Metrics
- `AWS/WAFV2` Namespace: `AllowedRequests`, `BlockedRequests`, `PassedRequests`, `BotControlAllowedData`, `BotControlBlockedData`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "WAF-SCP-001",
  "service": "WAF",
  "category": "Managed Rule & Scope-Down Optimization",
  "resource_id": "arn:aws:wafv2:us-east-1:123456789012:regional/webacl/prod-cloudfront-acl/12345",
  "resource_name": "prod-cloudfront-acl",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "managed_rule_group": "AWSManagedRulesBotControlRuleSet (Targeted)",
    "scope_down_enabled": false,
    "monthly_inspected_requests_millions": 65.0,
    "monthly_cost_usd": 660.00
  },
  "recommended_config": {
    "scope_down_enabled": true,
    "target_path": "/api/v1/login",
    "projected_monthly_inspected_requests_millions": 0.8,
    "projected_monthly_cost_usd": 18.00
  },
  "financial_impact": {
    "monthly_savings_usd": 642.00,
    "annual_savings_usd": 7704.00,
    "savings_percentage": 97.2
  },
  "risk_assessment": {
    "risk_level": "Zero",
    "reason": "Scope-down statement restricts Bot Control evaluation to login routes; static assets bypass rule."
  },
  "implementation": {
    "scope": "Security / DevOps",
    "effort_estimate": "30 minutes",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Bot Control Scope-Down Statements** | 8 | $9,500.00 | $9,234.00 | 97.2% | Zero |
| **Regional Web ACL Sharing** | 12 | $1,800.00 | $1,620.00 | 90.0% | Zero |
| **WAF Log Filtering (BLOCK Only)** | 10 | $6,200.00 | $5,890.00 | 95.0% | Low |
| **IPSet Rule Consolidation** | 15 | $1,500.00 | $1,497.00 | 99.8% | Zero |
| **Total** | **45** | **$19,000.00** | **$18,241.00** | **96.0%** | -- |
