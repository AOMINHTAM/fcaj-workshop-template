---
title: "Week 3 Worklog"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:
* Design and deploy a Multi-AZ **Amazon Virtual Private Cloud (VPC)** network architecture.
* Master routing mechanisms between Public and Private Subnets using **Internet Gateways (IGW)** and **NAT Gateways**.
* Implement defense-in-depth network security using **Security Groups (Stateful)** and **Network ACLs (Stateless)**.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| :---: | :--- | :---: | :---: | :--- |
| **Mon (Day 2)** | - Study **Amazon VPC** fundamentals and IPv4 CIDR block planning (`10.0.0.0/16`) <br> - Design a dual-tier subnet topology (Public & Private Subnets) across 2 Availability Zones | 12/10/2026 | 12/10/2026 | <https://cloudjourney.awsstudygroup.com/> |
| **Tue (Day 3)** | - **Hands-on VPC Provisioning:** <br>&emsp; + Create custom VPC (`10.0.0.0/16`) <br>&emsp; + Provision 2 Public Subnets and 2 Private Subnets across 2 AZs for High Availability | 13/10/2026 | 13/10/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html> |
| **Wed (Day 4)** | - **Gateway & Route Table Configuration:** <br>&emsp; + Attach **Internet Gateway (IGW)** and configure Public Route Table (`0.0.0.0/0 -> igw-xxxx`) <br>&emsp; + Deploy **NAT Gateway** with Elastic IP in Public Subnet and route outbound Private Subnet traffic | 14/10/2026 | 14/10/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html> |
| **Thu (Day 5)** | - Configure **Security Groups** enforcing Least Privilege (restricting SSH port 22 and internal CIDR traffic) <br> - Set up subnet-level **Network ACLs (NACLs)** | 15/10/2026 | 15/10/2026 | <https://cloudjourney.awsstudygroup.com/> |
| **Fri (Day 6)** | - Launch test EC2 instances in Public and Private Subnets to verify routing isolation and outbound NAT connectivity <br> - **Clean up NAT Gateway, Elastic IP, and EC2 instances** to prevent hourly billing charges | 16/10/2026 | 16/10/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 3 Achievements:
* Deployed a production-grade Multi-AZ Amazon VPC architecture isolating backend workloads inside Private Subnets.
* Mastered packet routing across IGW, NAT Gateway, and Route Tables, as well as the operational differences between Stateful Security Groups and Stateless NACLs.
* Gained hands-on network security skills directly applicable to configuring VPC Security Groups (Port `5439`) for Amazon Redshift Serverless.
