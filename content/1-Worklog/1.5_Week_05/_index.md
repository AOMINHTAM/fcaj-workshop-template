---
title: "Week 5 Worklog"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:
* Profile the real-world **Olist Brazilian E-Commerce Dataset** (~120 MB, ~100,000 orders across 9 relational tables) and standardize the local **Ubuntu (WSL2)** Linux development environment.
* Automate raw dataset synchronization into the **Amazon S3 Landing Zone** (`s3://.../raw/`).
* Provision **Amazon Redshift Serverless** optimized for **FinOps (4 RPUs)**, configure IAM Role delegation via AWS STS (`RedshiftS3ReadOnlyRole`), and execute high-throughput MPP batch loading via the **`COPY`** command.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| :---: | :--- | :---: | :---: | :--- |
| **Mon (Day 2)** | - Analyze entity relationships across the 9 **Olist E-Commerce** CSV datasets (`orders`, `order_items`, `order_payments`, `order_reviews`, `customers`, `products`, `sellers`, `geolocation`, `product_category_name_translation`) <br> - Set up **Ubuntu (WSL2)**, secure AWS CLI v2 credentials (`chmod 600`), and isolate secrets in git-ignored `.env` files | 26/10/2026 | 26/10/2026 | <https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce> |
| **Tue (Day 3)** | - Provision a private Amazon S3 bucket (`olist-raw-data-amt-us-east-1-...`) with **100% Block Public Access** <br> - Write `scripts/upload_to_s3.sh` to validate local files and synchronize all 9 CSVs into `s3://.../raw/` | 27/10/2026 | 27/10/2026 | <https://docs.aws.amazon.com/cli/latest/reference/s3/> |
| **Wed (Day 4)** | - Deploy **Amazon Redshift Serverless** (`ecommerce-workgroup`) with a minimum base capacity of **4 RPUs** (cutting compute costs by 50% vs. 8 RPUs and scaling to \$0 when idle) + configure **AWS Budget Alert (\$5/month)** <br> - Create `RedshiftS3ReadOnlyRole` IAM Role (AWS STS `AssumeRole`, Least Privilege, zero hardcoded keys) | 28/10/2026 | 28/10/2026 | <https://docs.aws.amazon.com/redshift/latest/mgmt/serverless-whatis.html> |
| **Thu (Day 5)** | - Configure **VPC Security Group** inbound rules on port `5439` and troubleshoot local routing conflicts caused by Cloudflare WARP blocking Redshift connections <br> - Author DDL script `scripts/create_raw_tables.sql` creating the `raw_data` schema (resolving Redshift's reserved keyword `raw`) across 9 tables | 29/10/2026 | 29/10/2026 | <https://docs.aws.amazon.com/redshift/latest/dg/r_pg_keywords.html> |
| **Fri (Day 6)** | - Execute `scripts/copy_raw_tables.sql` for parallel MPP ingestion from S3 into Redshift using the **`COPY`** command (`IAM_ROLE`, `FORMAT AS CSV`, `IGNOREHEADER 1`, `TIMEFORMAT 'auto'`) <br> - Audit row counts (**99,441 orders**, **112,650 order items**) and resolve duplicate-load issues to guarantee ingestion idempotency | 30/10/2026 | 30/10/2026 | <https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html> |

### Week 5 Achievements:
* Provisioned a cost-optimized (**4 RPUs**), IAM-secured Cloud Data Warehouse on **Amazon Redshift Serverless**.
* Automated the end-to-end **EL (Extract-Load)** ingestion pipeline from local storage through Amazon S3 into Redshift's `raw_data` schema.
* Gained hands-on troubleshooting experience resolving network perimeter issues (VPC Security Group port `5439`, Cloudflare WARP tunnel conflicts) and enforcing idempotent batch loading.
