---
title: "AWS Networking"
---

## Executive Summary

AWS networking centers on VPC design, secure connectivity, and global routing. These building blocks control isolation, access, and performance for cloud workloads. AWS networking defines how cloud resources communicate with each other, the internet, and on-premises systems.

It controls:
- Isolation of workloads
- Traffic routing
- Security boundaries
- Hybrid connectivity

## Amazon VPC (Virtual Private Cloud)

A Virtual Private Cloud (VPC) is a logically isolated network within AWS where you define your own IP address range and control network architecture.

- **Subnets:** Segments of a VPC (public or private)
- **Route Tables:** Control traffic routing within the VPC
- **Internet Gateway (IGW):** Enables internet access
- **NAT Gateway:** Allows private subnets outbound internet access
- **Security Groups:** Instance-level firewall
- **Network ACLs:** Subnet-level firewall

## Internet Access and Private Connectivity

AWS provides multiple ways to connect networks depending on security and performance needs.

- **Internet Gateway:** Enables communication between VPC resources and the public internet.
- **Virtual Private Gateway and VPN:** Encrypted VPN connectivity between on-premises networks and AWS.

### Additional Private Connectivity Options

- **AWS Client VPN:** Remote workforce access.
- **Site-to-Site VPN:** Connects on-premises data centers to AWS.
- **AWS PrivateLink:** Private access to AWS services.
- **AWS Direct Connect:** Dedicated high-bandwidth private connection.

## Network Security

AWS provides layered network security controls at both subnet and instance levels.

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

Amazon Route 53 is a scalable DNS service that routes user traffic to AWS resources and external endpoints.

### Amazon CloudFront

Amazon CloudFront is a content delivery network (CDN) that caches content at edge locations worldwide to reduce latency and improve performance.
