# Google Cloud Platform — Services Master Map

> **Objective:** Map every GCP service, understand how it's charged, and identify cost-optimization opportunities.
>
> **How to use this file:** Each service has a `Research Status` column. As we deep-dive into individual services, we'll update the status and link to the dedicated research file for that service.

---

## Legend

| Symbol | Meaning |
|--------|---------|
| ⬜ | Not started |
| 🟡 | In progress |
| ✅ | Research complete |

---

## 1. Compute

Core infrastructure for running workloads — VMs, containers, serverless.

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 1 | **[Compute Engine](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/compute_engine.md)** | Per-second (1-min minimum) | vCPUs, memory, GPUs, local SSDs, persistent disks, OS licenses | ✅ | SUDs & CUDs available; Spot VMs up to 91% cheaper |
| 2 | **[Google Kubernetes Engine (GKE)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/gke.md)** | Per-second + management fee | Cluster management fee + underlying Compute Engine resources | ✅ | Autopilot mode charges per pod resources |
| 3 | **[Cloud Run](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_run.md)** | Per-request + per-resource-second | vCPU-seconds, memory GiB-seconds, requests, networking | ✅ | Generous free tier; scales to zero |
| 4 | **[Cloud Run Functions](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_functions.md)** | Per-invocation + per-resource-second | Invocations, compute time (GB-seconds), networking | ✅ | Formerly Cloud Functions; 2M free invocations/month |
| 5 | **[App Engine](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/app_engine.md)** | Per-instance-hour | Instance hours, storage, outbound data | ✅ | Standard env has free tier; Flexible env like Compute Engine |
| 6 | **[Bare Metal Solution](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/bare_metal_solution.md)** | Monthly subscription | Server configs, storage, networking | ✅ | For specialized workloads (e.g., Oracle) |
| 7 | **[Sole-Tenant Nodes](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/sole_tenant_nodes.md)** | Per-node-hour | Node type, vCPUs, memory | ✅ | Dedicated physical servers; compliance use cases |
| 8 | **[VMware Engine](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/vmware_engine.md)** | Per-node-hour | Node type, private cloud infrastructure | ✅ | Migrate VMware workloads as-is |
| 9 | **[Batch](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/batch.md)** | No additional charge | Underlying Compute Engine resources only | ✅ | Managed batch processing scheduler |
| 10 | **[Cloud Workstations](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_workstations.md)** | Per-hour | Machine type, storage, idle timeout | ✅ | Managed dev environments |

---

## 2. Storage

Object, file, and block storage services.

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 11 | **[Cloud Storage](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_storage.md)** | Per GB-month + operations | Storage class (Standard/Nearline/Coldline/Archive), operations, retrieval, egress, early deletion | ✅ | Autoclass can auto-transition objects |
| 12 | **[Persistent Disk](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/persistent_disk.md)** | Per GB-month provisioned | Disk type (pd-standard, pd-ssd, pd-balanced), snapshots | ✅ | Attached to Compute Engine / GKE |
| 13 | **[Filestore](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/filestore.md)** | Per TB provisioned | Tier (Basic/Enterprise), capacity, snapshots | ✅ | Managed NFS; minimum capacity requirements |
| 14 | **[Cloud Storage for Firebase](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_storage_for_firebase.md)** | Per GB stored + operations | Storage, downloads, operations | ✅ | Mobile/web app focused |
| 15 | **[Backup and DR](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/backup_and_dr.md)** | Per GB protected | Protected data volume, appliance hours | ✅ | Centralized backup management |
| 16 | **[NetApp Volumes](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/netapp_volumes.md)** | Per TB provisioned | Service level (Standard/Premium/Extreme), capacity | ✅ | Enterprise NAS workloads |

---

## 3. Databases

Managed relational and NoSQL database services.

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 17 | **[Cloud SQL](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_sql.md)** | Per-hour + storage | Instance type (vCPU/memory), storage, backups, HA, read replicas, egress | ✅ | MySQL, PostgreSQL, SQL Server |
| 18 | **[AlloyDB](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/alloydb.md)** | Per-hour + storage | vCPUs, memory, storage, I/O, backups | ✅ | PostgreSQL-compatible, 4x faster than standard PG |
| 19 | **[Cloud Spanner](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_spanner.md)** | Per-node-hour + storage | Processing units (nodes), storage, egress | ✅ | Globally distributed; granular scaling with PUs |
| 20 | **[Firestore](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/firestore.md)** | Per-operation + storage | Document reads/writes/deletes, stored data, egress | ✅ | Serverless NoSQL; free tier available |
| 21 | **[Bigtable](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_bigtable.md)** | Per-node-hour + storage | Node count, storage (SSD/HDD), egress | ✅ | Wide-column NoSQL for large analytical workloads |
| 22 | **[Memorystore](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/memorystore.md)** | Per-GB-hour | Instance size, tier (Basic/Standard), engine (Redis/Valkey/Memcached) | ✅ | In-memory caching |
| 23 | **[Firebase Realtime Database](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/firebase_realtime_database.md)** | Per-GB stored + per-GB downloaded | Storage, downloads, simultaneous connections | ✅ | JSON-based realtime sync |
| 24 | **[Database Migration Service](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/database_migration_service.md)** | Free for migrations | Only pay for destination database resources | ✅ | Migrations to Cloud SQL, AlloyDB |

---

## 4. Networking

VPC, load balancing, CDN, DNS, and connectivity.

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 25 | **[Virtual Private Cloud (VPC)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/vpc.md)** | No charge for VPC itself | Egress, firewall rules (policy-based), flow logs | ✅ | Foundation service; egress charges apply |
| 26 | **[Cloud Load Balancing](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_load_balancing.md)** | Per-hour + per-GB processed | Forwarding rules, ingress data processed | ✅ | Global and regional options |
| 27 | **[Cloud CDN](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_cdn.md)** | Per-GB served + per-request | Cache egress, cache fill, HTTP/HTTPS requests, cache invalidations | ✅ | Content delivery network |
| 28 | **[Cloud DNS](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_dns.md)** | Per-zone + per-query | Managed zones, DNS queries | ✅ | Managed authoritative DNS |
| 29 | **[Cloud NAT](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_nat.md)** | Per-hour + per-GB | NAT gateway hours, data processed | ✅ | Outbound internet for private instances |
| 30 | **[Cloud Armor](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_armor.md)** | Per-policy + per-rule + per-request | Security policies, rules, requests evaluated, adaptive protection | ✅ | WAF & DDoS protection |
| 31 | **[Cloud Interconnect](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_interconnect.md)** | Per-hour + per-port | Port speed (10G/100G), VLAN attachments, egress (discounted vs internet) | ✅ | Dedicated/Partner connections |
| 32 | **[Cloud VPN](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_vpn.md)** | Per-tunnel-hour + egress | VPN tunnels, data processed | ✅ | Site-to-site encrypted connectivity |
| 33 | **[Network Service Tiers](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/network_tiers.md)** | Per-GB egress (tiered pricing) | Premium Tier (Google backbone) vs Standard Tier (public internet) | ✅ | Affects all egress pricing |
| 34 | **[Service Directory](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/service_directory.md)** | Per-endpoint registered | Endpoints, resolve requests | ✅ | Service registry and discovery |
| 35 | **[Traffic Director](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/traffic_director.md)** | Free (in GA) | No separate charge | ✅ | Managed control plane for service mesh |
| 36 | **[Private Service Connect](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/private_service_connect.md)** | Per-hour + per-GB | Endpoints, data processed | ✅ | Private connectivity to Google/3rd-party services |
| 37 | **[Network Connectivity Center](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/network_connectivity_center.md)** | Per-hour | Hub, spokes, data processed | ✅ | Multi-cloud network hub |
| 38 | **[Packet Mirroring](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/packet_mirroring.md)** | Per-GB mirrored | Mirrored data volume | ✅ | Network traffic inspection |

---

## 5. Data Analytics

Data warehousing, processing, and analytics.

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 39 | **[BigQuery](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/bigquery.md)** | On-demand (per-TB scanned) or Capacity (slots) | Queries (TB scanned), active storage, long-term storage, streaming inserts, BI Engine | ✅ | 1 TB free queries/month; flat-rate slots for predictability |
| 40 | **[Dataflow](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_dataflow.md)** | Per-vCPU-hr + per-GB-hr + per-shuffle-GB | Worker vCPUs, memory, shuffle data processed, Streaming Engine | ✅ | Apache Beam runner; batch & stream |
| 41 | **[Dataproc](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_dataproc.md)** | Per-vCPU-hr (discounted) + underlying VMs | Dataproc premium + Compute Engine resources | ✅ | Managed Spark/Hadoop |
| 42 | **[Pub/Sub](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/pubsub.md)** | Per-TiB delivered + per-operation | Message delivery, storage, snapshots & seek, egress | ✅ | Messaging and event streaming |
| 43 | **[Dataprep (by Trifacta)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/dataprep.md)** | Per-unit (based on data processed) | Unit consumption based on complexity and volume | ✅ | Visual data preparation |
| 44 | **[Data Fusion](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/data_fusion.md)** | Per-hour (by edition) | Edition (Developer/Basic/Enterprise), pipeline hours | ✅ | Visual ETL/ELT; built on CDAP |
| 45 | **[Dataplex](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/dataplex.md)** | Per-CU-hour | Compute units for discovery, profiling, quality | ✅ | Data governance and management |
| 46 | **[Datastream](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/datastream.md)** | Per-GB processed | Change data capture volume | ✅ | Serverless CDC replication |
| 47 | **[Looker](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/looker.md)** | Subscription (annual) | Users, platform edition | ✅ | Enterprise BI platform |
| 48 | **[Looker Studio](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/looker_studio.md)** | Free | No charge for the tool itself | ✅ | Self-service dashboards (data source costs apply) |
| 49 | **[Composer](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/composer.md)** | Per-hour (environment) | Environment size, vCPU, memory, storage, DAG processing | ✅ | Managed Apache Airflow |
| 50 | **[BigQuery Data Transfer Service](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/bigquery_data_transfer_service.md)** | Free (for most sources) | Only charges for BigQuery storage and queries | ✅ | Scheduled data imports |

---

## 6. AI & Machine Learning

AI platform, pre-trained APIs, and ML infrastructure.

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 51 | **[Vertex AI](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/vertex_ai.md)** | Per-resource-hour + per-prediction | Training (machine type hrs), predictions (node hrs or per-prediction), storage | ✅ | Unified ML platform; AutoML & custom training |
| 52 | **[Vertex AI Search & Conversation](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/vertex_ai_search_and_conversation.md)** | Per-query + per-document | Queries, indexed documents, storage | ✅ | AI-powered search and chat |
| 53 | **[Gemini API (Vertex AI)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/gemini_api.md)** | Per-input/output token | Input tokens, output tokens, model size | ✅ | LLM inference |
| 54 | **[Cloud TPU](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_tpu.md)** | Per-TPU-hour | TPU type (v4/v5e/v5p), on-demand vs reserved | ✅ | Custom ML accelerators |
| 55 | **[Cloud GPUs](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_gpus.md)** | Per-GPU-hour | GPU type (A100/H100/L4), on-demand vs Spot vs CUD | ✅ | Attached to Compute Engine |
| 56 | **[Natural Language API](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/speech_and_vision_apis.md)** | Per-1,000 text records | Text records processed, feature type | ✅ | NLP analysis |
| 57 | **[Vision AI](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/speech_and_vision_apis.md)** | Per-image or per-minute (video) | Images processed, video minutes, feature type | ✅ | Image/video analysis |
| 58 | **[Speech-to-Text](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/speech_and_vision_apis.md)** | Per-15-second increment | Audio duration, model (standard/enhanced), features | ✅ | Audio transcription |
| 59 | **[Text-to-Speech](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/speech_and_vision_apis.md)** | Per-million characters | Characters, voice type (Standard/WaveNet/Neural2) | ✅ | Speech synthesis |
| 60 | **[Translation API](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/speech_and_vision_apis.md)** | Per-million characters | Characters translated, model type | ✅ | Language translation |
| 61 | **[Document AI](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/document_ai.md)** | Per-page processed | Pages, processor type | ✅ | Document parsing and extraction |
| 62 | **[Recommendations AI](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/recommendations_ai.md)** | Per-prediction request | Prediction requests, training | ✅ | Personalized recommendations |
| 63 | **[Dialogflow](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/dialogflow.md)** | Per-request | Text/voice requests, edition (CX/ES) | ✅ | Conversational AI agents |
| 64 | **[AutoML](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/automl.md)** | Per-node-hour (training & prediction) | Training hours, prediction node hours | ✅ | Custom model training (via Vertex AI) |

---

## 7. Security & Identity

Identity management, encryption, threat detection.

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 65 | **[Identity and Access Management (IAM)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/free_security_services.md)** | Free | No charge | ✅ | Foundation access control |
| 66 | **[Cloud KMS](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_kms.md)** | Per-key-version + per-operation | Active key versions, cryptographic operations, HSM keys | ✅ | Key management and encryption |
| 67 | **[Secret Manager](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/secret_manager.md)** | Per-secret-version + per-access | Active secret versions, access operations | ✅ | Secrets storage |
| 68 | **[Security Command Center](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/security_command_center.md)** | Tiered (Standard free / Premium %) | Standard (free) or Premium (% of GCP spend) | ✅ | Central security and risk dashboard |
| 69 | **[Cloud Armor](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_armor.md)** | Per-policy + per-request | (See Networking section) | ✅ | WAF & DDoS |
| 70 | **[Identity-Aware Proxy (IAP)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/free_security_services.md)** | Free | No separate charge | ✅ | Zero-trust access to apps |
| 71 | **[Certificate Authority Service](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/certificate_authority_service.md)** | Per-CA + per-certificate | CA tier (DevOps/Enterprise), certificates issued | ✅ | Private certificate management |
| 72 | **[Binary Authorization](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/free_security_services.md)** | Free | No charge | ✅ | Deploy-time security policy for GKE |
| 73 | **[BeyondCorp Enterprise](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/free_security_services.md)** | Per-user-month | Users, features | ✅ | Zero-trust solution |
| 74 | **[Assured Workloads](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/free_security_services.md)** | Free (some features) | No separate charge (premium support may apply) | ✅ | Compliance controls |
| 75 | **[reCAPTCHA Enterprise](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/recaptcha_enterprise.md)** | Per-assessment | Assessments per month (first 10K free) | ✅ | Bot detection |
| 76 | **[Web Security Scanner](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/free_security_services.md)** | Free | No charge (App Engine, GKE, Compute Engine) | ✅ | Vulnerability scanning |
| 77 | **[Google SecOps (Chronicle)](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/google_secops.md)** | Per-GB ingested or subscription | Data ingestion volume | ✅ | SIEM & SOAR |

---

## 8. Operations & Observability

Monitoring, logging, tracing, profiling.

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 78 | **[Cloud Monitoring](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/operations_suite.md)** | Free (allotment) + per-metric | Monitoring data volume, custom metrics, MQL queries | ✅ | First 150 MB free/project/month |
| 79 | **[Cloud Logging](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/operations_suite.md)** | Per-GiB ingested | Log bytes ingested above free allotment (50 GiB/project) | ✅ | First 50 GiB free; retention pricing |
| 80 | **[Cloud Trace](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_trace.md)** | Per-span ingested | Trace spans (first 2.5M free) | ✅ | Distributed tracing |
| 81 | **[Cloud Profiler](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/free_operations_services.md)** | Free | No charge | ✅ | Continuous CPU/memory profiling |
| 82 | **[Error Reporting](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/free_operations_services.md)** | Free | No charge | ✅ | Error aggregation and alerting |
| 83 | **[Cloud Debugger](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/free_operations_services.md)** | Free (deprecated) | No charge | ✅ | Being replaced by Snapshot Debugger |

---

## 9. Developer Tools & CI/CD

Build, deploy, and manage applications.

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 84 | **[Cloud Build](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_build.md)** | Per-build-minute | Build minutes (first 120 min/day free), machine type | ✅ | Serverless CI/CD |
| 85 | **[Artifact Registry](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/artifact_registry.md)** | Per-GB stored + per-GB egress | Storage, data transfer | ✅ | Container/package registry |
| 86 | **[Cloud Deploy](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/cloud_deploy.md)** | Per-delivery-pipeline + per-target | Active delivery pipelines, active targets | ✅ | Continuous delivery to GKE/Cloud Run |
| 87 | **[Cloud Source Repositories](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/workflows_and_scheduler.md)** | Free (up to limits) | Users, storage, egress | ✅ | Private Git repos |
| 88 | **[Container Registry](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/artifact_registry.md)** | Per-GB stored (Cloud Storage pricing) | Storage, egress | ✅ | Legacy — use Artifact Registry |
| 89 | **[Cloud Scheduler](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/workflows_and_scheduler.md)** | Per-job-per-month | Jobs (3 free per account) | ✅ | Managed cron jobs |
| 90 | **[Cloud Tasks](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/workflows_and_scheduler.md)** | Per-million operations | Task operations (first 1M free) | ✅ | Asynchronous task execution |
| 91 | **[Eventarc](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/workflows_and_scheduler.md)** | Free (during preview/GA) | No separate charge (costs from triggering services) | ✅ | Event-driven architecture |
| 92 | **[Workflows](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/workflows_and_scheduler.md)** | Per-step execution | Internal steps, external steps (HTTP calls) | ✅ | Serverless orchestration |

---

## 10. API Management

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 93 | **[Apigee](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/api_management.md)** | Subscription (tiered) | API calls, environments, edition (Standard/Enterprise/Enterprise+) | ✅ | Full-lifecycle API management |
| 94 | **[API Gateway](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/api_management.md)** | Per-million API calls | API calls, data transfer | ✅ | Lightweight gateway for Cloud Run / Cloud Functions |
| 95 | **[Endpoints](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/api_management.md)** | Per-million API calls | API calls (first 2M free) | ✅ | API management for gRPC/REST |

---

## 11. Migration & Transfer

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 96 | **[Migration Center](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/migration_and_transfer.md)** | Free | No charge for assessment | ✅ | Discovery, assessment, planning |
| 97 | **[Migrate to Virtual Machines](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/migration_and_transfer.md)** | Free (service) | Pay only for destination resources | ✅ | VM migration from on-prem/other clouds |
| 98 | **[Migrate to Containers](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/migration_and_transfer.md)** | Free (service) | Pay only for GKE resources | ✅ | Containerize existing VMs |
| 99 | **[Storage Transfer Service](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/migration_and_transfer.md)** | Free (service) | Pay only for destination storage + egress from source | ✅ | Online data transfer |
| 100 | **[Transfer Appliance](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/migration_and_transfer.md)** | Per-appliance rental | Appliance fee + data ingestion | ✅ | Physical data transfer for petabyte-scale |
| 101 | **[BigQuery Data Transfer Service](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/bigquery_data_transfer_service.md)** | Free | BQ storage/query costs only | ✅ | SaaS data imports |
| 102 | **[Database Migration Service](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/database_migration_service.md)** | Free | Destination DB costs only | ✅ | DB migration tool |

---

## 12. Hybrid & Multi-Cloud

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 103 | **[Anthos / GKE Enterprise](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/hybrid_and_multi_cloud.md)** | Per-vCPU-month | vCPUs managed, environment (on-prem/cloud) | ✅ | Multi-cloud container management |
| 104 | **[Google Distributed Cloud](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/hybrid_and_multi_cloud.md)** | Monthly subscription | Node configuration, edge/connected mode | ✅ | GCP infrastructure at the edge |
| 105 | **[Anthos Config Management](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/hybrid_and_multi_cloud.md)** | Included with GKE Enterprise | Per Anthos pricing | ✅ | Policy-as-code for multi-cluster |
| 106 | **[Anthos Service Mesh](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/hybrid_and_multi_cloud.md)** | Included with GKE Enterprise | Per Anthos pricing | ✅ | Managed Istio-based service mesh |

---

## 13. Serverless Integration

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 107 | **[Cloud Pub/Sub](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/pubsub.md)** | Per-TiB delivered | (See Data Analytics section) | ✅ | Messaging backbone |
| 108 | **[Cloud Tasks](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/workflows_and_scheduler.md)** | Per-million operations | (See Developer Tools section) | ✅ | Async task queues |
| 109 | **[Cloud Scheduler](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/workflows_and_scheduler.md)** | Per-job-month | (See Developer Tools section) | ✅ | Cron-as-a-service |
| 110 | **[Workflows](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/workflows_and_scheduler.md)** | Per-step | (See Developer Tools section) | ✅ | Orchestration |
| 111 | **[Eventarc](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/workflows_and_scheduler.md)** | Free | (See Developer Tools section) | ✅ | Event routing |

---

## 14. Management & Governance

| # | Service | Billing Model | Key Pricing Dimensions | Research Status | Notes |
|---|---------|--------------|----------------------|----------------|-------|
| 112 | **[Resource Manager](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/management_and_governance.md)** | Free | No charge | ✅ | Org / folder / project hierarchy |
| 113 | **[Cloud Billing](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/management_and_governance.md)** | Free | No charge | ✅ | Billing accounts and budgets |
| 114 | **[Cloud Console](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/management_and_governance.md)** | Free | No charge | ✅ | Web UI |
| 115 | **[Cloud Shell](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/management_and_governance.md)** | Free (weekly limits) | No charge within limits; storage persists 120 days | ✅ | Browser-based terminal |
| 116 | **[Deployment Manager](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/management_and_governance.md)** | Free | No charge (pay for deployed resources) | ✅ | Infrastructure-as-code (legacy) |
| 117 | **[Config Connector](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/management_and_governance.md)** | Free | No charge | ✅ | Kubernetes-style resource management |
| 118 | **[Service Catalog](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/management_and_governance.md)** | Free | No charge | ✅ | Governed service provisioning |
| 119 | **[Cloud Quotas](file:///Users/anjukumari/Desktop/Cloud Providers Cost Analysis/gcp/management_and_governance.md)** | Free | No charge | ✅ | Quota management |

---

## 15. Data Transfer & Networking Egress

> **⚠️ IMPORTANT:** Egress costs are often the hidden cost driver. Ingress (data coming in) is free. Egress (data going out) is charged.

| Traffic Type | Cost |
|-------------|------|
| Ingress (inbound) | **Free** |
| Egress to same zone | **Free** |
| Egress to different zone (same region) | ~$0.01/GB |
| Egress between regions (within US/EU) | ~$0.01/GB |
| Egress to internet | **$0.08–$0.23/GB** (varies by destination & volume) |
| Egress via Interconnect | Discounted vs internet egress |
| Premium Tier (Google backbone) | Higher egress cost, better performance |
| Standard Tier (public internet) | Lower egress cost, lower performance |

---

## 16. Cost Optimization Levers (Cross-Cutting)

These apply across many services and are critical for cost reduction:

| Lever | Applicable Services | Potential Savings |
|-------|-------------------|-------------------|
| **Committed Use Discounts (CUDs)** | Compute Engine, Cloud SQL, GKE, Cloud Run | Up to 57% (1yr) / 70% (3yr) |
| **Sustained Use Discounts (SUDs)** | Compute Engine (N1, N2 families) | Up to 30% automatic |
| **Spot / Preemptible VMs** | Compute Engine, GKE, Dataproc | Up to 60–91% |
| **Autoscaling** | GKE, Cloud Run, Compute Engine MIGs | Variable — right-size to demand |
| **Storage class management** | Cloud Storage (Autoclass, lifecycle rules) | Up to 80%+ on cold data |
| **Right-sizing recommendations** | Compute Engine, Cloud SQL | 10–50% typical |
| **Active Assist** | Cross-service | Automated recommendations |
| **Billing budgets & alerts** | All | Prevent bill shock |
| **BigQuery flat-rate / editions** | BigQuery | Predictable costs at scale |
| **Reserved capacity** | Bigtable, Spanner, Cloud SQL | Similar to CUDs |

---

## Research Tracking Summary

| Category | Total Services | ⬜ Not Started | 🟡 In Progress | ✅ Complete |
|----------|---------------|---------------|----------------|-------------|
| Compute | 10 | 0 | 0 | 10 |
| Storage | 6 | 0 | 0 | 6 |
| Databases | 8 | 0 | 0 | 8 |
| Networking | 14 | 0 | 0 | 14 |
| Data Analytics | 12 | 0 | 0 | 12 |
| AI & ML | 14 | 0 | 0 | 14 |
| Security & Identity | 13 | 0 | 0 | 13 |
| Operations | 6 | 0 | 0 | 6 |
| Developer Tools | 9 | 0 | 0 | 9 |
| API Management | 3 | 0 | 0 | 3 |
| Migration & Transfer | 7 | 0 | 0 | 7 |
| Hybrid & Multi-Cloud | 4 | 0 | 0 | 4 |
| Management & Governance | 8 | 0 | 0 | 8 |
| **TOTAL** | **114** | **0** | **0** | **114** |

---

> **💡 Next Steps:** Pick a service category or a specific high-spend service to deep-dive into first. I'll create a dedicated `<Service_Name>_Research.md` file with detailed pricing tiers, cost optimization strategies, and actionable recommendations for each one.
