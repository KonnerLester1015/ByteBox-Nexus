---
title: "AWS Foundations"
---

## Executive Summary

AWS foundations cover the core concepts that apply to every workload: global infrastructure, service access methods, and infrastructure as code. These concepts shape how you design for availability, latency, cost, and compliance.

## Cloud Computing
### What Is Cloud Computing?

Cloud computing is the on-demand delivery of computing resources—such as servers, storage, databases, and networking—over the internet with pay-as-you-go pricing.

Instead of purchasing and maintaining physical hardware, organizations can rent resources from a cloud provider like Amazon Web Services (AWS) and scale them as needed.

### Benefits of Cloud Computing

- Trade upfront capital expense (CapEx) for variable expense (OpEx)
- Benefit from economies of scale
- Stop guessing capacity
- Increase speed and agility
- Focus on core business activities
- Go global in minutes

## AWS Global Infrastructure

### Regions, Availability Zones, and Edge Locations

- **Regions:** Geographic areas with multiple, isolated data centers.
- **Availability Zones (AZs):** Independent data centers within a Region.
- **Edge Locations:** CDN endpoints for low-latency content delivery.

### Region Selection Considerations

- Compliance and regulatory requirements
- Latency and proximity to users
- Service availability by Region
- Cost differences
- Data sovereignty

### Shared Responsibility Model

AWS is responsible for the security **of** the cloud, including physical infrastructure, networking, and managed services.

Customers are responsible for security **in** the cloud, including data protection, IAM permissions, and operating system patching.

### Key Terms Summary

| Term | Definition |
|------|-----------|
| Region | Geographic area containing multiple Availability Zones |
| Availability Zone | One or more isolated data centers within a Region |
| Edge Location | Point of presence used for content delivery |
| Infrastructure as Code | Managing infrastructure using templates |
| Shared Responsibility Model | Security responsibilities divided between AWS and the customer |

## Service Access and Management

- **AWS Management Console:** Web-based management UI.
- **AWS CLI:** Command-line interface for automation.
- **AWS SDKs:** Language-specific APIs for application integration.

## Infrastructure as Code

Infrastructure as Code (IaC) is the practice of defining infrastructure using declarative templates rather than manually creating resources.

### AWS CloudFormation

AWS CloudFormation uses YAML or JSON templates to provision and manage AWS resources consistently and repeatedly.
