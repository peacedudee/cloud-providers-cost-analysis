# AWS Service Cost Research: AWS Certificate Manager (ACM) & AWS Private CA

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Certificate Manager (ACM) simplifies the provisioning, management, and deployment of public and private Secure Sockets Layer/Transport Layer Security (SSL/TLS) certificates for use with AWS services and internal connected resources. ACM automates domain validation and certificate renewals. 

* **Public Certificates (Integrated AWS Services):** 100% Free for integrated AWS services (e.g., CloudFront, Application Load Balancers, API Gateway).
* **Exportable Public Certificates:** Issued by ACM for deployment on non-integrated endpoints, EC2 instances, containers, or on-premises servers (billed per certificate).
* **AWS Private Certificate Authority (AWS Private CA):** A dedicated service for creating private CA hierarchies to issue internal microservices mTLS, IoT, and VPN certificates (billed via a monthly base CA fee + per-certificate issuance fees).

---

## 2. Billing Mechanics
1. **Public SSL/TLS Certificates (Integrated Services):** **100% Free ($0.00)**. Zero charge for provisioning, deploying, or renewing certificates attached natively to AWS services.
2. **Exportable Public Certificates:** Charged a flat issuance/renewal fee based on domain structure (Fully Qualified Domain Name vs. Wildcard). Certificates have a 198-day validity period and trigger automated renewal notifications via Amazon EventBridge 45 days prior to expiration.
3. **AWS Private Certificate Authority (AWS Private CA):** Billed hourly per provisioned Private CA instance, plus tiered volume fees for each private certificate issued.

---

## 3. Key Cost Dimensions

### A. Public Certificates (us-east-1 Rates)
* **Integrated AWS Services (CloudFront, ALB, NLB, API Gateway, App Runner):**
  * *Cost:* **100% Free ($0.00)**.
  * *Includes:* Single domain, Multi-Domain (SAN), and Wildcard certificates (`*.yourdomain.com`). Includes automatic managed domain validation renewals.
* **Exportable Public Certificates (On-Premises / EC2 / Containers):**
  * *Fully Qualified Domain Name (FQDN):* **$7.00 per certificate**.
  * *Wildcard Certificate:* **$79.00 per certificate**.
  * *Validity:* 198 days (ACM handles automated re-issuance and sends EventBridge events for manual/automated external deployment).

### B. AWS Private CA (Private Certificate Authority)
Used for internal microservices mTLS, IoT devices, or VPN authentication.
* **Private CA Monthly Base Fee:** **$400.00 per month** per active Private CA (prorated hourly at ~$0.548/hour).
* **Private Certificate Issuance Fees:**
  * *First 1,000 private certificates/month:* **$0.75 per certificate**.
  * *Next 9,000 private certificates/month:* **$0.35 per certificate**.
  * *Over 10,000 private certificates/month:* **$0.001 per certificate**.

---

## 4. Detailed Pricing Rates (us-east-1)

| ACM Certificate Type | Target Environment | Monthly Base Fee | Issuance / Renewal Fee |
|----------------------|--------------------|------------------|------------------------|
| **Public ACM Cert (Integrated)** | CloudFront / ALB / API Gateway | **Free ($0.00)** | **Free ($0.00)** |
| **Exportable Public (FQDN)** | On-Premises / External / EC2 | **Free ($0.00)** | **$7.00 per cert** |
| **Exportable Public (Wildcard)** | On-Premises / External / EC2 | **Free ($0.00)** | **$79.00 per cert** |
| **Private CA Instance** | Internal Mesh / mTLS / VPN | **$400.00 / month** | $0.75 / cert (1st 1K)<br>$0.35 / cert (next 9K)<br>$0.001 / cert (>10K) |

---

## 5. AWS Free Tier Coverage
* **Integrated Public ACM Certificates:** Always 100% free for all integrated AWS services.
* **AWS Private CA:** 30-day free trial for your first Private CA.

---

## 6. Common Cost Hotspots & Pitfalls
* **Provisioning an AWS Private CA for Development Testing:** Creating an AWS Private CA ($400.00/month flat base charge) in developer sandbox environments to issue self-signed internal test certificates.
* **Buying Paid Commercial SSL Certificates Elsewhere:** Purchasing commercial SSL certificates from third-party registrars (e.g., DigiCert or Sectigo for $100–$300/year) and importing them into AWS, when free integrated ACM certificates ($0.00) or low-cost exportable ACM certificates could be used.
* **Unused Active Private CAs:** Leaving test or completed migration Private CAs active in AWS accounts, accumulating $400.00/month per CA continuously.

---

## 7. Actionable Cost Optimization Strategies
1. **Terminate TLS at Integrated AWS Services (CloudFront / ALB):**
   * Attach **Free ACM Public Certificates ($0.00)** to CloudFront distributions, Application Load Balancers, or API Gateways.
   * *Benefit:* 100% free certificate generation, zero maintenance, and automatic managed domain validation renewals.
2. **Avoid AWS Private CA in Non-Production Environments:**
   * Do not deploy AWS Private CA ($400.00/month) for non-production testing environments.
   * Use open-source tools (such as **Let's Encrypt**, **mkcert**, or **step-ca**) or free integrated ACM public certificates with staging subdomains (`staging.yourdomain.com`).
   * **The Savings:** Saves $400.00/month per dev environment ($4,800/year).
3. **Delete Inactive Private CAs:** Audit your AWS accounts for idle Private CAs. If an internal migration project has finished, delete the Private CA to immediately stop the **$400.00/month recurring base charge**.
