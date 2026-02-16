---
title: "Amazon Web Services (AWS)"
---

## Executive Summary

Amazon Web Services (AWS) provides a comprehensive suite of cloud services designed to support modern application development, global infrastructure deployment, virtual networks, scalable compute, and flexible storage. These services enable organizations to innovate faster, improve reliability, enhance security, and reduce the operational burden of managing physical hardware.

This document introduces the foundational AWS concepts that form the basis of cloud computing on AWS, including compute services, storage services, networking, global infrastructure, scalability, containers, and serverless architecture. It provides a structured understanding of how AWS components work together to deliver secure, scalable, and highly available cloud solutions.

## AWS Compute

### What Is Cloud Compute?

Cloud compute refers to running applications using virtualized computing resources provided on demand by AWS. These resources replace traditional on-premises hardware and enable rapid deployment, automated scaling, cost optimization, and operational efficiency.

AWS compute offerings fall into three categories:

- **Unmanaged Compute:** You manage the OS and application stack (e.g., EC2).
- **Managed Compute:** AWS manages orchestration infrastructure (e.g., ECS and EKS).
- **Serverless Compute:** You provide only the code; AWS manages everything else (e.g., Lambda, Fargate).

### Why Use AWS Compute?

Using AWS compute services provides:

- Rapid provisioning of resources
- High availability and fault tolerance
- Global reach across many Regions
- Cost efficiency through flexible pricing models
- Performance elasticity via automatic scaling
- Reduced operational overhead

### Amazon EC2 (Elastic Compute Cloud)

Amazon EC2 provides scalable virtual machines. AWS offers multiple instance families optimized for specific workloads:

| Instance Type | Description | Example Use Cases |
| ------------- | ----------- | ----------------- |
| General Purpose | Balanced compute, memory, and networking | Web servers, dev/test |
| Compute Optimized | High-performance CPU | ML inference, batch processing |
| Memory Optimized | High memory-to-CPU ratio | Databases, analytics |
| Accelerated Computing | GPUs or FPGAs | ML training, 3D rendering |
| Storage Optimized | High I/O performance | Large NoSQL or data warehouses |

### EC2 Pricing Options

- **On-Demand:** Pay per second without commitments.
- **Reserved Instances:** Deep discounts for long-term usage.
- **Spot Instances:** Use spare AWS capacity at up to 90% off.
- **Savings Plans:** Flexible, commitment-based cost savings.
- **Dedicated Instances/Hosts:** Single-tenant hardware.

### AWS Service Access

AWS resources can be managed through:

- **AWS Management Console**
- **AWS CLI**
- **AWS SDKs** for development languages

## Scaling and Elasticity

### Scalability vs. Elasticity

- **Scalability:** Long-term ability to increase capacity.
- **Elasticity:** Real-time automatic adjustment of resources.

### EC2 Auto Scaling

Auto Scaling adjusts instance counts using:

- Minimum capacity
- Desired capacity
- Maximum capacity

It supports dynamic and predictive scaling to optimize performance and cost.

### Elastic Load Balancing (ELB)

Elastic Load Balancing distributes incoming traffic across compute resources to improve fault tolerance and optimize utilization. It works with Auto Scaling for seamless scaling.

## Serverless Compute

### AWS Lambda

AWS Lambda runs code without provisioning servers. AWS handles scaling, patching, and fault tolerance. You pay only for compute time consumed.

Common use cases include:

- Event-driven processing
- Backend APIs
- File transformation workflows
- Automation tasks

## Containers on AWS

### Containers vs Virtual Machines

Containers are lightweight environments containing application code and dependencies. They share the underlying OS, start quickly, and provide consistent deployment experiences. Virtual machines include a full OS stack and are heavier but more isolated.

### AWS Container Services

| Service | Description |
| ------- | ----------- |
| Amazon ECS | AWS-native container orchestration |
| Amazon EKS | Managed Kubernetes |
| Amazon ECR | Private container image registry |
| AWS Fargate | Serverless compute for ECS and EKS |

## Microservices Architecture

Monolithic applications combine all functionality into one tightly coupled system. Microservices break applications into independently deployable services. This improves resilience, scalability, deployment speed, and fault isolation.

## AWS Global Infrastructure

### Regions, AZs, and Edge Locations

- **Regions:** Separate geographic areas that contain multiple isolated data centers.
- **Availability Zones (AZs):** Independent data centers within a Region.
- **Edge Locations:** CDN endpoints used for low-latency content delivery.

### Region Selection Considerations

- Compliance and regulatory requirements  
- Latency and user proximity  
- Service availability  
- Cost differences  
- Data sovereignty  

### AWS Infrastructure Advantages

- High availability  
- Global scalability  
- Rapid deployment  
- Operational agility  

### CloudFormation

AWS CloudFormation allows you to define infrastructure as code for consistent, repeatable, automated deployments.

## AWS Networking

### Amazon VPC (Virtual Private Cloud)

A VPC provides an isolated virtual network in AWS where you define routing, IP ranges, subnets, internet access, and security controls.

### Internet Gateway

Allows communication between VPC resources and the public internet.

### Virtual Private Gateway and VPN

A Virtual Private Gateway enables encrypted VPN connections between on-premises networks and AWS, allowing secure data transfer over the public internet.

### Additional Private Connectivity Options

- **AWS Client VPN:** Remote workforce access
- **Site-to-Site VPN:** Connects on-premises data centers to AWS
- **AWS PrivateLink:** Private access to AWS services
- **AWS Direct Connect:** Dedicated high-bandwidth private connection to AWS

## Network Security

### Network ACLs

- Stateless packet filtering
- Subnet-level control
- Default ACL allows all; custom ACL denies all until rules added

### Security Groups

- Stateful packet filtering  
- Instance-level control  
- Default inbound denied, outbound allowed  
- Automatically allows return traffic  

## Global Networking and Content Delivery

### Amazon Route 53

A global DNS service used for domain registration, DNS routing, health checks, and connecting users to AWS-hosted or on-premises infrastructure.

### Amazon CloudFront

A global content delivery network that caches data at edge locations to reduce latency and improve application performance.

## AWS Storage Services

### Overview of Storage Types

AWS provides multiple storage models to support diverse workloads:

- **Block Storage:** Raw block devices attached to compute resources  
- **Object Storage:** Flat, scalable storage for unstructured data  
- **File Storage:** Shared file systems accessible over NFS/SMB  
- **Hybrid Storage:** On-premises integrations backed by AWS  

Each model addresses different performance, durability, and accessibility needs.

## Block Storage

Block storage provides persistent, low-latency storage that behaves like a physical disk attached to an EC2 instance.

There are two primary block storage options:

### Amazon EC2 Instance Store

An instance store is non-persistent block storage physically attached to the EC2 host server. It offers:

- Extremely low-latency performance  
- High I/O throughput  
- No additional cost (included with instance type)  
- Temporary storage that is lost when the instance stops  

Best for:

- Temporary buffers  
- Caches  
- Scratch space  
- High-speed ephemeral processing  

Instance store volumes cannot retain data after the instance stops or terminates.

### Amazon Elastic Block Store (EBS)

EBS provides persistent block storage for EC2 instances. It offers:

- Consistent low-latency performance  
- Ability to attach/detach volumes  
- Resizing without downtime  
- Encryption and snapshotting  
- Integration with EC2 Auto Scaling groups  

EBS is ideal for:

- Databases  
- File systems  
- Enterprise applications requiring durability  

### EBS Snapshots

EBS snapshots are point-in-time backups stored redundantly across multiple Availability Zones using Amazon S3.

Benefits include:

- Incremental backups  
- Disaster recovery  
- Data migration  
- Cloning new volumes  

New snapshots store only changed blocks since the last one, reducing cost and time.

### Snapshot Automation with Data Lifecycle Manager (DLM)

Data Lifecycle Manager can automate:

- Snapshot schedules  
- Retention rules  
- Automated cleanup of old snapshots  

This is essential for environments with many EBS volumes.

## Object Storage

### Amazon Simple Storage Service (S3)

Amazon S3 provides durable, highly available object storage with virtually unlimited scalability.

Key characteristics:

- Stores data as objects in buckets  
- 99.999999999% durability  
- Versioning and lifecycle policies  
- Seamless integration with other AWS services  
- Granular access control  
- Encryption at rest and in transit  

### S3 Objects and Buckets

An S3 object contains:

- Data  
- Metadata  
- A unique key (identifier)  

Buckets are global, uniquely named containers for storing objects. They form the basis of access control and organization.

### Security in S3

- **Bucket Policies:** Resource-based policies attached to buckets  
- **Identity-Based Policies:** IAM permissions granting users access  
- **Encryption at Rest:** Protects stored data  
- **Encryption in Transit:** Secures data moving between clients and S3  

### S3 Storage Classes

AWS offers multiple storage classes optimized for different access patterns:

1. **S3 Standard** – Frequent access  
2. **S3 Intelligent-Tiering** – Automated tiering across access levels  
3. **S3 Standard-IA** – Infrequent access  
4. **S3 One Zone-IA** – Single-AZ infrequent access  
5. **S3 Glacier Instant Retrieval** – Archive with millisecond access  
6. **S3 Glacier Flexible Retrieval** – Low-cost archives with minutes-to-hours retrieval  
7. **S3 Glacier Deep Archive** – Lowest-cost, long-term archival storage  

## File Storage

### Amazon Elastic File System (EFS)

EFS is a fully managed, scalable NFS file system accessible by multiple EC2 instances concurrently. It automatically grows and shrinks as files are added or removed.

### EFS Storage Classes

1. **EFS Standard** and **Standard-IA** – Multi-AZ, high durability  
2. **EFS One Zone** and **One Zone-IA** – Lower cost, single-AZ  
3. **EFS Archive** – Lowest-cost file storage tier for rarely accessed data  

### EFS Lifecycle Management

Lifecycle policies include:

- **Transition to IA:** Moves files not accessed for 30 days  
- **Transition to Archive:** Moves files not accessed for 90 days  
- **Transition to Standard:** Optional promotion back to Standard when accessed  

## Hybrid Storage

### AWS Storage Gateway

Storage Gateway integrates on-premises environments with cloud-backed storage.

Three types of gateways:

1. **S3 File Gateway**  
   - Provides SMB/NFS file shares  
   - Stores data in S3 with local caching  

2. **Volume Gateway**  
   - Exposes cloud-backed iSCSI volumes  
   - Supports cached and stored volume modes  
   - Creates EBS snapshots for backups  

3. **Tape Gateway**  
   - Virtual tape library compatible with backup software  
   - Replaces physical tape systems  

# AWS Database Services

Modern applications require data systems that are scalable, reliable, and flexible. AWS provides a broad portfolio of fully managed database services tailored to different data models and workload patterns. Selecting the right database type can greatly improve performance, durability, and operational efficiency.

---

## Relational Databases (SQL)

Relational databases organize data into tables with defined relationships and schemas. They are ideal for transactional workloads, financial systems, inventory systems, and applications requiring strong consistency.

### Amazon RDS (Relational Database Service)

Amazon RDS is a fully managed service that automates common administrative tasks such as provisioning, backups, patching, and hardware maintenance. It supports several popular engines including MySQL, PostgreSQL, MariaDB, SQL Server, and Oracle.

**Why RDS Is Important**

- Removes undifferentiated heavy lifting so teams can focus on application logic  
- Provides automated backups and Multi-AZ failover for high availability  
- Enables scaling through larger instance classes or read replicas  
- Improves security through encryption, network isolation, and IAM integration  

RDS is ideal for teams migrating existing applications into the cloud or deploying new transactional systems requiring structured data.

---

## Amazon Aurora

Amazon Aurora is a MySQL- and PostgreSQL-compatible relational database engineered for high performance and high availability. Aurora separates compute from storage, automatically replicating data across multiple Availability Zones.

**Key Benefits**

- Up to 5× the performance of MySQL with minimal code changes  
- Fault-tolerant storage that self-heals and auto-scales  
- Continuous backups with no impact on performance  
- High availability with automatic failover  

Aurora is a strong fit for modern, large-scale applications that require strong consistency and enterprise-grade reliability.

---

## NoSQL Databases

NoSQL databases manage data using flexible structures such as key-value, document, or graph models. They are ideal for large-scale applications requiring high throughput, low latency, and dynamic schemas.

### Amazon DynamoDB

DynamoDB is a fully managed key-value and document database delivering single-digit millisecond performance at any scale. It automatically manages partitioning, scaling, fault tolerance, and performance tuning.

**Why DynamoDB Matters**

- Serverless with no infrastructure to manage  
- Scales seamlessly to millions of requests per second  
- Includes built-in security, encryption, and access control  
- DynamoDB Streams enables event-driven architectures  
- Ideal for session stores, real-time data, IoT, and serverless applications  

---

## Document Databases

### Amazon DocumentDB (MongoDB-Compatible)

Amazon DocumentDB stores and queries JSON-like documents with flexible schemas. It is designed for applications that frequently evolve their data model and rely on document-oriented structures.

**Key Advantages**

- MongoDB API compatibility  
- Automatic scaling of compute and storage  
- Continuous backups and high availability  
- Ideal for catalogs, content management, and user profile systems  

---

## In-Memory Databases and Caches

### Amazon ElastiCache (Redis / Valkey / Memcached)

ElastiCache improves application performance by storing frequently accessed data in memory (RAM) rather than disk.

**Use Cases**

- Caching query results  
- Session stores  
- Leaderboards and gaming applications  
- Real-time analytics  
- Reducing load on primary databases  

ElastiCache provides microsecond latency and automatic failover for highly available caching layers.

---

## Graph Databases

### Amazon Neptune

Amazon Neptune is a purpose-built graph database optimized for storing and querying highly connected datasets.

**Ideal For**

- Social networks  
- Fraud detection  
- Recommendation systems  
- Knowledge graphs  

Neptune can traverse complex relationships quickly, making it ideal for applications where connections are as important as the data itself.

---

## Backup and Data Protection

### AWS Backup

AWS Backup centralizes and automates backup policies across services such as EBS, RDS, DynamoDB, EFS, and FSx.

**Benefits**

- Consistent, automated backups across diverse workloads  
- Meets compliance and retention requirements  
- Provides cross-Region backup copying  
- Reduces operational overhead with a unified dashboard  

---

# AWS AI/ML and Data Analytics

Artificial Intelligence (AI) and Machine Learning (ML) help organizations turn raw data into predictions, insights, and automation. AWS provides a layered suite of services that support all levels of expertise—from fully managed AI APIs to custom-built ML frameworks.

---

## Understanding AI and ML

### Artificial Intelligence (AI)

AI refers to systems that perform tasks requiring human-like intelligence, such as understanding language, identifying objects, or making decisions.

### Machine Learning (ML)

ML is a subset of AI where models learn patterns from historical data and use those patterns to make predictions on new data.

**Typical ML Workflow**

1. Collect and prepare data  
2. Train and evaluate a model  
3. Deploy the model  
4. Continuously monitor and improve performance  

---

# AI/ML Architecture Layers in AWS

AWS organizes its ML services into three tiers, each serving different user skill levels and sophistication requirements.

---

## Tier 1 — AI Services (Pre-Built Models)

AI services provide ready-made intelligence without requiring ML expertise. These services solve common business problems through simple API calls.

### Language AI

- **Amazon Comprehend** — Extracts sentiment, key phrases, entities, and insights from text.  
- **Amazon Transcribe** — Converts speech to text with speaker identification and custom vocabularies.  
- **Amazon Translate** — Provides real-time and batch language translation.  
- **Amazon Polly** — Converts text into natural-sounding speech for accessibility or voice-enabled apps.

### Vision and Search AI

- **Amazon Rekognition** — Detects faces, objects, scenes, and text in images and videos.  
- **Amazon Textract** — Extracts structured and unstructured data from forms and documents.  
- **Amazon Kendra** — Enterprise search using natural language understanding to deliver precise answers.

### Conversational and Personalization AI

- **Amazon Lex** — Builds conversational chatbot experiences with ASR and NLU.  
- **Amazon Personalize** — Generates real-time, personalized recommendations using historical user behavior.

---

## Tier 2 — ML Services (Build, Train, Deploy Your Own Models)

### Amazon SageMaker

SageMaker is a fully managed machine learning platform that simplifies the entire ML lifecycle—including data preparation, model training, deployment, and monitoring.

**Key Capabilities**

- SageMaker Studio IDE for managing ML workflows  
- Built-in algorithms and notebooks  
- Automated model tuning and training job tracking  
- Real-time and batch inference endpoints  
- SageMaker JumpStart for deploying pretrained models  

SageMaker is ideal for data scientists and ML engineers wanting more control without managing servers.

---

## Tier 3 — Frameworks and Custom ML Infrastructure

Advanced users can run their own ML frameworks using:

- TensorFlow  
- PyTorch  
- MXNet  

These run on services such as:

- Amazon EC2 ML-optimized instances  
- Amazon EMR for distributed data processing  
- Amazon ECS or Amazon EKS for containerized ML workloads  

This tier is designed for organizations that require complete control over training environments.

---

# Deep Learning and Generative AI

### Deep Learning

Deep learning uses multi-layer neural networks to recognize complex patterns. It powers many modern AI capabilities including voice assistants, image recognition, and NLP models.

### Generative AI

Generative AI uses foundation models (FMs) to create text, images, audio, and more. These models are trained on massive datasets and can be adapted to a variety of tasks.

**AWS Generative AI Services**

- **Amazon Bedrock** — Fully managed service for deploying and customizing foundation models from AWS and third-party providers.  
- **SageMaker JumpStart** — Provides prebuilt FMs and templates for quick deployment.  
- **Amazon Q** — An enterprise-ready AI assistant for business and development workflows.

---

# Data Analytics

Data analytics is the process of transforming raw data into insights. Both AI/ML and analytics require clean, accessible data.

## The ETL Process

1. **Extract** — Gather data from source systems  
2. **Transform** — Clean, enrich, and standardize data  
3. **Load** — Store data into a destination like a data warehouse  

AWS supports ETL and data pipelines with services like AWS Glue, Amazon Kinesis, Amazon EMR, Lambda, and Step Functions.

---

# AWS Security

Security in AWS is based on the shared responsibility model: AWS secures the cloud, and customers secure what they run in the cloud. AWS provides a broad set of tools to help customers enforce identity controls, governance, encryption, and threat monitoring.

---

## Identity and Access Control

### AWS IAM (Identity and Access Management)

IAM manages authentication and authorization for AWS resources.

**What IAM Provides**

- Fine-grained access controls  
- Identity-based and resource-based policies  
- Role-based access for AWS services or external identities  
- MFA enforcement  
- Secure authentication without long-term credentials  

### Best Practices

- Enforce least privilege  
- Use IAM Roles instead of access keys  
- Enable MFA for all user accounts  
- Avoid using the root user  
- Use Access Analyzer to detect overly permissive policies  

---

## AWS Identity Center

AWS Identity Center enables centralized workforce authentication and authorization across AWS Organizations and enterprise applications.

**Why It's Important**

- Provides single sign-on (SSO)  
- Integrates with identity providers like Okta or Microsoft Entra ID  
- Simplifies access management across multiple AWS accounts  

---

## Secrets and Configuration Security

### AWS Secrets Manager

Securely stores and rotates:

- API keys  
- Database passwords  
- OAuth tokens  

### AWS Systems Manager Parameter Store

Stores application configurations and encrypted parameters used by applications and automation workflows.

---

## Encryption and Key Management

### AWS Key Management Service (KMS)

KMS provides centralized creation, management, rotation, and auditing of encryption keys used across AWS services such as:

- S3  
- EBS  
- RDS  
- DynamoDB  
- Lambda  

Encryption at rest and in transit is a foundational security requirement for protecting data.

---

## Network Security

### Security Groups

- Stateful firewalls applied at the instance or resource level  
- Automatically allow return traffic  
- Ideal for controlling inbound/outbound access at a granular level  

### Network ACLs

- Stateless filters applied at the subnet level  
- Must explicitly allow return traffic  
- Provide an additional layer of security beyond Security Groups  

---

## Threat Detection and Continuous Monitoring

AWS provides several fully managed services to help customers identify threats, perform investigations, and maintain compliance.

- **Amazon GuardDuty** — Intelligent threat detection for accounts and workloads  
- **Amazon Inspector** — Automated vulnerability scanning for EC2, Lambda, and container images  
- **AWS Security Hub** — Centralized visibility into security posture across AWS  
- **Amazon Detective** — Helps find root causes behind security events  

These services minimize the need for building and maintaining custom security detection logic.