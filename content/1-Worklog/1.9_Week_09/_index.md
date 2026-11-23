---
title: "Week 9 Worklog"
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objectives:
* Upgrade the AWS Data Pipeline from **V4 (`v4.0.1`)** to **V5 (`5.0.0` / `5.0.1` — Production CloudOps & Resilience)** aligned with **AWS First Cloud AI Journey (FCAJ)** standards using 100% declarative **AWS CloudFormation IaC**.
* Implement production-grade **Fault Tolerance & Active Observability**: isolate poison messages via **Amazon SQS Dead-Letter Queue (DLQ)**, configure **4 Amazon CloudWatch Alarms + Amazon SNS Email Alerting**, and build the **`ops.pipeline_ingestion_audit`** ledger with **Deterministic Safe ACK**.
* Execute live end-to-end verification on **AWS & Airflow 3** across both Happy Path and Fault-Tolerance scenarios (Empty CSV → DLQ → Alarm → SNS → DLQ Redrive), harden edge cases (`5.0.1`), and finalize bilingual technical documentation (**V5 Deep Dive, Operations Runbook, Business Case, Retrospective**).

### Tasks Completed This Week:
* **100% Declarative AWS CloudFormation IaC (`infra/cloudformation/template.yaml`):**
  * Authored **15 core resources** + **4 conditional FinOps/Redshift resources** (`DeployRedshift=false` by default, `PubliclyAccessible=false`, `/32` CIDR restriction, automatic password management via **AWS Secrets Manager**) + **16 Outputs**, enabling **Native Stack Auto-Rollback** and automated **`cfn-lint`** validation in GitHub Actions CI.
  * Resolved dual **Circular Dependencies** (`S3RawBucket ↔ SqsMainQueuePolicy` and `SqsMainQueue ↔ SqsDeadLetterQueue`) via `!Sub`-constructed ARNs combined with `DependsOn: SqsMainQueuePolicy`.
  * Applied 5-layer S3 Data Lake security: `AES256` encryption, all 4 Public Access Block flags, `S3RawBucketPolicy` denying non-HTTPS requests (`aws:SecureTransport: "false"`), **S3 Versioning** + Lifecycle rules (`archive/` 90-day expiry, noncurrent versions 30-day expiry, orphan `DeleteMarker` & 7-day incomplete multipart cleanup), plus paginated `scripts/empty_versioned_bucket.py` for 1-command `$0` teardown (`make cfn-delete`).
  * Enforced split **Least-Privilege IAM** with region suffixes (`*-${AWS::Region}`): `RedshiftS3ReadOnlyRole` (`raw/*` read-only with `aws:SourceAccount` condition), `AirflowRuntimePolicy` (no `DELETE` permission on `archive/*`), and `DlqOperatorPolicy` (isolated `StartMessageMoveTask` redrive permissions).
* **Fault Tolerance & Active Observability (SQS DLQ, CloudWatch Alarms & SNS):**
  * Provisioned **Amazon SQS Dead-Letter Queue** (`olist-v5-s3-raw-arrival-dlq-dev`, `maxReceiveCount=3`, 14-day retention, `RedriveAllowPolicy: byQueue`) to isolate poison/malformed CSV events and eliminate infinite retry loops.
  * Configured **4 CloudWatch Alarms** (`TreatMissingData: notBreaching`): `DlqMessagesVisibleAlarm` (≥ 1 within 1m), `MainQueueOldestMessageAlarm` (> 600s SLA), `MainQueueInFlightAlarm` (≥ 1 sustained for 15m), and `MainQueueBacklogAlarm` (≥ 20 over 3m), routed through **`PipelineAlertTopicPolicy`** to **Amazon SNS Email**.
* **Application Reliability Hardening, Airflow 3 Bug Fix & Audit Ledger (`5.0.0` → `5.0.1`):**
  * Diagnosed and fixed a live **Airflow 3.3 Task SDK** issue in `include/sqs_parser.py`: `context["triggering_asset_events"]` is a `TriggeringAssetEventsAccessor` inheriting `collections.abc.Mapping` (not `dict`), updated to check `isinstance(..., Mapping)`.
  * Implemented the **`ops.pipeline_ingestion_audit`** ledger in Redshift Serverless tracking `SWAPPED → ARCHIVED → COMPLETED / FAILED`; added **Two-Phase Batch Staging**, **CSV Header Schema Guard** (`validate_csv_header`), **Empty-Stage Guard** (`ensure_non_empty_stage`), same-transaction `DELETE + INSERT` + `SWAPPED` commit, cross-run deterministic archive paths (`archive/<safe_audit_id>/<filename>`), fail-closed `VisibilityTimeout=900s` heartbeat, deferred multi-record SQS ACK, and `verify_safe_to_ack_when_raw_missing`.
* **Live AWS Verification, Releases `5.0.0` / `5.0.1` & Documentation:**
  * Deployed `olist-v5-pipeline-stack` live in `us-east-1` (`CREATE_COMPLETE`), confirmed SNS email subscription, and verified both live E2E scenarios:
    * *Happy Path E2E:* Replayed `product_category_name_translation.csv` into S3 `raw/` → Airflow 3 `AssetWatcher` automatically ran **17/17 tasks PASS** (`00:03:52`), 11 dbt schema tests passed on Redshift, and the audit ledger recorded `SWAPPED → ARCHIVED → COMPLETED` (`71` rows).
    * *Fault Tolerance & DLQ Redrive:* Uploaded a 0-row CSV → `ensure_non_empty_stage` blocked the swap and rolled back (preserving all `71` rows), logged `FAILED` in audit, and routed the poison message to DLQ after 3 attempts → CloudWatch Alarms transitioned to `In alarm` and SNS delivered a real Gmail alert → restored valid CSV + executed `aws sqs start-message-move-task` → recovery run achieved **17/17 tasks PASS**, SQS Main/DLQ returned to `0/0`, and **all 4/4 Alarms returned to `OK`**.
  * Expanded automated testing to **21/21 Python unit tests** + **3 DAG pytests** + Ruff + `cfn-lint` + offline `dbt parse` passing 100% in GitHub Actions CI; merged PR #4 (`692050c`), cut releases **`5.0.0`** and **`5.0.1`**; delivered Draw.io diagram `architecture_v5.drawio` and bilingual documentation (**V5 Deep Dive, Operations Runbook with 14-scenario Test Matrix, Business Case, and Retrospective**).

### Week 9 Achievements:
* Achieved **100% reproducible Infrastructure as Code (AWS CloudFormation)** with split least-privilege IAM, native stack rollback, 1-command versioned bucket clean-up, and Always Free-aligned operating cost (~$0.003/month S3 storage).
* Verified the complete production fault-tolerance and observability lifecycle live on AWS: **Empty CSV → Empty-Stage Guard Rollback → SQS DLQ → CloudWatch Alarms → SNS Email Alert → DLQ Redrive Recovery**.
* Resolved a real **Airflow 3.3 Task SDK** compatibility bug (`TriggeringAssetEventsAccessor`), hardened 13 cross-review edge cases (`21/21` unit tests PASS), and published **Releases `5.0.0` / `5.0.1`** with complete bilingual CloudOps documentation.

### References:
* <https://github.com/AOMINHTAM/aws-data-pipeline>
* <https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html>
* <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html>
* <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html>
* <https://docs.aws.amazon.com/sns/latest/dg/welcome.html>
