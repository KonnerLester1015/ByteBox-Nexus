---
title: "AWS Security"
---

## Executive Summary

AWS security is built on a shared responsibility model in which AWS secures the underlying cloud infrastructure while customers are responsible for securing their data, identities, applications, and configurations.

AWS provides services and features for identity and access management, encryption, monitoring, threat detection, and compliance.

**Managed Security Services**

| Service | Purpose |
|--------|--------|
| AWS IAM | Identity and access management |
| AWS KMS | Encryption key management |
| Amazon GuardDuty | Threat detection |
| AWS Security Hub | Centralized security findings |
| Amazon Inspector | Vulnerability assessments |
| AWS Config | Configuration auditing |
| AWS CloudTrail | API activity logging |
| AWS Shield | DDoS protection |
| AWS WAF | Web application firewall |
| Amazon Macie | Sensitive data discovery |

## Shared Responsibility Model

AWS is responsible for the security **of** the cloud, including:

- Physical data centers
- Hardware
- Networking infrastructure
- Managed service platforms

Customers are responsible for security **in** the cloud, including:

- IAM configuration
- Data encryption
- Operating system patching
- Network security rules
- Application security

## Identity and Access Management (IAM)

AWS Identity and Access Management (IAM) controls who can access AWS resources and what actions they can perform. AWS recommends using IAM Identity Center and temporary credentials whenever possible instead of creating long-term IAM users.

### Core IAM Components

- **Users:** Individual identities.
- **Groups:** Collections of users.
- **Roles:** Temporary identities assumed by users or services.
- **Policies:** JSON documents defining permissions.

### Policy Types

- **Identity-Based Policies**
- **Resource-Based Policies**
- **Service Control Policies (SCPs)** for AWS Organizations
- **Permissions Boundaries**

## Encryption

AWS supports encryption both at rest and in transit.

### Encryption at Rest

Data is encrypted when stored using services such as AWS Key Management Service (KMS).

### Encryption in Transit

Transport Layer Security (TLS) encrypts data moving between clients and AWS services.

### AWS KMS

AWS KMS centrally manages encryption keys used across AWS services.

## Logging and Monitoring

- **AWS CloudTrail:** Records AWS API calls.
- **Amazon CloudWatch:** Monitors metrics and logs.
- **AWS Config:** Tracks resource configuration changes.

## Security Best Practices

- Enable multi-factor authentication (MFA)
- Follow the principle of least privilege
- Rotate credentials regularly
- Use roles instead of long-term access keys
- Encrypt sensitive data
- Enable logging and monitoring
- Automate compliance checks

## Key Terms Summary

| Term | Definition |
|------|-----------|
| IAM | Identity and access management service |
| Policy | JSON document defining permissions |
| Role | Temporary identity with permissions |
| KMS | Managed encryption key service |
| CloudTrail | API activity logging |
| GuardDuty | Threat detection service |
| Security Hub | Centralized security findings |