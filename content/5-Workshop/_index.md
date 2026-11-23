---
title: "Workshop: AWS E-Commerce Modern Data Pipeline (V5)"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

### Workshop Overview
This hands-on workshop provides a step-by-step, reproducible guide to building an **AWS E-Commerce Modern Data Pipeline (V5 — Production CloudOps & Resilience)** using the real-world **Olist Brazilian E-Commerce Dataset** (~120 MB, 9 relational tables, 99,441 orders) aligned with **AWS First Cloud AI Journey (FCAJ)** standards:

* **100% Infrastructure as Code (AWS CloudFormation):** Declarative provisioning, static CI linting (`cfn-lint`), Native Auto-Rollback, and 1-command teardown (`make cfn-delete`).
* **Pure Event-Driven & Fault-Tolerant ELT:** Amazon S3 Event Notifications → Amazon SQS Main Queue + Dead-Letter Queue (DLQ) → Apache Airflow 3 `AssetWatcher` → Amazon Redshift Serverless (4 RPUs) → dbt Core Kimball Star Schema.
* **Observability, Audit & FinOps:** 4 Amazon CloudWatch Alarms + Amazon SNS Email alerts, the `ops.pipeline_ingestion_audit` ledger with Deterministic Safe ACK, and FinOps cost governance under `$2/month`.
* **Reproducible Source Repository:** <https://github.com/AOMINHTAM/aws-data-pipeline>

---

### 📑 6-Chapter Workshop Outline:
1. **[5.1 Workshop Overview & V5 Architecture](5.1-workshop-overview/):** Business problem, custom Draw.io architecture diagram, end-to-end data flow, and **AWS Well-Architected Framework** alignment.
2. **[5.2 Prerequisites & Environment Setup](5.2-prerequiste/):** Configuring Ubuntu WSL2, Docker Desktop, Astro CLI, AWS CLI v2, Olist dataset, and `.env` secrets.
3. **[5.3 AWS Infrastructure Provisioning via CloudFormation IaC](5.3-infra-iac/):** Deploying `infra/cloudformation/template.yaml` (15 core + 4 optional resources), resolving dual circular dependencies, 5-layer S3 security, SQS + DLQ, 4 CloudWatch Alarms + SNS, and split Least-Privilege IAM.
4. **[5.4 Redshift Serverless, Airflow 3 & dbt Core Pipeline](5.4-pipeline-dwh/):** Namespace IAM Role association, `raw_data` & `ops.pipeline_ingestion_audit` DDL, Airflow 3 `AssetWatcher`, Two-Phase Atomic `TEMP TABLE` Swap, and dbt Incremental Star Schema.
5. **[5.5 E2E Verification, Fault Tolerance, Cost Estimation & Demo](5.5-testing-observability/):** Live Happy Path (`17/17` tasks), 0-row CSV → DLQ → CloudWatch Alarm → SNS Email → DLQ Redrive validation, CI/CD (`21/21` unit tests), **AWS Pricing Calculator**, and **Demo Video**.
6. **[5.6 Resource Clean-up to $0.00](5.6-cleanup/):** Stopping Airflow Triggerer, running `make cfn-delete` (purging S3 Object Versions via `empty_versioned_bucket.py`), removing standalone Redshift Serverless resources, and verifying zero lingering charges.
