# Cost-Cutting Playbook: Amazon ELB (Elastic Load Balancing)

> **Companion File:** [elb.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/elb/elb.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon Elastic Load Balancing (ELB) distributes application traffic across targets using **Application Load Balancer (ALB)**, **Network Load Balancer (NLB)**, or **Gateway Load Balancer (GLB)**. Billing combines a flat hourly base fee ($0.0252/hr = **$18.40/mo per ALB**) with Capacity Units (LCUs for ALB at $0.008/LCU-hr; NPUs for NLB at $0.006/NPU-hr).

Key billing traps include:
1. **Unused / Idle Load Balancers:** Leaving idle load balancers running in dev/staging accounts ($18.40/mo base fee + $3.65/mo per assigned public IPv4 address = **$22.05/mo per idle ALB**).
2. **ALB Rule Evaluation Storms:** Configuring hundreds of path/header routing rules on ALBs processing high request rates (1 LCU per 1,000 rule evaluations/sec), which can generate **$1,000s/month in unexpected LCU capacity surcharges**.
3. **Lambda Target Processed Bytes Tax:** Routing traffic to Lambda targets via ALB, where processed bytes are metered at **0.4 GB per LCU** (2.5x more expensive than EC2 targets).

This playbook provides **18 actionable strategies** across six operational categories, delivering an estimated **30–70% reduction in total ELB spend**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Delete Idle / Orphaned Load Balancers (`ActiveConnectionCount = 0`):** Reclaims $18.40/mo base fee + $3.65/mo IPv4 tax per idle load balancer ($22.05/mo total).
2. **Consolidate Microservices onto Shared ALBs (Host/Path Routing):** Consolidate 10 separate application ALBs onto 1 shared ALB to eliminate 9 flat hourly base fees (**saving $165.60/month**).
3. **Migrate Lambda Routing from ALB to API Gateway HTTP APIs:** Eliminates flat ALB base fees ($18.40/mo) and bypasses the 2.5x Lambda target LCU processed bytes penalty.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Load Balancers)

#### 1. Decommission Idle & Orphaned Load Balancers
- **What:** Identify and delete ALBs, NLBs, and GLBs with `ActiveConnectionCount = 0` over the last 14 days.
- **Why It Saves Money:** Every idle ALB costs **$18.40/month** in base hourly fees plus **$3.65/month** for every assigned public IPv4 address ($22.05/mo per idle load balancer). 20 idle dev load balancers waste **$441.00/month**.
- **Detailed Implementation Steps:**
  1. Audit active connections via CloudWatch across all load balancers.
  2. List load balancers with zero connections:
     ```bash
     aws elbv2 describe-load-balancers --query "LoadBalancers[*].{ARN:LoadBalancerArn,Name:LoadBalancerName,DNS:DNSName}"
     ```
  3. Delete orphaned load balancer:
     ```bash
     aws elbv2 delete-load-balancer --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/idle-alb/12345
     ```
- **Estimated Savings:** **$22.05 per month saved** per idle load balancer.
- **Risk Level:** Zero risk (connection count verified = 0).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** 14-day CloudWatch metrics verification.

#### 2. Release Unused Target Groups & Listener Rules
- **What:** Delete orphaned Target Groups containing 0 registered targets (`UnHealthyHostCount` or `HealthyHostCount = 0`).
- **Why It Saves Money:** Reclaims administrative hygiene and prevents rule evaluation clutter.
- **Detailed Implementation Steps:**
  1. Delete target group: `aws elbv2 delete-target-group --target-group-arn arn-xxx`.
- **Estimated Savings:** Infrastructure cleanliness.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Target group audit.

---

### 2. Architecture & Load Balancer Consolidation

#### 3. Consolidate Microservices onto Shared ALBs (Host & Path Routing)
- **What:** Replace individual dedicated ALBs provisioned per microservice with a **Single Shared ALB** utilizing **Host-Based Routing** (`HostHeader`) or **Path-Based Routing** (`PathPattern`).
- **Why It Saves Money:** Running 20 dedicated ALBs for 20 microservices costs 20 × $18.40 = **$368.00/month** in flat base fees alone. Consolidating 20 applications onto 1 shared ALB costs **$18.40/month** — an immediate **$349.60/month savings** (95% base fee reduction)!
- **Detailed Implementation Steps:**
  1. Add listener rules on central shared ALB in Terraform:
     ```hcl
     resource "aws_lb_listener_rule" "service_a" {
       listener_arn = aws_lb_listener.front_end.arn
       priority     = 10
       action {
         type             = "forward"
         target_group_arn = aws_lb_target_group.service_a.arn
       }
       condition {
         host_header {
           values = ["service-a.company.com"]
         }
       }
     }
     ```
  2. Point DNS records to shared ALB, then terminate single-service ALBs.
- **Estimated Savings:** **95% reduction** in ALB flat hourly base fees.
- **Risk Level:** Low.
- **Implementation Scope:** Infrastructure Engineer / DevOps
- **Prerequisites:** Host/Path routing architecture alignment.

#### 4. Migrate Lambda Target Routing from ALB to API Gateway HTTP APIs
- **What:** Re-route web traffic targeting AWS Lambda functions from Application Load Balancers to **API Gateway HTTP APIs**.
- **Why It Saves Money:**
  1. ALB charges a flat $18.40/mo base fee. API Gateway HTTP APIs have **zero flat base fee** ($0.00).
  2. ALB meters Lambda target processed bytes at **0.4 GB per LCU** (2.5x more expensive capacity unit rating than EC2 targets).
  3. HTTP APIs cost flat $1.00/M requests with no processed bytes capacity penalty.
- **Detailed Implementation Steps:**
  1. Create HTTP API with Lambda Integration in Terraform.
  2. Update CNAME record to point to HTTP API domain endpoint.
- **Estimated Savings:** **60–85% overall cost reduction** for Lambda web endpoints.
- **Risk Level:** Low.
- **Implementation Scope:** Software Engineer / DevOps
- **Prerequisites:** HTTP API feature parity verification.

#### 5. Use Network Load Balancers (NLB) for Simple TCP/UDP Workloads
- **What:** Replace ALBs with **Network Load Balancers (NLBs)** for applications that perform simple TCP/UDP port forwarding without HTTP header inspection or TLS termination requirements.
- **Why It Saves Money:** NLB capacity units (NPUs at **$0.006/NPU-hr**) are **25% cheaper** than ALB capacity units (LCUs at **$0.008/LCU-hr**), and NLBs handle 800 new connections/sec per NPU (vs 25 new connections/sec per LCU on ALB — a **32x connection capacity efficiency**!).
- **Detailed Implementation Steps:**
  1. Deploy NLB target group and listener on TCP port 443/80.
- **Estimated Savings:** 25–60% capacity fee reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Infrastructure Engineer
- **Prerequisites:** No HTTP layer 7 routing rules required.

---

### 3. LCU & Rule Evaluation Optimization

#### 6. Optimize ALB Listener Rules (Prevent Rule Evaluation Storms)
- **What:** Minimize the number of listener rules evaluated per request on high-traffic ALBs. Move static URL redirects, header rewrites, or complex path filtering to **CloudFront Functions** at the edge or within application code.
- **Why It Saves Money:** ALB calculates LCUs based on `RuleEvaluations` = (Number of rules processed × request rate). Processing 80 rules across 5,000 requests/sec requires **400 LCUs**, costing **$3.20/hour = $2,336.00/month** in LCU capacity surcharges! Moving rules to CloudFront drops LCU count to 1 LCU ($0.008/hr = $5.84/mo).
- **Detailed Implementation Steps:**
  1. Audit ALB CloudWatch metric `RuleEvaluations`.
  2. Order listener rules so high-volume default routes match early in evaluation priority.
  3. Move URL redirects to CloudFront Functions.
- **Estimated Savings:** **80–99% reduction** in rule evaluation LCU charges ($1,000s/mo saved on high-traffic ALBs).
- **Risk Level:** Low.
- **Implementation Scope:** DevOps / Web Developer
- **Prerequisites:** CloudFront distribution fronting ALB.

#### 7. Enable Connection Keep-Alive to Reduce New Connection LCUs
- **What:** Configure HTTP client applications and CloudFront origins to use persistent **HTTP Keep-Alive connections** (`Connection: keep-alive`).
- **Why It Saves Money:** 1 LCU allows 25 new connections/sec, but allows **3,000 active connections/sec**. Reusing existing connections reduces the `NewConnections` LCU metric by 99%.
- **Detailed Implementation Steps:**
  1. Set Keep-Alive timeout on upstream origins and client SDKs to 60+ seconds.
- **Estimated Savings:** 50–80% reduction in new connection LCU charges.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Software Engineer
- **Prerequisites:** Client/origin HTTP Keep-Alive support.

---

### 4. Cross-AZ & Data Egress Optimization

#### 8. Enable Cross-Zone Load Balancing Alignments
- **What:** Review Cross-Zone Load Balancing settings on NLBs based on target AZ distribution.
- **Why It Saves Money:** On NLB, Cross-Zone Load Balancing charges standard cross-AZ data transfer fees (**$0.01/GB**). On ALB, cross-zone load balancing is included for free. If targets are evenly distributed across AZs, disabling NLB cross-zone load balancing saves $0.01/GB on all cross-AZ target traffic.
- **Detailed Implementation Steps:**
  1. Disable cross-zone load balancing on evenly balanced NLBs:
     ```bash
     aws elbv2 modify-load-balancer-attributes \
       --load-balancer-arn arn-nlb-xxx \
       --attributes Key=load_balancing.cross_zone.enabled,Value=false
     ```
- **Estimated Savings:** $10.00 per 1 TB of NLB traffic.
- **Risk Level:** Low (targets must be balanced evenly across AZs).
- **Implementation Scope:** Infrastructure Engineer
- **Prerequisites:** Multi-AZ target symmetry.

#### 9. Offload Data Bandwidth to CloudFront CDN
- **What:** Place Amazon CloudFront in front of public ALBs.
- **Why It Saves Money:** CloudFront data egress rates ($0.085/GB) are cheaper than direct AWS internet egress ($0.09/GB), and edge caching offloads processed bytes from reaching the ALB.
- **Detailed Implementation Steps:**
  1. Create CloudFront distribution with ALB origin.
- **Estimated Savings:** 15–40% bandwidth cost reduction.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CloudFront deployment.

---

### 5. IPv4 & Public Address Cost Control

#### 10. Deploy Private-Facing Load Balancers inside VPCs
- **What:** Set load balancer scheme to `internal` (private IP only) for internal service-to-service microservices.
- **Why It Saves Money:** Prevents allocating public IPv4 addresses, avoiding the **$0.005/hour ($3.65/month per IP)** public IPv4 surcharge on internal load balancers.
- **Detailed Implementation Steps:**
  1. Set `scheme = "internal"` in Terraform load balancer definition.
- **Estimated Savings:** $3.65/mo per IP saved across private subnets.
- **Risk Level:** Zero risk (internal services do not require public internet access).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC routing configuration.

#### 11. Adopt Dualstack (IPv6) Enabled Load Balancers
- **What:** Convert public load balancers to `dualstack` mode, supporting native IPv6 client traffic.
- **Why It Saves Money:** Prepares infrastructure for shifting away from expensive public IPv4 addresses ($3.65/mo per IP).
- **Detailed Implementation Steps:**
  1. Set ip_address_type = `dualstack`.
- **Estimated Savings:** Future-proofs against public IPv4 address price increases.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Subnet IPv6 CIDR blocks attached.

---

### 6. Observability, Security & Governance

#### 12. Disable ALB Access Logs on Non-Production Load Balancers
- **What:** Turn off S3 Access Logging on dev/staging ALBs.
- **Why It Saves Money:** Prevents accumulating millions of minor S3 log files and S3 PUT request fees ($0.005/1k) on non-prod environments.
- **Detailed Implementation Steps:**
  1. Disable access logs attribute:
     ```bash
     aws elbv2 modify-load-balancer-attributes \
       --load-balancer-arn arn-xxx \
       --attributes Key=access_logs.s3.enabled,Value=false
     ```
- **Estimated Savings:** Eliminates non-prod S3 logging costs.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Audit requirement check.

#### 13. Optimize AWS WAF Integration on ALBs
- **What:** Attach AWS WAF Web ACLs at the CloudFront edge rather than directly on the ALB where possible.
- **Why It Saves Money:** Blocks malicious traffic at the edge before it consumes ALB LCU processing capacity.
- **Detailed Implementation Steps:**
  1. Move WAF Web ACL association to CloudFront distribution.
- **Estimated Savings:** Protects against ALB LCU capacity spikes during bot floods.
- **Risk Level:** Low.
- **Implementation Scope:** Security / DevOps
- **Prerequisites:** CloudFront fronting ALB.

#### 14. Standardize Target Group Health Check Intervals
- **What:** Increase target group health check intervals from 10 seconds to 30 seconds.
- **Why It Saves Money:** Reduces internal health check request chatter and target CPU overhead by 66%.
- **Detailed Implementation Steps:**
  1. Set `HealthCheckIntervalSeconds = 30` in target group attributes.
- **Estimated Savings:** Reduces target background CPU load.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 15. Enforce CloudWatch Alarms for High LCU Billing Metrics
- **What:** Set CloudWatch alarm on `ConsumedLCUs` (> 10 LCUs).
- **Why It Saves Money:** Provides instant alert before sudden application traffic spikes generate multi-thousand dollar LCU bills.
- **Detailed Implementation Steps:**
  1. Put CloudWatch metric alarm on `AWS/ApplicationELB` `ConsumedLCUs`.
- **Estimated Savings:** Proactive billing risk protection.
- **Risk Level:** Zero risk.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** SNS topic setup.

#### 16. Implement Automated Cleanup Scripts for Branch Testing ALBs
- **What:** Auto-delete temporary feature-branch ALBs created by CI/CD pipelines when pull requests are merged/closed.
- **Why It Saves Money:** Prevents forgotten PR test ALBs from billing $18.40/mo base fees indefinitely.
- **Detailed Implementation Steps:**
  1. Add `terraform destroy` or `aws elbv2 delete-load-balancer` step in GitHub Actions / GitLab CI cleanup workflow.
- **Estimated Savings:** 100% of abandoned PR test load balancer costs.
- **Risk Level:** Zero risk.
- **Implementation Scope:** DevOps Engineer
- **Prerequisites:** CI/CD cleanup pipeline integration.

#### 17. Leverage Free Tier Load Balancer Allocations
- **What:** Verify 750 free monthly ALB/NLB hours for new accounts (first 12 months).
- **Why It Saves Money:** Maximize initial free tier credits.
- **Detailed Implementation Steps:**
  1. Track free tier hours in Billing Console.
- **Estimated Savings:** Free baseline testing.
- **Risk Level:** Zero.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** None.

#### 18. Audit De-registered Target Deregistration Delays
- **What:** Reduce target group deregistration delay (connection draining timeout) from 300 seconds down to 30 seconds for stateless microservices.
- **Why It Saves Money:** Accelerates auto-scaling down-scaling speed, terminating excess EC2/container instances 4.5 minutes faster.
- **Detailed Implementation Steps:**
  1. Set `deregistration_delay.timeout_seconds = 30` in target group attributes.
- **Estimated Savings:** Faster EC2/container capacity termination.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Target application request duration < 30 seconds.

---

## Cross-Service Synergies

```
[ Application Load Balancer ] 
        │
        ├──(ALB Consolidation)─> [ Shared Host/Path Routing ] (Saves $165.60/mo per 10 consolidated ALBs)
        │
        ├──(Lambda Routing)────> [ API Gateway HTTP APIs ] (Bypasses flat ALB base fee & 2.5x LCU tax)
        │
        └──(Rule Offloading)───> [ CloudFront Functions ] (Saves $1,000s/mo on Rule Evaluation LCUs)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `LoadBalancerUsage`, `LCUUsage`, `NPUUsage`, `PublicIPv4:InUseAddress`.
- `line_item_resource_id`: ELB Load Balancer ARN (`arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/xxx/123`).

### B. CloudWatch Metrics
- `AWS/ApplicationELB` Namespace: `ActiveConnectionCount`, `NewConnectionCount`, `ProcessedBytes`, `RuleEvaluations`, `ConsumedLCUs`, `TargetResponseTime`.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "ELB-CNS-001",
  "service": "ELB",
  "category": "Architecture & Load Balancer Consolidation",
  "resource_id": "arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/microservice-auth/a1b2c3d4",
  "resource_name": "microservice-auth-alb",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "deployment": "Dedicated ALB per microservice (15 total ALBs)",
    "monthly_base_fee_usd": 276.00,
    "ipv4_address_fee_usd": 54.75,
    "monthly_cost_usd": 330.75
  },
  "recommended_config": {
    "deployment": "Consolidate 15 microservices onto 1 Shared Host-Routed ALB",
    "projected_monthly_cost_usd": 22.05
  },
  "financial_impact": {
    "monthly_savings_usd": 308.70,
    "annual_savings_usd": 3704.40,
    "savings_percentage": 93.3
  },
  "risk_assessment": {
    "risk_level": "Low",
    "reason": "Microservices share single ALB using Host-Header routing; zero application downtime."
  },
  "implementation": {
    "scope": "Infrastructure Engineer / DevOps",
    "effort_estimate": "2-3 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **ALB Consolidation (Host/Path)** | 12 | $4,800.00 | $4,478.40 | 93.3% | Low |
| **Idle ALB Decommissioning** | 15 | $330.75 | $330.75 | 100.0% | Zero |
| **Rule Evaluation LCU Tuning** | 5 | $8,500.00 | $7,650.00 | 90.0% | Low |
| **Lambda Routing -> HTTP APIs** | 8 | $3,200.00 | $2,400.00 | 75.0% | Low |
| **Total** | **40** | **$16,830.75** | **$14,859.15** | **88.3%** | -- |
