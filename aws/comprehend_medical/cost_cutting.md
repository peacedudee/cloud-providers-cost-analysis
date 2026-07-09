# Cost-Cutting Playbook: Amazon Comprehend Medical
> **Companion File:** [comprehend_medical.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/comprehend_medical/comprehend_medical.md)
> **Last Updated:** July 2026
---
## Executive Summary
Amazon Comprehend Medical is a powerful, serverless HIPAA-eligible NLP service that charges based on the volume of text analyzed. Because pricing varies massively by API operation (e.g., standard NERe is $100 per million characters, while RxNorm linking is just $2.50 per million characters) and request sizes are rounded up to 100-character units, the majority of cost savings come from optimizing payload size, stringing small requests together, and selecting the most specific API endpoint for the task. This playbook outlines 17 strategies to minimize Comprehend Medical costs through waste elimination, architectural adjustments, and rightsizing.

## Strategy Categories
### 1. Waste Elimination
### 2. Rightsizing
### 3. Commitment Discounts
### 4. Architecture Changes
### 5. Scheduling & Auto-Scaling
### 6. Pricing Model Optimization
### 7. Network & Data Transfer Optimization
---
## Cross-Service Synergies
- **Amazon S3 & DynamoDB:** Used to cache processed clinical notes to prevent redundant API invocations.
- **AWS Lambda:** Used for pre-processing, regex filtering, and text stitching before invoking Comprehend Medical.
- **AWS VPC Endpoints:** Keeps Comprehend Medical traffic off expensive NAT Gateways.
- **AWS CloudWatch & Budgets:** Essential for monitoring request volumes and preventing runaway recursive loops.
---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
Identify `lineItem/ProductCode` as `AmazonComprehendMedical` and examine `lineItem/Operation` (e.g., `DetectEntitiesV2`, `InferRxNorm`) and `lineItem/UsageAmount` to track character unit consumption.
### B. CloudWatch Metrics
Monitor `SuccessfulRequestCount` and `ClientErrors` in the AWS/ComprehendMedical namespace to identify inefficient retry loops and baseline traffic patterns.
### C. AWS Config / Trusted Advisor
Not typically used for Comprehend Medical directly, but useful for ensuring that S3 buckets and Lambda functions interacting with the service are properly secured and configured.
### D. Company Policies
Review data privacy and HIPAA compliance rules to determine if PHI detection is strictly necessary for all internal workloads.
### E. IaC (Optional)
Review Terraform/CloudFormation to ensure VPC Endpoints (PrivateLink) are configured for Comprehend Medical to avoid NAT data processing charges.
---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "COMPMED-001",
  "strategy_name": "Call Specific Ontology Linking APIs Directly",
  "estimated_savings_percentage": 97.5,
  "risk_level": "Low",
  "effort_level": "Low"
}
```
### Summary Report Table

#### 1. Pre-Clean Medical Documents Locally
- **What:** Use local scripts or regex within your application to strip out non-clinical text such as HTML tags, hospital letterheads, footers, system telemetry logs, and boilerplate privacy agreements before sending the document to the API.
- **Why It Saves Money:** Comprehend Medical bills per 100-character unit. Standard NERe costs $0.01 per unit. Removing 500 characters of boilerplate per document saves $0.05 per document, which scales massively across millions of records.
- **Implementation Steps:**
  1. Analyze a sample of clinical documents to identify consistent non-clinical boilerplate.
  2. Implement regex or parsing logic in your data pipeline (e.g., AWS Lambda) to strip this text.
  3. Send only the cleaned clinical narrative to Comprehend Medical.
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Predictable document structures.

#### 2. Stitch Short Transcripts Prior to Ingestion
- **What:** Concatenate multiple short text inputs (e.g., 15-character doctor voice commands or short prescriptions) into a single larger block separated by delimiters (like `|`) before calling the API.
- **Why It Saves Money:** Every API request has a hard 1-unit (100 characters) minimum charge. Sending five 20-character requests costs 5 units. Stitching them into one 100-character request costs 1 unit.
- **Implementation Steps:**
  1. Identify workloads sending very short text fragments.
  2. Update the application logic to batch these fragments into a single string (up to a reasonable limit like 1,000 characters).
  3. Parse the delimited response back into individual records.
- **Estimated Savings:** Up to 80%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload can tolerate slight batching latency.

#### 3. Call Specific Ontology Linking APIs Directly
- **What:** Instead of calling standard NERe ($0.01/unit) and then calling an ontology linking API on the output, call the specific linking API (like RxNorm or ICD-10-CM) directly.
- **Why It Saves Money:** Linking APIs perform both entity recognition and code mapping in a single pass. RxNorm ($0.00025/unit) is 40x cheaper than standard NERe ($0.01/unit).
- **Implementation Steps:**
  1. Audit application code for chained API calls (e.g., `DetectEntitiesV2` followed by `InferRxNorm`).
  2. Refactor the code to exclusively call `InferRxNorm` or `InferICD10CM`.
- **Estimated Savings:** 95-97.5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Use case is focused on specific ontologies (medications, diagnoses).

#### 4. Restrict API Payloads to Relevant Sections
- **What:** Parse clinical documents locally and extract only the relevant sections (e.g., "History of Present Illness", "Medications", "Assessment and Plan") to send to the API, ignoring standard physical exam checkboxes or demographics.
- **Why It Saves Money:** Drastically reduces the total character count processed, cutting per-unit billing.
- **Implementation Steps:**
  1. Implement a lightweight local NLP or regex parser to identify section headers.
  2. Extract only the sections required for downstream analysis.
  3. Pass the truncated text to Comprehend Medical.
- **Estimated Savings:** 40-60%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Structured or semi-structured clinical documents.

#### 5. Use RxNorm for Medication Extraction Instead of SNOMED-CT
- **What:** If the primary goal of the NLP extraction is to identify medications and dosages, use the RxNorm linking API instead of the SNOMED-CT API.
- **Why It Saves Money:** RxNorm is billed at a flat $0.00025 per unit, whereas SNOMED-CT linking costs $0.00750 per unit (30x more expensive).
- **Implementation Steps:**
  1. Review the clinical requirements of the output data.
  2. Switch the API endpoint from `InferSNOMEDCT` to `InferRxNorm` if medication codes are sufficient.
- **Estimated Savings:** 96%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Downstream systems accept RxNorm ontologies.

#### 6. Bypass PHI Detection for Internal-Only Data
- **What:** If clinical data remains strictly inside a secure, HIPAA-compliant database and is not shared with third parties or researchers, bypass the PHI detection API step.
- **Why It Saves Money:** Saves $14.00 per Million characters by skipping the `DetectPHI` API.
- **Implementation Steps:**
  1. Review data compliance policies with the Security/Governance team.
  2. Disable the PHI detection pipeline for data that does not leave the secure boundary.
- **Estimated Savings:** 100% of PHI detection costs
- **Risk Level:** High (Compliance Risk)
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Approval from Security and Compliance teams.

#### 7. Use Open-Source Regex for Simple PHI
- **What:** Filter out obvious Protected Health Information (SSNs, standard phone numbers, dates, email addresses) using open-source regex libraries before sending complex clinical text to Comprehend Medical's PHI detection.
- **Why It Saves Money:** Reduces the volume of text sent to the expensive $0.00140/unit PHI API. Comprehend Medical should only be used for complex context-based PHI (like finding a patient name within a paragraph).
- **Implementation Steps:**
  1. Deploy a pre-processing lambda function with regex patterns for structured PHI.
  2. Mask these findings locally.
  3. Only send remaining unstructured text to `DetectPHI`.
- **Estimated Savings:** 15-25%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Strong regex libraries and testing frameworks.

#### 8. Store Results in Persistent Storage (Caching)
- **What:** Cache the JSON outputs from Comprehend Medical APIs in S3 or DynamoDB, keyed to a cryptographic hash (e.g., SHA-256) of the clinical document text.
- **Why It Saves Money:** If the exact same document or note is processed again, the system fetches the cached JSON rather than paying for a duplicate API call.
- **Implementation Steps:**
  1. Hash the incoming text string.
  2. Check DynamoDB/S3 for the hash.
  3. Return cached result if present; otherwise, call the API and store the result.
- **Estimated Savings:** 10-20%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** S3 or DynamoDB infrastructure.

#### 9. Consolidate Usage for Volume Tier Discounts
- **What:** Route all Comprehend Medical API calls across the organization through a centralized AWS account or ensure AWS Organizations consolidated billing is active.
- **Why It Saves Money:** Prices drop significantly at higher volumes. Standard NERe drops from $0.01 to $0.005 (>1M units) and $0.001 (>2M units). Pooling usage unlocks these tiers faster.
- **Implementation Steps:**
  1. Verify AWS Organizations consolidated billing is enabled.
  2. Alternatively, create an internal centralized microservice for NLP processing.
- **Estimated Savings:** 50-90%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** AWS Organizations.

#### 10. Filter Empty or Invalid Records Prior to Ingestion
- **What:** Add data validation checks to drop nulls, empty strings, whitespace-only strings, or garbled non-text records before sending them to the API.
- **Why It Saves Money:** Avoids the 1-unit (100 character) minimum charge for requests that contain no actionable data.
- **Implementation Steps:**
  1. Add a conditional check prior to invocation: `if len(text.strip()) > 5: call_api(text)`
- **Estimated Savings:** 1-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None.

#### 11. Implement VPC Endpoints (AWS PrivateLink)
- **What:** Route Comprehend Medical API traffic originating from private subnets through a VPC Interface Endpoint rather than a NAT Gateway.
- **Why It Saves Money:** NAT Gateway data processing charges ($0.045/GB) are significantly higher than VPC Endpoint charges ($0.01/GB).
- **Implementation Steps:**
  1. Create an Interface VPC Endpoint for `com.amazonaws.region.comprehendmedical`.
  2. Ensure Security Groups allow traffic from your subnets.
- **Estimated Savings:** 75% on Data Transfer out of private subnets
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC Architecture.

#### 12. Keep API Calls Within the Same AWS Region
- **What:** Ensure the compute resources (EC2, EKS, Lambda) invoking the Comprehend Medical API are located in the same AWS region as the Comprehend Medical endpoint.
- **Why It Saves Money:** Prevents cross-region data transfer out fees ($0.02/GB).
- **Implementation Steps:**
  1. Map where your clinical data originates and where your workloads run.
  2. Ensure the SDK uses the local region client (e.g., `us-east-1` to `us-east-1`).
- **Estimated Savings:** 100% of cross-region transfer fees
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-region architecture analysis.

#### 13. Utilize Asynchronous Batch Invocations for Archives
- **What:** For bulk historical records or non-time-sensitive data, use asynchronous batch APIs (e.g., `StartMedicalTranscriptionJob`) instead of real-time synchronous APIs.
- **Why It Saves Money:** While it doesn't directly change the per-unit Comprehend Medical cost, it allows for optimal text packing and avoids idle compute (EC2/Lambda) waiting for synchronous real-time responses.
- **Implementation Steps:**
  1. Dump clinical notes to an S3 bucket.
  2. Trigger a batch job to process the entire bucket.
  3. Retrieve results from the output S3 bucket.
- **Estimated Savings:** 5-15% (Compute savings)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Data can tolerate asynchronous processing latency.

#### 14. Mock APIs During Development and Testing
- **What:** Use local stubbed responses or mock APIs for Comprehend Medical during local developer testing and CI/CD automated test runs.
- **Why It Saves Money:** Developers running test suites won't trigger real API costs at $100/Million characters.
- **Implementation Steps:**
  1. Implement a toggle in your application configuration to bypass AWS APIs locally.
  2. Return static, pre-recorded JSON responses that match the Comprehend Medical schema.
- **Estimated Savings:** 100% of Dev/Test API costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Local development environment setup.

#### 15. Implement API Concurrency Limits and Budget Alarms
- **What:** Set up AWS Budgets and CloudWatch alarms on Comprehend Medical usage metrics to catch runaway recursive loops. Implement concurrency limits on AWS Lambda functions that trigger Comprehend Medical.
- **Why It Saves Money:** Prevents accidental infinite loops (e.g., an S3 trigger that reads and writes to the same bucket, invoking Comprehend Medical repeatedly) from generating massive unexpected bills.
- **Implementation Steps:**
  1. Create an AWS Budget targeted specifically at the Comprehend Medical service.
  2. Set Reserved Concurrency limits on Lambda functions in the ingestion pipeline.
- **Estimated Savings:** Unquantifiable (Risk Mitigation)
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps | FinOps Team
- **Prerequisites:** AWS Budgets.

#### 16. Evaluate Custom Comprehend for Non-Clinical Ontologies
- **What:** If you only need to extract very specific non-standard entities (e.g., internal hospital department codes) that aren't strict clinical ontologies, training a Custom Amazon Comprehend (standard) entity recognizer might be cheaper at scale than using Comprehend Medical.
- **Why It Saves Money:** Standard Comprehend custom entity recognition is $0.0001 per unit. Comprehend Medical NERe is $0.01 per unit (100x difference).
- **Implementation Steps:**
  1. Evaluate if your extraction needs strictly require clinical NLP.
  2. If not, provide training data to Amazon Comprehend (Standard) to build a custom recognizer.
- **Estimated Savings:** Up to 99%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Clean training data for custom entity recognition.

#### 17. Negotiate Enterprise Discount Program (EDP)
- **What:** Roll Comprehend Medical usage into a larger AWS Enterprise Discount Program (EDP) commitment.
- **Why It Saves Money:** AWS EDPs offer a flat percentage discount across all eligible services in exchange for a committed annual spend.
- **Implementation Steps:**
  1. Forecast Comprehend Medical usage for the next 1-3 years.
  2. Work with your AWS Account Manager to include this in the EDP minimum commit.
- **Estimated Savings:** 5-15%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** High overall AWS spend (typically $1M+ annually).
