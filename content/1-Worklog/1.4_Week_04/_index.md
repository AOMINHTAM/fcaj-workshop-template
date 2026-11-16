---
title: "Week 4 Worklog"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:
* Design a multi-tier **Medallion Data Lake Architecture** on **Amazon S3** secured with **SSE-KMS** and optimized via **S3 Lifecycle Rules**.
* Automate metadata cataloging and schema discovery using **AWS Glue Data Catalog** and **AWS Glue Crawlers**.
* Execute serverless analytics queries with **Amazon Athena** and benchmark columnar performance using partitioned **Apache Parquet**.

### Tasks Completed This Week:
* **Medallion Data Lake on Amazon S3:** Created a structured S3 bucket partitioned into `bronze-raw/`, `silver-cleaned/`, and `gold-curated/` layers; enforced **100% Block Public Access** and default encryption with **SSE-KMS**.
* **S3 Lifecycle Management:** Configured lifecycle transitions moving Bronze objects to `S3 Standard-IA` after 30 days and archiving to `S3 Glacier Flexible Retrieval` after 90 days; staged sample CSV/JSON and partitioned Apache Parquet files (`year=YYYY/month=MM`).
* **Automated Schema Discovery with AWS Glue:** Initialized an **AWS Glue Data Catalog** database, provisioned service IAM roles, and executed **AWS Glue Crawlers** to automatically infer schemas and discover dataset partition keys.
* **Serverless Analytics with Amazon Athena:** Set up Athena with a designated S3 query result bucket; authored and executed analytical SQL aggregations directly against S3 tables registered in the Glue Catalog.
* **Columnar Parquet Performance Benchmarking:** Quantified query execution gains comparing raw CSV against partitioned **Apache Parquet**, confirming an **~85% reduction in Data Scanned** and substantially lower query latency; cleaned up temporary test resources.

### Week 4 Achievements:
* Fully established a secure, cost-optimized 3-tier Data Lake (`Bronze`, `Silver`, `Gold`) on Amazon S3.
* Automated metadata governance and schema evolution using AWS Glue Crawlers and Glue Data Catalog.
* Quantified the performance gains of columnar **Apache Parquet** format and partitioning on Amazon Athena: ~85% scan reduction and near-instant query turnaround.

### References:
* <https://cloudjourney.awsstudygroup.com/>
* <https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html>
* <https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html>
* <https://docs.aws.amazon.com/athena/latest/ug/what-is.html>
* <https://docs.aws.amazon.com/athena/latest/ug/performance-tuning.html>
