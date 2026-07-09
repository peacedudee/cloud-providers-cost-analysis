# Cost-Cutting Playbook: AWS Schema Conversion Tool
> **Companion File:** [sct.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/sct/sct.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Schema Conversion Tool (SCT) is a 100% free downloadable desktop application that automates the conversion of heterogeneous database schemas, database code (stored procedures, functions), and data warehouse structures from expensive commercial engines (Oracle, SQL Server, Teradata) to cost-effective open-source AWS targets (Amazon Aurora, RDS PostgreSQL/MySQL, Amazon Redshift). While the tool itself incurs no licensing or usage fees, it is a primary driver for massive cloud cost savings. The cost-cutting strategies in this playbook focus on maximizing the use of SCT to eliminate commercial database licensing fees, avoiding expensive manual database refactoring consulting, and optimizing the ancillary infrastructure (EC2, network transfer) used to host SCT and its data extraction agents during the migration process.

## Strategy Categories
### 1. Waste Elimination
*   **SCT-01: Run SCT on Local Machines:** Instead of provisioning dedicated Amazon EC2 instances to run the SCT client application, install and run it locally on developers' machines (available for Windows, macOS, and Linux) to completely avoid compute costs.
*   **SCT-02: Terminate Post-Migration Infrastructure:** Ensure that any EC2 instances provisioned specifically for running SCT data extraction agents for data warehouse migrations are immediately terminated once the migration is complete.
*   **SCT-03: Avoid Manual Consulting Fees:** Maximize the automated schema and code conversion capabilities of SCT to reduce or eliminate the need for expensive third-party database consultants or hundreds of manual engineering hours.
*   **SCT-04: Exclude Unused Objects:** Use the SCT Assessment Report to identify obsolete or unused database objects, schemas, or tables in the source database and deliberately exclude them from the migration, saving target storage and compute costs.

### 2. Rightsizing
*   **SCT-05: Rightsize Extraction Agent Instances:** If using EC2 instances to host SCT Data Extraction Agents for large data warehouse migrations, monitor their CPU and Memory usage and rightsize the instances to avoid over-provisioning compute resources.
*   **SCT-06: Target Database Rightsizing via Assessment:** Use the performance and complexity metrics from the SCT Migration Assessment Report to correctly estimate the required size of the target database (Amazon RDS/Aurora), rather than blindly matching the source hardware.
*   **SCT-07: Consolidate Extraction Agents:** For smaller data warehouse migrations, consolidate multiple SCT extraction agents onto fewer, appropriately sized EC2 instances instead of running a separate instance for every agent.

### 3. Commitment Discounts
*   **SCT-08: Delay Target Instance RIs:** When migrating to Amazon RDS or Aurora using SCT, delay purchasing Reserved Instances (RIs) for the target database until *after* the conversion is complete, the data is migrated, and the actual performance footprint of the converted schema has been profiled in production for 2-4 weeks.
*   **SCT-09: Spot Instances for Batch Extraction:** If conducting asynchronous, fault-tolerant batch data extractions from legacy data warehouses using SCT agents, consider running those agents on Amazon EC2 Spot Instances to save up to 90% on compute costs during the migration.

### 4. Architecture Changes
*   **SCT-10: Eliminate Commercial DB Licenses:** Convert legacy commercial databases (Oracle, Microsoft SQL Server) to open-source target engines (Amazon Aurora PostgreSQL/MySQL) to eliminate commercial software license fees, which can cost $3,700+ per core.
*   **SCT-11: Utilize SCT Extension Packs:** Apply SCT Extension Packs to automatically emulate specialized commercial database functions (e.g., Oracle's `NVL`, `DECODE`) in PostgreSQL, avoiding the exorbitant architectural refactoring costs required to manually rewrite that logic.
*   **SCT-12: Modernize Data Warehouses:** Use SCT to convert legacy on-premises data warehouses (Teradata, Netezza, IBM DB2) to Amazon Redshift, benefiting from cloud-native scalability, separation of compute and storage, and lower overall total cost of ownership.
*   **SCT-13: Re-architect Complex Procedures to Serverless:** For the 10-20% of stored procedures that SCT flags as unable to be converted automatically, evaluate re-architecting them into AWS Lambda functions instead of spending heavy engineering hours writing complex PL/pgSQL.
*   **SCT-14: Migrate NoSQL Databases:** Utilize SCT to migrate from self-managed NoSQL databases like Apache Cassandra to fully managed Amazon DynamoDB for a scalable, serverless pricing model.

### 5. Scheduling & Auto-Scaling
*   **SCT-15: Schedule Off-Peak Extraction:** Schedule SCT Data Extraction Agents to run their heavy queries against the source data warehouse during off-peak hours. This prevents the need to scale up the source hardware to handle migration-induced load.
*   **SCT-16: Auto-Stop Migration EC2 Instances:** Implement scheduling scripts (e.g., AWS Instance Scheduler) to automatically stop any EC2 instances hosting SCT or its extraction agents during weekends or non-working hours if the migration team is not actively running conversions.
*   **SCT-17: Auto-Scale Extraction Fleets:** For massive data warehouse migrations, put SCT extraction agents into an EC2 Auto Scaling Group to scale the number of agents horizontally based on the backlog of extraction tasks, spinning them down when idle.

### 6. Pricing Model Optimization
*   **SCT-18: Target Aurora Serverless v2:** If the SCT Assessment Report identifies that the source database workload is highly variable and spiky, select Amazon Aurora Serverless v2 as the conversion target to pay only for the database capacity actively used, rather than a provisioned peak capacity.
*   **SCT-19: Evaluate Open-Source RDS vs. Aurora:** Based on the complexity assessed by SCT and business requirements, choose the cheaper Amazon RDS open-source engine (e.g., RDS PostgreSQL) over Amazon Aurora if the high-availability and extreme performance features of Aurora are not strictly required.

### 7. Network & Data Transfer Optimization
*   **SCT-20: Localize Migration Agents:** Deploy SCT extraction agents in the same AWS Region (and ideally the same VPC) as the target Amazon Redshift cluster or S3 bucket to minimize cross-region or internet data transfer costs.
*   **SCT-21: Integrate with AWS Snowball:** For multi-terabyte or petabyte data warehouse migrations, configure SCT data extraction agents to write data directly to an AWS Snowball Edge device rather than transferring it over the internet or Direct Connect, significantly reducing outbound bandwidth utilization and time.
*   **SCT-22: Enable Agent Compression:** Ensure that data compression is enabled in SCT extraction agents when extracting and sending data to Amazon S3/Redshift to reduce network transfer time and minimize intermediate S3 storage costs.

---
## Cross-Service Synergies
*   **AWS Database Migration Service (DMS):** SCT converts the structural schema and code, while DMS handles the ongoing replication and migration of the actual row data. Optimizing them together ensures a fast, cost-effective migration.
*   **Amazon EC2:** Hosting SCT extraction agents efficiently on right-sized or Spot EC2 instances reduces the overhead compute cost of the migration itself.
*   **Amazon S3 & Redshift:** SCT optimizes the target schema for Redshift and can compress data stored in S3 during the extraction phase, reducing storage footprints.
*   **AWS Snow Family:** Using Snowball alongside SCT extraction agents bypasses expensive network data transfer for massive data warehouse migrations.

---
## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
*   `lineItem/ProductCode`: Filter by `AmazonEC2` and `AWSDataTransfer` to find costs associated with infrastructure provisioned specifically for the migration.
*   `resourceTags/user:Project`: Tags indicating instances used for "Database-Migration" or "SCT-Agents".

### B. CloudWatch Metrics
*   `CPUUtilization` and `MemoryUtilization` for EC2 instances running SCT or extraction agents to determine rightsizing opportunities.
*   `NetworkOut` and `NetworkIn` for tracking the volume of data being moved by extraction agents.

### C. AWS Config / Trusted Advisor
*   Identify long-running EC2 instances tagged for migration that should be terminated (Low Utilization EC2 Instances check).
*   Verify unassociated Elastic IPs left behind after terminating SCT extraction instances.

### D. Company Policies
*   Dates for upcoming commercial database license renewals (Oracle/SQL Server) to prioritize conversion timelines using SCT.
*   Security requirements dictating where SCT can be installed (local vs. cloud VDI).

### E. IaC (Optional)
*   Terraform modules or CloudFormation templates used to deploy the target databases (RDS/Aurora/Redshift) to ensure they match the optimized recommendations from the SCT assessment.

---
## Output Schema
### Finding Record (JSON)
```json
{
  "finding_id": "SCT-001",
  "category": "Waste Elimination",
  "service": "AWS Schema Conversion Tool",
  "title": "Terminate post-migration SCT infrastructure",
  "description": "EC2 instance running SCT extraction agents is still active but idle post-migration.",
  "severity": "Medium",
  "estimated_savings_mrr": 150.00,
  "action_type": "Terminate",
  "resource_id": "i-0abcd1234efgh5678",
  "resolution_steps": [
    "Verify database migration is completely finished.",
    "Terminate the EC2 instance hosting the SCT extraction agent.",
    "Delete any unattached EBS volumes or Elastic IPs."
  ]
}
```

### Summary Report Table

| Finding ID | Category | Title | Severity | Estimated Monthly Savings | Resource ID |
|------------|----------|-------|----------|---------------------------|-------------|
| SCT-001 | Waste Elimination | Terminate post-migration SCT infrastructure | Medium | $150.00 | `i-0abcd1234efgh5678` |
| SCT-002 | Architecture Changes | Eliminate Commercial DB Licenses | High | $7,400.00 | `Oracle-DB-Prod` |
| SCT-003 | Rightsizing | Rightsize Extraction Agent Instances | Low | $45.00 | `i-09876xyz54321` |
