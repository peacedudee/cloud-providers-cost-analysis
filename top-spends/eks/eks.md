# AWS Service Cost Research: Amazon EKS (Elastic Kubernetes Service)

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon EKS is a managed Kubernetes service that simplifies running Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane. EKS is highly powerful but has a higher baseline cost compared to ECS due to its flat cluster management fee, version support tiers, and Kubernetes-specific networking overhead.

---

## 2. Billing Mechanics
EKS billing is divided into two distinct parts:
1.  **EKS Control Plane Fee:** A flat hourly rate for each active EKS cluster.
    *   *Standard Support:* Billed during the first 14 months of a version's release.
    *   *Extended Support:* Billed once a version passes 14 months of age.
2.  **Compute Worker Nodes:** Billed based on the worker node deployment mode:
    *   *Managed Node Groups (EC2):* Standard EC2 compute and storage rates.
    *   *Fargate Profiles (Serverless):* Billed per second for the vCPU and memory allocated to active pods.
    *   *EKS Auto Mode:* AWS automatically manages worker nodes using a Karpenter-based system. You pay standard EC2 rates plus an Auto Mode node management fee.
    *   *EKS Hybrid Nodes:* Billed per vCPU-hour for on-premises or edge servers connected to the EKS control plane.

---

## 3. Key Cost Dimensions

### A. EKS Control Plane Fee & Extended Support
*   **Standard Support Fee:** AWS charges a flat **$0.10 per hour** (~$73.00/month) for each active EKS cluster.
*   **Extended Support Fee:** For clusters running Kubernetes versions older than 14 months, AWS charges an additional **$0.60 per hour**, bringing the total control plane fee to **$0.70 per hour** (~$511.00/month).

### B. Worker Nodes (EC2, Fargate, and Auto Mode)
*   **EC2 Node Groups:** Billed standard EC2 rates. You pay for the full instance capacity regardless of pod utilization.
*   **Fargate Profiles:** Billed per second for pod allocations.
    *   *Memory Overhead:* AWS automatically adds a **256 MB memory overhead** to each pod's resource requests for Kubernetes agents (`kubelet`, `kube-proxy`, and CSI drivers).
    *   *Resource Rounding:* The sum of pod request + 256 MB memory overhead is rounded up to the nearest Fargate configuration tier.
*   **EKS Auto Mode:**
    *   *EC2 Costs:* Billed standard EC2 rates (eligible for Spot, RIs, and Savings Plans).
    *   *Node Management Fee:* Billed per second (with a 1-minute minimum) at **~10–12% of the On-Demand price** of the underlying instance type launched.

### C. Pod-to-Pod Cross-AZ Data Transfer
*   **The Cost:** Traffic between pods running on different worker nodes in *different* Availability Zones (AZs) is billed at **$0.01 per GB in each direction** ($0.02/GB roundtrip).

### D. Automatically Provisioned Load Balancers
*   Exposing services via `Type: LoadBalancer` automatically provisions an AWS Elastic Load Balancer (ALB/NLB), charging base hourly rates (~$16.42/month) plus data processing fees for **each** balancer.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Dimension | Billing Basis | Rate (us-east-1) | Monthly Cost (Est. per unit) |
|-------------------|---------------|------------------|------------------------------|
| **EKS Control Plane (Standard)** | Per cluster hour | **$0.10 / hour** | ~$73.00 / month |
| **EKS Control Plane (Extended)** | Per cluster hour | **$0.70 / hour** | ~$511.00 / month |
| **EKS Auto Mode Node Fee** | Per running instance second | **~10-12% of instance On-Demand rate**| Varies by instance size |
| **Worker Nodes (EC2)** | Per running VM hour | *See EC2 rates* | Varies by instance type |
| **Worker Nodes (Fargate)** | Per pod vCPU/Memory second | *See Fargate rates* | Varies by pod CPU/Memory specs |
| **EBS Storage (gp3)** | Per GB-month provisioned | **$0.08 / GB-month** | $8.00 per 100 GB |

### Example Monthly Cost Calculation (Extended Support Impact)
*Workload: A developer maintains 3 idle staging EKS clusters. Two are running the latest standard support Kubernetes version, and one runs a legacy version under Extended Support.*

*   **Standard Support Clusters Cost:**
    $$\text{Standard Fee} = 2\text{ clusters} \times \$0.10/\text{hour} \times 730\text{ hours} = \$146.00$$
*   **Extended Support Cluster Cost:**
    $$\text{Extended Fee} = 1\text{ cluster} \times \$0.70/\text{hour} \times 730\text{ hours} = \$511.00$$
*   **Total Monthly Control Plane Cost:** **$657.00/month** (The legacy cluster represents 77% of the total control plane spend).

---

## 5. AWS Free Tier Coverage
*   **EKS Control Plane:** No free tier coverage.
*   **Worker Nodes / Load Balancers:** Eligible for standard EC2, EBS, and ELB free tiers (if configurations match micro-instance limits).

---

## 6. Common Cost Hotspots & Pitfalls
*   **The Extended Support Trap:** Operating legacy Kubernetes versions past standard support, multiplying the control plane fee by **7x** ($0.10/hr to $0.70/hr).
*   **Service LoadBalancer Proliferation:** Creating a new AWS Load Balancer for every Kubernetes service instead of using a shared Ingress Controller.
*   **Cross-AZ Data Transfer Storms:** Distributing pods randomly across Availability Zones, forcing inter-pod queries to travel across AZs ($0.01/GB).
*   **Unused EBS Volumes from PVCs:** Deleting namespaces or deployments without deleting the underlying EBS volumes when PVC retention is set to `Retain`.

---

## 7. Actionable Cost Optimization Strategies
1.  **Strict Kubernetes Version Lifecycles:**
    *   Implement a schedule to upgrade EKS clusters at least once a year.
    *   **The Savings:** Bypasses EKS Extended Support fees, saving **$438.00/month per cluster**.
2.  **Consolidate Services onto a Shared Ingress Controller:**
    *   Use a single shared Application Load Balancer (ALB) with the **AWS Load Balancer Controller** or NGINX Ingress. Route external traffic to multiple services using host- or path-based rules.
    *   **The Savings:** Eliminates the flat ~$16.42/month fee and IPv4 address charges ($3.65/month) for dozens of independent load balancers.
3.  **Enable Topology-Aware Routing:**
    *   Configure Topology-Aware Hints in Kubernetes to instruct Service routing to prefer endpoints within the same Availability Zone.
    *   **The Savings:** Eliminates the **$0.01/GB** inter-AZ data transfer charge for intra-cluster traffic.
4.  **Adopt EKS Auto Mode or Karpenter for Nodes:**
    *   Use **Karpenter** (or enable **EKS Auto Mode**) to dynamically provision right-sized worker nodes, bin-pack containers efficiently, and terminate idle hosts immediately.
5.  **Schedule Developer Clusters to Scale to Zero:**
    *   Use tools like *kube-green* to scale deployment replicas and worker node pools to zero during nights and weekends.
