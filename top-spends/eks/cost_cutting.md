# Cost-Cutting Playbook: Amazon EKS (Elastic Kubernetes Service)

> **Companion File:** [eks.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/eks/eks.md)  
> **Last Updated:** July 2026

---

## Executive Summary

Amazon EKS is a managed Kubernetes service with a complex multi-layered billing structure: flat control plane fees ($0.10/hr baseline vs $0.70/hr Extended Support), compute worker node groups (EC2 / Fargate / Auto Mode), pod-to-pod cross-AZ network egress ($0.01/GB), and per-service LoadBalancer provisioning.

This playbook provides **20 actionable strategies** across seven operational categories, yielding an estimated **35–70% reduction in total EKS cluster infrastructure costs**.

### Top 3 Quick Wins (< 1 Day Implementation)
1. **Upgrade Legacy Kubernetes Versions:** Eliminates the **7x EKS Extended Support multiplier** ($0.70/hr vs $0.10/hr), saving $438/month per cluster.
2. **Consolidate Services onto Shared Ingress Controller:** Replaces individual `Type: LoadBalancer` services with 1 shared ALB, saving ~$20/month per service.
3. **Enable Topology-Aware Hints in Kubernetes:** Routes intra-cluster pod traffic locally within the same AZ, eliminating $0.01/GB cross-AZ transfer fees.

---

## Strategy Categories

### 1. Waste Elimination (Zombie Resources)

#### 1. Upgrade EKS Clusters to Avoid Extended Support Surcharges
- **What:** Establish an automated cluster upgrade lifecycle to keep EKS Kubernetes versions within the 14-month Standard Support window.
- **Why It Saves Money:** Running Kubernetes versions older than 14 months incurs **EKS Extended Support**, raising the control plane fee from **$0.10/hour ($73/mo)** to **$0.70/hour ($511/mo)** — a **7x price multiplier** ($438/mo penalty per cluster).
- **Implementation Steps:**
  1. Audit cluster versions: `aws eks list-clusters` and check `version`.
  2. Flag clusters running versions in Extended Support.
  3. Schedule cluster upgrades using EKS managed node group auto-upgrades or Terraform.
- **Estimated Savings:** $438.00 per cluster per month (85.7% control plane cost reduction).
- **Risk Level:** Medium (requires application regression testing against newer Kubernetes APIs).
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CI/CD pipeline compatibility testing.

#### 2. Clean Up Orphaned EBS Persistent Volume Claims (PVCs)
- **What:** Locate and delete unattached EBS volumes created by Kubernetes `PersistentVolumeClaims` where the underlying pod/deployment has been deleted but reclaim policy was set to `Retain`.
- **Why It Saves Money:** Orphaned gp3 EBS PVC volumes bill continuously at $0.08/GB-month with 0 utility.
- **Implementation Steps:**
  1. Run kubectl plugin `kubectl-pv-cleaner` or AWS CLI to find `available` EBS volumes tagged with `kubernetes.io/created-for/pvc-name`.
  2. Delete orphaned volumes: `aws ec2 delete-volume --volume-id vol-xxx`.
- **Estimated Savings:** 100% of orphaned storage costs ($0.08/GB-mo).
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Verification that volume data is no longer needed.

#### 3. Decommission Idle Developer or Staging EKS Clusters
- **What:** Identify and tear down developer test clusters that are no longer actively used.
- **Why It Saves Money:** An idle EKS cluster costs $73/mo for control plane + underlying node group compute.
- **Implementation Steps:**
  1. Audit pod invocation and API request metrics over 30 days.
  2. Terminate unused clusters via Terraform/eksctl.
- **Estimated Savings:** 100% of cluster spend.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Developer confirmation.

---

### 2. Rightsizing

#### 4. Deploy Karpenter for Just-in-Time Node Bin-Packing
- **What:** Replace Cluster Autoscaler and fixed EC2 Managed Node Groups with **Karpenter** for dynamic, right-sized worker node provisioning.
- **Why It Saves Money:** Karpenter evaluates unschedulable pods and provisions the exact instance type/size required, bin-packing pods efficiently and terminating empty nodes immediately, improving node utilization from ~30% to > 80%.
- **Implementation Steps:**
  1. Install Karpenter helm chart in EKS cluster.
  2. Configure Karpenter `NodePool` allowing diverse instance families (`c8g`, `m8g`, `r8g`).
  3. Enable `consolidationPolicy: WhenEmptyOrUnderutilized`.
- **Estimated Savings:** 30–50% reduction in worker node EC2 spend.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** EKS IAM OIDC provider enabled.

#### 5. Right-Size Pod CPU and Memory Requests/Limits
- **What:** Audit Kubernetes pod `resources.requests` and `resources.limits` using Goldilocks or Kubecost, reducing over-allocated pod requests to match actual container usage.
- **Why It Saves Money:** Kubernetes schedules pods based on requested resources, not actual usage. Over-requesting CPU/RAM forces the autoscaler to launch extra EC2 nodes unnecessarily.
- **Implementation Steps:**
  1. Deploy Kubecost or Fairwinds Goldilocks.
  2. Adjust deployment YAML manifests based on 85th percentile usage metrics.
- **Estimated Savings:** 25–45% worker node footprint reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Prometheus / Metrics Server installed.

---

### 3. Commitment Discounts

#### 6. Cover EKS EC2 Worker Nodes with Compute Savings Plans
- **What:** Apply Compute Savings Plans or EC2 Instance Savings Plans to the underlying EC2 worker node pools.
- **Why It Saves Money:** Slashes worker node compute rates by **up to 66–72%**.
- **Implementation Steps:**
  1. Analyze steady-state EKS node pool CPU/RAM baseline.
  2. Purchase Compute Savings Plan for 70% of baseline.
- **Estimated Savings:** 20–66% off On-Demand EC2 rates.
- **Risk Level:** Low.
- **Implementation Scope:** FinOps Team
- **Prerequisites:** Baseline spend stability.

---

### 4. Architecture Changes

#### 7. Migrate EKS Node Pools to AWS Graviton (ARM64)
- **What:** Transition EC2 worker node groups from x86 (`c6i`/`m6i`) to Graviton (`c8g`/`m8g`/`t4g`).
- **Why It Saves Money:** Graviton worker nodes are **20% cheaper** and provide higher container throughput.
- **Implementation Steps:**
  1. Add ARM64 node pools to EKS cluster.
  2. Ensure multi-arch Docker images (`linux/amd64` and `linux/arm64`).
  3. Add `nodeSelector: kubernetes.io/arch: arm64` to deployment manifests.
- **Estimated Savings:** 20% direct worker node cost reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Multi-arch container builds.

#### 8. Consolidate Per-Service LoadBalancers into a Shared Ingress Controller
- **What:** Replace individual `Type: LoadBalancer` Kubernetes services (which provision separate ALBs/NLBs per service) with a single shared **AWS Load Balancer Controller** or NGINX Ingress Controller.
- **Why It Saves Money:** Each provisioned ALB costs ~$16.42/mo base + $3.65/mo public IP fee = ~$20.00/mo. Running 30 individual load balancers costs $600/mo. A single shared Ingress Controller costs $20/mo total.
- **Implementation Steps:**
  1. Install AWS Load Balancer Controller.
  2. Convert `Service` manifests to `Type: ClusterIP` and route external traffic via single `Ingress` object with path/host rules.
- **Estimated Savings:** Saves ~$20.00/month per Kubernetes service.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ingress Controller installation.

#### 9. Evaluate EKS Auto Mode for Node Management
- **What:** Enable **EKS Auto Mode** to let AWS automatically handle worker node provisioning, patching, and Karpenter-based autoscaling.
- **Why It Saves Money:** Optimizes bin-packing dynamically while eliminating manual node management overhead.
- **Implementation Steps:**
  1. Enable EKS Auto Mode on target cluster.
  2. Allow AWS to manage node pools.
- **Estimated Savings:** 20–35% infrastructure efficiency gains.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Compatible Kubernetes version.

---

### 5. Scheduling & Auto-Scaling

#### 10. Implement Kube-Green to Scale Dev/Staging Clusters to Zero Off-Hours
- **What:** Install **kube-green** or **Kube-downscaler** to automatically scale deployment replicas to 0 and terminate worker nodes outside office hours.
- **Why It Saves Money:** Cuts non-production cluster worker node compute spend by **70%** (saving 118 idle hours/week).
- **Implementation Steps:**
  1. Deploy `kube-green` operator via Helm.
  2. Create CRD rule: `SleepMode` active Monday–Friday 7 PM to 7 AM and all weekend.
- **Estimated Savings:** 65–70% off non-prod compute spend.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Non-prod namespace isolation.

---

### 6. Pricing Model Optimization

#### 11. Deploy Spot Instances for Stateless Worker Node Pools
- **What:** Configure Karpenter or Managed Node Groups to use Spot instances for interruption-tolerant pod workloads.
- **Why It Saves Money:** Spot worker nodes offer discounts up to **90% off On-Demand rates** (average 70% savings).
- **Implementation Steps:**
  1. Add Spot instance types to Karpenter NodePool definitions.
  2. Implement `topology.kubernetes.io/zone` spreading and pod disruption budgets (PDBs).
- **Estimated Savings:** 70–90% off worker node compute.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Pods must handle graceful SIGTERM termination within 2 minutes.

#### 12. Mix On-Demand Base with Spot Scaling via Capacity Providers
- **What:** Configure a 20% On-Demand baseline for core pods, scaling remaining capacity on Spot instances.
- **Why It Saves Money:** Blends high availability for baseline traffic with maximum Spot savings for peak scaling.
- **Implementation Steps:**
  1. Configure Karpenter weights or ASG mixed instances policies.
- **Estimated Savings:** 50–70% overall compute discount.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Karpenter configuration.

---

### 7. Network & Data Transfer Optimization

#### 13. Enable Topology-Aware Hints to Eliminate Cross-AZ Pod Traffic
- **What:** Enable `topology.kubernetes.io/zone` hints on Kubernetes services to prefer routing traffic to pods within the same Availability Zone.
- **Why It Saves Money:** Cross-AZ pod-to-pod traffic incurs **$0.01/GB in each direction ($0.02/GB roundtrip)**. Intra-AZ traffic is **100% FREE ($0.00/GB)**.
- **Implementation Steps:**
  1. Set `service.kubernetes.io/topology-mode: Auto` on Service annotations.
  2. Ensure pods are evenly distributed across AZs.
- **Estimated Savings:** 100% of inter-pod cross-AZ data transfer fees ($20/TB).
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Kubernetes 1.24+ cluster.

#### 14. Deploy ECR Interface Endpoints to Avoid NAT Gateway Pull Fees
- **What:** Deploy Interface VPC Endpoints for ECR inside the EKS VPC.
- **Why It Saves Money:** Image pulls from ECR by nodes in private subnets pass through NAT Gateways ($0.045/GB). VPC Endpoints drop transfer fees to $0.01/GB.
- **Implementation Steps:**
  1. Create VPC Endpoints for `ecr.api`, `ecr.dkr`, and S3 (Gateway Endpoint).
- **Estimated Savings:** **78% reduction** in image pull network costs.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** EKS private subnet nodes.

#### 15. Enforce Pod Co-Location via Pod Affinity for High-Throughput Microservices
- **What:** Use `podAffinity` or `podAntiAffinity` rules to place tightly coupled, chatty microservices on the same EC2 worker node or in the same AZ.
- **Why It Saves Money:** Communication between pods on the same node uses loopback/shared memory ($0.00/GB), eliminating network fees entirely.
- **Implementation Steps:**
  1. Add `podAffinity` targeting partner app labels in deployment spec.
- **Estimated Savings:** 100% network fee elimination for co-located services.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Pod resource sizing.

#### 16. Restrict Container Insights Log Verbosity in CloudWatch
- **What:** Configure FluentBit to filter out health check logs (`/healthz`, `/livez`) before sending stdout to CloudWatch Logs.
- **Why It Saves Money:** Health check logs generate millions of unneeded log events ($0.50/GB ingestion fee).
- **Implementation Steps:**
  1. Add grep filter in FluentBit ConfigMap to drop 200 OK health check paths.
- **Estimated Savings:** 30–60% reduction in EKS CloudWatch logging spend.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** FluentBit daemonset access.

#### 17. Deploy Cilium eBPF for High-Performance Local Networking
- **What:** Replace standard AWS VPC CNI with Cilium CNI using eBPF for kube-proxy replacement and local routing.
- **Why It Saves Money:** Accelerates packet processing, reduces CPU overhead on worker nodes, and optimizes local pod-to-pod routing.
- **Implementation Steps:**
  1. Deploy Cilium in eBPF mode on EKS cluster.
- **Estimated Savings:** 10–15% worker node CPU overhead reduction.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CNI migration testing.

#### 18. Use Private IPv4 Pod CIDR Blocks (VPC CNI Secondary Subnets)
- **What:** Configure AWS VPC CNI with `AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG` to use secondary non-routable RFC1918/CGNAT IP blocks (e.g. 100.64.0.0/16) for pods.
- **Why It Saves Money:** Prevents consuming valuable public or primary VPC IPv4 addresses, avoiding public IP surcharges ($0.005/hr).
- **Implementation Steps:**
  1. Attach secondary CIDR to VPC.
  2. Configure `ENIConfig` custom networking in VPC CNI.
- **Estimated Savings:** Prevents IPv4 exhaustion and address fees.
- **Risk Level:** Medium.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** VPC CNI custom networking setup.

#### 19. Set Up Automated EKS Cluster Autoscaler / Karpenter Metrics Monitoring
- **What:** Monitor `karpenter_nodes_created`, `karpenter_nodes_terminated`, and node utilization metrics in Prometheus/Grafana.
- **Why It Saves Money:** Provides real-time visibility into scaling efficiency and cost leaks.
- **Implementation Steps:**
  1. Import Karpenter Grafana dashboard.
- **Estimated Savings:** Continuous optimization enablement.
- **Risk Level:** Zero.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Prometheus/Grafana stack.

#### 20. Optimize EKS Storage Classes (gp2 -> gp3 PVCs)
- **What:** Update the default Kubernetes `StorageClass` from `gp2` to `gp3`.
- **Why It Saves Money:** `gp3` storage costs **$0.08/GB-mo** vs `gp2` at **$0.10/GB-mo** (20% savings).
- **Implementation Steps:**
  1. Deploy EBS CSI Driver.
  2. Create `gp3` StorageClass and set as default: `kubectl annotate storageclass gp3 storageclass.kubernetes.io/is-default-class=true`.
- **Estimated Savings:** 20% direct PVC storage cost reduction.
- **Risk Level:** Low.
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** EBS CSI Driver installed.

---

## Cross-Service Synergies

```
[ EKS Control Plane ] (Upgrade Kubernetes version to avoid $0.70/hr Extended Support)
        │
        ├──(Node Provisioning)──> [ Karpenter + Graviton + Spot ] (70% compute savings)
        │
        ├──(Pod Egress)─────────> [ Topology-Aware Hints ] (Eliminates $0.01/GB cross-AZ fees)
        │
        └──(Ingress Gateway)────> [ Shared ALB Ingress ] (Saves $20/mo per service vs LoadBalancer)
```

---

## Required Input Data for Real-World Analysis

### A. AWS Cost & Usage Report (CUR 2.0)
- `line_item_usage_type`: `EKS-Hours`, `EKS-Hours:extended`, `BoxUsage`, `Fargate-vCPU-Hours`.
- `line_item_resource_id`: EKS Cluster ARN / EC2 Instance ID.

### B. CloudWatch & Kubernetes Metrics
- Metrics: `node_cpu_utilization`, `node_memory_utilization`, `pod_cpu_utilization`, `pod_memory_utilization`.
- Tools: Kubecost, Prometheus metrics exporter.

---

## Output Schema

### Finding Record (JSON)

```json
{
  "finding_id": "EKS-VER-001",
  "service": "EKS",
  "category": "Waste Elimination",
  "resource_id": "arn:aws:eks:us-east-1:123456789012:cluster/dev-microservices-cluster",
  "resource_name": "dev-microservices-cluster",
  "account_id": "123456789012",
  "region": "us-east-1",
  "current_config": {
    "kubernetes_version": "1.26",
    "support_status": "Extended Support",
    "hourly_control_plane_cost_usd": 0.70,
    "monthly_control_plane_cost_usd": 511.00
  },
  "recommended_config": {
    "target_kubernetes_version": "1.30",
    "support_status": "Standard Support",
    "hourly_control_plane_cost_usd": 0.10,
    "projected_monthly_control_plane_cost_usd": 73.00
  },
  "financial_impact": {
    "monthly_savings_usd": 438.00,
    "annual_savings_usd": 5256.00,
    "savings_percentage": 85.7
  },
  "risk_assessment": {
    "risk_level": "Medium",
    "reason": "Requires API deprecation audit prior to cluster version upgrade."
  },
  "implementation": {
    "scope": "Engineer/DevOps",
    "effort_estimate": "2-4 hours",
    "automation_eligible": true
  }
}
```

### Summary Report Table

| Strategy Category | Findings Count | Total Current Monthly Spend | Projected Monthly Savings | Avg Savings % | Primary Risk |
|---|---|---|---|---|---|
| **Waste Elimination (Extended Support)**| 4 | $2,044.00 | $1,752.00 | 85.7% | Medium |
| **Rightsizing (Karpenter/Bin-Pack)**| 12 | $14,500.00 | $5,800.00 | 40.0% | Medium |
| **Architecture (Shared Ingress/Graviton)**| 18 | $6,800.00 | $2,720.00 | 40.0% | Low |
| **Pricing Model (Spot Node Pools)**| 8 | $9,200.00 | $6,440.00 | 70.0% | Medium |
| **Total** | **42** | **$32,544.00** | **$16,712.00** | **51.3%** | -- |
