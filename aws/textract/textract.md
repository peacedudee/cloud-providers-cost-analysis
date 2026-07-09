# AWS Service Cost Research: Amazon Textract

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Textract is a machine learning service that automatically extracts text, handwriting, and data from scanned documents. It goes beyond simple optical character recognition (OCR) to identify, understand, and extract data from tables, forms, and specific document types (like invoices, receipts, and identity documents). Textract is serverless and billed purely on a pay-as-you-go model per page analyzed, with pricing based on the extraction feature selected.

---

## 2. Billing Mechanics
Amazon Textract bills monthly based on the number of pages processed:
1.  **Page Count:** Calculated per page processed in PDF, TIFF, PNG, or JPEG formats.
2.  **Extraction Features:** Pricing varies heavily depending on the API feature used (e.g. basic OCR text extraction vs. structured form field detection).
3.  **Tiered Discounts:** Page-rate pricing drops automatically for high-volume accounts in a calendar month.

---

## 3. Key Cost Dimensions

### A. API Extraction Rates (us-east-1 Tiers)
*   **Detect Document Text API (Basic OCR):**
    *   *Rate:* **$1.50 per 1,000 pages** ($0.0015 per page).
    *   *Ideal for:* Simple plain-text extractions.
*   **Analyze Document API - Tables (Extracts structured tabular data):**
    *   *Rate:* **$15.00 per 1,000 pages** ($0.015 per page).
*   **Analyze Document API - Forms (Extracts key-value pairs like name, address):**
    *   *Rate:* **$50.00 per 1,000 pages** ($0.050 per page).
*   **Analyze Document API - Queries (Uses natural language prompts to pull specific fields):**
    *   *Rate:* **$15.00 per 1,000 pages** ($0.015 per page).
*   **Analyze Document API - Layout (Identifies headers, footers, paragraphs):**
    *   *Rate:* **$10.00 per 1,000 pages** ($0.010 per page).
*   **Analyze Expense API (Extracts data from invoices and receipts):**
    *   *Rate:* **$10.00 per 1,000 pages** ($0.010 per page).
*   **Analyze ID API (Extracts data from driver's licenses and passports):**
    *   *Rate:* **$25.00 per 1,000 pages** ($0.025 per page).

---

## 4. Detailed Pricing Rates (us-east-1 Volume Tiers)

| Textract Feature | Price per Page (0 to 1M pages/mo) | Price per 1,000 Pages | Price per Page (1M+ pages/mo) |
|------------------|-----------------------------------|-----------------------|------------------------------|
| **Detect Text (OCR)**| **$0.0015** | **$1.50** | $0.00060 |
| **Analyze Tables** | **$0.0150** | **$15.00** | $0.01000 |
| **Analyze Forms** | **$0.0500** | **$50.00** | $0.04000 |
| **Analyze Queries**| **$0.0150** | **$15.00** | $0.01000 |
| **Analyze Layout** | **$0.0100** | **$10.00** | $0.00500 |
| **Analyze Expense**| **$0.0100** | **$10.00** | $0.00800 |

---

## 5. AWS Free Tier Coverage
Textract features a **3-month free tier** for new accounts:
*   **Detect Document Text:** 1,000 free pages per month.
*   **Analyze Document (Forms/Tables/Queries):** 100 free pages per month.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Running Analyze Document (Forms) on Simple Text Pages:** Using the expensive **Analyze Forms API ($50.00/1,000 pages)** for multi-page documents where only the first page contains form fields, and subsequent pages contain standard paragraphs.
*   **Processing Empty or Duplicated Pages:** Submitting un-sorted document packages to Textract, paying the full page rate for blank pages, cover letters, or legal disclaimers.
*   **Unneeded Query Extractions:** Combining Tables, Forms, and Queries APIs in a single call when a cheaper subset of APIs would extract all needed data.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enforce Smart Page Classification (Pre-Filtering):**
    *   Before sending a document to the expensive **Forms ($50/1K)** or **Tables ($15/1K)** APIs, use a lightweight, local script or the cheap **Detect Text OCR ($1.50/1K)** to identify page contents.
    *   Classify which pages actually contain forms or tables.
    *   Extract and send only those specific pages to the expensive APIs.
    *   **The Savings:** Reduces the table/form page volume by 50–70% in multi-page document pipelines.
2.  **Prune Blank and Irrelevant Pages Locally:**
    *   Scan PDFs locally using simple libraries (like PyPDF or PDFMiner).
    *   Detect and discard blank pages, signature blocks, cover pages, or disclaimer pages.
    *   Do not upload these pages to Textract to avoid paying the page-tax.
3.  **Evaluate Queries API over Forms API:**
    *   If you only need to extract 2 or 3 specific key-value pairs (e.g. "Total Amount" and "Due Date") from a complex page, use the **Queries API ($15.00/1,000 pages)** instead of the **Forms API ($50.00/1,000 pages)**.
    *   **The Savings:** Cuts processing costs by **70%** for targeted extractions.
4.  **Batch Processing for Non-Interactive Pipelines:** For large archives, run asynchronous batch processing queue systems (`StartDocumentAnalysis`) which handle workloads reliably without endpoint timeouts.
