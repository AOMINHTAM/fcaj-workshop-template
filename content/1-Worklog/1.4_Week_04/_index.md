---
title: "Week 4 Worklog"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:
* Architect a multi-layered **Medallion Data Lake** on **Amazon S3** secured with **SSE-KMS** encryption and automated **S3 Lifecycle Rules**.
* Automate schema discovery and metadata governance using **AWS Glue Data Catalog** and **AWS Glue Crawlers**.
* Execute serverless analytical SQL queries with **Amazon Athena** and benchmark **Apache Parquet** columnar storage performance.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| :---: | :--- | :---: | :---: | :--- |
| **Mon (Day 2)** | - Study **Medallion Data Lake Architecture** (`Bronze` → `Silver` → `Gold`) on Amazon S3 <br> - Provision S3 Data Lake prefixes (`bronze-raw/`, `silver-cleaned/`, `gold-curated/`) with **Block Public Access** and **SSE-KMS** encryption | 19/10/2026 | 19/10/2026 | <https://cloudjourney.awsstudygroup.com/> |
| **Tue (Day 3)** | - Configure **S3 Lifecycle Rules** transitioning Bronze objects to `S3 Standard-IA` after 30 days and `S3 Glacier Flexible Retrieval` after 90 days <br> - Upload raw CSV/JSON and partitioned **Apache Parquet** (`year=YYYY/month=MM`) datasets to S3 | 20/10/2026 | 20/10/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html> |
| **Wed (Day 4)** | - Create a unified database in **AWS Glue Data Catalog** <br> - Configure Glue IAM Role and execute an **AWS Glue Crawler** to infer schemas and register partition keys from S3 | 21/10/2026 | 21/10/2026 | <https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html> |
| **Thu (Day 5)** | - Configure **Amazon Athena** query result location on S3 with a 7-day expiration policy <br> - Run serverless analytical SQL queries against Glue Catalog tables | 22/10/2026 | 22/10/2026 | <https://docs.aws.amazon.com/athena/latest/ug/what-is.html> |
| **Fri (Day 6)** | - Benchmark query latency and **Data Scanned** volume on Athena comparing unpartitioned CSV vs. partitioned **Apache Parquet** <br> - Audit and clean up test resources | 23/10/2026 | 23/10/2026 | <https://docs.aws.amazon.com/athena/latest/ug/performance-tuning.html> |

### Week 4 Achievements:
* Deployed a secure, cost-tiered 3-layer Data Lake (`Bronze`, `Silver`, `Gold`) on Amazon S3.
* Automated metadata cataloging and partition discovery using AWS Glue Crawlers.
* Validated an **~85% reduction in scanned data volume** and faster query execution on Amazon Athena by leveraging partitioned Apache Parquet formats.
