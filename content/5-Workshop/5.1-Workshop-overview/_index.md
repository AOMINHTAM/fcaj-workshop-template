---
title: "5.1 Workshop Overview & V5 Architecture"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### Workshop Objectives
By completing this workshop, you will deploy and verify a production-style AWS Data Pipeline capable of:
* Automatically ingesting e-commerce CSV drops into **Amazon S3** via pure event-driven triggers (eliminating idle cron polling).
* Protecting **Amazon Redshift Serverless** against `0`-row or schema-drifted CSVs, isolating poison events into an **SQS DLQ**, and dispatching real-time email alerts via **CloudWatch + SNS**.
* Transforming data into an incremental **Kimball Star Schema** (**dbt Core `delete+insert`**) backed by automated quality tests and the `ops.pipeline_ingestion_audit` ledger.

---

### V5 System Architecture (Custom Draw.io Diagram with AWS Icons)

![V5 Architecture — Production CloudOps & Resilience](/fcaj-workshop-template/images/workshop/architecture_v5.png)

#### End-to-End Data Flow:
1. **Landing & Event Push:** Raw Olist CSVs sync into `s3://<bucket>/raw/*.csv` → S3 emits `s3:ObjectCreated:*` events into **Amazon SQS Main Queue** (`VisibilityTimeout=900s`).
2. **Zero-Cron Trigger:** **Airflow 3 `AssetWatcher`** (`SqsSensorTrigger`, 20s Long Polling) wakes the `ecommerce_aws_data_pipeline` DAG only when messages arrive (idle = `0` DAG runs, `$0` Redshift compute).
3. **Two-Phase Atomic Ingestion:** `load_raw_from_sqs` validates CSV headers (`validate_csv_header`), runs MPP `COPY` from S3 (via STS `RedshiftS3ReadOnlyRole`) into a `TEMP TABLE`, enforces `staged_rows > 0` (`ensure_non_empty_stage`), and commits `DELETE + INSERT` plus `status = 'SWAPPED'` in `ops.pipeline_ingestion_audit` inside a **single `autocommit=False` transaction**.
4. **Incremental Star Schema Transformation:** **Astronomer Cosmos** executes **dbt Core** across 7 Staging Views (`public_staging`) and 3 Marts Tables (`fct_orders`, `dim_customers`, `dim_products`) alongside automated schema tests (`unique`, `not_null`, `relationships`, `accepted_values`).
5. **Deterministic Archive & Safe ACK:** Once dbt passes 100%, `archive_and_ack` copies objects to `archive/<safe_audit_id>/<filename>`, verifies destination existence, records `ARCHIVED → COMPLETED`, removes `raw/` objects, and issues SQS `DeleteMessage`.
6. **Fault Isolation & Active Alerting:** After `maxReceiveCount=3` failures, SQS routes poison messages to **SQS DLQ** (14-day retention) → **CloudWatch Alarms** enter `ALARM` → **Amazon SNS** delivers an instant email alert; operators remediate and trigger **DLQ Redrive**.
7. **Decoupled BI Export:** `scripts/export_to_csv.py` extracts 4 curated CSV datasets (~56 MB) powering offline Power BI dashboards at `$0` idle RPU cost.

---

### AWS Well-Architected Framework Alignment

| Well-Architected Pillar | Concrete V5 Implementation |
| :--- | :--- |
| **Operational Excellence** | 100% declarative **CloudFormation IaC** (`cfn-lint` CI, Change Sets, Native Auto-Rollback), bilingual **Operations Runbook**, and `ops.pipeline_ingestion_audit` state tracking. |
| **Security** | 3-way **Least-Privilege IAM** (`RedshiftS3ReadOnlyRole` `raw/*`-only, `AirflowRuntimePolicy` without `archive/*` delete, `DlqOperatorPolicy`); S3 100% Block Public Access, `AES256`, HTTPS-only `S3RawBucketPolicy`; Redshift `/32` Security Group & Secrets Manager. |
| **Reliability** | **Two-Phase TEMP TABLE Swap** (`autocommit=False`), **CSV Header + Empty-Stage Guard**, **SQS DLQ (`maxReceiveCount=3`) + DLQ Redrive**, **4 CloudWatch Alarms + SNS**, S3 Versioning, and 30-minute **Redshift Serverless Recovery Points**. |
| **Performance Efficiency** | Parallel **MPP `COPY`** from S3 into Redshift Serverless, **dbt Incremental (`delete+insert`)** with 4-source `affected_order_ids` CTE avoiding full rebuilds, and 20s SQS Long Polling. |
| **Cost Optimization** | **Redshift Serverless 4 RPUs** auto-sleeping at `$0` idle compute, `DeployRedshift=false` lab toggle, paused weekly full-refresh DAG, S3 Lifecycle (30/90 days), `$5/mo` AWS Budget, and 1-command `make cfn-delete` teardown to `$0`. |
