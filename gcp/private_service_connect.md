# Private Service Connect (PSC) Cost Optimization

Google Cloud Private Service Connect (PSC) allows you to connect consumer VPC networks directly to producer services (Google APIs, third-party services in the GCP Marketplace, or shared internal services in other VPCs) privately using internal IP addresses. While PSC is a powerful security tool, running duplicate endpoints or routing high volumes of standard Google API traffic through PSC rather than Private Google Access (PGA) can drive up network costs.

---

## 1. Private Service Connect Billing Mechanics

PSC pricing is calculated based on:
1. **Endpoint Hourly Fee:** Billed hourly for each active PSC endpoint you deploy in your VPC (approx. $0.01 per hour, or ~$7.30/month per endpoint).
2. **Data Processed (per GB):** Billed for data transferred through the PSC endpoint in both directions (approx. $0.01 per GB).
3. **Service Attachment Hourly Fee:** Billed for host/producer service attachments.

---

## 2. Core Cost-Optimization Levers

### A. Prioritize Private Google Access (PGA) over PSC for Google APIs
* **The Cost Trap:** You can configure PSC to access Google APIs (like BigQuery or GCS) using dedicated internal IP addresses. However, PSC charges an hourly endpoint fee, whereas standard API access does not.
* **The Solution:** For accessing standard Google APIs, use **Private Google Access (PGA)**.
* **The Benefit:** PGA is a subnet toggle that is **100% free**—it does not charge any hourly endpoint fees and does not charge any data processing fees. Reserve PSC only for:
  * Accessing third-party partner services (like Snowflake or MongoDB Atlas).
  * Consolidating endpoints across VPCs.
  * Cases where you must control the exact internal IP address used for Google APIs due to strict firewall rules.

### B. Consolidate PSC Endpoints
* **The Waste:** Creating separate PSC endpoints for the same service in every subnet or VPC in your organization.
* **Action:** Consolidate endpoints. Deploy a single PSC endpoint in a shared "Services VPC" and use VPC Peering, Hub-Spoke Transit, or internal DNS routing to allow other VPCs and subnets to route traffic through the single shared endpoint.
* **The Benefit:** Reduces the total number of active endpoints, saving on hourly endpoint hosting fees.

### C. Delete Idle PSC Endpoints & Attachments
* **Action:** Run a periodic gcloud script to identify PSC endpoints with zero data throughput over the last 30 days and decommission them.
  ```bash
  # List all PSC endpoints in a project
  gcloud compute forwarding-rules list --filter="target:all-google-apis OR target:vpc-sc"
  ```

---

## 3. Private Service Connect Audit Checklist

1. [ ] **Google API Routing Review:** Verify that standard Google API traffic (GCS, BigQuery) is routed via the free **Private Google Access** rather than PSC where IP constraints allow.
2. [ ] **Endpoint Consolidation:** Audit your GCP organization for duplicate PSC endpoints pointing to the same service. Group them under a shared VPC routing structure.
3. [ ] **Stale Endpoint Sweep:** Run CLI sweeps to locate and delete PSC forwarding rules that are inactive or have zero data transfer.
4. [ ] **Monitoring Data Processed:** Track PSC data processing metrics in Cloud Monitoring to identify high-throughput connections that could be optimized at the application layer.
