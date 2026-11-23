---
title: "5.5 E2E Verification, Fault Tolerance, Cost Estimation & Demo"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

### 1. Live Verification Scenario 1 — Happy Path E2E (`17/17` Tasks PASS)
Replay `product_category_name_translation.csv` from `archive/` into `s3://<bucket>/raw/`:
```bash
aws s3 cp \
  s3://$S3_BUCKET_NAME/archive/product_category_name_translation.csv \
  s3://$S3_BUCKET_NAME/raw/product_category_name_translation.csv
```
* **Observed Live Result (Nov 18, 2026):** `AssetWatcher` triggered automatically with zero empty runs, completing **17/17 tasks PASS** (`00:03:52`), passing **11 dbt schema tests** on Redshift Serverless, recording **`SWAPPED → ARCHIVED → COMPLETED`** (`71` rows) in `ops.pipeline_ingestion_audit`, archiving deterministically, and ACKing SQS cleanly to `0`.

![Airflow 3 Event-Driven Run 17/17 Tasks PASS](/fcaj-workshop-template/images/workshop/airflow_v5_asset_watcher_run.png)

![Redshift ops.pipeline_ingestion_audit Ledger](/fcaj-workshop-template/images/workshop/redshift_v5_ops_audit_table.png)

---

### 2. Live Verification Scenario 2 — Fault Tolerance (0-Row CSV → DLQ → Alarm → SNS → Redrive)
1. **Simulate Poison Message (Header-Only 0-Row CSV):** Upload a 0-row CSV to `s3://<bucket>/raw/product_category_name_translation.csv`.
   * `ensure_non_empty_stage` raises `ValueError`, **rolls back cleanly to preserve all `71` existing rows** in `raw_data.product_category_name_translation`, and logs `status='FAILED', rows_loaded=0` in `ops.pipeline_ingestion_audit`.

![Empty-Stage Guard Blocking 0-Row CSV and Preserving Target Table](/fcaj-workshop-template/images/workshop/airflow_v5_empty_stage_guard_log.png)

2. **SQS DLQ Isolation & 4 CloudWatch Alarms + SNS Email:**
   * After `maxReceiveCount=3`, Amazon SQS moves the poison message into **`olist-v5-s3-raw-arrival-dlq-dev`** (`1` message available).
   * Within 1 minute, **CloudWatch Alarms** (`dlq-messages-visible`, `backlog-age`, `inflight-sustained`) flip to **`In alarm`**, and **Amazon SNS** delivers a real alert email to Gmail.

![Amazon SQS Dead-Letter Queue Isolating Poison Message](/fcaj-workshop-template/images/workshop/sqs_v5_dlq_messages.png)

![CloudWatch 4 Alarms Entering In Alarm State](/fcaj-workshop-template/images/workshop/cloudwatch_v5_alarms_in_alarm.png)

![Real-Time Amazon SNS Alert Email Delivered to Gmail](/fcaj-workshop-template/images/workshop/gmail_v5_sns_dlq_alarm.png)

3. **Recovery via SQS DLQ Redrive (`StartMessageMoveTask`):**
   * Restore the valid `71`-row CSV and redrive the isolated message back to `SqsMainQueue`:
   ```bash
   aws sqs start-message-move-task \
     --source-arn "$DLQ_ARN" \
     --destination-arn "$MAIN_QUEUE_ARN" \
     --region us-east-1
   ```
   * **Observed Live Result:** The recovery DAG run passed **17/17 tasks**, logged `SWAPPED → ARCHIVED → COMPLETED`, drained SQS Main & DLQ to `0/0`, and **all 4/4 CloudWatch Alarms automatically recovered to `OK`**.

---

### 3. Automated CI/CD & Cross-Review Hardening (`5.0.1`)
Every push/PR to `main` executes `.github/workflows/ci.yml` at `$0` AWS cost:
* **Ruff Lint** (`0` warnings) + **`cfn-lint`** validating `infra/cloudformation/template.yaml`.
* **21/21 Python Unit Tests PASS** (`tests/test_ingestion_guards.py` 11 tests + `tests/test_sqs_parser.py` 10 tests).
* **Offline `dbt parse`** structural SQL/YAML verification + **Astro Docker Build & 3 DAG Pytests**.

---

### 4. Cost Estimation & FinOps Optimization (AWS Pricing Calculator)

Official cost estimate built on **[AWS Pricing Calculator](https://calculator.aws/#/estimate?id=77a934d9011f072f5b013defff9c61ac61f895ef)** (`us-east-1`, Upfront `$0.00`, List Total **`$3.51 USD/month`** / **`$42.12 USD/12 months`**):

| AWS Service | Sized Configuration (Pricing Calculator) | List Monthly Cost | FinOps Optimization Applied |
| :--- | :--- | :---: | :--- |
| **Amazon S3 (S3 Standard)** | 1 GB + 1,000 PUT/COPY/LIST + 5,000 GET | `$0.03` | Lifecycle expires `archive/` in 90d, noncurrent versions in 30d, incomplete multiparts in 7d. |
| **Amazon SQS (Main + DLQ)** | Standard Queue 0.15M requests/month | `$0.06` | Effectively `$0.00` under 1M Always Free requests; 20s Long Polling; `0` reqs after `astro dev stop`. |
| **Amazon CloudWatch** | 4 Standard-Resolution Alarm Metrics | `$0.40` | Effectively `$0.00` under 10 free Standard Alarms/month. |
| **Amazon SNS** | 100 Email/Email-JSON Notifications | `$0.00` | Free within 1,000 email notifications/month. |
| **Amazon Redshift Serverless** | 4 Base RPUs (`0.0656h/day` ≈ 2h/month) + 1 GB Storage | `$3.02` | Auto-sleeps at `$0` idle; `DeployRedshift=false` toggle; paused full-refresh DAG; offline CSV `exports/`. |
| **Total (Pricing Calculator)** | **AWS Budgets Alert `$5.00/month`** | **`$3.51 / month`** | **Effective net spend after Always Free `~$3.02/mo` (covered by `$200` Credit)** |

* **Official AWS Pricing Calculator Estimate Link:** <https://calculator.aws/#/estimate?id=77a934d9011f072f5b013defff9c61ac61f895ef>
* **Exported Calculator Reports:** [Official PDF](/fcaj-workshop-template/files/aws_pricing_calculator.pdf) | [Detailed CSV](/fcaj-workshop-template/files/aws_pricing_calculator.csv)

![AWS Pricing Calculator Cost Estimate Screenshot](/fcaj-workshop-template/images/workshop/aws_pricing_calculator.png)

---

### 5. Live Production Evidence & Demo Video

> **Pre-Submission Action (Mandatory for Demo Score):** Record a 5–8 minute end-to-end walkthrough matching `architecture_v5.png` (CloudFormation Outputs → S3 CSV upload → Airflow 3 `AssetWatcher` → Redshift `ops.pipeline_ingestion_audit` & dbt → Empty CSV DLQ/Alarm/Redrive → Clean-up) and embed the public YouTube/Drive link below.

* **End-to-End Demo Video (5–8 mins):** *(Paste public YouTube / Google Drive demo link here)*
* **Online Portfolio Case Study:** <https://aominhtam-portfolio.vercel.app/projects/aws-data-pipeline.html>
* **GitHub Repository (Releases `5.0.0` / `5.0.1`):** <https://github.com/AOMINHTAM/aws-data-pipeline>
