# Google Kubernetes Engine (GKE) Cost Optimization & Research

Google Kubernetes Engine (GKE) is a highly scalable managed Kubernetes service. While it makes orchestration seamless, it is easy to accumulate massive waste due to misconfigured autoscalers, over-provisioned pod requests, and unmanaged idle node overhead. This file breaks down GKE's cost components and optimization strategies.

---

## 1. GKE Billing Modes: Standard vs. Autopilot

GKE offers two operation modes, which use completely different billing paradigms:

### A. GKE Standard (Infrastructure-centric Billing)
* **What you pay for:** You pay for the underlying Google Compute Engine VMs (nodes), Persistent Disks, GPUs, and load balancers that you provision in your cluster, **regardless of whether pods are actually running on them**.
* **Cluster Management Fee:** Flat fee of **$0.10 per cluster per hour** ($73/month). One zonal cluster's fee is waived per billing account.
* **Cost Risk:** High. If your pods only use 20% of your node's CPU, you still pay 100% of the node's cost. You bear the cost of compute headroom and idle capacity.

### B. GKE Autopilot (Resource-centric Billing)
* **What you pay for:** You pay **only for the CPU, memory, and ephemeral storage requested by your running pods**. You are billed per second of pod execution.
* **Cluster Management Fee:** Billed at the same **$0.10 per cluster per hour**. The free tier (one free cluster per billing account, up to $74.40/month in credits) **does apply** to Autopilot clusters.
* **System Overhead:** System pods (kube-system, logging agents, ingress controllers) and node-level OS overhead are **completely free**.
* **Cost Risk:** Medium. If developers request too much CPU/memory in their deployment manifests, you pay for that waste, even if the application code is idle.

---

## 2. Workload Right-sizing (Pods)

Workload right-sizing is the most impactful cost optimization in Kubernetes.

### The Request vs. Limit Pitfall
* **Requests:** The amount of CPU and Memory guaranteed to the pod. Kubernetes uses this to schedule the pod onto a node. **In GKE Standard, requests dictate node capacity. In GKE Autopilot, requests dictate your direct bill.**
* **Limits:** The maximum CPU and Memory a pod can consume.
* **The Waste:** Teams often set Pod Requests equal to peak workload spikes or arbitrary "safe" values, leading to massive idle capacity.

### Rightsizing Strategy
1. **Vertical Pod Autoscaler (VPA):**
   * Deploy the VPA in **`Off` (Recommendation)** mode.
   * VPA monitors actual resource usage and recommends the optimal `CPU requests` and `Memory requests`.
   * For non-critical services (dev/test), configure VPA in **`Auto`** mode to automatically resize pods on restart.
2. **Horizontal Pod Autoscaler (HPA):**
   * Scale the *number* of pods based on CPU/Memory utilization or custom metrics (e.g. Pub/Sub queue depth).
   * **Cost Tip:** Set a reasonable minimum pod count (e.g., 1 or 2 instead of 5 for non-prod) to allow down-scaling during idle periods.

---

## 3. GKE Standard Cluster Node Optimization

If running GKE Standard, you must manage node utilization aggressively:

### A. Configure Cluster Autoscaler (CA)
* The Cluster Autoscaler dynamically adds or removes node instances from your node pools based on pending pods.
* **Tactic: Optimize Autoscaler Profile**
  * Change the CA profile from `balanced` (default) to **`optimize-utilization`**.
  * This profile prioritizes packing pods tightly on fewer nodes and aggressively scaling down underutilized nodes, at the expense of slightly longer scale-up times.

### B. Node Auto-Provisioning (NAP)
* Standard Cluster Autoscaler only scales existing node pools. If a pod requires a specific resource shape (e.g. high memory), and no pool matches it, the pod remains pending.
* **Tactic:** Enable Node Auto-Provisioning. NAP automatically creates and deletes node pools with VM shapes tailored to your pending pods' exact requirements, maximizing resource packing.

### C. Spot VM Node Pools
* Create a dedicated node pool comprised of **Spot VMs** (saving up to 91%).
* Use Kubernetes **Taints and Tolerations** and **Node Affinities** to run stateless, batch, or non-production pods on the Spot pool.
* **Caution:** Ensure critical system pods (DNS, Ingress, Cert-Manager) do *not* run on Spot node pools to prevent cluster disruptions.

---

## 4. GKE Autopilot Optimization Tactics

Autopilot eliminates node-level management but shifts the optimization focus entirely to Pod manifests.

* **Minimize Requests:** Since you are billed on requests, audit and lower CPU/memory requirements in all manifests.
* **Use Autopilot Spot Pods:**
  * Add the toleration and node selector for spot to your pod template:
    ```yaml
    spec:
      nodeSelector:
        cloud.google.com/gke-spot: "true"
      tolerations:
      - key: "cloud.google.com/gke-spot"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"
    ```
  * Spot Pods are discounted by up to 60-70% compared to standard Autopilot pods.
* **Tune Ephemeral Storage:** Autopilot pods get 1 GiB of ephemeral storage free. If you request more, you are billed. Only request extra storage if strictly necessary.

---

## 5. Networking & Storage Optimization in GKE

* **Topology-Aware Routing:**
  * When a pod calls a service, Kubernetes can route the request to a pod in a different availability zone, triggering inter-zone data egress charges ($0.01/GB).
  * **Tactic:** Set `trafficDistribution: PreferClose` in your Kubernetes Service definition to keep traffic within the same zone.
* **Shared Load Balancers:**
  * By default, every Kubernetes `Service` of type `LoadBalancer` spins up a separate GCP Load Balancer (approx. $18/month base charge).
  * **Tactic:** Use a single **Ingress Controller** (like NGINX Ingress or GCP Gateway API) to route external traffic to hundreds of internal services through a single load balancer.
* **PV Lifecycle Audit:**
  * When a namespace or a deployment is deleted, the associated `PersistentVolumeClaim` (PVC) might be left behind, keeping the persistent disk active and billing.
  * **Tactic:** Audit orphaned Persistent Volumes in GCP console or CLI regularly.

---

## 6. FinOps Visibility & Chargeback

GKE makes it hard to see costs by team/project because nodes run workloads from multiple namespaces.

* **Enable GKE Cost Allocation (Cost Attribution):**
  * GKE has a built-in feature that exports resource usage (CPU, Memory, Storage) by **Namespace** and **Kubernetes Labels** to the Cloud Billing BigQuery dataset.
  * This allows FinOps teams to construct dashboards (e.g., Looker Studio) showing exactly which microservice or engineering team is driving cluster spend.
