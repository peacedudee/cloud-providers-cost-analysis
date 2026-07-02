# Cloud GPUs Cost Optimization & Research

Google Cloud allows you to attach NVIDIA GPUs (including T4, L4, A100, H100, and Blackwell) to Compute Engine VMs, GKE clusters, and Vertex AI nodes. GPUs are highly sought-after, expensive accelerators. Idle GPU resources and selecting oversized GPU types for lightweight tasks are the main sources of cloud budget waste in ML pipelines.

---

## 1. GPU Billing Mechanics

GPU charges are added directly to the underlying VM cost:
1. **GPU Hourly Fee:** Billed per-second based on the GPU model and quantity attached to the VM.
2. **Underlying VM Cost:** Standard CPU, RAM, and Disk charges apply.
3. **Provisioning Model:**
   * **On-Demand:** Standard rates.
   * **Spot VMs:** Attach GPUs to Spot VMs to save **60% to 80%** on the GPU hourly fee.
   * **CUDs:** 1-year and 3-year Committed Use Discounts apply to GPU instances for steady-state workloads.

---

## 2. Core Cost-Optimization Levers

### A. Right-Size GPU Models for the Workload
Selecting the wrong GPU tier can waste thousands of dollars.
* **NVIDIA T4 (approx. $0.35/hr):** Best for small model inference, lightweight training, and basic image processing.
* **NVIDIA L4 (approx. $1.00/hr):** Excellent price-to-performance for mid-sized LLM inference, video transcoding, and graphics workloads.
* **NVIDIA A100/H100 (approx. $3.00–$5.00+/hr):** High-cost accelerators designed for massive LLM training and high-concurrency enterprise inference.
* **Action:** Perform benchmarking. Do not default to A100s. If an L4 GPU can meet your target latency (e.g., in a customer chatbot), migrate to it to save **70-80%** on hardware costs.

### B. Configure GKE GPU Pools to Scale to Zero
* **The Waste:** Running a GKE node pool containing 3 active GPU instances 24/7 because of a single service, billing for GPUs during nights and weekends when there is zero traffic.
* **Action:** Enable **Autoscaling** on GKE GPU node pools and set the minimum node count to **0**.
  ```bash
  gcloud container node-pools update gpu-pool \
      --cluster=production-cluster \
      --enable-autoscaling --min-nodes=0 --max-nodes=5
  ```
* **How it works:** When no pods are requesting GPU resources (`resources.limits.nvidia.com/gpu`), GKE will automatically scale the node pool down to 0, stopping GPU billing.

### C. Enable Multi-Instance GPU (MIG) for Co-location
* NVIDIA A100 and H100 GPUs support MIG, which partition a single physical GPU into up to 7 independent GPU instances.
* **Action:** Use MIG in GKE to split one A100 GPU among multiple small inference services, rather than provisioning a separate physical GPU for each service.
* **The Benefit:** Increases hardware utilization and divides the high baseline cost of the A100 among multiple workloads.

---

## 3. Cloud GPU Audit Checklist

1. [ ] **GPU Model Benchmarking:** Confirm that running inference services are matched to the cheapest GPU model (e.g., T4 or L4) that meets performance SLA requirements.
2. [ ] **Scale-to-Zero GKE Pools:** Verify that all GKE GPU node pools have `min-nodes` configured to 0.
3. [ ] **MIG Enablement:** Evaluate A100/H100 workloads for Multi-Instance GPU (MIG) opportunities to share physical cards.
4. [ ] **Spot GPU for Batch/Training:** Ensure offline batch rendering or custom training runs utilize Spot GPUs.
