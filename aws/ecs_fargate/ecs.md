# AWS Service Cost Research: Amazon ECS (Elastic Container Service)

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon ECS is a fully managed container orchestration service that helps you deploy, manage, and scale containerized applications. Unlike GKE or Amazon EKS, there is **no cluster management fee** for ECS itself; pricing is determined entirely by the launch type you use to run your containers.

---

## 2. Billing Mechanics
ECS orchestration is **free**. You pay only for the underlying compute, storage, and networking resources used to run your containers. There are three launch types and one managed model:
*   **EC2 Launch Type:** You run container tasks on virtual machines that you manage. You pay standard EC2 instance, EBS storage, and network transfer rates.
*   **Fargate Launch Type:** Serverless model. You do not manage VMs. You pay for the specific vCPU and memory allocated to your running containers (see [fargate.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/fargate.md) for details).
*   **ECS Anywhere:** Run ECS containers on your own physical, on-premises hardware. AWS charges a flat rate of **$0.01025 per hour** for each active managed instance.
*   **ECS Managed Instances:** If you use ECS to manage EC2 instances automatically (provisioning, patching, and scaling), you pay standard EC2 instance charges plus a management fee of **$0.02 per hour** per managed instance.

---

## 3. Key Cost Dimensions

### A. EC2 Launch Type Resources
Under the EC2 launch type, you pay for the instances in your cluster whether they are empty or fully loaded. Cost drivers include:
*   **EC2 Compute Hours:** Hourly cost of the selected VM types.
*   **EBS Storage:** Boot and data volumes attached to the EC2 instances.
*   **Networking:** Public IP addresses ($0.005/hr each) and cross-AZ data transfer within the cluster.

### B. ECS Anywhere (On-Premises)
*   **The Charge:** Billed at a flat rate of **$0.01025 per hour** (~$7.48/month) per registered on-premises server.
*   **Limits:** The first 2,200 instance-hours per customer account per month are free.
*   **Egress:** You pay standard outbound data transfer fees for any logs or metrics sent from your on-premises servers back to AWS (e.g., CloudWatch, Systems Manager).

### C. Amazon ECR (Elastic Container Registry)
ECS downloads container images from ECR.
*   **Storage:** Billed at **$0.10 per GB-month** for storing container images.
*   **Data Transfer:** Pulling images from ECR inside the same region is **free**. Pulling images to the internet or across regions is subject to standard egress rates.

### D. Elastic Load Balancing (ELB)
Most ECS services run behind an Application Load Balancer (ALB) or Network Load Balancer (NLB).
*   Charged per load balancer hour (~$0.0225/hr in `us-east-1`) plus capacity units (LCU/NCU) based on connections and traffic processed.

### E. CloudWatch Logs Ingestion
Containers typically stream standard output/error to CloudWatch Logs (using the `awslogs` log driver).
*   CloudWatch charges **$0.50 per GB** of logs ingested and **$0.03 per GB-month** for storage (in `us-east-1`). This is a major, often hidden, cost driver.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Dimension | Billing Basis | Rate (us-east-1) | Cost Control Strategy |
|-------------------|---------------|------------------|-----------------------|
| **ECS Cluster Fee** | Management | **Free ($0.00)** | N/A |
| **ECS Anywhere** | Per active instance hour | **$0.01025 / hour** | First 2,200 hours/month are free |
| **ECS Managed Instances**| EC2 + Management | **EC2 + $0.020 / hour** | Prune idle managed servers |
| **ECR Storage** | Per GB-month stored | **$0.1000 / GB-month** | Implement lifecycle policies to delete old tags |
| **Fargate Launch Type** | Per task vCPU/Memory second | *See Fargate pricing* | Use Fargate Spot and right-size task sizes |
| **EC2 Launch Type** | VM and EBS allocation | *See EC2 pricing* | Use instance autoscaling and Spot fleets |

---

## 5. AWS Free Tier Coverage
*   **ECS Orchestration:** Always free.
*   **ECS Anywhere:** 2,200 free instance-hours per month (always free, shared across all on-premises servers).
*   **ECR:** 500 MB of private image storage per month for 12 months. ECR also offers a public registry with 50 GB of free storage.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Low EC2 Host Utilization (Bin-Packing Failure):** Running container tasks on a fleet of large EC2 instances where average CPU/Memory utilization is very low. You are billed for the full capacity of the underlying EC2 instances.
*   **ECR Image Bloat:** Storing hundreds of old container image tags (e.g., from every CI/CD commit). Docker images are often large (500MB - 2GB+), and registries easily accumulate TBs of storage, leading to high ECR fees.
*   **NAT Gateway Charges on Fargate:** Fargate tasks running in private subnets require a NAT Gateway to download images from ECR (if VPC Endpoints are not configured) or talk to external APIs. ECR pulls of large images through a NAT Gateway incur the $0.045/GB NAT processing fee.

---

## 7. Actionable Cost Optimization Strategies
1.  **Configure ECR Lifecycle Policies:**
    *   Set up rules to automatically delete untagged images or keep only the last $N$ images.
    *   **The Savings:** Stops image storage fees from growing linearly over time.
2.  **Optimize EC2 Task Placement (Bin-Packing):**
    *   If using the EC2 launch type, configure the **binpack** placement strategy for your services. This instructs ECS to pack tasks onto as few EC2 instances as possible, allowing Auto Scaling to terminate empty instances at the edge of the cluster.
3.  **Implement Fargate Spot:**
    *   For dev/test environments and queue-processing services that can handle minor interruptions, use Fargate Spot Capacity Providers to achieve a **70% discount** on compute.
4.  **Right-Size ECS Task Definitions:**
    *   Set memory and CPU limits in your task definitions close to actual peak requirements. Do not over-provision (e.g., allocating 4 vCPUs to a service that never uses more than 0.5 vCPUs). Use ECS container insights to monitor utilization.
5.  **Use VPC Endpoints for ECR Pulls:**
    *   If running Fargate in private subnets, configure interface VPC endpoints for ECR and S3 (gateway). This allows Fargate to pull container images privately, completely avoiding NAT Gateway data processing fees.
