# Cost-Cutting Playbook: Amazon Managed Blockchain
> **Companion File:** [managed_blockchain.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/managed_blockchain/managed_blockchain.md)
> **Last Updated:** July 2026

---

## Executive Summary
Amazon Managed Blockchain (AMB) provides fully managed infrastructure for public blockchain networks (AMB Access), serverless multi-chain data lookups (AMB Query), and private enterprise networks (Hyperledger Fabric). Costs primarily stem from dedicated node hourly rates, Hyperledger membership fees, and serverless API query volumes. This playbook outlines actionable strategies to optimize AMB deployments, focusing on migrating from dedicated nodes to serverless queries, aggressively pruning idle dev/test memberships, and rightsizing node types for non-production environments.

## Strategy Categories

### 1. Waste Elimination

#### AMB-1. Delete Idle Hyperledger Fabric Memberships
- **What:** Terminate Hyperledger Fabric network memberships that are no longer actively used for development or testing.
- **Why It Saves Money:** Each Hyperledger membership incurs a $0.30/hour fee ($219.00/month), regardless of whether transactions are occurring.
- **Implementation Steps:**
  1. Identify inactive Hyperledger Fabric networks using CloudWatch Metrics for transaction volume.
  2. Confirm with the development team that the environment is no longer needed.
  3. Delete the member and associated peer nodes via the AWS Management Console or AWS CLI.
- **Estimated Savings:** 100% of the $219/month idle membership cost.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Backup of chaincode/data if needed.

#### AMB-2. Terminate Unused Dedicated Peer Nodes
- **What:** Identify and delete dedicated AMB Access peer nodes that have zero transaction activity.
- **Why It Saves Money:** A running `bc.c5.xlarge` node costs $198.56/month. If it's not processing transactions or serving read queries, it is wasted spend.
- **Implementation Steps:**
  1. Audit CloudWatch metrics for AMB node CPU utilization and request count.
  2. Flag nodes with consistent zero or near-zero activity over a 7-day period.
  3. Delete the node.
- **Estimated Savings:** $70 to $1,088+ per month per node depending on instance type.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Metric monitoring enabled.

#### AMB-3. Remove Orphaned VPC Endpoints
- **What:** Clean up VPC Endpoints (AWS PrivateLink) previously created to access deleted AMB networks.
- **Why It Saves Money:** VPC Endpoints incur hourly charges (~$7.20/month per AZ) plus data processing fees, even if the underlying AMB service is gone.
- **Implementation Steps:**
  1. Cross-reference existing VPC Endpoints with active AMB network IDs.
  2. Delete VPC Endpoints pointing to defunct networks.
- **Estimated Savings:** $7.20 - $21.60 per month per orphaned endpoint.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Access to VPC dashboard.

### 2. Rightsizing

#### AMB-4. Rightsize Dev/Test Nodes to `bc.t3.small`
- **What:** Use the smallest available node type (`bc.t3.small`) for non-production workloads instead of default production sizes.
- **Why It Saves Money:** A `bc.t3.small` costs $0.096/hour ($70.08/month) compared to a `bc.c5.xlarge` at $0.272/hour ($198.56/month) or larger.
- **Implementation Steps:**
  1. Review the instance types of all non-production peer nodes.
  2. Provision a new `bc.t3.small` node for the dev/test network.
  3. Update application endpoints and terminate the larger node.
- **Estimated Savings:** 65-90% per dev/test node.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Infrastructure as Code templates updated.

#### AMB-5. Downsize Over-Provisioned Production Nodes
- **What:** Analyze production AMB Access nodes and downsize them if CPU and memory utilization are consistently low.
- **Why It Saves Money:** Moving from a `bc.c5.4xlarge` ($1.088/hr) to a `bc.c5.2xlarge` ($0.544/hr) cuts costs exactly in half ($397/month savings per node).
- **Implementation Steps:**
  1. Use CloudWatch to monitor node CPU utilization for at least 2 weeks.
  2. If peak utilization is < 30%, test the workload on a smaller instance size.
  3. Provision the smaller node, redirect traffic, and delete the larger node.
- **Estimated Savings:** 50-75% per downsized node.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Load testing on smaller nodes.

#### AMB-6. Select the Right Instance Family (Compute vs. General Purpose)
- **What:** Choose between C5 (Compute Optimized) and M5 (General Purpose) based on workload requirements.
- **Why It Saves Money:** If your node requires more memory but not CPU, an M5 might prevent you from over-provisioning a massive C5 just for RAM, right-sizing the ratio.
- **Implementation Steps:**
  1. Profile the memory vs. CPU usage of your blockchain node.
  2. Select the optimal family (e.g., `bc.m5.2xlarge` vs `bc.c5.2xlarge`).
- **Estimated Savings:** 10-20%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload profiling.

### 3. Commitment Discounts

#### AMB-7. Leverage AWS Free Tier for POCs
- **What:** Utilize the AMB Free Tier for new proof-of-concept deployments instead of paying on-demand rates.
- **Why It Saves Money:** New AWS accounts get 250 free hours of `bc.t3.small` peer node usage and 1 Million JSON-RPC requests per month for the first 2 months.
- **Implementation Steps:**
  1. Spin up POCs in a newly created AWS account linked via AWS Organizations.
  2. Ensure the node type selected is strictly `bc.t3.small`.
  3. Terminate before the 250 hours (~10 days of 24/7 runtime) expire.
- **Estimated Savings:** $24 per month per POC.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** New AWS Account eligibility.

#### AMB-8. Enterprise Discount Program (EDP) Inclusion
- **What:** Ensure Amazon Managed Blockchain spend is factored into AWS EDP negotiations.
- **Why It Saves Money:** While AMB doesn't support Reserved Instances or Compute Savings Plans natively, high AMB spend contributes to overall EDP volume discounts (typically 9-15% across all services).
- **Implementation Steps:**
  1. Aggregate projected yearly AMB spend.
  2. Provide forecasts to AWS account managers during EDP renewals.
- **Estimated Savings:** 9-15% discount on AMB spend.
- **Risk Level:** Low
- **Implementation Scope:** Procurement/Leadership
- **Prerequisites:** EDP qualification thresholds met.

### 4. Architecture Changes

#### AMB-9. Migrate to AMB Query Serverless APIs for Data Reads
- **What:** Replace dedicated 24/7 AMB Access nodes with the serverless AMB Query API for reading blockchain data (token balances, events).
- **Why It Saves Money:** A dedicated node costs ~$198.56/month. AMB Query costs $0.50 per 1 Million API requests. For low-to-medium query volumes, the cost difference is massive.
- **Implementation Steps:**
  1. Identify applications currently using dedicated nodes purely for multi-chain data lookups.
  2. Refactor the application code to use the AMB Query API.
  3. Terminate the dedicated node.
- **Estimated Savings:** Up to 99% (e.g., $198 down to $0.50/month).
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Workload is read-heavy and supported by AMB Query.

#### AMB-10. Implement a Caching Layer for AMB Query
- **What:** Deploy ElastiCache (Redis) or use application-level caching to store frequently accessed blockchain data.
- **Why It Saves Money:** Avoids repeatedly paying the $0.50 per 1M API requests for identical multi-chain data lookups (e.g., historical block data that doesn't change).
- **Implementation Steps:**
  1. Identify highly repetitive queries (e.g., block headers, historical smart contract state).
  2. Implement a TTL-based cache using Redis or Memcached.
  3. Route queries to the cache first before hitting AMB Query.
- **Estimated Savings:** 20-50% on API request costs.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Caching infrastructure in place.

#### AMB-11. Use Serverless APIs over Hyperledger Fabric for Simple Ledgers
- **What:** Evaluate if a full Hyperledger Fabric network is necessary, or if AWS QLDB (Quantum Ledger Database) can serve the purpose.
- **Why It Saves Money:** AMB Hyperledger requires $219/month membership fees + node hourly costs. QLDB is serverless and pay-per-transaction, which can be exponentially cheaper for simple centralized ledger needs.
- **Implementation Steps:**
  1. Assess the business need for decentralization (are multiple untrusted parties actually running nodes?).
  2. If centralized trust is acceptable, migrate to AWS QLDB.
- **Estimated Savings:** 60-90% depending on transaction volume.
- **Risk Level:** High
- **Implementation Scope:** Architect/Engineer
- **Prerequisites:** Trust model allows for a centralized verifiable ledger.

#### AMB-12. Consolidate Hyperledger Fabric Memberships
- **What:** Share a single Hyperledger Fabric membership across multiple dev/test projects using different channels, rather than spawning separate networks.
- **Why It Saves Money:** Avoids paying multiple $0.30/hour ($219/mo) membership fees for isolated sandbox environments.
- **Implementation Steps:**
  1. Provision a centralized shared dev/test Hyperledger network.
  2. Use Fabric Channels to isolate data and smart contracts for different teams.
- **Estimated Savings:** $219/month per eliminated membership.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of Fabric Channel isolation.

### 5. Scheduling & Auto-Scaling

#### AMB-13. Ephemeral Dev/Test Environments via Infrastructure as Code
- **What:** Use Terraform or CloudFormation to spin up AMB nodes and memberships only when actively developing or testing, and tear them down overnight/weekends.
- **Why It Saves Money:** Running a node and membership for 40 hours a week instead of 168 hours saves ~75% of the cost.
- **Implementation Steps:**
  1. Script the creation of AMB memberships and nodes.
  2. Implement CI/CD pipelines to deploy environments on-demand.
  3. Schedule automated teardowns at 6 PM daily or on Fridays.
- **Estimated Savings:** 65-75% on dev/test infrastructure.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Fully automated IaC.

#### AMB-14. Auto-Scale AMB Access Serverless APIs
- **What:** Rely on the inherently auto-scaling nature of AMB Access Serverless APIs rather than provisioning multiple dedicated nodes for peak loads.
- **Why It Saves Money:** Dedicated nodes must be provisioned for peak capacity, paying for idle compute. Serverless APIs only charge per million requests.
- **Implementation Steps:**
  1. Profile request volume to find the breakeven point between dedicated nodes and serverless requests.
  2. Route traffic to the serverless API endpoint.
- **Estimated Savings:** Variable, up to 50% for bursty workloads.
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Application supports standard JSON-RPC endpoints.

### 6. Pricing Model Optimization

#### AMB-15. Optimize Request Complexity for Serverless APIs
- **What:** Batch requests or optimize the types of RPC calls made to AMB Access serverless APIs, as complex requests may be billed at higher rates or consume more throughput.
- **Why It Saves Money:** Reducing the absolute number of API calls or avoiding highly complex queries reduces the "per million" billing multiplier.
- **Implementation Steps:**
  1. Review application logic for N+1 query problems.
  2. Use multicall smart contracts to bundle multiple read operations into a single API request.
- **Estimated Savings:** 30-60% on API costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Smart contract development knowledge.

#### AMB-16. Consolidate AWS Accounts for Free Tier Exhaustion
- **What:** Carefully manage how POCs are distributed to maximize the 2 months of AMB Free Tier without unnecessarily spawning empty accounts.
- **Why It Saves Money:** Ensures that you are getting the full 250 hours per month for testing before committing to paid usage.
- **Implementation Steps:**
  1. Track AWS account creation dates.
  2. Deploy AMB tests strictly in accounts under 2 months old.
- **Estimated Savings:** Minor ($48/POC), but effective for strict budgets.
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** AWS Organizations.

### 7. Network & Data Transfer Optimization

#### AMB-17. Ensure Same-Region Data Access
- **What:** Deploy applications (EC2/EKS/Lambda) in the same AWS Region as the Amazon Managed Blockchain network.
- **Why It Saves Money:** Avoids cross-region data transfer fees ($0.01 - $0.02 per GB) when querying heavy blockchain data.
- **Implementation Steps:**
  1. Check the region of the AMB network (e.g., us-east-1).
  2. Ensure client applications are also hosted in us-east-1.
- **Estimated Savings:** 100% of cross-region egress fees.
- **Risk Level:** Low
- **Implementation Scope:** Architect
- **Prerequisites:** Multi-region deployment capabilities.

#### AMB-18. Optimize Hyperledger Fabric Payload Sizes
- **What:** Minimize the size of data written to the Hyperledger Fabric ledger to reduce storage and cross-AZ egress costs.
- **Why It Saves Money:** Blockchain storage grows indefinitely. Writing large payloads (e.g., images, large JSONs) to the ledger bloats AMB standard storage ($0.10/GB-month) and increases peer-to-peer gossip network transfer costs.
- **Implementation Steps:**
  1. Store large payloads in Amazon S3.
  2. Write only the S3 URI and a cryptographic hash (SHA-256) of the payload to the Hyperledger ledger.
- **Estimated Savings:** 80-99% on AMB storage and data transfer costs.
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Integration with Amazon S3.

---

## Cross-Service Synergies
- **Amazon S3:** Use S3 to store large off-chain payloads, storing only cryptographic hashes on AMB to save expensive blockchain storage costs (Strategy AMB-18).
- **Amazon ElastiCache / Redis:** Implement caching in front of AMB Query APIs to reduce duplicate API request billing (Strategy AMB-10).
- **AWS QLDB:** A serverless alternative for centralized verifiable ledgers, bypassing AMB Hyperledger membership fees entirely (Strategy AMB-11).
- **AWS CloudFormation / Terraform:** Essential for scheduling the automated teardown of ephemeral AMB dev/test nodes (Strategy AMB-13).

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
- **Line Items:** `Amazon Managed Blockchain`
- **Operations:** `RunNode`, `MembershipFee`, `QueryData`, `APICalls`
- **Cost Metrics:** UnblendedCost, UsageQuantity

### B. CloudWatch Metrics
- `CPUUtilization` and `MemoryUtilization` for dedicated AMB nodes.
- `RequestCount` for API/RPC traffic to identify unused nodes.

### C. AWS Config / Trusted Advisor
- AWS Config rules to flag Hyperledger memberships that have been active for > 30 days in dev/test accounts.

### D. Company Policies
- Environment scheduling policies (e.g., all dev environments must tear down at 6 PM).
- Guidelines on acceptable use of AMB vs QLDB.

### E. IaC (Optional)
- Terraform state files (`aws_managedblockchain_member`, `aws_managedblockchain_node`) to verify instance types across environments.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "AMB-1",
  "service": "Amazon Managed Blockchain",
  "category": "Waste Elimination",
  "strategy_name": "Delete Idle Hyperledger Fabric Memberships",
  "monthly_savings_usd": 219.00,
  "risk_level": "Medium",
  "effort_level": "Low",
  "action_required": "Delete membership for non-production network ID n-XXXXXXXX via AWS Console.",
  "resource_ids": ["n-XXXXXXXX"]
}
```

### Summary Report Table

| Finding ID | Strategy | Target Resource | Estimated Savings | Risk |
|------------|----------|-----------------|-------------------|------|
| AMB-1 | Delete Idle Hyperledger Memberships | Dev Network n-XXX | $219/mo | Medium |
| AMB-4 | Rightsize Dev/Test Nodes | Node nd-XXX | $128/mo | Low |
| AMB-9 | Migrate to AMB Query Serverless | App Tier Read Queries | $198/mo | Medium |
