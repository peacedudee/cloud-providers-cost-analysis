# AWS Service Cost Research: AWS Shield

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Shield is a managed Distributed Denial of Service (DDoS) protection service that guards applications running on AWS. Shield provides dynamic threat detection and automatic inline mitigations that minimize application downtime and latency. AWS Shield is available in two tiers: **Shield Standard** (free automatic baseline defense) and **Shield Advanced** (enterprise managed DDoS protection with financial cost safeguards).

---

## 2. Billing Mechanics
1.  **Shield Standard:** **100% Free ($0.00)**. Automatically active across all AWS accounts and services.
2.  **Shield Advanced:**
    *   **Flat Monthly Subscription:** **$3,000.00 per month** per organization (managed via AWS Organizations).
    *   **Data Transfer Out (DTO) Fee:** Billed per GB of data transfer out from protected resources (CloudFront, ALB, NLB, Elastic IP, Route 53).
    *   **Contract Commitment:** Requires a 1-year subscription commitment.

---

## 3. Key Cost Dimensions

### A. Shield Standard vs. Shield Advanced (us-east-1)
*   **Shield Standard:**
    *   *Cost:* **Free ($0.00)**.
    *   *Protection:* Defends against common Layer 3 and Layer 4 infrastructure attacks (SYN floods, UDP reflection, ICMP floods) at the AWS border.
*   **Shield Advanced:**
    *   *Base Subscription:* **$3,000.00 per month** (consolidated across all linked accounts in AWS Organizations).
    *   *Protected Data Transfer Out (DTO) Rates:*
        *   *First 100 TB DTO / month:* **$0.050 per GB**.
        *   *Next 400 TB DTO / month:* **$0.030 per GB**.
        *   *Next 500 TB DTO / month:* **$0.025 per GB**.

### B. Built-in Cost Bundles with Shield Advanced
Subscribing to Shield Advanced includes several financial credits that offset other AWS bills:
1.  **Free AWS WAF Usage:** WAF Web ACL fees ($5.00/mo), Rule fees ($1.00/mo), and WAF request processing fees ($0.60/M) are **100% waived** for resources protected by Shield Advanced.
2.  **DDoS Cost Protection (Financial Safeguard):** If a DDoS attack causes your EC2 instances, ALBs, CloudFront distributions, or EKS clusters to auto-scale out, AWS provides service credits to cover the spike in compute/bandwidth bills incurred during the attack.

---

## 4. Detailed Pricing Rates (us-east-1)

| Shield Tier | Base Monthly Fee | Protected Data Transfer Out (First 100 TB)| Included Benefits |
|-------------|------------------|-------------------------------------------|-------------------|
| **Shield Standard**| **Free ($0.00)** | Standard AWS Egress Rates | L3/L4 DDoS protection |
| **Shield Advanced**| **$3,000.00** | **$0.050 / GB** | L3/L4/L7 defense, Free WAF, Cost Reimbursement |

---

## 5. AWS Free Tier Coverage
*   **Shield Standard:** Always 100% free for all AWS accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Subscribing to Shield Advanced for Small Web Applications:** Subscribing to Shield Advanced ($3,000/mo flat) for small or medium workloads where Shield Standard + CloudFront + WAF ($5–$30/mo) provides 99.9% of required security.
*   **Multiple Individual Subscriptions:** Subscribing to Shield Advanced in multiple standalone AWS accounts without consolidating accounts under **AWS Organizations**, incurring $3,000/mo *per account* instead of $3,000/mo *per organization*.

---

## 7. Actionable Cost Optimization Strategies
1.  **Rely on Shield Standard + CloudFront + WAF Rate Rules (For 99% of Apps):**
    *   Instead of spending $3,6000/year on Shield Advanced, combine:
        *   **Shield Standard ($0.00)** for network layer attack absorption.
        *   **Amazon CloudFront** for global edge capacity and IP masking.
        *   **AWS WAF Rate-Based Rules ($1.00/rule)** to automatically block client IPs making >2,000 requests/5 min.
    *   **The Savings:** Provides enterprise-grade Layer 7 DDoS mitigation for less than **$50.00/month**.
2.  **Consolidate Accounts under AWS Organizations for Shield Advanced:**
    *   If your enterprise requires Shield Advanced (for 24/7 SRT access or compliance), enable it through an **AWS Organizations Master Account**.
    *   The single **$3,000.00/month** subscription fee automatically covers all linked child accounts across the entire organization.
3.  **Claim DDoS Cost Protection Credits After Attacks:** If you are a Shield Advanced subscriber and experience a DDoS attack that triggers auto-scaling compute bills, immediately submit a ticket to AWS Support to request **DDoS Cost Protection credits** to reimburse the unexpected instance fees.
