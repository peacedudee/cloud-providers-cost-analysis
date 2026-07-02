# AWS Service Cost Research: Amazon EKS (Elastic Kubernetes Service)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon EKS is a managed Kubernetes service that simplifies running Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane. EKS is highly powerful but has a higher baseline cost compared to ECS due to its flat cluster management fee and Kubernetes-specific networking overhead.

---

## 2. Billing Mechanics
EKS billing is divided into two distinct parts:
1.  **EKS Control Plane Fee:** A flat hourly rate for each active EKS cluster you create.
2.  **Compute Worker Nodes:** The compute, storage, and networking resources you provision to run your Kubernetes workloads (pods).

---

## 3. Key Cost Dimensions

### A. EKS Control Plane Fee & Extended Support
*   **Standard Control Plane Charge:** AWS charges a flat **$0.10 per hour** (approximately **$73.00/month**) for each active Amazon EKS cluster.
*   **Extended Support Surcharge:** Standard EKS support is active for 14 months after a Kubernetes version's release. If you run a cluster on a Kubernetes version in **Extended Support** (versions older than 14 months), AWS charges an additional **$0.60 per hour** per cluster. This brings the total control plane fee to **$0.70 per hour** (approximately **$511.00/month**), making upgrades a critical cost lever.
*   **Scope:** This charge applies the moment the cluster is created, even if it has zero worker nodes or runs no workloads.

### B. Worker Nodes (EC2 vs. Fargate)
Kubernetes pods must run on physical or virtual compute nodes:
*   **Managed Node Groups (EC2):** You pay standard EC2 rates for the instances in your node groups. You are billed for the full instance capacity regardless of pod utilization.
*   **Fargate Profiles (Serverless):** You pay for the CPU and memory resources requested by your Kubernetes pods, plus a 4-vCPU-second and 4-GB-second minimum. Fargate runs pods on dedicated serverless micro-VMs.
    *   *Fargate Kubernetes Overhead:* When running EKS pods on Fargate, AWS automatically adds a **256 MB memory overhead** to your pod's resource requests for the Kubernetes agent (`kubelet`, `kube-proxy`, and `csi-driver`). 
    *   *Resource Sizing Rounding:* The combined pod request + 256 MB memory overhead is rounded up to the nearest Fargate resource configuration tier (e.g. 0.25 vCPU/0.5 GB, 0.25 vCPU/1 GB, etc.). You are billed for the rounded-up Fargate task size, which can result in paying for more RAM than specified in the pod spec.

### C. Pod-to-Pod Cross-AZ Data Transfer
In Kubernetes, workloads are often spread across multiple Availability Zones for high availability.
*   **The Toll:** Traffic between pods running on different worker nodes in *different* AZs is billed at **$0.01 per GB in each direction** ($0.02/GB roundtrip).
*   If your microservices are constantly talking to each other across AZs (e.g., frontend pod in AZ-A calling backend pod in AZ-B), this data transfer fee can become a primary cost driver.

### D. Automatically Provisioned Load Balancers
In Kubernetes, exposing a service via `Type: LoadBalancer` automatically provisions an AWS Elastic Load Balancer (usually an NLB or ALB).
*   Each service exposes a public endpoint that adds an hourly charge (~$16.42/month in base fees) plus data processing costs. If you create 20 services with `Type: LoadBalancer`, you are paying for 20 separate load balancers.

### E. Persistent Volumes (PV)
Stateful applications in EKS mount Persistent Volumes.
*   Typically backed by **Amazon EBS** (billed per GB provisioned) or **Amazon EFS** (billed per GB consumed). Standard storage rates apply.

### F. Control Plane Logging
EKS allows you to export logs (API server, audit, authenticator, controller manager, scheduler) to CloudWatch.
*   Audit logs in a busy cluster can easily generate gigabytes of data per day. Standard CloudWatch ingestion rates ($0.50/GB) apply, making control plane logging a frequent budget hotspot.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Dimension | Billing Basis | Rate (us-east-1) | Monthly Cost (Est. per unit) |
|-------------------|---------------|------------------|------------------------------|
| **EKS Control Plane (Standard)** | Per cluster hour | **$0.10 / hour** | ~$73.00 / month |
| **EKS Control Plane (Extended)** | Per cluster hour | **$0.70 / hour** | ~$511.00 / month |
| **Worker Nodes (EC2)** | Per running VM hour | *See EC2 rates* | Varies by instance type |
| **Worker Nodes (Fargate)** | Per pod vCPU/Memory second | *See Fargate rates* | Varies by pod CPU/Memory specs |
| **EBS Storage (gp3)** | Per GB-month provisioned | **$0.08 / GB-month** | $8.00 per 100 GB |
| **EFS Storage (Standard)** | Per GB-month consumed | **$0.30 / GB-month** | $30.00 per 100 GB |

---

## 5. AWS Free Tier Coverage
*   **EKS Control Plane:** No free tier coverage.
*   **Worker Nodes / Load Balancers:** Eligible for standard EC2, EBS, and ELB free tiers (if configurations match micro-instance limits).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Accumulated Cluster Fees (Idle Clusters):** Leaving unused EKS clusters running in dev, staging, or test environments. Multiple sandbox clusters running 24/7 can quickly add up (e.g., 5 clusters = ~$365/month idle baseline).
*   **Service LoadBalancer Proliferation:** Creating a new AWS Load Balancer for every Kubernetes service instead of using a shared **Ingress Controller** (like NGINX Ingress or AWS Load Balancer Controller).
*   **Cross-AZ Data Transfer Storms:** Distributing pods randomly across AZs without zone-awareness, leading to constant database queries and API calls traveling across AZ boundaries.
*   **Unused EBS Volumes from PVCs:** Deleting a Kubernetes namespace or deployment does *not* automatically delete the underlying EBS volumes if the PersistentVolumeClaim (PVC) retention policy is set to `Retain`. These volumes sit orphaned and charge storage fees.

---

## 7. Actionable Cost Optimization Strategies
1.  **Implement an Ingress Controller:** Consolidate external ingress. Use a single shared **Application Load Balancer (ALB)** with an Ingress Controller (like NGINX or the AWS Load Balancer Controller) to route traffic to multiple internal services using host- or path-based rules.
2.  **Adopt Karpenter for Node Autoscaling:** Replace the traditional Cluster Autoscaler with **Karpenter**. Karpenter is a high-performance Kubernetes node autoscaler that directly provisions the cheapest, right-sized EC2 instances for pending pods, bin-packs efficiently, and terminates underutilized nodes in seconds.
3.  **Enforce Topology-Aware Routing:** Turn on **Topology-Aware Hints** (or Topology-Aware Routing) in Kubernetes. This configuration instructs Service routing to prefer endpoints (pods) within the *same* Availability Zone, keeping traffic local and eliminating inter-AZ transfer charges.
4.  **Consolidate Non-Production Environments:** Instead of spinning up separate EKS clusters for Dev, QA, and Staging (adding $73/month per cluster), consolidate them into a single EKS cluster. Use **Kubernetes Namespaces** and network policies to isolate environments securely.
5.  **Use Spot Instances for Worker Nodes:** Run stateless node groups on EC2 Spot instances. Karpenter supports mixed-instance policies that can automatically fallback to On-Demand instances if Spot capacity is unavailable.
6.  **Schedule Dev Clusters:** Use open-source tools (like *Kubedown* or *Kube-green*) to scale down deployment replicas to zero and scale down worker node pools to zero during nights and weekends.
