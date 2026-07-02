# AWS Service Cost Research: Amazon ELB (Elastic Load Balancing)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Elastic Load Balancing (ELB) automatically distributes incoming application traffic across multiple targets (such as EC2 instances, ECS containers, IP addresses, and Lambda functions) in one or more Availability Zones. ELB offers three primary managed load balancers: **Application Load Balancer (ALB)**, **Network Load Balancer (NLB)**, and **Gateway Load Balancer (GLB)**. ELB billing is multi-dimensional, combining a flat hourly base fee with a capacity charge based on the highest usage metric (connections, bytes, rules).

---

## 2. Billing Mechanics
Every active load balancer is billed on a monthly cycle based on two dimensions:
1.  **Load Balancer Hourly Base Fee:** A flat fee charged per hour or partial hour that a load balancer is running.
2.  **Capacity Unit Charges:** Hourly charges based on the capacity consumed, measured in:
    *   **LCU (Load Balancer Capacity Units)** for ALB.
    *   **NPU (Network Load Balancer Capacity Units)** for NLB.
    *   **GBCU (Gateway Load Balancer Capacity Units)** for GLB.
*   *The Billing Rule:* Capacity units are calculated by evaluating four different dimensions (new connections, active connections, processed bytes, and rule evaluations). You are billed **only for the single dimension with the highest usage**.

---

## 3. Key Cost Dimensions

### A. Application Load Balancer (ALB - us-east-1)
Best for HTTP/HTTPS web application routing.
*   **Hourly Base Fee:** **$0.0252 per hour** (~$18.40/month).
*   **LCU Hourly Fee:** **$0.008 per LCU-hour**.
*   **1 LCU** represents the highest value of:
    *   *New Connections:* Up to **25** new connections per second.
    *   *Active Connections:* Up to **3,000** active connections per minute.
    *   *Processed Bytes:* Up to **1 GB** for targets on EC2/containers, or **0.4 GB** for Lambda targets.
    *   *Rule Evaluations:* Up to **1,000** rules evaluated (Number of rules processed × request rate).

### B. Network Load Balancer (NLB - us-east-1)
Best for high-performance, ultra-low latency TCP/UDP routing.
*   **Hourly Base Fee:** **$0.0225 per hour** (~$16.42/month).
*   **NPU Hourly Fee:** **$0.006 per NPU-hour** (25% cheaper capacity fee than ALB).
*   **1 NPU** represents the highest value of:
    *   *New Connections/Flows:* Up to **800** new connections per second.
    *   *Active Connections/Flows:* Up to **100,000** active connections per minute.
    *   *Processed Bytes:* Up to **1 GB** processed.

### C. Gateway Load Balancer (GLB - us-east-1)
Best for deploying virtual firewalls or security appliances.
*   **Hourly Base Fee:** **$0.0125 per hour** (~$9.12/month).
*   **GBCU Hourly Fee:** **$0.004 per GBCU-hour**.
*   **1 GBCU** represents the highest value of:
    *   *New Connections/Flows:* Up to **600** new connections per second.
    *   *Active Connections/Flows:* Up to **20,000** active connections per minute.
    *   *Processed Bytes:* Up to **0.4 GB** processed.

---

## 4. Detailed Pricing Rates (us-east-1)

| Load Balancer Type | Hourly Base Rate | Capacity Unit Rate | Baseline Monthly Base Cost |
|--------------------|------------------|---------------------|---------------------------|
| **ALB (Application)** | **$0.0252 / hour** | **$0.0080 / LCU-hour**| ~$18.40 / month |
| **NLB (Network)** | **$0.0225 / hour** | **$0.0060 / NPU-hour**| ~$16.42 / month |
| **GLB (Gateway)** | **$0.0125 / hour** | **$0.0040 / GBCU-hour**| ~$9.12 / month |

---

## 5. AWS Free Tier Coverage
*   **ALB:** 750 hours/month shared with 15 LCUs (new accounts, first 12 months).
*   **NLB:** 750 hours/month shared with 15 NPUs (new accounts, first 12 months).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Rule Evaluation Storms (ALB):** Creating complex path-based or header-based routing rules (e.g., 50 rules) on an ALB that processes millions of requests. The rule evaluation dimension (rules × requests) can scale LCU counts rapidly.
    *   *Example:* If you have 80 rules and process 5,000 requests/second, you evaluate 400,000 rules/sec, which requires **400 LCUs**, costing **$2,304.00/month** in rule evaluation capacity alone!
*   **Unused Load Balancers:** Leaving idle ALBs or NLBs running in dev/staging accounts. Each idle load balancer costs ~$16–$18/month in flat base fees plus IPv4 address charges ($3.65/month per IP).
*   **Lambda Targets on ALB:** Routing high-volume web requests to Lambda functions via ALB. Processed bytes are metered at **0.4 GB per LCU** (compared to 1.0 GB for EC2), which multiplies LCU charges by 2.5x.

---

## 7. Actionable Cost Optimization Strategies
1.  **Optimize ALB Routing Rules (Avoid Rule Storms):** 
    *   Minimize the number of listener rules configured on your ALB.
    *   Move complex routing logic (e.g. host redirects or path rewrites) to your application code or use **CloudFront Functions** at the edge to prevent ALB rule evaluation LCU charges.
2.  **Delete Idle/Orphaned Load Balancers:** Set up AWS Config rules or scripts to identify load balancers with `ActiveConnectionCount = 0` over the last 14 days and delete them.
3.  **Consolidate Applications onto Shared ALBs:** Instead of deploying a separate ALB for every microservice, use a single shared ALB with **Host-Based Routing** (using wildcards or subdomains) or **Path-Based Routing** to consolidate up to 100 applications under one base hourly charge.
4.  **Use NLB for Non-HTTP TCP Traffic:** If your application only needs simple TCP port forwarding (without TLS termination or HTTP header inspection), deploy an **NLB**. NLB baseline hourly fees and capacity units ($0.006/NPU vs. $0.008/LCU) are cheaper, and NLBs handle massive traffic volumes far more cost-effectively.
5.  **Use API Gateway HTTP APIs for Lambda Routing:** If you are routing traffic to AWS Lambda, use **API Gateway HTTP APIs** instead of an ALB. This avoids both the flat ALB hourly base fee and the 2.5x Lambda target LCU processed bytes penalty.
