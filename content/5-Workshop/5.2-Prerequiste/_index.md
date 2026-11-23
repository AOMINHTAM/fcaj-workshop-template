---
title: "5.2 Prerequisites & Environment Setup"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

### 1. AWS Account & Permissions
* An **AWS account** (recommended Region: **`us-east-1`** for consistent service availability) protected with an **AWS Budgets** alert (`$5/month` or learning budget cap).
* A dedicated CLI IAM User (never use the Root user) with permissions to manage CloudFormation, S3, SQS, CloudWatch, SNS, IAM, and Redshift Serverless.
* A valid email address (`ALERT_EMAIL`) to confirm the **Amazon SNS** alert subscription.

---

### 2. Local Workstation Tools (Windows WSL2 Ubuntu / Linux / macOS)
1. **Ubuntu on WSL2** (on Windows) with `git`, `make`, and `python3` (≥ 3.10).
2. **Docker Desktop** running with the WSL2 backend enabled.
3. **AWS CLI v2** configured via `aws configure`:
   ```bash
   aws --version
   aws sts get-caller-identity
   ```
4. **Astro CLI** to run **Apache Airflow 3 (Astronomer Runtime 3.3)** locally in Docker:
   ```bash
   astro version
   ```

---

### 3. Clone the Repository & Download the Olist Dataset
1. Clone the repository and validate local AWS CLI connectivity + offline tests/linting:
   ```bash
   git clone https://github.com/AOMINHTAM/aws-data-pipeline.git
   cd aws-data-pipeline

   # Verify AWS identity and run offline Ruff + cfn-lint + 21 unit tests ($0 AWS)
   make aws-verify
   make verify
   ```
2. Download the **Brazilian E-Commerce Public Dataset by Olist** from Kaggle (<https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce>) containing 9 relational CSV files (~120 MB):
   * `olist_orders_dataset.csv`
   * `olist_order_items_dataset.csv`
   * `olist_order_payments_dataset.csv`
   * `olist_order_reviews_dataset.csv`
   * `olist_customers_dataset.csv`
   * `olist_products_dataset.csv`
   * `olist_sellers_dataset.csv`
   * `olist_geolocation_dataset.csv`
   * `product_category_name_translation.csv`
3. Store the 9 CSV files in your local data folder for S3 upload in Chapter 5.4.

---

### 4. Prepare the `.env` Configuration File (Git-Ignored)
Never hard-code AWS keys or Redshift passwords in source files. After deploying the CloudFormation stack in **Chapter 5.3**, populate `.env` (git-ignored) from the stack Outputs:

```env
DBT_REDSHIFT_PASSWORD=<your-redshift-admin-password>
AWS_DEFAULT_REGION=us-east-1
AWS_ACCESS_KEY_ID=<your-aws-access-key>
AWS_SECRET_ACCESS_KEY=<your-aws-secret-key>
S3_BUCKET_NAME=<S3RawBucketName-from-stack-outputs>
SQS_QUEUE_NAME=<SqsMainQueueName-from-stack-outputs>
SQS_QUEUE_URL=<SqsMainQueueUrl-from-stack-outputs>
REDSHIFT_IAM_ROLE=<RedshiftS3ReadOnlyRoleArn-from-stack-outputs>
AIRFLOW_CONN_AWS_DEFAULT=aws://
```
