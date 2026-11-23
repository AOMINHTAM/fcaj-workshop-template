---
title: "Blog 2: Migrating to Pure Event-Driven Ingestion (S3 → SQS → Airflow 3) & dbt Incremental"
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

- **Topic:** Upgrading an AWS Data Pipeline from Scheduled Polling to **Pure Event-Driven (`AssetWatcher`)**, **Atomic `TEMP TABLE` Swap**, and **dbt Incremental (`delete+insert`)**
- **Target Community Group:** AWS Study Group VN (First Cloud AI Journey)
- **Status:** ⏳ Draft ready — pending Facebook publication & official permalink
- **Official Facebook Permalink:** *(Paste permalink after publishing to AWS Study Group VN)*
- **Project Source Code (GitHub):** <https://github.com/AOMINHTAM/aws-data-pipeline>

---

### Introduction

In my first post migrating from Snowflake to AWS S3 & Redshift Serverless, the primary limitation was manual/static loading and scheduled polling. In **V4 (`v4.0.1`)**, I solved three classic Cloud Data Engineering challenges: **eliminating wasteful cron polling**, **preventing table wipes from empty CSVs**, and **optimizing warehouse compute via dbt Incremental modeling**.

---

### 1. Replacing `*/5` Polling with Pure Event-Driven S3 → SQS → Airflow 3 `AssetWatcher`

* **Why Polling Hurts:** Fixed-interval `S3KeySensor` schedules create hundreds of `SKIPPED` DAG runs when no files arrive, add unnecessary S3 `LIST` calls, and risk waking compute unnecessarily.
* **Push-Based Event-Driven Design:**
  * Enabled **Amazon S3 Event Notifications** (`s3:ObjectCreated:*`, filtered by `prefix: raw/` and `suffix: .csv`) pushing directly to **Amazon SQS (`olist-s3-raw-arrival-queue`)** via a Resource-based Queue Policy locked to `aws:SourceArn` and `aws:SourceAccount`.
  * Upgraded orchestration to **Airflow 3 `Asset` + `AssetWatcher`** using `SqsSensorTrigger` (`20s` Long Polling, `delete_message_on_reception=False`, `VisibilityTimeout=900s`, `schedule=None`).
  * **FinOps Impact:** Idle queues generate **`0` DAG runs** and **`$0` Redshift compute**, while new CSV drops trigger end-to-end processing in under 20 seconds.

---

### 2. Why You Should Avoid `TRUNCATE + COPY` on Amazon Redshift

* In Redshift, `TRUNCATE` auto-commits immediately and **cannot be rolled back**. If a `0`-row or corrupted CSV lands in S3, production tables are wiped clean.
* **V4 Solution (Atomic `TEMP TABLE` Swap + Empty-Stage Guard):**
  1. Run `COPY` into an isolated `TEMP TABLE` first.
  2. Enforce `ensure_non_empty_stage` (`include/ingestion_guards.py`) verifying `staged_rows > 0`.
  3. Execute `DELETE + INSERT` inside an explicit `autocommit=False` transaction before `commit()`. Any failure triggers `conn.rollback()`, leaving production rows untouched.
  4. Archive objects to `archive/` and issue SQS `DeleteMessage` only after dbt quality tests pass.

---

### 3. Solving Two Classic `dbt Core` Incremental Challenges (`delete+insert`)

1. **Late-Arriving Updates on `fct_orders`:** Older orders often receive delayed deliveries or late `review_score` updates. I built a 4-source `affected_order_ids` CTE (`stg_orders`, `stg_order_items`, `stg_payments`, `stg_reviews`) combined with a **3-day lookback window** and `incremental_strategy='delete+insert'`.
2. **Preserving Customer Lifetime Value (LTV) on `dim_customers`:** Olist generates a new `customer_id` per order even for repeat buyers (`customer_unique_id`). `dim_customers` keeps `customer_id` grain for clean `1:1` Star Schema joins with `fct_orders` while re-aggregating full order history per affected `customer_unique_id` so LTV and `Returning Customer` flags stay 100% accurate.

---

### 4. Key Outcomes & Next Up (V5)
* Passed 100% of dbt schema tests (`unique`, `not_null`, `relationships`, `accepted_values`) and automated CI checks on GitHub Actions at `$0` AWS cost.
* Exported **4 clean BI datasets** (~54–56 MB, 344,483 rows) to decouple Power BI from Redshift Serverless compute.
* **Up next (V5):** Migrating to **100% AWS CloudFormation IaC**, isolating poison messages with **SQS DLQ**, adding **4 CloudWatch Alarms + SNS Email**, and building the **`ops.pipeline_ingestion_audit`** ledger.

* **GitHub Repository:** <https://github.com/AOMINHTAM/aws-data-pipeline>
