# Cloud TPU Cost Optimization & Research

Google Cloud Tensor Processing Units (TPUs) are custom-designed application-specific integrated circuits (ASICs) accelerated for deep learning workloads (TensorFlow, PyTorch, JAX). Running TPU VM pod slices is one of the most expensive undertakings in Google Cloud, making idle resource control and execution automation essential.

---

## 1. Cloud TPU Billing Dimensions

TPU costs are calculated based on:
1. **TPU Generation & Chip Count:** Billed hourly based on the TPU type and slice size (e.g. a `v5e-4` has 4 TPU v5e chips, whereas a `v5p-8` has 8 TPU v5p chips).
2. **Pricing Model:**
   * **On-Demand:** Standard hourly rates.
   * **Spot TPUs:** Up to **60% to 70% cheaper** than on-demand. Subject to preemption.
   * **Committed Use Discounts (CUDs):** 1-year and 3-year commitments for steady-state training capacity.

---

## 2. Core Cost-Optimization Levers

### A. Use Spot TPUs for Training with Checkpointing
Deep learning training runs are highly tolerant of interruptions if checkpointing is configured correctly.
* **Action:** Configure your TPU VM creation commands or Kubernetes manifests to use **Spot provisioning**.
* **The Benefit:** Saves up to **70%** on chip billing.
* **Requirement:** Ensure your training script (e.g., in PyTorch or JAX) regularly writes checkpoints to a Cloud Storage bucket (e.g., every 30 minutes) so that if the Spot VM is preempted, the next worker can resume from the last saved state without losing progress.

### B. Enforce Ephemeral Run Pipelines (Vertex AI Custom Training)
* **The Waste:** Machine learning engineers provision a persistent TPU VM, SSH into it to experiment, run a 4-hour training script, and then forget to delete the VM, leaving it billing 24/7 over the weekend.
* **The Solution:** Run training through **Vertex AI Custom Jobs**.
* **Action:** Package your training script into a Docker container and submit it as a Vertex AI custom training job specifying TPU worker pools.
* **The Benefit:** Vertex AI automatically provisions the TPU VMs, mounts storage, runs the container, and **instantly tears down the TPU VM the second the job succeeds or fails**. You pay only for the exact seconds of training execution.

### C. Right-Size Topology During Experimentation
* **Tactic:** Do not debug training loops or pipeline scripts on a massive, expensive TPU Pod (e.g. `v5p-32`). Do all initial code validation, dataset pipe checks, and learning rate warm-ups on the smallest available slice (e.g. `v5e-4` or `v4-8`). Scale up to larger multi-slice topologies only when the model training loop is proven stable.

---

## 3. Cloud TPU Audit Checklist

1. [ ] **Spot VM Preference:** Verify that all non-production and training workloads utilize Spot TPUs with checkpointing.
2. [ ] **Persistent TPU Review:** Audit all active TPU VMs. Stop or delete any VM that has been running for > 24 hours without active CPU/TPU utilization.
3. [ ] **Vertex AI Pipeline Migration:** Migrate manual training on persistent TPU VMs to serverless Vertex AI Custom Jobs.
4. [ ] **TPU CUD Alignment:** Ensure steady-state foundation model hosting is covered by active TPU committed use agreements.
