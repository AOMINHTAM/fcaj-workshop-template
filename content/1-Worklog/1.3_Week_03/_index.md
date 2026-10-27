---
title: "Week 3: Amazon VPC Networking & Cloud Infrastructure Security"
weight: 3
---

### 🇬🇧 Week 03 Worklog

**1. Tasks Completed:**
- **Custom VPC Architecture Design:**
  - Provisioned a custom Virtual Private Cloud (VPC) with CIDR block `10.0.0.0/16` across 2 Availability Zones (AZs) for high availability.
  - Created a dual-tier subnet hierarchy: 2 Public Subnets (for Internet-facing entry points) and 2 Private Subnets (for isolated data processing workloads).
- **Routing & Gateway Configuration:**
  - Deployed an **Internet Gateway (IGW)** attached to the VPC and associated with the Public Route Table (`0.0.0.0/0 -> igw-xxxx`).
  - Provisioned a managed **NAT Gateway** in the public subnet with an Elastic IP (EIP) to allow secure outbound traffic for private workloads while blocking unsolicited inbound connections.
  - Configured Private Route Tables to direct default egress traffic (`0.0.0.0/0`) through the NAT Gateway.
- **Network Security & Defense-in-Depth:**
  - Configured stateful **Security Groups (SG)** enforcing least-privilege port access (SSH port 22 restricted to administrative IP, internal traffic restricted to VPC CIDR).
  - Configured stateless **Network Access Control Lists (NACLs)** as an additional subnet boundary defense layer.
  - Launched test EC2 instances in both public and private subnets to validate routing paths, internet egress, and isolation.

**2. Key Deliverables & Results:**
- Successfully established an enterprise-grade, multi-AZ network foundation tailored for scalable data pipelines.
- Enforced zero-trust network boundaries preventing direct public exposure of backend data compute nodes.
- Verified end-to-end routing and network security compliance via hands-on connectivity testing.

**3. References:**
- FCAJ Lab Guide: Networking on AWS - Amazon VPC Architecture.
- AWS Documentation: VPC with Public and Private Subnets (NAT).
- AWS Well-Architected Framework: Reliability and Security Pillars.
