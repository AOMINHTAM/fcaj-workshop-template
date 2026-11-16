---
title: "Week 3 Worklog"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:
* Understand and design a cloud networking infrastructure using a Multi-AZ **Amazon Virtual Private Cloud (VPC)**.
* Master routing between Public Subnets and Private Subnets via **Internet Gateway (IGW)** and **NAT Gateway**.
* Implement multi-layered network security using **Security Groups (Stateful)** and **Network ACLs (Stateless)**.

### Tasks Completed This Week:
* **VPC Architecture Planning & CIDR Allocation:** Designed an Amazon VPC network layout allocating the `10.0.0.0/16` IPv4 CIDR block; structured a 2-tier topology across two Availability Zones for High Availability.
* **Subnet Provisioning:** Provisioned the custom `10.0.0.0/16` VPC, dividing it into 2 isolated Public Subnets and 2 Private Subnets spanning 2 distinct AZs.
* **Gateways & Route Tables Configuration:**
  * Attached an **Internet Gateway (IGW)** and established a Public Route Table directing outbound Internet traffic (`0.0.0.0/0 → igw-xxxx`).
  * Deployed a **NAT Gateway** with an allocated Elastic IP (EIP) in the Public Subnet; configured the Private Route Table to grant outbound Internet egress for software patching while blocking inbound unsolicited traffic.
* **Multi-layer Network Defense (Security Groups & NACLs):**
  * Configured **Security Groups (Stateful)** following Least Privilege rules, restricting SSH port 22 and internal inter-tier traffic.
  * Configured **Network ACLs (Stateless)** providing boundary-level subnet filtering.
* **Network Verification & Resource Clean-Up:** Launched test EC2 instances in both subnets to validate routing flows; completed clean-up by deleting the NAT Gateway, releasing Elastic IPs, and terminating instances to avoid ongoing hourly costs.

### Week 3 Achievements:
* Fully built a production-grade Multi-AZ Amazon VPC, isolating data assets safely within Private Subnets.
* Acquired a deep understanding of packet paths across IGW, NAT Gateways, Route Tables, and the operational differences between Security Groups and NACLs.
* Gained hands-on proficiency in network access rules, directly preparing for secure port `5439` connectivity on Amazon Redshift in subsequent weeks.

### References:
* <https://cloudjourney.awsstudygroup.com/>
* <https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html>
* <https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html>
