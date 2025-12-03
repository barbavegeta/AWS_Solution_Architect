# Solution Architect - AWS Migration Design  
High-level AWS architecture design for migrating two on‑premises workloads, a three‑tier web application and a Hadoop‑based analytics environment, into a cloud‑native, fully managed solution on AWS.  

The focus is on **scalability**, **high availability**, **security**, and **reduced operational overhead** by leaning on managed services instead of self‑managed infrastructure.

**AWS Tech stack**: S3 · CloudFront · ECS Fargate · Aurora · ElastiCache · EMR · Glue · Athena · Redshift · QuickSight

---
## Scenario
An on‑prem data centre currently hosts:
- A customer‑facing three‑tier web application.
- A Hadoop analytics cluster processing operational and historical data.
The goal is to modernise both workloads by:
- Moving customer traffic and application logic to AWS.
- Migrating data and batch analytics to a cloud‑native data lake and analytics stack.
- Maintaining secure, hybrid connectivity to the on‑prem environment during migration.
---
## Architecture diagram
![AWS Architecture Diagram](aws_architecture_diagram-Solution_Architect.png)
The diagram shows:
- **Web / API traffic path** from users through CloudFront and WAF into a containerised app on ECS Fargate backed by Aurora.
- **Data ingestion path** from on‑prem files/HDFS into an S3 data lake via DataSync.
- **Analytics path** from S3 through Glue/EMR to Athena/Redshift/QuickSight.
---
## Web Application Architecture (Three‑Tier Design)
### Edge & Frontend
- **Amazon S3 - Static Website Hosting**  
  Hosts static assets (HTML/CSS/JS) for the web front‑end.
- **Amazon CloudFront**  
  Global CDN distribution over HTTPS for low‑latency content delivery.
- **AWS WAF (Web Application Firewall)**  
  Protects the application edge from common web exploits (SQL injection, XSS, bots).
### Application Layer
- **Application Load Balancer (ALB)**  
  - Terminates HTTPS from CloudFront.
  - Routes dynamic `/api` traffic to the containerised backend.
- **Amazon ECS on Fargate - Java App**  
  - Runs stateless Java microservices as containers.
  - Auto‑scales based on traffic and CPU/memory metrics.
  - No EC2 server management.
- **Amazon ElastiCache for Redis**  
  - In‑memory cache for sessions, hot data and frequently used queries.
  - Reduces load and latency against Aurora.
- **Amazon SQS (Queue)**  
  - Asynchronous decoupling of backend tasks (e.g. emails, report generation, ETL triggers).
  - Improves responsiveness of the synchronous API.
### Database Layer
- **Amazon Aurora MySQL - Multi‑AZ**  
  - Managed relational database with automatic backups, failover and replication.
  - Multi‑AZ deployment for high availability.
- **AWS Secrets Manager**  
  - Manages and rotates DB credentials.
  - Injects secrets into ECS tasks via IAM roles instead of hard‑coding environment variables.
### Observability
- **Amazon CloudWatch Logs & Metrics**  
  - Collects application logs, infrastructure metrics and ALB access logs.
  - Drives alarms for error rates, latency and resource saturation.
- **AWS X‑Ray**  
  - End‑to‑end tracing to identify slow calls and bottlenecks across the application path.
---
## Data & Analytics Architecture (Hadoop Modernisation)
The analytics workload is migrated from a self‑managed Hadoop cluster to AWS‑managed services around an **S3 data lake**.
### Ingestion & Storage
- **On‑prem Files or HDFS → AWS DataSync → S3 Data Lake**  
  - DataSync securely transfers batch files and HDFS exports into S3.
  - S3 becomes the central data lake for raw and curated datasets.
### Metadata & Processing
- **AWS Glue Data Catalog**  
  - Central schema and table metadata for data stored in S3.
  - Enables schema‑on‑read for Athena, EMR and Redshift Spectrum.
- **Amazon EMR (Spark / Hive)**  
  - Elastic cluster for batch processing and ETL.
  - Uses Spark/Hive on top of S3 as storage instead of HDFS.
  - Can be configured with transient clusters that spin up for jobs and then terminate.
### Query & Analytics Consumers
- **Amazon Athena**  
  - Serverless interactive SQL queries directly against S3 data via Glue Catalog.
  - Ideal for ad‑hoc analysis, data exploration and lightweight reporting.
- **Amazon Redshift**  
  - Columnar data warehouse for highly structured, performance‑sensitive workloads.
  - Consumes curated data from S3 (via COPY or external tables).
- **Amazon QuickSight**  
  - BI and dashboarding tool connecting to Athena/Redshift.
  - Provides interactive dashboards for business stakeholders.
---
## Hybrid Connectivity
- **Site‑to‑Site VPN / AWS Direct Connect** (shown at the left of the diagram)  
  - Secure connectivity between the on‑prem data centre and AWS VPC.
  - Supports phased migration and hybrid workloads during transition.
---
## End‑to‑End Flow
1. **User requests** hit CloudFront over HTTPS.
2. CloudFront forwards:
   - Static content requests to **S3 static website**.
   - Dynamic `/api` traffic via **ALB** to **ECS Fargate**.
3. ECS containers:
   - Read/write data via **Aurora MySQL**.
   - Use **ElastiCache** for fast cached data.
   - Offload long‑running tasks to **SQS**.
4. All application activity is visible in **CloudWatch Logs/Metrics** and **X‑Ray**.
5. **DataSync** migrates operational and historical data from on‑prem files/HDFS into the **S3 data lake**.
6. **Glue** crawlers catalog the datasets.
7. **EMR** jobs transform and enrich data, writing curated outputs back to S3 (often partitioned).
8. **Athena** and **Redshift** query the curated layer; **QuickSight** sits on top for BI dashboards.
---
## AWS Services Grouped by Layer
- **Edge & Frontend**  
  - Amazon S3, Amazon CloudFront, AWS WAF
- **Application Layer**  
  - Application Load Balancer  
  - Amazon ECS on Fargate (Java app)  
  - Amazon ElastiCache (Redis)  
  - Amazon SQS
- **Database & Secrets**  
  - Amazon Aurora MySQL (Multi‑AZ)  
  - AWS Secrets Manager
- **Data & Analytics**  
  - AWS DataSync, Amazon S3 (Data Lake), AWS Glue Data Catalog  
  - Amazon EMR (Spark / Hive)  
  - Amazon Athena, Amazon Redshift, Amazon QuickSight
- **Security, Monitoring & Hybrid Connectivity**  
  - AWS WAF, IAM, Security Groups  
  - Amazon CloudWatch Logs & Metrics, AWS X‑Ray  
  - Site‑to‑Site VPN / AWS Direct Connect (for on‑prem integration)
---
## Deployment & Cost Considerations
This repository focuses on the **architecture design**, not a full IaC implementation, but the stack is suitable for:
- **Infrastructure as Code**  
  - Provisioning via AWS CloudFormation or Terraform modules.
  - Version‑controlled templates for repeatable environments (dev / test / prod).
- **CI/CD for the Application**  
  - Pipelines using AWS CodePipeline + CodeBuild / GitHub Actions to build and deploy Docker images to ECS.
- **Cost Optimisation**  
  - Auto Scaling for ECS services and Aurora capacity.
  - S3 lifecycle policies (Standard → Infrequent Access → Glacier) for historical data.
  - Spot Instances for EMR clusters where tolerant to interruption.
  - QuickSight SPICE caching for predictable BI costs.
  - AWS Budgets + Cost Explorer for monitoring usage and alerts.
---
## Purpose of this Repository
- Demonstrates the ability to design a **realistic migration architecture** from on‑prem to AWS.
- Shows understanding of:
  - Multi‑tier web design with managed container and database services.
  - Modern data lake & analytics patterns on AWS.
  - Security, observability, and hybrid connectivity best practices.
- Includes a single high‑level diagram and this design document; additional IaC or sample configs can be layered on later if needed.
