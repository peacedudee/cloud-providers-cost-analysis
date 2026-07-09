# Cost-Cutting Playbook: Amazon Braket
> **Companion File:** [braket.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/braket/braket.md)
> **Last Updated:** July 2026

---
## Executive Summary
This playbook outlines comprehensive strategies for optimizing costs when using Amazon Braket. Given Braket's unique pricing model—based on flat per-task fees, variable per-shot execution fees on physical Quantum Processing Units (QPUs), and per-minute execution fees on managed cloud simulators—the most impactful optimizations require algorithmic awareness. Key focuses include batching shots to avoid redundant task fees, leveraging free local simulators for debugging, and applying strict lifecycle controls on surrounding classical compute (like SageMaker Notebooks) and storage (like S3).

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
Amazon Braket does not operate in isolation. It relies heavily on **Amazon SageMaker** (for managed Jupyter notebooks), **Amazon S3** (for storing task and shot results), and **Amazon EC2** (for classical compute in Hybrid Jobs). Optimizing Braket architectures inherently drives down associated SageMaker compute and S3 storage costs.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
### B. CloudWatch Metrics
### C. AWS Config / Trusted Advisor
### D. Company Policies
### E. IaC (Optional)

---
## Output Schema
### Finding Record (JSON)
### Summary Report Table

#### 1. BRAKET-001. Use Local Python SDK Simulators for Testing
- **What:** Run and debug quantum circuits on the developer's local machine using `LocalSimulator()` before dispatching them to the cloud.
- **Why It Saves Money:** Local simulator executions are completely free ($0.00). In contrast, managed cloud simulators (SV1, DM1, TN1) cost $0.075 per minute, and physical QPUs charge flat task fees ($0.30) plus per-shot fees.
- **Implementation Steps:** 
  1. Install the Amazon Braket Python SDK locally.
  2. Replace physical QPU or cloud simulator ARNs in the code with `LocalSimulator()`.
  3. Run validation, syntax checking, and logical debugging tests locally.
  4. Switch back to cloud simulators or physical QPUs only for large-scale or final hardware executions.
- **Estimated Savings:** 90-100% on prototyping and debugging costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Developer workstation capable of running small-scale quantum simulations locally.

#### 2. BRAKET-002. Batch Circuit Executions into Single Quantum Tasks
- **What:** Configure quantum algorithms to request multiple shots (e.g., `shots=1000`) within a single task submission rather than submitting multiple separate tasks.
- **Why It Saves Money:** Braket charges a flat $0.30 per task request fee regardless of the shot count. Submitting 1,000 separate tasks of 1 shot each on IonQ costs $310.00 ($0.30 task + $0.01 shot × 1000). Batching 1,000 shots into a single task costs just $10.30 ($0.30 task + $10.00 shots).
- **Implementation Steps:** 
  1. Review quantum circuit execution loops in the algorithmic codebase.
  2. Modify the code to submit a single task using the `shots=N` parameter, avoiding iterating to submit `N` individual tasks.
  3. Validate circuit logic to ensure it can successfully process batched result arrays.
- **Estimated Savings:** Up to 96% on QPU execution costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Algorithms architected to handle bulk statistical shot returns.

#### 3. BRAKET-003. Stop Idle Braket Managed Notebooks
- **What:** Configure lifecycle policies to automatically stop Braket Managed Notebook instances when idle.
- **Why It Saves Money:** Braket notebooks run on SageMaker and are billed hourly (e.g., `ml.t3.medium` at $0.05/hour). Leaving them running 24/7 when active coding only happens for a few hours a day results in over 70% wasted spend.
- **Implementation Steps:** 
  1. Navigate to the SageMaker console or use the AWS CLI.
  2. Create a lifecycle configuration script that checks for idle Jupyter kernels.
  3. Attach the lifecycle script to all active Braket notebook instances.
  4. Ensure notebooks are automatically shut down after a defined period (e.g., 60 minutes) of inactivity.
- **Estimated Savings:** 60-80% on Notebook costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Notebooks deployed with IAM roles supporting lifecycle execution permissions.

#### 4. BRAKET-004. Terminate Unused Braket Managed Notebooks
- **What:** Identify and permanently delete Braket notebook instances that have not been used in the last 30 days.
- **Why It Saves Money:** Even if a notebook is in a "Stopped" state, the underlying EBS volumes attached to it continue to incur monthly storage charges. Deleting unused notebooks stops all associated computing and storage billing.
- **Implementation Steps:** 
  1. Review CloudWatch metrics to audit notebook activity.
  2. Identify instances showing zero activity over a 30-day period.
  3. Backup necessary local notebook data, algorithms, and models to Amazon S3.
  4. Terminate the unused notebook instances and delete their attached EBS volumes.
- **Estimated Savings:** 100% on unused resource holding costs
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Robust data backup policies for notebook content.

#### 5. BRAKET-005. Rightsize Braket Notebook Instances
- **What:** Downscale the instance type used for Braket Managed Notebooks to smaller, cheaper variants if local compute demands are minimal.
- **Why It Saves Money:** Moving from an `ml.t3.xlarge` ($0.20/hour) to an `ml.t3.medium` ($0.05/hour) reduces notebook running costs by 75%. Since the heavy computational lifting is offloaded to QPUs or Managed Simulators, large local notebook instances are rarely necessary.
- **Implementation Steps:** 
  1. Analyze CloudWatch CPU and Memory utilization metrics for active Braket notebooks.
  2. Stop the notebook instance in the console.
  3. Modify the instance type to a smaller, cost-effective size.
  4. Restart the notebook and verify that algorithmic development performance remains unaffected.
- **Estimated Savings:** 50-75% on Notebook instance costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** None

#### 6. BRAKET-006. Utilize Cheaper QPUs for Initial Hardware Validation
- **What:** Run preliminary physical QPU tests on lower-cost hardware providers before scaling to more expensive architectures.
- **Why It Saves Money:** Rigetti QPU usage costs $0.00035 per shot, while IonQ and QuEra cost $0.01000 per shot (approximately 28x more expensive per shot). Using Rigetti for initial physical debugging saves significant funds.
- **Implementation Steps:** 
  1. Configure the application logic to parameterize the QPU Device ARN.
  2. Set the default Device ARN to the Rigetti QPU for the initial hardware testing phase.
  3. Switch to IonQ or QuEra only for final production runs, or when a specific QPU topology (like trapped ion or neutral atom) is strictly required for the algorithmic approach.
- **Estimated Savings:** Up to 95% on physical execution shots during validation
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Agnostic quantum algorithms that can transpile and run on different physical QPU architectures.

#### 7. BRAKET-007. Clean Up Excess Amazon S3 Quantum Task Results
- **What:** Implement S3 lifecycle policies to expire or transition old quantum task results to cheaper storage tiers.
- **Why It Saves Money:** Amazon Braket automatically saves all task outputs to Amazon S3. Over time, millions of tiny JSON output files from quantum shots can accumulate, leading to steadily rising S3 Standard storage costs.
- **Implementation Steps:** 
  1. Identify the default S3 bucket used by Braket (typically formatted as `amazon-braket-<region>-<account-id>`).
  2. Create an S3 Lifecycle rule targeting the task results prefix.
  3. Transition data older than 30 or 60 days to S3 Glacier Instant Retrieval, or expire and delete them if the statistical results are no longer needed.
- **Estimated Savings:** 50-80% on S3 storage costs for Braket results
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** None

#### 8. BRAKET-008. Optimize Quantum Circuit Depth for Managed Simulators
- **What:** Reduce the number of gates and overall circuit complexity when using SV1, DM1, or TN1 cloud simulators.
- **Why It Saves Money:** Managed simulators are billed per minute ($0.075/min). Deep, complex circuits with many qubits take exponentially longer to simulate, which rapidly increases the execution duration and associated cost.
- **Implementation Steps:** 
  1. Use circuit optimization passes and transpilers (like Qiskit integration) to reduce the gate count.
  2. Eliminate redundant gates (e.g., consecutive H gates) before submitting the circuit to the simulator.
  3. Monitor simulator execution times in CloudWatch to track duration improvements.
- **Estimated Savings:** 20-50% on simulator execution costs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Advanced understanding of quantum circuit transpilation and optimization.

#### 9. BRAKET-009. Maximize the Amazon Braket Free Tier
- **What:** Ensure that smaller simulation workloads are spread across accounts or months to stay within the 1-hour free simulator limit.
- **Why It Saves Money:** All AWS accounts receive 1 free hour of SV1/DM1/TN1 simulator execution time per month indefinitely. Leveraging this equates to $4.50 of free value per month per account.
- **Implementation Steps:** 
  1. Track simulator usage metrics via AWS Cost Explorer or Braket console dashboards.
  2. Schedule automated CI/CD simulation tests to run within the 1-hour free tier budget.
  3. Avoid running long, unoptimized simulations that rapidly burn through the free tier limit early in the month.
- **Estimated Savings:** $4.50 per month per account
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** None

#### 10. BRAKET-010. Select the Correct Simulator Architecture (SV1 vs TN1)
- **What:** Match the quantum circuit structure to the most efficient cloud simulator (State Vector vs. Tensor Network).
- **Why It Saves Money:** SV1 simulates the full state vector and is ideal for highly entangled circuits with up to 34 qubits. TN1 uses tensor networks and is much faster/cheaper for circuits with local entanglement or sparse connectivity up to 50 qubits. Picking the wrong simulator can drastically increase simulation execution time (billed at $0.075/min).
- **Implementation Steps:** 
  1. Analyze the entanglement structure and qubit count of the quantum circuit.
  2. Route highly entangled, small-qubit circuits to the SV1 simulator.
  3. Route sparse, large-qubit circuits to the TN1 simulator.
  4. Compare execution times and adjust the targeting logic accordingly.
- **Estimated Savings:** 30-60% on simulator execution time
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Deep understanding of circuit entanglement characteristics and tensor networks.

#### 11. BRAKET-011. Implement Braket Hybrid Jobs for Variational Algorithms
- **What:** Use Amazon Braket Hybrid Jobs instead of orchestrating classical-quantum loops from a local machine or unmanaged notebook.
- **Why It Saves Money:** Variational algorithms (like QAOA or VQE) require tight, iterative feedback loops between classical compute and QPUs. Running this from outside the AWS network incurs network latency, extending QPU lock times. Hybrid jobs run on managed EC2 instances adjacent to the QPU, minimizing latency and overall execution time.
- **Implementation Steps:** 
  1. Refactor variational algorithms to use the `@hybrid_job` decorator within the Braket SDK.
  2. Submit the algorithm as a Hybrid Job.
  3. Monitor execution to ensure classical compute and QPU time are minimized.
- **Estimated Savings:** 10-30% on QPU execution and classical compute time
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Algorithms that rely on iterative classical-quantum feedback loops.

#### 12. BRAKET-012. Use Spot Instances within Braket Hybrid Jobs
- **What:** Configure Braket Hybrid Jobs to utilize Amazon EC2 Spot Instances for the classical compute portion of the workload.
- **Why It Saves Money:** Spot instances can discount classical EC2 compute costs by up to 90% compared to standard On-Demand prices.
- **Implementation Steps:** 
  1. In the Hybrid Job configuration codebase, set `instance_config` to enable Spot capacity.
  2. Ensure the classical algorithm can handle potential interruptions (though Hybrid Job durations are usually short enough to avoid Spot reclamations).
  3. Execute the Hybrid Job and monitor the billing discount.
- **Estimated Savings:** Up to 90% on the classical compute portion of Hybrid Jobs
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Fault-tolerant classical workloads in the hybrid loop.

#### 13. BRAKET-013. Purchase SageMaker Savings Plans for Notebooks
- **What:** Apply SageMaker Savings Plans to cover the underlying compute costs of Braket Managed Notebooks.
- **Why It Saves Money:** Braket notebooks run on underlying SageMaker compute instances. A 1-year or 3-year SageMaker Savings Plan can provide up to a 64% discount on the compute hourly rate in exchange for a monetary commitment.
- **Implementation Steps:** 
  1. Review continuous baseline usage for Braket/SageMaker notebooks in AWS Cost Explorer.
  2. Calculate the hourly spend commitment required.
  3. Purchase a SageMaker Savings Plan through the AWS Billing console.
- **Estimated Savings:** Up to 64% on Notebook compute costs
- **Risk Level:** High (Financial Commitment)
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Consistent, predictable notebook usage.

#### 14. BRAKET-014. Implement Budget Alerts and Anomaly Detection
- **What:** Set up AWS Budgets and AWS Cost Anomaly Detection specifically filtered for Amazon Braket spending.
- **Why It Saves Money:** A runaway programmatic loop submitting tasks to IonQ ($300+ per 1000 tasks) can drain thousands of dollars in hours. Alerts catch these architectural or coding errors before the bill heavily escalates.
- **Implementation Steps:** 
  1. Open AWS Budgets in the Billing Console.
  2. Create a custom budget filtered by `Service = Amazon Braket`.
  3. Set an alert threshold (e.g., $50/day).
  4. Enable AWS Cost Anomaly Detection with alerts sent to a FinOps Slack/Email channel.
- **Estimated Savings:** Prevents 100% of unbounded runaway costs
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** SNS Topics configured for alerting.

#### 15. BRAKET-015. Avoid Polling Overheads in Custom Architectures
- **What:** When orchestrating Braket tasks via Step Functions or custom Lambda scripts, use event-driven architectures (EventBridge) rather than continuous polling.
- **Why It Saves Money:** Continuously polling the Braket API for task completion status using AWS Lambda can incur high Lambda execution costs, especially since QPUs can have queue times stretching for hours.
- **Implementation Steps:** 
  1. Configure Amazon EventBridge rules to listen for Braket Task State Change events.
  2. Trigger downstream Lambda functions or Step Functions only when the task enters a `COMPLETED`, `FAILED`, or `CANCELLED` state.
  3. Remove `while(status == QUEUED)` polling loops from the orchestrating codebase.
- **Estimated Savings:** Up to 99% on surrounding Lambda/Compute polling costs
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** EventBridge and IAM setup experience.

#### 16. BRAKET-016. Monitor and Cancel Stalled Tasks Promptly
- **What:** Build automated checks or workflows to cancel quantum tasks that are stuck in the queue for too long if the results are no longer time-relevant or the test suite failed.
- **Why It Saves Money:** If a developer submits a massive job to a QPU queue, realizes a mistake, and closes their laptop, the job will still execute hours later and bill the account. Cancelling it while it is `QUEUED` incurs zero execution charges.
- **Implementation Steps:** 
  1. Use the Braket SDK to track task ARNs during execution.
  2. Implement an abort command or CI/CD script that calls `cancel_quantum_task()` for all pending jobs when a test suite fails or a developer manually aborts.
- **Estimated Savings:** 100% savings on aborted erroneous tasks
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Task ID tracking mechanism in local environment or CI/CD pipeline.

#### 17. BRAKET-017. Leverage Reserved Instances for Surrounding Compute
- **What:** Use EC2 Reserved Instances or Compute Savings Plans for dedicated classical EC2 instances that interface heavily with Braket, bypassing managed notebooks altogether.
- **Why It Saves Money:** If a team runs a 24/7 orchestration layer for Braket, deploying it on a standard EC2 instance covered by a 3-year Compute Savings Plan is significantly cheaper than using a SageMaker Notebook or on-demand EC2.
- **Implementation Steps:** 
  1. Assess continuous compute requirements for Braket orchestration.
  2. Migrate from Managed Notebooks to standard EC2 if appropriate.
  3. Purchase a Compute Savings Plan covering the EC2 instance family usage.
- **Estimated Savings:** Up to 72% on classical orchestration compute
- **Risk Level:** High (Financial Commitment)
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Self-managed infrastructure capabilities.

#### 18. BRAKET-018. Tag Braket Resources for Cost Allocation
- **What:** Apply mandatory cost allocation tags to all Braket Tasks, Notebooks, and S3 Buckets.
- **Why It Saves Money:** Without tagging, FinOps cannot attribute runaway Braket costs to a specific team, project, or researcher. Tagging enables chargebacks and internal accountability, which naturally drives down wasteful spending.
- **Implementation Steps:** 
  1. Define a tagging taxonomy (e.g., `Project`, `CostCenter`, `Researcher`).
  2. Use AWS Tag Policies to enforce tagging on Braket creation API calls.
  3. Activate tags in the AWS Billing console for visibility within AWS Cost Explorer.
- **Estimated Savings:** 10-20% through behavioral accountability
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Organization-wide tagging strategy.
