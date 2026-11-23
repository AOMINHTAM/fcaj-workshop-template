---
title: "Week 4 Worklog"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:
* Understand object storage fundamentals on **Amazon S3** (Buckets, Objects, Prefixes, Regions, and global naming rules).
* Practice provisioning and securing an **Amazon S3 Bucket** with **100% Block Public Access**, default encryption (**SSE-S3 / SSE-KMS**), and structured folder prefixes.
* Explore **S3 Storage Classes** and configure basic **S3 Lifecycle Rules**, followed by clean-up of all test resources.

### Tasks Completed This Week:
* **Amazon S3 Core Concepts:** Studied S3 object storage architecture, storage tiers (`S3 Standard`, `S3 Standard-IA`, `S3 Glacier`), access controls (Block Public Access, Bucket Policies), and server-side encryption options.
* **First S3 Bucket Provisioning & Security:** Created a hands-on S3 bucket in the AWS Console, enforced **100% Block Public Access**, and enabled default encryption (**SSE-S3 / SSE-KMS**).
* **Folder Hierarchy & Object Operations:** Organized layered folder prefixes (`bronze-raw/`, `silver-cleaned/`, `gold-curated/`), and practiced uploading, inspecting metadata, downloading, and deleting sample datasets (CSV/JSON) via AWS Console and basic AWS CLI commands.
* **S3 Lifecycle Management & Versioning:** Tested S3 Versioning and configured Lifecycle rules to transition objects (`S3 Standard-IA` after 30 days, `S3 Glacier Flexible Retrieval` after 90 days) to practice cloud storage cost optimization.
* **Resource Clean-up:** Removed all test objects, object versions, and temporary buckets after completing the lab to prevent unintended charges.

### Week 4 Achievements:
* Gained solid hands-on familiarity with **Amazon S3** bucket provisioning, private-by-default security controls, encryption, and object management.
* Understood prefix organization and **S3 Lifecycle Rules** as the storage foundation for building the Data Lake landing zone in subsequent weeks.

### References:
* <https://cloudjourney.awsstudygroup.com/>
* <https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html>
* <https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html>
* <https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html>
