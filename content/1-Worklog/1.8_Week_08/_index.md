---
title: "Week 8 Worklog"
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Objectives:
* Optimize data transformation performance via **Incremental Modeling** with **dbt Core (`dbt-redshift`)**, adopting the `delete+insert` strategy to minimize **Amazon Redshift Serverless** compute overhead.
* Handle **Late-arriving data** using a **3-day lookback window** on the Fact table (`fct_orders`), while normalizing dimensional grain at `customer_unique_id` level on `dim_customers` to accurately compute customer **Lifetime Value (LTV)**.
* Elevate software engineering rigor: Develop a comprehensive **Unit Test** suite with **Pytest**, implement continuous integration (**CI/CD**) via **GitHub Actions**, and export 4 clean datasets to hand over for the **Data Analytics (DA)** phase.

### Tasks Completed This Week:
* **dbt Incremental Modeling for Fact Table:** Updated DDL to add the metadata column `_ingested_at TIMESTAMP DEFAULT SYSDATE` across all 7 Staging tables in Redshift Serverless; refactored `fct_orders.sql` into `materialized='incremental'` with `unique_key='order_id'`, implementing `delete+insert` and an `affected_order_ids` CTE unioning changes across 4 upstream tables (`stg_orders`, `stg_order_items`, `stg_payments`, `stg_reviews`) with a 3-day lookback window.
* **Dimensional Grain Normalization & Drift Reconciliation DAG:** Refactored `dim_customers.sql` into `materialized='incremental'` with `unique_key='customer_id'` and `delete+insert`; resolved dimensional grain by disaggregating and re-aggregating at the unique individual customer level (`customer_unique_id`), ensuring cumulative **Lifetime Value (LTV)** and segmentation (`Returning` vs `One-Time`) remain accurate; authored the drift-reconciliation DAG `dags/ecommerce_weekly_full_refresh_dag.py` (paused by default) for scheduled full rebuilds.
* **Unit Testing Suite with Pytest:** Authored 7 isolated unit tests for the SQS parsing module (`tests/test_sqs_parser.py`) covering single CSV parsing, multiple records, non-CSV filtering, S3 URL-encoded key decoding, and malformed payload handling; authored unit test cases for the ingestion safety guard (`tests/test_ingestion_guards.py`) validating `ensure_non_empty_stage` behavior.
* **CI/CD Pipeline & Curated Analytical Extraction:** Built a **CI Pipeline** with **GitHub Actions** (`.github/workflows/ci.yml`), automating linting (Flake8), unit tests (Pytest), and SQL validation (`dbt parse`) on every pull request and commit to `main`; developed `scripts/export_to_csv.py` with SSL-encrypted Redshift connectivity, extracting 4 curated analytical datasets (~54 MB, 344,483 rows) into `exports/`.
* **End-to-End Technical Verification & Handover:** Executed comprehensive verification: confirmed 100% Unit Tests (7/7 passed), 100% dbt Tests (9/9 passed), and a green GitHub Actions CI build; finalized the Data Engineering Technical Acceptance Report, handed over curated datasets to the Data Analytics (DA) team, and prepared the analytics plan for Week 9.

### Week 8 Achievements:
* Drastically reduced transformation runtimes and Redshift Serverless compute costs by migrating from full rebuilds to dbt Incremental Modeling (`delete+insert`).
* Robustly resolved two classic data engineering edge cases: late-arriving payment and review updates via multi-table 3-day lookback CTE, and customer LTV grain integrity via `customer_unique_id`.
* Elevated repository software engineering standards with 7 Pytest cases, ingestion safety guards, and automated GitHub Actions CI testing achieving 100% All-Green status.
* Successfully exported 4 clean analytical datasets (~54 MB, 344,483 rows), completing the Data Engineering lifecycle and handing over to the Data Analytics phase (Week 9).

### References:
* <https://docs.getdbt.com/docs/build/incremental-models>
* <https://docs.getdbt.com/terms/kimball-dimensional-modeling>
* <https://docs.pytest.org/en/stable/>
* <https://docs.github.com/en/actions>
