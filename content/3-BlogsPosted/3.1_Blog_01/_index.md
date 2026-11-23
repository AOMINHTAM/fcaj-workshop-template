---
title: "Blog 1: Migrating a Data Pipeline from Snowflake to AWS S3 & Redshift Serverless"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

- **Original Post Title (Vietnamese):** *Chê Snowflake hết Free Trial, mình dọn nguyên cái Pipeline sang AWS S3 & Redshift và cái kết =)))*
- **Community Group:** AWS Study Group VN (First Cloud AI Journey)
- **Published Date:** November 11, 2026 (20:21)
- **Official Facebook Permalink:** <https://www.facebook.com/groups/awsstudygroupfcj/permalink/2334049077360051/>
- **Project Source Code (GitHub):** <https://github.com/AOMINHTAM/aws-data-pipeline>

---

### Introduction

When my Snowflake free trial expired, I decided to migrate and rebuild my existing `dbt_snowflake` data pipeline onto the **AWS** ecosystem—documenting hands-on architectural lessons and inviting technical feedback from senior engineers in the **AWS Study Group VN** community.

![AWS S3 and Redshift Serverless Data Pipeline Architecture](/fcaj-workshop-template/images/blogs/blog-01-architecture.png)

---

### 1. Core Migration Stack: Amazon S3 & Redshift Serverless

* **Region Selection:** Choosing **`us-east-1`** proved essential for full AWS service availability and seamless feature integration.
* **Amazon S3 (Data Lake — Landing Zone):** Served as the central raw landing zone where Olist E-Commerce source datasets (9 CSV files, ~100,000 orders) were uploaded via Bash and AWS CLI (`aws s3 sync`) at minimal storage cost prior to warehouse ingestion.
* **Amazon Redshift Serverless (Data Warehouse):** Replaced Snowflake effectively with **RPU-based (4 RPUs)** FinOps billing—charging only while queries execute and scaling down to `$0` compute cost during idle periods.
* **S3 + Redshift + dbt Core Integration:**
  * Provisioned a least-privilege **IAM Role (`RedshiftS3ReadOnlyRole`)** via STS `AssumeRole` so Redshift could ingest raw CSVs directly from S3 into `raw_data` using the high-throughput **`COPY` (MPP Bulk Load)** command.
  * Connected **dbt Core (`dbt-redshift`)** to the Redshift Workgroup Endpoint to push heavy SQL transformations down into Redshift—building `raw_data` → `public_staging` → `public_marts` (**Kimball Star Schema** with 9/9 automated data quality tests).
* **Real-World Troubleshooting Lesson:** During initial S3-to-Redshift loading, accidentally executing the `COPY` command **4 times** duplicated all dataset rows by 4x. Dropping and cleanly reloading the tables solved the baseline issue immediately—laying the groundwork for idempotent ingestion and atomic table swaps in later iterations.

---

### 2. Demystifying Ubuntu/Linux, Docker & Apache Airflow

* **Understanding Ubuntu/Linux:** Previously an unfamiliar topic, building this project inside **WSL2 Ubuntu** clarified how the Linux kernel operates under the hood and powers containerized workflows with **Docker**.
* **Apache Airflow (Astro CLI & Astronomer Cosmos):** Orchestrated dbt transformation models cleanly as native Airflow tasks via Astronomer Cosmos, bringing clear dependency visibility across warehouse layers.

---

### 3. Personal Reflections & Next Steps

* **Key Observation:** AWS abstracts away heavy infrastructure management, meaning the most time-consuming engineering work was configuring networking, **IAM permissions**, and cross-service trust relationships accurately.
* **Initial Limitation (at the V2 post milestone):** Source CSVs were still static, meaning ingestion was not yet end-to-end event-driven with Airflow.
* **Planned Follow-ups:** Evolving the pipeline toward automated event-driven triggers and adding downstream **BI & Analytics** visualization layers in future posts.

---

### 4. Conclusion & Links

Whether building on AWS, GCP, or Azure, the underlying platform matters less than **sound architectural thinking and data engineering fundamentals**. Special thanks to mentors and peers in **First Cloud AI Journey (FCAJ)** and **Xóm Data** for the knowledge and support throughout this build.

* **Official Facebook Post (AWS Study Group VN):** <https://www.facebook.com/groups/awsstudygroupfcj/permalink/2334049077360051/>
* **GitHub Repository:** <https://github.com/AOMINHTAM/aws-data-pipeline>
