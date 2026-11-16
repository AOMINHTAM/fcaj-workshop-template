---
title: "Week 7 Worklog"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:
* Transition data ingestion from traditional cron-based polling to an event-driven architecture (**Event-Driven S3 Ingestion**) to optimize cloud infrastructure costs (**FinOps $0 compute** during idle periods).
* Integrate **Amazon S3 Event Notifications** with an **Amazon SQS** queue (`olist-s3-raw-arrival-queue`) and orchestrate triggers using **Apache Airflow 3 `Asset` + `AssetWatcher` (`SqsSensorTrigger` Long Polling 20s)**, completely eliminating `*/5` cron schedules.
* Engineer an **Atomic TEMP TABLE Staging Swap** mechanism backed by an **Ingestion Guard** (`ensure_non_empty_stage`) to prevent empty or malformed files from overwriting production staging tables on **Amazon Redshift Serverless**.

### Tasks Completed This Week:
* **FinOps Cost Analysis & Event-Driven Architecture Design:** Evaluated the cost drawbacks of traditional polling models (`S3KeySensor` and cron `schedule="*/5 * * * *"`) in cloud environments: wasted Airflow worker resources and unneeded Redshift Serverless RPU compute during idle periods; drafted the Event-Driven Ingestion architecture: S3 (`raw/*.csv`) → S3 Event Notification → Amazon SQS Queue → Airflow 3 `AssetWatcher`.
* **Amazon SQS Provisioning & S3 Event Routing:** Provisioned the Amazon SQS queue `olist-s3-raw-arrival-queue` on the AWS Console with Visibility Timeout (300s) and Message Retention (4 days); configured S3 Event Notifications on the bucket to route `s3:ObjectCreated:*` events (filter prefix `raw/`) to the SQS queue ARN and granted SQS Access Policy to the S3 Principal.
* **Airflow 3 Asset & AssetWatcher Implementation:** Installed Astro CLI and upgraded to the Apache Airflow 3 runtime on Docker and WSL2; authored the primary DAG leveraging **Airflow 3 `Asset` & `AssetWatcher`** with `SqsSensorTrigger` (Long Polling `wait_time_seconds=20`, `poke_interval=20`), triggering DAG runs automatically upon new S3 arrivals without any cron schedule (`schedule=None`).
* **Atomic Staging Swap & Ingestion Guard Module:** Built the ingestion safety module `include/ingestion_guards.py` implementing `ensure_non_empty_stage` (`staged_rows > 0`); implemented the **Atomic Staging Swap**: executed S3 `COPY` into a `TEMP TABLE` with transactional atomicity (`autocommit=False`), overwriting the production staging table only when data is verified non-empty, preventing data corruption.
* **SQS Message Lifecycle & End-to-End Verification:** Implemented post-verification acknowledgment: deleted SQS messages (`delete_message`) and archived S3 objects (`archive/<timestamp>/`) only after downstream validation and checks passed; executed end-to-end testing with sample Olist uploads, verifying the Airflow DAG triggers within < 20 seconds, safely loads data, and acknowledges queue messages.

### Week 7 Achievements:
* Successfully deployed an Event-Driven Ingestion architecture on AWS: S3 → SQS → Airflow 3 `AssetWatcher`, eliminating 100% of idle DAG executions and achieving FinOps compliance ($0 Redshift RPU compute when no new data arrives).
* Guaranteed staging data immutability and integrity using the Atomic Staging Swap and `ensure_non_empty_stage` guard, eliminating the risk of empty/corrupt files wiping production tables.
* Established a reliable SQS message lifecycle, acknowledging and deleting messages only after downstream validation and archiving succeed.

### References:
* <https://aws.amazon.com/blogs/big-data/>
* <https://docs.aws.amazon.com/AmazonS3/latest/userguide/EventNotifications.html>
* <https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/assets.html>
* <https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html>
