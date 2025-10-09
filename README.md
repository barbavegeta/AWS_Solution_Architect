# Solution Architect – AWS Migration Design

This repository showcases a **complete AWS architecture solution** for migrating two on-premises workloads, a **three-tier web application** and a **Hadoop-based analytics environment**, into a **fully managed, cloud-native design** on AWS.  
The project emphasizes scalability, high availability, security, and minimal operational overhead through modern managed services.

---

## Overview

The architecture demonstrates how legacy, on-prem environments can be modernized using AWS services.  
It replaces physical servers with **resilient, fault-tolerant cloud infrastructure** while maintaining a clear separation of web and analytics workloads.

![AWS Architecture Diagram](aws_architecture_diagram-Solution_Architect.png)

---

## Web Application Architecture (Three-Tier Design)

### Frontend
- **Amazon S3** hosts static assets (HTML, CSS, JS) configured for **static website hosting**.
- **Amazon CloudFront** delivers global content distribution over HTTPS for low latency and improved security.

### Backend
- **Application Load Balancer (ALB)** routes dynamic requests to containerized workloads.
- **Amazon ECS (Fargate)** runs Java application containers with automatic scaling and zero server management.
- **Amazon ElastiCache (Redis)** provides in-memory caching for frequently accessed data.
- **Amazon SQS** decouples asynchronous tasks to improve application responsiveness.

### Database Layer
- **Amazon Aurora MySQL (Multi-AZ)** ensures high availability with automated replication, backups, and failover.
- **AWS Secrets Manager** securely stores and rotates database credentials.

### Security & Observability
- **AWS WAF** protects the edge layer from common web exploits.
- **IAM roles and security groups** enforce least-privilege access.
- **Amazon CloudWatch** and **AWS X-Ray** provide performance monitoring, logging, and tracing.

---

## Data Analytics Architecture (Hadoop Modernization)

The analytics environment transitions from a self-managed Hadoop cluster to AWS’s managed big-data ecosystem.

### Data Ingestion
- **AWS DataSync** migrates on-premises datasets into an **Amazon S3 Data Lake**.

### Processing & Metadata
- **AWS Glue Data Catalog** automatically discovers and classifies datasets.
- **Amazon EMR (Spark/Hive)** provides scalable data processing while maintaining Hadoop compatibility.

### Querying & Analysis
- **Amazon Athena** runs serverless SQL queries directly on S3 data.
- **Amazon Redshift** supports analytical storage, complex joins, and warehouse queries.
- **Amazon QuickSight** visualizes insights through interactive BI dashboards.

---

## Integration Flow

1. **User traffic** enters via CloudFront → ALB → ECS → Aurora.  
2. **ElastiCache** accelerates responses and **SQS** manages asynchronous processing.  
3. **Enterprise data** flows from on-premises → DataSync → S3 → Glue → EMR → Athena/Redshift → QuickSight.

This ensures parallel yet integrated operation between web and analytics workloads.

---

## Outcome

**Migration Goals Achieved**
- Decoupled, fault-tolerant design across all tiers  
- Multi-AZ high availability  
- Cloud-native modernization for both web and analytics workloads  
- Minimal maintenance overhead through managed services  
- Comprehensive observability and integrated security

---

## AWS Services Used

| Category | Services |
|-----------|-----------|
| **Frontend & Delivery** | S3 · CloudFront |
| **Application Layer** | ECS Fargate · ALB · ElastiCache · SQS |
| **Database Layer** | Aurora MySQL · Secrets Manager |
| **Analytics** | DataSync · S3 · Glue · EMR · Athena · Redshift · QuickSight |
| **Security & Monitoring** | WAF · IAM · CloudWatch · X-Ray |

---

## Deployment & Cost Optimization

### Deployment Options
- Infrastructure can be provisioned using **AWS CloudFormation** or **Terraform** templates.  
- **CI/CD pipelines** (AWS CodePipeline + CodeBuild) can automate container deployment to ECS.  
- **S3 bucket policies** and **CloudFront OAC (Origin Access Control)** secure the web content layer.

### Cost Optimization Strategies
- Enable **Auto Scaling** on ECS and Aurora for demand-based resource allocation.  
- Use **S3 lifecycle rules** to transition infrequently accessed data to **S3 Glacier**.  
- Configure **Spot Instances** for EMR clusters to reduce compute costs up to 70%.  
- Leverage **QuickSight SPICE** for faster, cached BI performance with predictable pricing.  
- Apply **AWS Budgets** and **Cost Explorer** to monitor usage and set cost thresholds.
