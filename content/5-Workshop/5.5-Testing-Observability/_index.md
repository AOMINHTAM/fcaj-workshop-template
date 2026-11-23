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

> **Pre-Submission Action (Mandatory for Presentation Score):** Generate an official estimate on [AWS Pricing Calculator](https://calculator.aws/) matching the configuration below, paste the shareable URL here, and save a screenshot to `static/images/workshop/aws_pricing_calculator.png`.

| AWS Service | Sized Configuration | Est. Monthly Cost | FinOps Optimization Applied |
| :--- | :--- | :---: | :--- |
| **AWS CloudFormation & IAM** | 1 Stack (15–19 resources) | `$0.00` | Free of charge; deterministic teardown prevents orphaned resources. |
| **Amazon S3 (Standard + Versioning)** | ~120 MB data + archive | `~$0.003` | Lifecycle expires `archive/` in 90d, noncurrent versions in 30d, incomplete multiparts in 7d. |
| **Amazon SQS (Main + DLQ)** | ≤ 104k requests/month | `$0.00` | Well within 1M Always Free requests; 20s Long Polling; `0` reqs after `astro dev stop`. |
| **Amazon CloudWatch & SNS** | 4 Standard Alarms + Email | `$0.00` | Within Always Free tier (10 Standard Alarms + 1,000 emails/month). |
| **Amazon Redshift Serverless** | 4 RPUs (~1–2 RPU-hours/mo) | `~$0.75 – $1.50` | Auto-sleeps at `$0` idle; `DeployRedshift=false` toggle; paused full-refresh DAG; offline CSV `exports/`. |

* **Official AWS Pricing Calculator Estimate Link:** *(Paste your `https://calculator.aws/` share link here)*
* **AWS Pricing Calculator Screenshot:** *(Pending `static/images/workshop/aws_pricing_calculator.png`)*

---

### 5. Live Production Evidence & Demo Video

> **Pre-Submission Action (Mandatory for Demo Score):** Record a 5–8 minute end-to-end walkthrough matching `architecture_v5.png` (CloudFormation Outputs → S3 CSV upload → Airflow 3 `AssetWatcher` → Redshift `ops.pipeline_ingestion_audit` & dbt → Empty CSV DLQ/Alarm/Redrive → Clean-up) and embed the public YouTube/Drive link below.

* **End-to-End Demo Video (5–8 mins):** *(Paste public YouTube / Google Drive demo link here)*
* **Online Portfolio Case Study:** <https://aominhtam-portfolio.vercel.app/projects/aws-data-pipeline.html>
* **GitHub Repository (Releases `5.0.0` / `5.0.1`):** <https://github.com/AOMINHTAM/aws-data-pipeline>
