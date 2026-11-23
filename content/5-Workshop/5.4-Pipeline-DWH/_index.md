---
title: "5.4 Redshift Serverless, Airflow 3 & dbt Core Pipeline"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

### 1. Associate the V5 IAM Role with Amazon Redshift Serverless
In default FinOps mode (`DeployRedshift=false`, reusing an existing 4-RPU Redshift Serverless workgroup), associate the CloudFormation-provisioned `RedshiftS3ReadOnlyRole` with your Namespace so `COPY` strictly reads `raw/*`:

```bash
ROLE_ARN=$(aws cloudformation describe-stacks \
  --stack-name olist-v5-pipeline-stack \
  --region us-east-1 \
  --query "Stacks[0].Outputs[?OutputKey=='RedshiftS3ReadOnlyRoleArn'].OutputValue" \
  --output text)

aws redshift-serverless update-namespace \
  --namespace-name "${REDSHIFT_NAMESPACE:-ecommerce-namespace}" \
  --iam-roles "$ROLE_ARN" \
  --default-iam-role-arn "$ROLE_ARN" \
  --region us-east-1
```

*(Ensure the Workgroup's VPC Security Group allows port `5439` only from your workstation `/32` CIDR and disable conflicting VPN/Cloudflare WARP tunnels during local testing).*

---

### 2. Initialize Raw Tables & the Audit Ledger DDL
Run both SQL scripts via Redshift Query Editor v2 (or `psql` over port `5439`):
1. `scripts/create_raw_tables.sql`: Creates the `raw_data` schema with 9 Olist tables plus `_ingested_at TIMESTAMP DEFAULT SYSDATE`.
2. `scripts/create_ops_audit_table.sql`: Creates **`ops.pipeline_ingestion_audit`** tracking `SWAPPED → ARCHIVED → COMPLETED / FAILED`, `rows_loaded`, `s3_key`, deterministic `archive_key`, and error messages (also auto-bootstrapped by the DAG if absent).

---

### 3. Airflow 3 (`AssetWatcher`) Orchestration & Ingestion Safety
The central DAG `dags/ecommerce_pipeline_dag.py` runs purely event-driven (`schedule=None`) across three tightly guarded stages:

1. **Zero-Cron Trigger (`Asset` + `AssetWatcher`):**
   * Uses `SqsSensorTrigger` (`20s` Long Polling, `delete_message_on_reception=False`, `VisibilityTimeout=900s`).
   * `include/sqs_parser.py` handles **Airflow 3.3 Task SDK**'s `TriggeringAssetEventsAccessor` via `isinstance(..., collections.abc.Mapping)` (fixing a real live `dict` check bug), enforces the flat `raw/<allowlisted_filename>.csv` contract, and extracts a cross-run `event_token`.
2. **Task 1 — `@task load_raw_from_sqs` (Two-Phase Atomic Staging Swap):**
   * *Phase A (Staging & Guards):* Validates CSV headers against `EXPECTED_CSV_HEADERS` (`validate_csv_header`), loads into `TEMP TABLE tmp_stage_<table>` via `COPY`, and enforces `ensure_non_empty_stage` (`staged_rows > 0`) for all batch files before touching any target `raw_data.*` table.
   * *Phase B (Atomic Commit & Audit):* Executes `DELETE + INSERT` and writes fail-closed `status = 'SWAPPED'` inside the **same `autocommit=False` transaction** (`conn.commit()`), followed by a `VisibilityTimeout=900s` heartbeat refresh. Any failure triggers `conn.rollback()` and logs `FAILED`.
3. **Task 2 — Astronomer Cosmos (`dbt Core`) & Kimball Star Schema:**
   * Transforms 9 `raw_data` sources → 7 Staging Views (`public_staging`) → 3 Marts Tables (`public_marts`):
     * **`fct_orders`** (`incremental`, `delete+insert`): 4-source `affected_order_ids` CTE (`stg_orders`, `stg_order_items`, `stg_payments`, `stg_reviews`) with 3-day lookback, computing `total_order_cost`, `primary_payment_type`, `is_delayed`, and `delivery_status` (`On Time`, `Delayed`, `Pending Delivery`).
     * **`dim_customers`** (`incremental`, `delete+insert`): Preserves `customer_id` grain for unambiguous 1:1 Star Schema joins with `fct_orders`, while window-aggregating **Customer Lifetime Value (LTV)** over `customer_unique_id`.
     * **`dim_products`** (`table`): English product catalog with physical dimensions and weights.
   * Validates warehouse contracts via automated dbt tests (`unique`, `not_null`, `relationships`, `accepted_values`).
4. **Task 3 — `@task archive_and_ack` (Deterministic Archive & Safe ACK):**
   * Refreshes SQS `VisibilityTimeout` with `fail_on_error=True`, copies files to deterministic `archive/<safe_audit_id>/<filename>`, verifies `check_for_key(dest_key)`, logs `ARCHIVED → COMPLETED`, deletes `raw/` objects, and **only then** calls `DeleteMessage` (deferring multi-record ACKs and blocking blind ACKs via `verify_safe_to_ack_when_raw_missing`).

![dbt Data Lineage Graph](/fcaj-workshop-template/images/workshop/dbt_lineage.png)

---

### 4. Start Airflow 3 & Trigger the Live Pipeline
```bash
# Start local Apache Airflow 3 (Astro CLI & Docker)
astro dev start

# Open Airflow UI at http://localhost:8080 and unpause ecommerce_aws_data_pipeline
# Keep ecommerce_dbt_full_refresh_maintenance paused by default ($0 FinOps)

# Sync Olist CSVs into S3 raw/ to trigger the event-driven pipeline
aws s3 sync ./data/ s3://$S3_BUCKET_NAME/raw/ --exclude "*" --include "*.csv"
```

Once the DAG completes, export **4 BI-ready CSVs** (~56 MB) into `exports/` for `$0`-idle offline Power BI analytics:
```bash
make export-data
```
