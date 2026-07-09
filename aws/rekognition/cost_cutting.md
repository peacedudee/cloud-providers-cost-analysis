# Cost-Cutting Playbook: Amazon Rekognition
> **Companion File:** [rekognition.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/rekognition/rekognition.md)
> **Last Updated:** July 2026

---

## Executive Summary
Amazon Rekognition offers powerful computer vision capabilities but can quickly become a significant expense if not carefully managed. The most common cost pitfalls involve continuous hosting of Custom Labels models ($2,920/month per active model) and inefficient processing techniques, such as using image APIs for high-framerate video frames. This playbook outlines 18 actionable strategies to eliminate waste, rightsize data payloads, optimize architectural decisions, and minimize associated data transfer costs.

---

## Strategy Categories

### 1. Waste Elimination
*   **REKOG-WE-001: Terminate Idle Custom Labels Hosting**
    *   **Description:** Custom Labels inference endpoints do not scale to zero automatically. Stop `ProjectVersions` outside of business hours or when not actively needed in non-production environments to avoid the $4.00/hour charge.
*   **REKOG-WE-002: Implement Hash-Based Image Caching**
    *   **Description:** Avoid re-analyzing duplicate images. Generate a cryptographic hash (e.g., MD5/SHA256) of incoming images and store the Rekognition results in DynamoDB or Redis. Check the cache before calling the API.
*   **REKOG-WE-003: Pre-filter Video Feeds with Edge Motion Detection**
    *   **Description:** For security or continuous monitoring, do not stream 24/7 static feeds to Rekognition Video. Use local edge devices to detect motion and only send segments containing activity.
*   **REKOG-WE-004: Delete Abandoned Custom Models**
    *   **Description:** While training and hosting are the primary costs, routinely audit and delete old, unused Custom Labels models to prevent accidental or orphaned hosting starts.

### 2. Rightsizing
*   **REKOG-RS-001: Resize Images Before Analysis**
    *   **Description:** Rekognition does not require raw, multi-megabyte images for accurate analysis. Downsize images to standard resolutions (e.g., 1080p) on the client side or via a lightweight Lambda function before calling the API.
*   **REKOG-RS-002: Compress Images and Videos**
    *   **Description:** Convert lossless formats (PNG, RAW) to optimized JPEGs or compressed MP4s to reduce S3 storage and cross-region/internet data transfer costs.
*   **REKOG-RS-003: Limit Bounding Box Processing**
    *   **Description:** If your workflow requires cropping multiple objects from an image based on Rekognition bounding boxes, perform the cropping locally or in a Lambda function rather than sending cropped segments back to Rekognition unnecessarily.

### 3. Commitment Discounts
*   **REKOG-CD-001: Leverage Enterprise Discount Program (EDP)**
    *   **Description:** Rekognition does not offer standalone Reserved Instances or Savings Plans. However, its usage contributes to and benefits from overall AWS EDP commitments. Ensure large-scale computer vision workloads are factored into EDP negotiations.

### 4. Architecture Changes
*   **REKOG-AC-001: Migrate from Image API (Frame-by-Frame) to Video API**
    *   **Description:** Do not extract frames from a video and send them individually to the Image API. A 1-hour 30fps video processed as images costs ~$108.00, whereas using the Rekognition Video API costs ~$6.00 (a 94% saving).
*   **REKOG-AC-002: Tiered AI Architecture (Edge AI + Rekognition Fallback)**
    *   **Description:** Deploy lightweight, open-source object detection models (e.g., YOLO) at the edge or on EC2/Lambda for basic classification. Only forward low-confidence results or complex tasks to Rekognition.
*   **REKOG-AC-003: Decouple Ingestion with SQS Batching**
    *   **Description:** Buffer incoming image analysis requests using Amazon SQS. This prevents API throttling and allows you to control the throughput of your analysis pipeline.
*   **REKOG-AC-004: Use Asynchronous Video Analysis APIs**
    *   **Description:** Always use `StartLabelDetection` and SNS for completion notifications rather than synchronous polling to save compute costs on the worker side.

### 5. Scheduling & Auto-Scaling
*   **REKOG-SA-001: Event-Driven Custom Labels Scaling**
    *   **Description:** Scale Custom Labels inference units based on the depth of an SQS queue. Trigger a Lambda to start the model when messages arrive, and stop it when the queue has been empty for a defined period.
*   **REKOG-SA-002: Scheduled Batch Processing for Non-Critical Workloads**
    *   **Description:** For historical image tagging, schedule processing jobs during off-peak hours to consolidate the underlying compute resources invoking the APIs, minimizing concurrent execution costs.

### 6. Pricing Model Optimization
*   **REKOG-PM-001: Optimize API Selection (Group 1 vs Group 2)**
    *   **Description:** Group 1 APIs (Labels, Moderation, Text) and Group 2 APIs (Face Search, Celebrity) have different billing classifications. Only call the specific API required for your use case to avoid compounding costs.
*   **REKOG-PM-002: Consolidate Volume for Tiered Pricing**
    *   **Description:** Rekognition Image API pricing tiers drop from $1.00 to $0.80 and then $0.40 per 1,000 images at high volumes. Centralize Rekognition usage across multiple organizational accounts into a single payer account to hit cheaper tiers faster.

### 7. Network & Data Transfer Optimization
*   **REKOG-ND-001: Co-Locate S3 and Rekognition in the Same Region**
    *   **Description:** Ensure that the S3 buckets hosting your media and the Rekognition API calls are executed in the exact same AWS Region to eliminate cross-region data transfer fees.
*   **REKOG-ND-002: Implement VPC Endpoints (PrivateLink) for Rekognition**
    *   **Description:** Route Rekognition API traffic through AWS PrivateLink VPC Endpoints. This prevents heavy media payloads from traversing and incurring per-GB processing charges on NAT Gateways.

---

## Cross-Service Synergies
*   **Amazon S3:** Use S3 Event Notifications to trigger Lambda-based Rekognition analysis, and S3 Lifecycle rules to archive processed media.
*   **AWS Lambda & EventBridge:** Automate the starting and stopping of expensive Custom Labels endpoints.
*   **Amazon SQS/SNS:** Decouple video processing state management, eliminating the need for idle EC2/Lambda polling.
*   **Amazon DynamoDB:** Store image hashes and metadata to act as an analysis cache, preventing duplicate API charges.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
*   `line_item_product_code`: "AmazonRekognition"
*   `line_item_usage_type`: Look for `CustomLabels-Inference-Hrs` and `Image-Analysis` variations.
*   `line_item_operation`: Differentiate between `DetectLabels`, `DetectFaces`, and Custom Labels operations.

### B. CloudWatch Metrics
*   `SuccessfulRequestLatency` (to determine if resizing is needed for faster processing).
*   `ClientErrors` (to detect faulty API calls wasting compute).
*   Custom metrics for SQS queue depth if using event-driven Custom Labels scaling.

### C. AWS Config / Trusted Advisor
*   Check VPC endpoint configurations to ensure PrivateLink is utilized for Rekognition traffic.
*   Audit S3 bucket locations relative to the region where Rekognition is invoked.

### D. Company Policies
*   Data retention requirements (how long original media must be kept).
*   Acceptable SLA for asynchronous batch processing vs. real-time processing.

### E. IaC (Optional)
*   Terraform/CloudFormation reviews to ensure Custom Labels infrastructure includes automated `stop` configurations or schedules.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "REKOG-WE-001",
  "category": "Waste Elimination",
  "resource_id": "arn:aws:rekognition:us-east-1:123456789012:project/my-custom-model/version/1",
  "issue": "Custom Labels model left running 24/7 in a development environment.",
  "recommendation": "Implement EventBridge scheduler to stop the inference unit outside of business hours.",
  "estimated_monthly_savings_usd": 2040.00,
  "effort": "Low"
}
```

### Summary Report Table
| Category | Finding ID | Issue Description | Recommendation | Est. Savings | Effort |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Waste Elimination | REKOG-WE-001 | Idle Custom Labels hosting | Schedule StopProjectVersion | High | Low |
| Architecture Changes | REKOG-AC-001 | Frame-by-frame image API usage | Switch to Rekognition Video API | High | Medium |
| Network Optimization | REKOG-ND-002 | NAT Gateway data transfer | Implement VPC Endpoint for Rekognition | Medium | Low |
| Rightsizing | REKOG-RS-001 | Large image payloads | Resize to 1080p before API call | Low | Low |
