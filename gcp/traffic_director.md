# Traffic Director Cost Optimization & Research

Google Cloud Traffic Director is a managed control plane for service mesh and load balancing, compatible with xDS APIs (like Envoy proxy). Traffic Director manages traffic routing, load balancing, and policy enforcement across VMs and containers. While Traffic Director itself is largely free of direct charges, the sidecar proxies and inter-service routing it orchestrates can drive significant resource waste.

---

## 1. Traffic Director Billing Mechanics

* **Service Control Plane Fee:** **Traffic Director is 100% free** of direct management fees.
* **Associated Infrastructure Costs:** You are billed for:
  1. **Envoy Proxy Sidecar Resources:** Each container pod in the mesh runs an Envoy sidecar proxy, consuming CPU and Memory.
  2. **Inter-Service Network Egress:** Billed standard rates when Traffic Director routes calls between services across different zones ($0.01/GB) or regions.

---

## 2. Core Cost-Optimization Levers

### A. Right-Size Envoy Proxy Container Requests
In a large service mesh (e.g. 500 pods), each pod runs a sidecar container. By default, developers often copy-paste resource settings, requesting e.g., 0.5 vCPU and 512 MB memory for the Envoy sidecar.
* **The Waste:** 500 pods x 512 MB = 250 GB of RAM allocated solely to proxy sidecars, most of which goes unused.
* **Action:** Audit actual Envoy CPU and memory usage using Cloud Monitoring. Right-size sidecar containers to minimum safe limits (e.g. 0.1 vCPU and 128 MB RAM).

### B. Enable Topology-Aware Routing (Keep Mesh Local)
* Without location rules, Traffic Director might route calls from a web service in `zone-a` to a backend service in `zone-b`, triggering inter-zone data transfer charges.
* **Action:** Configure Traffic Director routing rules to prefer local backends (endpoints in the same zone). Only failover to other zones or regions if local endpoints are unavailable.

---

## 3. Traffic Director Audit Checklist

1. [ ] **Sidecar Resource Audit:** Review CPU/Memory utilization of Envoy sidecar proxies in GKE/GCE. Apply VPA recommendations to sidecars.
2. [ ] **Zone Affinity Routing:** Verify Traffic Director endpoint groups are configured to prioritize local zone backends.
3. [ ] **Service Mesh Clean Up:** Confirm that decommissioned services are deleted from Traffic Director registries to prevent control plane overhead.
