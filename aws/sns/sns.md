# AWS Service Cost Research: Amazon SNS (Simple Notification Service)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon SNS (Simple Notification Service) is a managed pub/sub messaging service designed for high-throughput, many-to-many microservice-to-microservice fanout and end-user notification delivery. SNS allows publisher applications to send messages to a topic, which are then instantly delivered to subscriber endpoints—including SQS queues, Lambda functions, HTTP/S webhooks, Mobile Push notifications (APNS, FCM), Email, and SMS text messages. SNS is billed per API request and per delivery endpoint type.

---

## 2. Billing Mechanics
1.  **Publish & API Requests:** Billed per 1 Million API requests to SNS ($0.50 per 1 Million requests).
2.  **Notification Deliveries:** Billed per delivery to subscriber endpoints, with vastly different rates based on the target protocol (HTTP/S, SQS, Email, Mobile Push, SMS).
3.  **Free Tier:** First **1 Million API requests per month** are **100% Free ($0.00)**.

---

## 3. Key Cost Dimensions

### A. Delivery Endpoint Pricing (us-east-1 Rates)
*   **HTTP / HTTPS Webhooks & SQS Fanout:** **$0.06 per 100,000 deliveries** ($0.60 per 1 Million deliveries).
*   **Mobile Push Notifications (APNS / FCM):** **$0.50 per 1 Million deliveries**.
*   **Email / Email-JSON Notifications:** **$2.00 per 100,000 deliveries** ($20.00 per 1 Million deliveries).
*   **SMS Text Messages (Telecom Carrier Charges):** Destination-based rates ($0.0065 to $0.05+ per SMS).

### B. Math Comparison
Delivering 1 Million notifications via different protocols:
*   *Mobile Push:* **$0.50 total**
*   *HTTP / SQS:* **$0.60 total**
*   *Email:* **$200.00 total**
*   *SMS (US):* **$6,500.00 total**

---

## 4. Detailed Pricing Rates (us-east-1)

| SNS Feature / Protocol | Free Tier Allowance | Delivery Rate | Price per 1 Million Deliveries |
|------------------------|---------------------|---------------|--------------------------------|
| **API Requests** | 1M requests/mo | **$0.50 / 1M requests** | **$0.50** |
| **HTTP / SQS Delivery** | 100k deliveries/mo| **$0.06 / 100k** | **$0.60** |
| **Mobile Push Delivery**| 1M deliveries/mo | **$0.50 / 1M** | **$0.50** |
| **Email Delivery** | 1k deliveries/mo | **$2.00 / 100k** | **$200.00** |

---

## 5. AWS Free Tier Coverage
*   **Amazon SNS Free Tier:** Includes **1 Million free API requests**, 100,000 free HTTP/S deliveries, 100,000 free SQS deliveries, 1 Million free Mobile Push deliveries, and 1,000 free Email deliveries per month indefinitely.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Using Email / SMS Endpoints for High-Frequency Automated Alerts:**
    *   Subscribing raw developer email addresses or SMS phone numbers directly to high-volume system topics.
    *   Delivering 100,000 automated system alert emails/month costs **$200.00/month**, while SMS costs **$650.00+/month**!

---

## 7. Actionable Cost Optimization Strategies
1.  **Fanout SNS Topics to SQS Queues ($0.60/M) for Microservices:**
    *   Subscribe Amazon SQS queues to your SNS topics.
    *   *Why:* Delivering messages from SNS to SQS costs **$0.60 per 1 Million deliveries** (compared to $200/M for Email). Worker applications can then batch-read SQS messages for free.
    *   **The Savings:** Slashes application integration messaging bills by **99.7%**.
2.  **Replace SMS with Mobile Push Notifications:**
    *   Migrate user notifications from SMS text messages to **Mobile Push Notifications (APNS/FCM)** via SNS.
    *   **The Savings:** Slashes notification delivery costs from $6,500/M down to **$0.50 per 1 Million deliveries (99.99% discount)**.
3.  **Filter Messages with SNS Subscription Filter Policies:**
    *   Configure **SNS Subscription Filter Policies** on topic subscriptions.
    *   *Why:* Ensures subscriber endpoints receive ONLY the specific messages they need, eliminating unneeded delivery executions ($0.06/100K).
