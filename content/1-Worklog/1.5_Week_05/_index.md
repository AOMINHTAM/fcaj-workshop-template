---
title: "Week 5 Worklog"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:
* Explore the real-world **Olist Brazilian E-Commerce Dataset** (~120 MB, nearly 100,000 orders across 9 relational tables) and set up a standardized development environment on **Ubuntu (WSL2)**.
* Build an automated ingestion script staging raw data into an **Amazon S3 Landing Zone** (`s3://.../raw/`).
* Provision **Amazon Redshift Serverless** tuned for **FinOps cost efficiency (4 RPUs)**, delegate permissions via IAM Role and AWS STS (`RedshiftS3ReadOnlyRole`), and execute high-speed MPP batch loading using the **`COPY` command**.

### Tasks Completed This Week:
* **Dataset Exploration & Dev Environment Hardening:** Analyzed the schema relationships of the 9 Olist CSV tables; configured an Ubuntu WSL2 environment, secured AWS CLI credentials (`chmod 600`), and isolated environment variables in `.env`.
* **Automated S3 Raw Ingestion:** Created a private S3 bucket with 100% Block Public Access; authored the Bash script `scripts/upload_to_s3.sh` to validate and synchronize all 9 CSV files into `s3://.../raw/`.
* **Amazon Redshift Serverless Provisioning & FinOps:** Deployed `ecommerce-workgroup` with **4 RPUs base compute** (a 50% cost savings compared to default 8 RPUs, auto-pausing to $0 when idle) along with a **$5/month AWS Budget alert**; created the IAM Role `RedshiftS3ReadOnlyRole` using `sts:AssumeRole` following Least Privilege.
* **Network Troubleshooting & DDL Setup:** Configured VPC Security Groups opening port `5439`; diagnosed and resolved a routing conflict where Cloudflare WARP blocked Redshift endpoint access; authored DDL script `scripts/create_raw_tables.sql` initializing the `raw_data` schema for 9 source tables.
* **MPP Parallel Ingestion & Idempotency Controls:** Executed parallel `COPY` commands from S3 into Redshift (`IAM_ROLE`, `FORMAT AS CSV`, `IGNOREHEADER 1`, `TIMEFORMAT 'auto'`); verified record counts (**99,441 orders**, **112,650 order items**), and implemented idempotency patterns to prevent data duplication upon re-execution.

### Week 5 Achievements:
* Successfully deployed an **Amazon Redshift Serverless Cloud Data Warehouse**, secured via IAM Role STS delegation and cost-optimized at **4 RPUs**.
* Fully automated the **Extract & Load (EL)** phase from local development to Amazon S3 Landing Zone and parallel-loaded into Redshift's `raw_data` schema.
* Gained hands-on troubleshooting experience with network access (VPC Security Groups, port `5439`, Cloudflare WARP routing) and ensured batch idempotency.

### References:
* <https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce>
* <https://docs.aws.amazon.com/cli/latest/reference/s3/>
* <https://docs.aws.amazon.com/redshift/latest/mgmt/serverless-whatis.html>
* <https://docs.aws.amazon.com/redshift/latest/dg/r_pg_keywords.html>
* <https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html>
