# Packet Mirroring Cost Optimization & Research

Google Cloud Packet Mirroring clones network traffic (both ingress and egress) from specific sources (VMs, subnets, or GKE nodes), and forwards it to a destination (typically an Internal Passthrough Network Load Balancer) for security inspection, threat detection, or troubleshooting. Since Packet Mirroring charges are based on the volume of **data processed**, un-filtered traffic duplication can generate high data transfer and processing fees.

---

## 1. Packet Mirroring Billing Mechanics

Packet Mirroring charges are determined by a single primary usage metric:
1. **Data Processed (per GB):** Billed for the total volume of traffic cloned and forwarded to the target load balancer (approx. $0.01 per GB).
2. **Collector Infrastructure:** Standard charges apply for the destination Internal Load Balancer, forwarding rules, and the target collector VM instances.

---

## 2. Core Cost-Optimization Levers

### A. Apply Traffic Filters (Limit Mirrored Scope)
* **The Waste:** Mirroring all traffic (ingress, egress, internal database synchronization, backup traffic) from an entire subnet, generating terabytes of mirrored data.
* **The Solution:** Configure **Traffic Filters** in your mirroring policy.
* **Action:**
  * Define filters to only mirror specific protocols (e.g., TCP only, ignoring UDP/ICMP).
  * Filter by CIDR blocks (e.g., only mirror traffic going to or coming from public IP space, ignoring internal VPC-to-VPC traffic).
  * Filter by direction: mirror only ingress or egress traffic, not both.

### B. Disable in Non-Production
* **Action:** Turn off Packet Mirroring policies in all development, QA, and staging environments. Mirroring should be active strictly in production for threat detection, or temporarily enabled in staging for specific load-testing audits.

### C. Consolidate Collector Infrastructure
* **Action:** Deploy a single centralized pool of monitoring VMs behind an Internal Network Load Balancer in a shared services VPC. Configure mirroring policies across multiple consumer VPCs to target this single shared load balancer.
* **The Benefit:** Eliminates the overhead of running separate, under-utilized collector VM clusters in every project or environment.

---

## 3. Packet Mirroring Audit Checklist

1. [ ] **Active Policy Review:** Identify all active Packet Mirroring policies. Disable policies that are no longer needed.
2. [ ] **Filter Implementation:** Confirm that mirroring policies have active filters (CIDR blocks, protocols, direction) to minimize processed volume.
3. [ ] **Collector Resource Sizing:** Audit collector VM pools. Right-size collector instances or configure autoscaling to handle traffic spikes without idle overhead.
4. [ ] **Non-Prod Audit:** Ensure Packet Mirroring is turned off in dev/test projects.
