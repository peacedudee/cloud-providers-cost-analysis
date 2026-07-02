# Network Service Tiers Cost Optimization & Research

Google Cloud is unique in offering two distinct network routing options for public traffic: **Premium Tier** and **Standard Tier**. Because GCP defaults to Premium Tier for all newly provisioned VMs and Load Balancers, organizations routinely pay a 20% to 35% surcharge on network egress without realizing a cheaper alternative exists. This file covers network tier pricing and cost reduction.

---

## 1. Premium Tier vs. Standard Tier

| Metric | Premium Tier (Default) | Standard Tier (Cost-Optimized) |
| :--- | :--- | :--- |
| **Routing Path** | Routes traffic over Google's high-speed, private global fiber network. | Routes traffic over public ISP networks (transit internet). |
| **Ingress Point** | Traffic enters Google's network at the POP (Point of Presence) closest to the user. | Traffic enters Google's network at the GCP region gateway hosting the resource. |
| **Performance** | Lowest latency, lowest packet loss, maximum throughput. | Standard internet performance (comparable to other clouds). |
| **Supported Load Balancers** | Global External Load Balancing (HTTP/S, SSL, TCP), Regional Load Balancing. | Regional External Load Balancing only. |
| **IP Addresses** | Global Anycast IP addresses. | Regional Static/Ephemeral IP addresses. |
| **Egress Price** | Baseline premium rate (e.g. ~$0.08–$0.12/GB depending on volume). | **20% to 35% cheaper** than Premium Tier. |

---

## 2. Core Cost-Reduction Tactics

### A. Apply Organization Policies to Set Standard Tier in Dev/Staging
* **The Waste:** Staging and development environments rarely need Google's global fiber backbone. However, because GCP defaults to Premium Tier, dev VMs running unit tests or serving internal QA users are billed at the premium network rate.
* **The Solution:** Enforce the Standard network tier in non-production environments.
* **Implementation:** Apply the GCP Organization Policy constraint **`constraints/compute.defaultNetworkTier`** at the folder or project level for all non-production projects. Set the value to `STANDARD`. All VMs, Load Balancers, and Forwarding Rules created in these projects will automatically use the cheaper Standard Tier.

### B. Migrate Single-Region Production Workloads
* If your application is deployed in a single region (e.g. `us-central1`) and serves customers primarily in that same geographical area, the latency delta between Premium Tier and Standard Tier is negligible (often < 5-10ms).
* **Action:** Audit production regional external IP addresses. For single-region APIs or websites, change the network tier of the VM instance network interfaces or the external forwarding rules to **Standard Tier**.
* **Terraform Example:**
  ```hcl
  resource "google_compute_instance" "app_server" {
    name         = "web-server"
    machine_type = "e2-medium"
    zone         = "us-central1-a"
    
    network_interface {
      network = "default"
      access_config {
        network_tier = "STANDARD" # Change from default PREMIUM
      }
    }
  }
  ```

### C. Restructure Egress Patterns
Egress pricing is highly dependent on destination:
1. **Egress to Internet:** Billed at standard network tier rates (highest cost).
2. **Egress between GCP Regions:** Billed at $0.01 per GB (within North America) or higher for other continents (e.g. $0.02/GB within Europe).
   * *Tactic:* Co-locate data-generating pipelines (like Dataflow) in the same region as the database or storage bucket they query.
3. **Egress between Zones in the same Region:** Billed at $0.01 per GB.
   * *Tactic:* Use zone-affinity or keep high-throughput services within the same zone where HA is not strictly required.
4. **Egress to Google Services (e.g., BigQuery, Cloud Storage):**
   * *Tactic:* Enable **Private Google Access** on your VPC subnets. This allows VMs without public IPs to communicate with Google services over internal Google network routing, avoiding internet egress fees.

---

## 3. Network Tier Audit Checklist

1. [ ] **Org Policy Verification:** Verify that the `compute.defaultNetworkTier = STANDARD` org policy is active for all non-production GCP projects.
2. [ ] **VM Network Tier Sweep:** Run a CLI sweep to identify active VM external IPs configured for Premium Tier.
   ```bash
   gcloud compute instances list --format="table(name, zone, networkInterfaces[].accessConfigs[].networkTier)"
   ```
3. [ ] **Load Balancer Audit:** Check if regional Load Balancers are configured on Premium Tier when Standard Tier is sufficient.
4. [ ] **Private Google Access:** Confirm that "Private Google Access" is enabled on all active VPC subnets to prevent VMs from routing traffic through external IPs to call GCP APIs.
5. [ ] **Inter-Region Data Auditing:** Identify applications transfering data across regions. Work to consolidate those resources into a single region.
