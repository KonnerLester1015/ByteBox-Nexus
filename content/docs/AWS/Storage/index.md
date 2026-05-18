---
title: "AWS Storage"
---

## Executive Summary

AWS storage spans block, object, file, and hybrid models. Each option targets different performance, durability, and access requirements. AWS storage services provide scalable, durable, and highly available ways to store and retrieve data for applications, backups, and analytics workloads.

## Storage Types

- **Block Storage:** Raw block devices attached to compute resources.
- **Object Storage:** Flat, scalable storage for unstructured data.
- **File Storage:** Shared file systems accessible over NFS or SMB.
- **Hybrid Storage:** On-premises integrations backed by AWS.

## Storage Models Overview

| Type | Use Case | AWS Services | Key Trait |
|------|----------|--------------|-----------|
| Block Storage | OS volumes, databases | EBS, Instance Store | Low-latency, attached to compute |
| Object Storage | Unstructured data | S3 | Highly scalable, HTTP-based |
| File Storage | Shared file access | EFS | POSIX/NFS shared filesystem |
| Hybrid Storage | On-prem integration | Storage Gateway | Extends on-prem to cloud |

## Block Storage

Block storage provides low-level storage volumes that can be attached to compute resources like EC2.

### EC2 Instance Store

Non-persistent block storage physically attached to the EC2 host.

- Extremely low latency
- High I/O throughput
- Temporary storage lost on stop or terminate

Best for caches, scratch space, and temporary buffers.

### Amazon EBS (Elastic Block Store)

Amazon EBS provides persistent block storage volumes for EC2 instances. Persistent block storage for EC2 instances with resizing, encryption, and snapshotting.

#### EBS Snapshots

Point-in-time backups stored redundantly across multiple Availability Zones using Amazon S3.

#### Data Lifecycle Manager (DLM)

Automates snapshot schedules, retention, and cleanup.

## Object Storage

### Amazon S3

Amazon S3 is an object storage service designed for durability, scalability, and low-cost storage of any data type.

Key features/concepts:

- **Buckets:** Containers for objects
- **Objects:** Files stored in S3
- **Keys:** Unique object identifiers
- **Versioning:** Keeps multiple versions of objects
- **Lifecycle Policies:** Automates storage transitions

### S3 Storage Classes

| Class | Use Case |
|------|----------|
| S3 Standard | Frequently accessed data |
| Intelligent-Tiering | Unknown or changing access patterns |
| Standard-IA | Infrequent access, but fast retrieval |
| One Zone-IA | Lower cost, single AZ data |
| Glacier Instant | Archive with quick access |
| Glacier Flexible | Long-term archives |
| Glacier Deep Archive | Lowest cost, long retention |

## File Storage

### Amazon EFS

Amazon EFS provides scalable shared file storage that can be mounted by multiple EC2 instances simultaneously using NFS.

#### EFS Storage Classes

1. EFS Standard and Standard-IA
2. EFS One Zone and One Zone-IA
3. EFS Archive

#### EFS Lifecycle Management

Moves infrequently accessed files to lower-cost tiers and can promote them back to Standard.

## Hybrid Storage

Hybrid storage connects on-premises environments with AWS cloud storage, enabling data migration, backup, and integration.

### AWS Storage Gateway

Bridges on-premises environments with cloud-backed storage.

- **S3 File Gateway:** SMB or NFS shares backed by S3 with local caching.
- **Volume Gateway:** Cloud-backed iSCSI volumes with EBS snapshots.
- **Tape Gateway:** Virtual tape library for backup workflows.

## Key Terms Summary

| Term | Definition |
|------|-----------|
| EBS | Persistent block storage for EC2 |
| S3 | Object storage service |
| EFS | Shared file storage service |
| Bucket | Container for S3 objects |
| Snapshot | Backup of EBS volume |
| Storage Class | Tier of S3 or EFS storage |
| Storage Gateway | Hybrid storage integration service |