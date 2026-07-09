# Cost-Cutting Playbook: Amazon Textract
> **Companion File:** [textract.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/textract/textract.md)
> **Last Updated:** July 2026

---

## Executive Summary
Amazon Textract provides powerful machine learning-based document extraction, but its pay-per-page pricing model can quickly become expensive if unoptimized. The most significant costs stem from utilizing premium extraction features—like Analyze Forms ($50/1K pages)—on entire multi-page documents when only a fraction of those pages contain forms. This playbook outlines 21 strategic cost-cutting measures, heavily focusing on pre-filtering pages, right-sizing API usage (e.g., using Queries over Forms), and architectural adjustments to eliminate wasteful extractions and minimize overall page counts.

## Strategy Categories

### 1. Waste Elimination
*   **TEXTRACT-WE-001: Prune Blank Pages Locally**
    *   Scan PDFs locally (using lightweight libraries like PyPDF) to detect and discard blank pages before sending the document to Textract, avoiding the per-page tax.
*   **TEXTRACT-WE-002: Remove Irrelevant Pages**
    *   Strip out cover letters, legal disclaimers, or signature blocks locally if they don't contain the data you need to extract.
*   **TEXTRACT-WE-003: Deduplicate Documents**
    *   Implement document hashing in your ingest pipeline to identify and reject duplicate documents before they reach Textract.
*   **TEXTRACT-WE-004: Implement Result Caching**
    *   Store Textract JSON outputs in DynamoDB or S3. Check the cache for the document hash before initiating a new Textract job.
*   **TEXTRACT-WE-005: Avoid Blind Retries on Bad Data**
    *   If Textract fails to extract data due to poor image quality or unsupported formats, fail the job rather than continuously retrying the API.

### 2. Rightsizing
*   **TEXTRACT-RS-001: Pre-Filter with Basic OCR**
    *   Use the inexpensive Detect Document Text API ($1.50/1K) to identify page contents. Only send pages containing forms to the expensive Analyze Forms API ($50/1K).
*   **TEXTRACT-RS-002: Evaluate Queries API over Forms API**
    *   If extracting only a few specific fields (e.g., "Total Amount"), use the Queries API ($15/1K) instead of the heavy Forms API ($50/1K), saving 70%.
*   **TEXTRACT-RS-003: Downgrade from Forms to Layout API**
    *   If the goal is simply to identify headers, footers, or paragraphs for NLP segmentation, use the Layout API ($10/1K) rather than the Forms API.
*   **TEXTRACT-RS-004: Image Stitching (Consolidation)**
    *   For very small documents like receipts or ID cards, programmatically stitch multiple images into a single page grid before uploading to Textract to reduce the total page count.
*   **TEXTRACT-RS-005: Use Specialized APIs When Applicable**
    *   For invoices, use Analyze Expense ($10/1K) instead of Forms ($50/1K) or Queries ($15/1K) to achieve better accuracy at a lower price.

### 3. Commitment Discounts
*   **TEXTRACT-CD-001: Volume Tier Optimization**
    *   Consolidate Textract workloads within a single AWS account or AWS Organization to cross the 1M+ page threshold faster, triggering automatic volume tier pricing discounts.
*   **TEXTRACT-CD-002: Private Pricing Agreements (EDP)**
    *   If your organization processes millions of pages monthly, negotiate a private pricing agreement or Enterprise Discount Program (EDP) with AWS for custom Textract rates.

### 4. Architecture Changes
*   **TEXTRACT-AC-001: Asynchronous Batch Processing**
    *   For non-interactive pipelines or large archives, use `StartDocumentAnalysis` instead of synchronous APIs to handle workloads reliably and avoid API Gateway or Lambda timeout costs.
*   **TEXTRACT-AC-002: Open Source OCR Fallback Pipeline**
    *   For simple, high-quality typed text, run an open-source OCR (e.g., Tesseract) on a cheap Lambda first. Only fall back to Textract if the confidence score is below a certain threshold.
*   **TEXTRACT-AC-003: Step Functions for Orchestration**
    *   Manage complex OCR workflows with AWS Step Functions to gracefully handle rate limiting, callbacks, and retries without wasting Lambda execution time polling for Textract completion.
*   **TEXTRACT-AC-004: Event-Driven S3 Pipelines**
    *   Trigger Textract processing via S3 events (`s3:ObjectCreated`) rather than running polling servers to ensure compute is only used when a document is ready.

### 5. Scheduling & Auto-Scaling
*   **TEXTRACT-SA-001: SQS Queue Throttling**
    *   Buffer incoming Textract requests using Amazon SQS to smooth out spikes, preventing API throttling and the compute costs associated with handling retries.
*   **TEXTRACT-SA-002: Off-Peak Batch Processing**
    *   While Textract pricing is flat, processing low-priority document archives off-peak prevents the need for high concurrency limits (TPS) and reduces the strain on downstream databases.

### 6. Pricing Model Optimization
*   **TEXTRACT-PM-001: Avoid Unneeded API Combinations**
    *   Do not select all feature flags (Tables, Forms, Queries) in a single `AnalyzeDocument` call. Only request the specific features required for that document type to avoid stacked billing.
*   **TEXTRACT-PM-002: AWS Free Tier Utilization**
    *   Route developer sandboxes and low-volume testing environments to new AWS accounts to utilize the 3-month Textract Free Tier (1,000 OCR pages, 100 Analyze pages/mo).

### 7. Network & Data Transfer Optimization
*   **TEXTRACT-ND-001: Keep Processing in the Same Region**
    *   Ensure the S3 bucket containing the documents and the Lambda function calling Textract are in the same AWS region to completely avoid cross-region data transfer fees.

---

## Cross-Service Synergies
*   **Amazon S3:** Use S3 Lifecycle policies to automatically archive processed original documents to Glacier, and leverage S3 Object Tagging to mark documents as `TextractStatus: Processed` to prevent duplicate processing.
*   **AWS Lambda / Step Functions:** Combine Textract asynchronous APIs with Step Functions callback patterns (Task Tokens) to completely eliminate Lambda wait times.
*   **Amazon Comprehend:** Pre-process text extracted by cheap OCR with Comprehend to determine if a page needs further, expensive Textract processing (e.g., Forms).

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
*   `lineItem/ProductCode` = `AmazonTextract`
*   `lineItem/Operation` (to distinguish between DetectText, AnalyzeDocument, AnalyzeExpense).
*   `lineItem/UsageAmount` (to identify the exact number of pages processed per feature).

### B. CloudWatch Metrics
*   `SuccessfulRequestLatency` (to detect if synchronous calls are taking too long, indicating a need for async batching).
*   `ThrottledRequests` (to identify if API quotas are being hit, requiring SQS queuing).
*   `ServerErrorRate` (to track failed pages that you may still be paying compute retries for).

### C. AWS Config / Trusted Advisor
*   Check for Lambda timeout configurations that might be prematurely killing long-running synchronous Textract calls.

### D. Company Policies
*   Data retention policies (how long must extracted data and original images be kept?).
*   SLA requirements (do documents need synchronous real-time processing, or is an async queue acceptable?).

### E. IaC (Optional)
*   Review Terraform or CloudFormation configurations to ensure Textract API calls aren't hardcoded to request unnecessary feature sets (e.g., always requesting Forms when only Tables are needed).

---

## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "TEXTRACT-RS-001",
  "category": "Rightsizing",
  "title": "Pre-Filter with Basic OCR",
  "description": "Use Detect Document Text ($1.50/1K) to classify pages before sending relevant pages to Analyze Forms ($50/1K).",
  "estimated_savings_percentage": 50,
  "effort_level": "Medium",
  "risk_level": "Low"
}
```

### Summary Report Table
| Finding ID | Category | Strategy | Effort |
|------------|----------|----------|--------|
| TEXTRACT-WE-001 | Waste Elimination | Prune Blank Pages Locally | Low |
| TEXTRACT-RS-002 | Rightsizing | Evaluate Queries API over Forms API | Medium |
| TEXTRACT-PM-001 | Pricing Model | Avoid Unneeded API Combinations | Low |
| TEXTRACT-AC-001 | Architecture Changes | Asynchronous Batch Processing | High |
| TEXTRACT-ND-001 | Network & Data | Keep Processing in the Same Region | Low |
