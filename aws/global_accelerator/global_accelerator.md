# AWS Service Cost Research: AWS Global Accelerator

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Global Accelerator is a networking service that improves the availability, security, and performance of your local or global applications by routing user traffic over AWS's high-speed, congestion-free global network backbone. It provides static IP addresses that act as fixed entry points to your application endpoints (like ALBs, NLBs, or EC2 instances) in multiple regions. Global Accelerator bills on a hybrid model that combines a flat hourly fee with an incremental Data Transfer-Premium (DT-Premium) rate.

---

## 2. Billing Mechanics
Global Accelerator charges are billed on a monthly cycle based on two dimensions:
1.  **Fixed Hourly Fee:** Billed per hour or partial hour for every provisioned accelerator (standard or custom routing).
2.  **Data Transfer-Premium (DT-Premium) Fee:** Billed per GB of data transferred over the AWS global network. This fee is calculated only in the **dominant direction of traffic** (either inbound or outbound, whichever is higher) in any given hour.

---

## 3. Key Cost Dimensions

### A. Fixed Hourly Fee (us-east-1)
*   **The Rate:** **$0.025 per hour** (~$18.00/month) per accelerator.
*   **Idle Penalty:** Billed continuously from the moment the accelerator is provisioned until it is deleted, even if it is disabled or processes zero traffic.

### B. Data Transfer-Premium (DT-Premium)
*   **The Model:** This charge is a premium added *on top* of standard AWS data transfer egress charges from your origin resources (like EC2 or ALB).
*   **Dominant Direction Rule:** You are billed only for the traffic flow direction (ingress vs. egress) that is largest.
*   **Geographic Rates:** Billed per GB depending on the source client location and the destination AWS region:
    *   *US / Canada / Europe:* **$0.015 per GB**.
    *   *Asia Pacific / Japan:* **$0.035 to $0.045 per GB**.
    *   *South America / Africa:* **$0.050 per GB**.

---

## 4. Detailed Pricing Rates (us-east-1 Base Origin)

| Cost Component | Base Rate | Unit / Details |
|----------------|-----------|----------------|
| **Fixed Hourly Fee** | **$0.025 / hour** | ~$18.00 / month per accelerator |
| **DT-Premium (US/Europe)**| **$0.015 / GB** | In addition to S3/EC2 egress |
| **DT-Premium (Asia)** | **$0.035 / GB** | Dominant direction only |
| **DT-Premium (South America)**| **$0.050 / GB**| Dominant direction only |

### Example Monthly Cost Calculation (Dominant Direction Rule)
*Workload: A web application in us-east-1 is connected via Global Accelerator. Over the month, US and European users upload 200 GB of images (inbound) and download 800 GB of generated reports (outbound).*

*   **Dominant Direction Assessment:**
    *   Inbound: 200 GB
    *   Outbound: 800 GB
    *   *Result:* Outbound is the dominant direction.
*   **DT-Premium Calculation (at $0.015/GB for US/Europe):**
    $$\text{Premium Cost} = 800\text{ GB} \times \$0.015/\text{GB} = \$12.00$$
*   **Fixed Accelerator Fee:**
    $$\text{Fixed Fee} = 730\text{ hours} \times \$0.025 = \$18.25$$
*   **Total Monthly Cost:** **$30.25/month** (Inbound DT-Premium is $0.00 under the dominant direction rule).

---

## 5. AWS Free Tier Coverage
*   **AWS Global Accelerator:** No free tier is available. Standard billing applies immediately upon provisioning.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Leaving Idle/Disabled Accelerators Provisioned:** Creating an accelerator for deployment testing, disabling it, but forgetting to delete it. Disabled accelerators still bill at the full **$18.00/month** base rate.
*   **Routing High-Volume Static Assets:** Routing massive static file downloads through the Global Accelerator. Since static files do not benefit from dynamic routing or TCP acceleration (unlike APIs or dynamic web app portals), paying the extra $0.015/GB premium is a waste of budget.

---

## 7. Actionable Cost Optimization Strategies
1.  **Delete Inactive Accelerators:**
    *   Audit the Global Accelerator console. Identify and delete any accelerators that are disabled or show `0` traffic on CloudWatch.
2.  **Separate Static and Dynamic Traffic (CloudFront Bypass):**
    *   Do not route your entire application domain through Global Accelerator. Route static assets (images, stylesheets, media) through **Amazon CloudFront** (saving the $0.015/GB accelerator premium). Route only dynamic API calls and transactional endpoints through **Global Accelerator**.
3.  **Evaluate Site-to-Site VPN Acceleration:**
    *   If you use Accelerated Site-to-Site VPN, review the usage metrics. If the VPN does not experience latency-related throughput drops, disable the "Accelerated" flag to stop paying the Global Accelerator hourly and premium charges.
4.  **Use Regional ALBs First:**
    *   If your users are mostly located in a single region, route them directly to a local ALB via Route 53 latency records, bypassing Global Accelerator entirely.
