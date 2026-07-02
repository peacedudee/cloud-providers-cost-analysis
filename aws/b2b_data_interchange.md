# AWS Service Cost Research: AWS B2B Data Interchange

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS B2B Data Interchange is a fully managed Electronic Data Interchange (EDI) service that automates the transformation of EDI documents (such as X12 EDI transactions used in supply chain, logistics, retail, and healthcare) into JSON or XML formats. It replaces legacy on-premises EDI translation software, specialized third-party Value-Added Network (VAN) systems, or custom parsing scripts. B2B Data Interchange is serverless, billing based on EDI documents transformed.

---

## 2. Billing Mechanics
1. **Document Transformation Fee:** Billed per EDI document transformed into JSON or XML formats ($0.01 to $0.15 per document).
2. **No Baseline Subscriptions:** Serverless pay-as-you-go pricing with no monthly subscription fees or minimum usage commitments.
3. **Underlying Storage & Event Messaging:** Standard Amazon S3 storage fees apply for storing raw and transformed EDI files, and EventBridge charges apply for transformation event notifications.

---

## 3. Key Cost Dimensions

### A. Document Transformation Pricing (us-east-1)
* **Standard Document Transformation (Default JSON):** **$0.01 per document transformed**.
* **Custom Schema Transformation (Custom Mapping):** **$0.15 per document transformed**.

### B. Mathematical Cost Calculation Example
*Scenario: A logistics company processes 50,000 inbound EDI 850 (Purchase Orders) per month using standard JSON transformation.*

1. **Transformation Fee:**
   $$\text{Transformation Cost} = 50,000\text{ documents} \times \$0.01 = \$500.00\text{ / month}$$
2. **Total Monthly Cost:** **$500.00 / month** (excluding minor S3 storage fees).

---

## 4. Detailed Pricing Rates (us-east-1)

| Feature / Operation | Billed Unit | Rate (us-east-1) | Price for 10,000 EDI Documents |
|---------------------|-------------|------------------|--------------------------------|
| **Standard Transformation** | Per document | **$0.01** | **$100.00** |
| **Custom Schema Mapping** | Per document | **$0.15** | **$1,500.00** |

---

## 5. AWS Free Tier Coverage
* **AWS B2B Data Interchange Free Tier:** First **1,000 EDI document transformations** free during your first month of usage.

---

## 6. Common Cost Hotspots & Pitfalls
* **Duplicate EDI Document Processing:** Re-running transformation workflows on duplicate or retried EDI batches without deduplication filters, incurring extra $0.01–$0.15 transformation fees per document.
* **Defaulting to Custom Mapping When Standard JSON Suffices:** Using custom mapping schemas ($0.15/doc) when standard JSON transformation ($0.01/doc) followed by lightweight downstream code could accomplish the same result.

---

## 7. Actionable Cost Optimization Strategies
1. **Replace Commercial Third-Party EDI Software Licenses:**
   * Migrate legacy EDI integrations (e.g., Sterling B2B or BizTalk) to AWS B2B Data Interchange.
   * **The Savings:** Eliminates tens of thousands of dollars in annual software licensing and server maintenance.
2. **Leverage Standard JSON Transformation:** Standard transformation ($0.01/doc) is **93% cheaper** than custom schema mapping ($0.15/doc). Transform EDI to standard JSON first, then use lightweight Lambda functions to reshape JSON in memory.
3. **Archive Raw EDI Files to S3 Glacier:** Route raw EDI archive files to S3 Glacier Instant Retrieval ($0.004/GB-mo) for long-term audit compliance.
