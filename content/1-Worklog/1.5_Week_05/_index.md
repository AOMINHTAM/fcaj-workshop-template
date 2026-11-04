---
title: "Week 5: Cloud Data Warehouse Design with Amazon Redshift Serverless & S3 Data Ingestion"
weight: 5
---

### 🇬🇧 Week 05 Worklog

**1. Tasks Completed:**
- **Dataset Profiling & Development Environment Standardization:**
  - Analyzed the schema and entity relationships of the real-world **Olist Brazilian E-Commerce Dataset** (~120 MB, ~100,000 orders) spanning 9 relational tables: `orders`, `order_items`, `order_payments`, `order_reviews`, `customers`, `products`, `sellers`, `geolocation`, and `product_category_name_translation`.
  - Configured a standardized Linux development environment on **Ubuntu (WSL2)**, secured AWS CLI v2 credential files (`chmod 600`), and isolated sensitive connection parameters inside git-ignored `.env` files.
- **Amazon S3 Landing Zone & Automated Data Synchronization:**
  - Provisioned a private Amazon S3 bucket (`olist-raw-data-amt-us-east-1-...`) in `us-east-1` with **100% Block Public Access** enforced.
  - Developed an automated Bash ingestion script (`scripts/upload_to_s3.sh`) to validate local datasets and synchronize all 9 raw CSV files into the `s3://.../raw/` landing zone prefix.
- **Amazon Redshift Serverless Provisioning & FinOps Cost Optimization:**
  - Deployed **Amazon Redshift Serverless** (`ecommerce-workgroup`) configured with a minimum base capacity of **4 RPUs (Redshift Processing Units)** — cutting compute costs by 50% compared to standard 8-RPU deployments while automatically scaling down to \$0 during idle periods.
  - Configured **AWS Budgets** threshold alerts (\$5/month) to maintain strict financial governance over warehouse compute usage.
  - Implemented **Principle of Least Privilege** security: created the `RedshiftS3ReadOnlyRole` IAM Role leveraging AWS STS (`AssumeRole`), enabling Redshift to read objects from S3 with zero hardcoded access keys in SQL scripts.
  - Configured **VPC Security Group** inbound rules on port `5439` and diagnosed/resolved local network routing conflicts caused by Cloudflare WARP blocking Redshift JDBC/ODBC connections.
- **`raw_data` Schema Design & MPP Batch Ingestion:**
  - Authored DDL scripts (`scripts/create_raw_tables.sql`) to initialize the `raw_data` schema (working around Redshift's reserved keyword `raw`) and defined explicit data types across all 9 source tables.
  - Executed high-throughput parallel batch loading (`scripts/copy_raw_tables.sql`) using the Redshift **`COPY`** command authenticated via `IAM_ROLE` with `FORMAT AS CSV`, `IGNOREHEADER 1`, and `TIMEFORMAT 'auto'`.
  - Audited ingested row counts (**99,441 orders**, **112,650 order items**) and resolved duplicate-record issues caused by repeated `COPY` executions to guarantee ingestion idempotency.

**2. Key Deliverables & Results:**
- Successfully provisioned a cost-optimized (4 RPUs), IAM-secured Cloud Data Warehouse on Amazon Redshift Serverless.
- Built an end-to-end automated EL (Extract-Load) ingestion workflow from local CSV sources through Amazon S3 into Redshift's `raw_data` schema.
- Gained practical troubleshooting experience resolving Redshift network perimeter issues (Security Groups, Port 5439, WARP tunnel conflicts) and enforcing idempotent data loading.

**3. References:**
- Amazon Redshift Management Guide: Getting Started with Amazon Redshift Serverless & RPU Capacity.
- Amazon Redshift Database Developer Guide: Loading Data from Amazon S3 using the `COPY` Command.
- AWS IAM Documentation: Authorizing Amazon Redshift to Access Other AWS Services on Your Behalf.
