# Amazon EC2 - Reasoning & Heuristic Guidelines

This document provides the qualitative context, edge cases, and business reasoning for the LLM to apply when evaluating EC2 resources as part of the LangGraph Efficiency Agent.

## 1. Analysis Node Guidelines

When evaluating resources flagged by the `computational.json` rules, apply the following contextual reasoning:

### Risk Assessment & Edge Cases
- **Disaster Recovery (DR) Nodes:** If a node is tagged as `Environment: DR` or `Role: Standby`, ignore low CPU utilization (< 5%). It is expected to be idle until failover.
- **Bastion Hosts / Jump Boxes:** These often have near-zero CPU and network traffic. Do not terminate them if they are the only entry point into a private subnet.
- **Graviton Migration Safety:** While `computational.json` flags x86 Linux instances for Graviton4 migration, you must assess risk. If the workload uses proprietary binaries without ARM64 builds, flag the migration as **High Risk** and request engineering review.
- **EBS Volume Types:** Do not recommend moving high-throughput database workloads (e.g., heavily utilized `io2` or `io1`) to `gp3` without verifying IOPS and throughput caps.

## 2. Validation Node Guidelines

When checking recommendations against organizational policies:
- **Production Safety:** Never recommend termination of instances tagged `Environment: Production` without explicit RACI approval from the `Owner` tag.
- **Commitment Overlap (SP/RI):** If recommending termination of a large fleet, check if those instances are currently covered by standard (non-flexible) Reserved Instances. Terminating them without trading the RI will result in zero net savings.

## 3. Backlog Builder & New Lever Detection

### New Lever Detection
If you identify a cost-cutting mechanism that is **NOT** present in the standard lever database (e.g., a newly released AWS pricing update), output a `NEW_LEVER_CANDIDATE` block:
- **Example:** Public IPv4 Hourly Tax Elimination ($0.005/hr introduced in 2024). If you see high charges for IPv4 on private nodes, flag this as a new lever candidate.

### Remediation Plan Generation
For high-confidence recommendations, provide step-by-step remediation plans.
- **Example (gp2 to gp3):** Note that this is a zero-downtime online migration.
- **Example (Instance Type Change):** Note that this requires a brief maintenance window (stop/start). Include safety steps like creating a pre-migration AMI snapshot.
