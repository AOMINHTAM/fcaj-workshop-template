---
title: "Blog 3: Upgrading to V5 Production CloudOps with CloudFormation IaC, SQS DLQ & CloudWatch/SNS"
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

- **Topic:** Hardening an AWS Data Pipeline to **Production CloudOps (V5)** using **100% AWS CloudFormation IaC**, **SQS Dead-Letter Queue**, **4 CloudWatch Alarms + SNS**, and the **`ops.pipeline_ingestion_audit`** Ledger
- **Target Community Group:** AWS Study Group VN (First Cloud AI Journey)
- **Status:** ⏳ Draft ready — pending Facebook publication & official permalink
- **Official Facebook Permalink:** *(Paste permalink after publishing to AWS Study Group VN)*
- **Project Source Code (GitHub):** <https://github.com/AOMINHTAM/aws-data-pipeline>

---

### Introduction

While V4 delivered a strong Data Engineering Core, five real-world production gaps remained: manual AWS infrastructure without rollback, infinite retry risk on poison CSVs, no active alerting, coarse IAM roles, and retry ACK edge cases. **V5 (`5.0.0` / `5.0.1`)** resolves all five following **AWS First Cloud AI Journey (FCAJ)** standards while staying inside Always Free allowances (`~$0.003/month` S3).

![V5 Architecture — Production CloudOps & Resilience](/fcaj-workshop-template/images/workshop/architecture_v5.png)

---

### 1. Breaking Dual Circular Dependencies in AWS CloudFormation IaC

Authoring `infra/cloudformation/template.yaml` (15 core + 4 conditional resources) exposed two classic CloudFormation dependency cycles:
* **Cycle 1 (`S3RawBucket ↔ SqsMainQueuePolicy`):** S3 requires the Queue Policy before creation so `s3:TestEvent` succeeds, yet calling `!GetAtt S3RawBucket.Arn` inside the policy creates a circular dependency. **Fix:** Construct the bucket ARN with `!Sub` inside `SqsMainQueuePolicy` and add `DependsOn: SqsMainQueuePolicy` on `S3RawBucket`.
* **Cycle 2 (`SqsMainQueue ↔ SqsDeadLetterQueue`):** Similarly, `SqsDeadLetterQueue` builds the source ARN via `!Sub` in `RedriveAllowPolicy: byQueue`, guaranteeing clean creation order: `DLQ → Main Queue → Queue Policy → S3 Bucket`.
* **Versioned Bucket Teardown Gotcha:** Plain `aws s3 rm --recursive` only writes *DeleteMarkers*, causing `delete-stack` to fail with `BucketNotEmpty`. Integrating paginated `scripts/empty_versioned_bucket.py` into `make cfn-delete` purges all `Versions` + `DeleteMarkers` for 1-command `$0` clean-up.

---

### 2. 3-Way Least-Privilege IAM & 4 CloudWatch Alarms + SNS

* **Separation of Duties (IAM):** Split permissions across `RedshiftS3ReadOnlyRole` (`raw/*` read-only), `AirflowRuntimePolicy` (no `DELETE` on `archive/*`), and `DlqOperatorPolicy` (dedicated `StartMessageMoveTask` DLQ Redrive access).
* **4 CloudWatch Alarms (`TreatMissingData: notBreaching`):** Monitor `DlqMessagesVisible >= 1` (1m), `OldestMessage > 600s` (10m SLA), `InFlight >= 1` (sustained 15m), and `Backlog >= 20` (3m), publishing real-time emails through `PipelineAlertTopicPolicy` + **Amazon SNS**.

---

### 3. Catching a Live Airflow 3.3 Bug & Building `ops.pipeline_ingestion_audit`

* **Live Airflow 3.3 Discovery:** `context["triggering_asset_events"]` returned empty during live verification because Airflow 3 Task SDK exposes **`TriggeringAssetEventsAccessor`** (inheriting `collections.abc.Mapping`, **not `dict`**). Switching to `isinstance(..., Mapping)` in `include/sqs_parser.py` fixed event extraction cleanly (backed by 21/21 passing unit tests).
* **Audit Ledger & Safe ACK:** Every ingestion transitions through `SWAPPED → ARCHIVED → COMPLETED / FAILED` in `ops.pipeline_ingestion_audit`, archives to deterministic `archive/<safe_audit_id>/<filename>`, validates CSV headers (`validate_csv_header`), refreshes `VisibilityTimeout=900s` fail-closed, and prevents blind ACKs via `verify_safe_to_ack_when_raw_missing`.

---

### 4. Live Verification Results on AWS

* **Happy Path:** Replaying a CSV into `raw/` triggered Airflow 3 `AssetWatcher` automatically → **17/17 tasks PASS** (`00:03:52`), 11 dbt tests passed on Redshift, audit recorded `SWAPPED → ARCHIVED → COMPLETED`.
* **Fault Tolerance & DLQ Redrive:** Uploading a 0-row CSV triggered `ensure_non_empty_stage` rollback (preserving all 71 rows) → after 3 attempts the poison message moved to DLQ → CloudWatch Alarms turned `In alarm` and SNS emailed Gmail → restoring the CSV + running `aws sqs start-message-move-task` recovered **17/17 tasks PASS** and returned **4/4 Alarms to `OK`**.

* **GitHub Repository (Releases `5.0.0` / `5.0.1`):** <https://github.com/AOMINHTAM/aws-data-pipeline>
* **Portfolio Case Study:** <https://aominhtam-portfolio.vercel.app/projects/aws-data-pipeline.html>
