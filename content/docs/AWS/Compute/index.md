---
title: "AWS Compute"
---

## Executive Summary

AWS compute provides on-demand capacity across unmanaged, managed, and serverless models. These options let teams balance control, operational overhead, and scaling needs. Compute refers to the processing power required to run applications, including virtual machines, containers, and serverless functions.

AWS compute services allow you to provision and scale resources on demand without managing physical hardware.

## Compute Models

| Model | Level of Control | Management Responsibility | Example Services |
|------|------------------|---------------------------|------------------|
| Unmanaged Compute | High | You manage OS, runtime, scaling | Amazon EC2 |
| Managed Compute | Medium | AWS manages orchestration | Amazon ECS, Amazon EKS |
| Serverless Compute | Low | AWS manages everything except code | AWS Lambda, AWS Fargate |

## Compute Selection Guide

| Use Case | Best Option |
|----------|-------------|
| Full OS control | EC2 |
| Container orchestration | ECS or EKS |
| Run code on events | Lambda |
| Run containers without managing servers | Fargate |
| Kubernetes workloads | EKS |

### Amazon EC2

Amazon Elastic Compute Cloud (EC2) provides resizable virtual machines in the AWS Cloud. EC2 instances run in a Region and can be launched in multiple Availability Zones for high availability.

| Instance Type | Description | Example Use Cases |
| --- | --- | --- |
| General Purpose | Balanced compute, memory, and networking | Web servers, dev/test |
| Compute Optimized | High-performance CPU | ML inference, batch processing |
| Memory Optimized | High memory-to-CPU ratio | Databases, analytics |
| Accelerated Computing | GPUs or FPGAs | ML training, 3D rendering |
| Storage Optimized | High I/O performance | Large NoSQL, data warehouses |

#### EC2 Core Components

- **Amazon Machine Image (AMI):** Template used to launch instances
- **Instance Type:** Hardware configuration (CPU, memory, network)
- **Key Pair:** Secure SSH access method
- **Security Groups:** Instance-level firewall rules
- **Elastic IP:** Static public IP address

#### EC2 Pricing Options

- **On-Demand:** Pay per second without commitments.
- **Reserved Instances:** Long-term discounts.
- **Spot Instances:** Use spare capacity at steep discounts.
- **Savings Plans:** Flexible, commitment-based savings.
- **Dedicated Instances/Hosts:** Single-tenant hardware.

#### Scaling and Elasticity

- **Scalability:** Long-term ability to increase capacity.
- **Elasticity:** Real-time adjustment of resources.

#### EC2 Auto Scaling

Auto Scaling automatically adjusts the number of EC2 instances based on demand or defined policies.

It uses:

- **Minimum Capacity:** Lowest number of instances
- **Desired Capacity:** Target number of instances
- **Maximum Capacity:** Upper limit of scaling

### Elastic Load Balancing (ELB)

ELB distributes incoming traffic across multiple targets to improve availability and reliability.

#### Types of Load Balancers

- Application Load Balancer (ALB): HTTP/HTTPS traffic, Layer 7 routing
- Network Load Balancer (NLB): High-performance TCP/UDP traffic
- Gateway Load Balancer (GWLB): Network appliance integration

### Serverless Compute

#### AWS Lambda

AWS Lambda is an event-driven serverless compute service that runs code in response to triggers.

Common use cases:

- Event-driven processing
- Backend APIs
- File transformation workflows
- Automation tasks

### Containers on AWS

Containers package applications and dependencies into a lightweight, portable unit. AWS provides multiple services to run and manage containers depending on control and scalability needs.

| Service | Description |
| --- | --- |
| Amazon ECS | AWS-native container orchestration |
| Amazon EKS | Managed Kubernetes |
| Amazon ECR | Private container registry |
| AWS Fargate | Serverless compute for ECS and EKS |

### Microservices Architecture

Microservices split applications into independently deployable services to improve resilience, scalability, and deployment speed.

## Key Terms Summary

| Term | Definition |
|------|-----------|
| EC2 | Virtual machines in AWS |
| AMI | Template for EC2 instances |
| Auto Scaling | Automatic adjustment of compute capacity |
| ELB | Distributes traffic across targets |
| Lambda | Serverless event-driven compute |
| Container | Lightweight application packaging unit |
| Fargate | Serverless container runtime |