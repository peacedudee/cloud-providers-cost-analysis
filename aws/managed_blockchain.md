# AWS Service Cost Research: Amazon Managed Blockchain (AMB)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Managed Blockchain (AMB) is a fully managed service that lets you join public blockchain networks (Ethereum, Bitcoin) via **AMB Access**, query multi-chain data (token balances, smart contract events) via **AMB Query**, or set up private enterprise Hyperledger Fabric networks without managing underlying node infrastructure. AMB automates node provisioning, network scaling, peer discovery, and Certificate Authority management. AMB is billed per node instance-hour, request volume, and network membership.

---

## 2. Billing Mechanics
Amazon Managed Blockchain billing is structured across three product offerings:
1.  **AMB Access (Public & Private Peer Nodes):**
    *   *Node Hours:* Billed per hour per running peer node ($0.096 to $0.272 per hour).
    *   *JSON-RPC Requests:* Billed per 1 Million API requests ($0.10 per 1 Million requests).
2.  **AMB Query (Serverless Multi-Chain Data APIs):**
    *   Billed per 1 Million API query requests ($0.50 per 1 Million queries).
3.  **AMB Enterprise (Hyperledger Fabric):**
    *   *Network Membership Fee:* Billed per hour per network member ($0.30 per hour for Starter Edition = ~$219.00/mo).
    *   *Peer Node Rate:* Billed per hour (`bc.t3.small` at $0.096/hr; `bc.m5.large` at $0.384/hr).

---

## 3. Key Cost Dimensions

| AMB Product / Node Type | Billed Unit | Rate (us-east-1) | Monthly Price (24/7) |
|-------------------------|-------------|------------------|----------------------|
| **AMB Access (`bc.c5.xlarge`)**| Node Hour | **$0.272 / hour** | **$198.56 / month** |
| **AMB Query API** | Per 1M queries | **$0.50 / 1M queries** | **$0.50** (1M lookups) |
| **Hyperledger Membership**| Per member-hour | **$0.300 / hour** | **$219.00 / month** |
| **Peer Node (`bc.t3.small`)** | Node Hour | **$0.096 / hour** | **$70.08 / month** |

---

## 4. Detailed Pricing Rates (us-east-1)

*   **Node Rate (`bc.c5.xlarge`):** $0.272 per hour.
*   **Node Rate (`bc.t3.small`):** $0.096 per hour.
*   **JSON-RPC Request Rate:** $0.10 per 1 Million requests.
*   **AMB Query API Rate:** $0.50 per 1 Million queries.

---

## 5. AWS Free Tier Coverage
*   **Amazon Managed Blockchain Free Tier:** Includes **250 free hours of `bc.t3.small` peer node usage** and 1 Million JSON-RPC requests per month for 2 months for new accounts.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Provisioning Dedicated Peer Nodes for Simple Data Lookups:**
    *   Running dedicated 24/7 AMB Ethereum peer nodes (`bc.c5.xlarge` at $198.56/mo) just to query occasional ERC-20 token balances or block heights.

---

## 7. Actionable Cost Optimization Strategies
1.  **Use AMB Query Serverless APIs Instead of Provisioning Dedicated Nodes:**
    *   Use **AMB Query APIs ($0.50 per 1M queries)** for reading multi-chain data (token balances, transaction histories).
    *   *Why:* AMB Query is serverless, eliminating the need to manage $198.56/mo dedicated peer nodes.
    *   **The Savings:** Slashes blockchain data lookup costs from **$198.56/mo down to fractions of a dollar**.
2.  **Delete Dev/Test Hyperledger Fabric Memberships:** Delete non-production Hyperledger network memberships and peer nodes during idle development periods to stop the $0.30/hr membership fee.
