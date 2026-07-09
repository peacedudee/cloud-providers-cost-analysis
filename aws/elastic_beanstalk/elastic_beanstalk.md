# AWS Service Cost Research: AWS Elastic Beanstalk

> **Status:** ✅ Research Complete & Ground Truth Verified  
> **Last Updated:** July 2026

---

## 1. Service Overview
AWS Elastic Beanstalk is an easy-to-use service for deploying and scaling web applications and services developed with Java, .NET, PHP, Node.js, Python, Ruby, Go, and Docker on familiar servers such as Apache, Nginx, Passenger, and IIS.

---

## 2. Billing Mechanics
*   **Orchestration Cost:** There is **no additional charge ($0.00)** for using AWS Elastic Beanstalk.
*   **Resource Cost:** You pay only for the underlying AWS resources (such as EC2 instances, Elastic Load Balancers, S3 buckets, Auto Scaling groups, and Amazon RDS databases) that you configure to run your application.
*   **Billing Basis:** The cost profile is entirely derivative, driven by the individual rates of the resources Beanstalk provisions on your behalf.

---

## 3. Key Cost Dimensions

### A. Environment Architecture (Load-Balanced vs. Single-Instance)
*   **Load-Balanced Environment (Production Standard):** Beanstalk provisions an Auto Scaling Group, a Load Balancer (ALB), and at least two EC2 instances across Availability Zones.
    *   *Baseline Cost:* Flat monthly ALB charge (~$16.42/month) plus compute and storage for at least two EC2 instances and EBS volumes.
*   **Single-Instance Environment (Dev/Test Standard):** Beanstalk provisions a single EC2 instance with an Elastic IP address. No Load Balancer is created.
    *   *Baseline Cost:* One EC2 instance and EBS volume. The Elastic IP is free because it is attached to a running instance.

### B. Managed Platform Updates & Rolling Deploys
*   Beanstalk manages application deployments (e.g., rolling updates, immutable updates, or blue/green deployments).
*   **Immutable and Blue/Green Deploys:** During these updates, Beanstalk temporarily provisions a double-capacity fleet of EC2 instances and databases. You are billed for these temporary instances for the duration of the deployment.
*   *Note: If deployments frequently fail or roll back, these temporary compute hours can accumulate significant charges.*

### C. Integrated Database (RDS)
*   Beanstalk allows you to create an RDS database instance directly inside the environment configuration.
*   **The Database Risk:** If you delete the Beanstalk environment, **the RDS database is deleted as well** (unless a snapshot was configured). For production workloads, RDS should be created independently of the Beanstalk lifecycle to prevent accidental data loss and separate resource tracking.

### D. Egress & S3 Storage
*   Beanstalk stores application versions (source bundles) in an Amazon S3 bucket created automatically per region.
*   Every code upload adds S3 storage charges ($0.023/GB-month). If you deploy frequently, the version history can grow to gigabytes of data.

---

## 4. Detailed Pricing Rates (us-east-1)

| Billing Dimension | Billing Basis | Rate (us-east-1) | Cost Management Strategy |
|-------------------|---------------|------------------|--------------------------|
| **Beanstalk Fee** | Management | **Free ($0.00)** | N/A |
| **EC2 Instances** | Per running instance second | *See EC2 rates* | Use smaller instance types (e.g., t3/t4g/c8g) |
| **Load Balancer** | Per ALB hour + LCU | **~$0.0225 / hour** | Use Single-Instance mode in non-prod |
| **S3 Storage** | Per GB-month stored | **$0.023 / GB-month** | Set Beanstalk lifecycle rules to delete old versions |
| **RDS Database** | Per database instance hour | *See RDS rates* | Deploy database outside of Beanstalk |

---

## 5. AWS Free Tier Coverage
*   **Orchestration:** Always free.
*   **Underlying Resources:** Eligible for standard EC2, S3, RDS, and ELB free tiers (if configurations match micro-instance limits).

---

## 6. Common Cost Hotspots & Pitfalls
*   **Running Load Balancers in Dev/QA:** Keeping load-balanced configurations for developer sandboxes. A development environment that is rarely accessed does not need a $16.42/month load balancer or multiple EC2 hosts.
*   **Unbounded Version History in S3:** Beanstalk never deletes old application source versions by default. Years of deployment ZIP files accumulate in your account's Beanstalk S3 bucket, creating silent storage charges.
*   **Accidental Database Deletion (RDS Lifecycle):** Coupling RDS databases directly to Beanstalk environments in production, causing unnecessary billing complexity and data loss risk.
*   **Aggressive Scaling Triggers:** Configuring Auto Scaling rules with low thresholds (e.g., scale up if CPU > 40%). A minor traffic burst will trigger the provision of extra EC2 instances that remain active for hours.

---

## 7. Actionable Cost Optimization Strategies
1.  **Configure Single-Instance Mode for Non-Prod:**
    *   In Dev, Test, and Staging environments, set the Environment Type to **Single-Instance**.
    *   **The Savings:** Eliminates the hourly Load Balancer fee and halves your baseline compute cost.
2.  **Configure Beanstalk Application Version Lifecycle Rules:**
    *   In the Beanstalk console, set a lifecycle limit on application versions. Configure it to keep only the last 50 versions or delete versions when the total storage exceeds 5 GB.
    *   **The Savings:** Automatically purges old ZIP bundles from S3.
3.  **Decouple RDS Databases:**
    *   Never create RDS instances directly inside your Beanstalk environment config. Provision your database separately via the RDS console, and pass the connection endpoint to Beanstalk via environment properties.
    *   **The Savings:** Allows you to scale, stop, or manage the database independently.
4.  **Adopt Graviton (ARM) EC2 Instances:**
    *   Update the EC2 instance configuration in your Beanstalk platform settings to use Graviton instance families (e.g., `t4g.micro` or `t4g.medium`).
    *   **The Savings:** Offers a **20% savings** on compute hours.
5.  **Utilize Auto Scaling Schedules:**
    *   If you run load-balanced environments for QA or internal testing, configure scheduled scaling actions in the Auto Scaling group. Set it to scale down to 0 instances at 6:00 PM and scale back up to 1 or 2 instances at 8:00 AM on weekdays.
