# Document AI Cost Optimization & Research

Google Cloud Document AI is a document understanding platform that uses machine learning to extract structured data from unstructured documents (invoices, tax forms, contracts, receipts, identity documents). Because Document AI charges are calculated **per page processed**, running oversized documents or using premium processors for simple OCR tasks can quickly drive up processing costs.

---

## 1. Document AI Billing Mechanics

Document AI pricing is determined by the specific processor type and the volume of pages parsed:
1. **Standard OCR / Layout Parser:** Low cost (approx. $1.50 per 1,000 pages).
2. **Specialized Parsers (Invoices, Receipts, W2, ID Cards, Utility Bills):** High cost (approx. $50.00 to $150.00 per 1,000 pages).
3. **Custom Document Extractors:** Premium pricing per page parsed, plus training and hosting hourly fees for custom models.

---

## 2. Core Cost-Optimization Levers

### A. Extract Pages Pre-Ingestion (Split PDFs)
* **The Cost Trap:** A vendor uploads a 60-page PDF containing an invoice on page 1, followed by 59 pages of product catalogs or terms of service. Sending the entire PDF to the Invoice Parser costs $9.00 (60 pages x $0.15/page).
* **The Solution:** Split the document before calling the API.
* **Action:** Implement a lightweight pre-processing step (e.g., using a Python library like PyPDF in a Cloud Function). Extract only the first page (or pages containing key text like "Invoice", "Total") and send only those target pages to the Document AI processor.
* **The Savings:** Cuts processing costs by **over 90%** for documents containing excessive attachments.

### B. Use OCR + Regex for Simple Key-Value Extractions
* **The Waste:** Using the premium Invoice Parser ($150 per 1,000 pages) just to extract the invoice date or invoice number.
* **Action:** If the invoice formats are standardized or easily parsed via search, run the standard **Document AI OCR processor** ($1.50 per 1,000 pages) to extract the raw text, and use a simple regex pattern or Python parsing logic to find the fields.
* **The Benefit:** Reduces API costs by **99%** compared to using specialized neural parsers.

### C. Decommit Custom Processor Hosting Tiers
* If you train Custom Document Classifiers or Extractors, you can configure them to run on dedicated hosting capacity for fast latency.
* **Action:** Monitor utilization. If custom model endpoints are inactive or experience very low daily query volumes, downscale active hosting replicas or use standard serverless prediction endpoints.

---

## 3. Document AI Audit Checklist

1. [ ] **PDF Splitting Pre-Processor:** Implement page-splitting logic in ingestion pipelines to isolate pages containing targets.
2. [ ] **Processor Tier Alignment:** Review active processors. Replace specialized parsers with standard OCR where simple text extraction is sufficient.
3. [ ] **Endpoint Utilization Audit:** Audit custom processor active endpoints. Delete or downscale idle custom model deployments.
