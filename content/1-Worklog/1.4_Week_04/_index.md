---
title: "Week 4: Amazon S3 Data Lake Architecture & Serverless Analytics with Athena"
weight: 4
---

### 🇬🇧 Week 04 Worklog

**1. Tasks Completed:**
- **Amazon S3 Data Lake Foundation:**
  - Designed a multi-layered storage layout for the Data Lake following the medallion architecture pattern:
    - `bronze-raw/`: Ingested raw datasets in uncompressed format (JSON / CSV).
    - `silver-cleaned/`: Cleaned and standardized schema data.
    - `gold-curated/`: Partitioned columnar format (Apache Parquet) optimized for analytical queries.
  - Enforced strict bucket-level security: **Block Public Access** activated globally, S3 Bucket Policies restricting cross-account access, and default Server-Side Encryption with KMS keys (**SSE-KMS**).
  - Configured **S3 Lifecycle Rules** for automated storage tiering (transitioning Bronze data to S3 Standard-IA after 30 days and archiving historical objects to S3 Glacier Flexible Retrieval after 90 days to minimize ongoing storage overhead).
- **Metadata Management with AWS Glue:**
  - Created an **AWS Glue Data Catalog** database to maintain unified schema definitions.
  - Configured and executed an **AWS Glue Crawler** to crawl raw and partitioned datasets stored in S3, automatically inferring data schemas, partition keys (`year=YYYY/month=MM`), and metadata definitions.
- **Serverless Querying with Amazon Athena:**
  - Configured an Athena Query Result location bucket with lifecycle expiration.
  - Executed interactive serverless SQL queries against the crawled datasets directly in the S3 Data Lake without spinning up database servers.
  - Evaluated query execution performance and cost impact: compared scanned data volumes between raw unpartitioned CSV versus partitioned Apache Parquet format, observing a ~85% reduction in scanned data and significantly faster query latency.

**2. Key Deliverables & Results:**
- Fully deployed and secured a 3-tier Data Lake foundation on Amazon S3.
- Automated schema discovery and metadata synchronization using AWS Glue Crawlers.
- Mastered cost-optimized serverless big data analytics using Amazon Athena and columnar Parquet formats.

**3. References:**
- FCAJ Lab Guide: Building a Serverless Data Lake with Amazon S3, AWS Glue & Amazon Athena.
- AWS Prescriptive Guidance: Data Lake Storage Best Practices on Amazon S3.
- Amazon Athena User Guide: Performance Tuning and Partitioning.
