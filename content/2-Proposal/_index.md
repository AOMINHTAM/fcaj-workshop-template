---
title: "Proposal"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

### 1. Project Overview
* **Project Title:** **AWS E-Commerce Modern Data Pipeline (V5)** — A 100% **AWS CloudFormation IaC**, event-driven, fault-tolerant ELT platform with **Kimball Star Schema** modeling on **Amazon Redshift Serverless**.
* **Specialization Track:** Data & Analytics / Cloud Data Engineering (Workforce Bootcamp — First Cloud AI Journey 2026).
* **Real-World Dataset:** **Olist Brazilian E-Commerce Public Dataset** (~120 MB, 9 relational CSVs, 99,441 orders, 112,650 order items).
* **Source Repository:** <https://github.com/AOMINHTAM/aws-data-pipeline>

---

### 2. Problem Statement & Business Case
Raw operational exports from an e-commerce marketplace (such as Olist) arrive as **9 fragmented CSV files**, creating five critical business and engineering bottlenecks if consumed directly or loaded via ad-hoc scripts:

1. **Finance & Sales Reconciliation Gaps:** Orders span multiple `order_items` and installment `order_payments`, causing merchandise revenue, freight fees, invoice totals, and cash collected to diverge without a governed Fact model.
2. **Distorted Customer Count & Lifetime Value (LTV):** The source platform assigns a fresh `customer_id` per order rather than aggregating by `customer_unique_id`, misreporting repeat purchase rates and customer LTV.
3. **Missing Delivery SLA Metrics & Raw Schema Defects:** Raw timestamps lack standardized delivery duration, delay metrics, and SLA statuses (`delivery_status`) tied to `review_score`; product categories are in Portuguese with raw column typos (`product_name_lenght`).
4. **Data-Loss & Idle Cloud Billing Risks:** Naive `TRUNCATE + COPY` commits immediately in Redshift and cannot roll back when an empty or corrupt CSV arrives; fixed-schedule polling (`*/5`) wastes requests and compute when queues are idle.
5. **Lack of Production Reproducibility & Observability:** Manually provisioned AWS resources lack native rollback; poison messages can loop indefinitely; operations lack automated alarms and an ingestion audit ledger.

---

### 3. Objectives & Scope
* **100% Declarative Infrastructure as Code (IaC):** Provision, lint (`cfn-lint`), auto-rollback, and tear down all AWS resources to `$0` using **AWS CloudFormation** (`infra/cloudformation/template.yaml`).
* **Pure Event-Driven ELT (Zero Time Cron):** Connect **Amazon S3 Event Notifications → Amazon SQS (Main + DLQ) → Apache Airflow 3 `AssetWatcher`** (`Long Polling 20s`), eliminating idle DAG runs.
* **Transactional Safety & Auditability:** Enforce **Two-Phase Batch Staging**, **CSV Header Schema Guard**, **Empty-Stage Guard** (`ensure_non_empty_stage`), atomic `DELETE + INSERT` (`autocommit=False`), the **`ops.pipeline_ingestion_audit`** ledger, and **Deterministic Safe ACK**.
* **Kimball Star Schema Analytics Warehouse:** Build **dbt Core (`dbt-redshift`)** layers (7 Staging Views + 3 Marts Tables: 4-source incremental `fct_orders`, `customer_unique_id` LTV-preserving `dim_customers`, and `dim_products`) verified by automated dbt schema tests.
* **Active Observability & Decoupled BI:** Deploy **4 CloudWatch Alarms + Amazon SNS Email**, split **Least-Privilege IAM**, and export **4 clean analytical CSVs** (~56 MB) powering Power BI at `$0` idle Redshift RPU cost.

---

### 4. Preliminary Solution Architecture

![V5 Architecture — Production CloudOps & Resilience](/fcaj-workshop-template/images/workshop/architecture_v5.png)

#### AWS Services Used (8 Services — Exceeding the ≥ 3 Service Requirement):
| Functional Layer | AWS Service | Role in V5 Architecture |
| :--- | :--- | :--- |
| **IaC & Governance** | **AWS CloudFormation** | Provisions 15 core resources + 4 conditional resources with Change Sets & Native Auto-Rollback. |
| **Data Lake Storage** | **Amazon S3** | Hosts `raw/` (landing) and `archive/<audit_id>/` (replay) with Versioning, AES256, HTTPS-only BucketPolicy, and 30/90-day Lifecycle rules. |
| **Event Messaging & Resilience** | **Amazon SQS + DLQ** | Main queue (`VisibilityTimeout=900s`, 20s Long Polling) paired with `RedrivePolicy` (`maxReceiveCount=3`) into a 14-day DLQ supporting DLQ Redrive. |
| **Observability & Alerting** | **Amazon CloudWatch + Amazon SNS** | 4 Alarms monitoring DLQ arrivals, SLA message age (> 600s), sustained in-flight activity (15m), and backlog volume (≥ 20), notifying via SNS Email. |
| **Cloud Data Warehouse** | **Amazon Redshift Serverless** | FinOps-tuned 4 RPUs workgroup (`$0` idle compute) hosting Bronze/Silver/Gold schemas, `ops.pipeline_ingestion_audit`, and 30-minute Recovery Points. |
| **Security & FinOps** | **AWS IAM, VPC Security Group, AWS Budgets** | 3-way least-privilege IAM split (`RedshiftS3ReadOnlyRole`, `AirflowRuntimePolicy`, `DlqOperatorPolicy`), port `5439` scoping, and `$5/month` budget alerts. |

---

### 5. Implementation Plan & 12-Week Timeline

| Phase | Timeline | Key Activities | Deliverables |
| :--- | :---: | :--- | :--- |
| **Phase 1: AWS Foundations & FinOps** | Weeks 1 – 4 | Account setup, AWS Budgets, IAM Roles/STS, Multi-AZ Amazon VPC, Amazon S3 security & Lifecycle. | Secured AWS account, `$200` AWS Credits, FinOps guardrails. |
| **Phase 2: Data Engineering Core (V2 → V4)** | Weeks 5 – 8 | Olist ingestion into Redshift Serverless 4 RPUs, dbt Core Kimball Star Schema, Event-Driven S3 → SQS → Airflow 3 `AssetWatcher`, dbt Incremental + GitHub Actions CI + 4 CSV exports. | Release `v4.0.1`, 9/9 dbt tests PASS, all-green CI, 4 curated CSVs (~54–56 MB). |
| **Phase 3: Production CloudOps (V5)** | Week 9 | 100% CloudFormation IaC migration, SQS DLQ, 4 CloudWatch Alarms + SNS, `ops.pipeline_ingestion_audit` + Safe ACK, live E2E & Poison Redrive verification. | Releases `5.0.0` / `5.0.1`, 21/21 unit tests PASS, bilingual Runbook & Deep Dive docs. |
| **Phase 4: BI Analytics & FCAJ Packaging** | Weeks 10 – 12 | Power BI dashboard on curated CSVs, 3 AWS Study Group blogs, 6-chapter Hugo Workshop, AWS Pricing Calculator, Demo Video, and final FCAJ Portal submission. | Complete bilingual Portfolio/Workshop website submitted for FCAJ stamp approval. |

---

### 6. Budget & AWS Cost Estimation (FinOps)

Official cost estimate built on **[AWS Pricing Calculator](https://calculator.aws/#/estimate?id=77a934d9011f072f5b013defff9c61ac61f895ef)** in **`us-east-1` (US East — N. Virginia)** covering all V5 pipeline services:

| AWS Service | Sized Configuration in Calculator | List Monthly Cost | 12-Month Cost | Effective Spend w/ Always Free & Credits |
| :--- | :--- | :---: | :---: | :--- |
| **Amazon S3 (S3 Standard)** | 1 GB/month + 1,000 PUT/COPY/LIST + 5,000 GET/SELECT | `$0.03` | `$0.36` | `~$0.003` for ~120 MB Olist + 30/90-day Lifecycle |
| **Amazon SQS** | Standard queue: 0.15 million requests/month | `$0.06` | `$0.72` | `$0.00` (within 1M requests/month Free Tier) |
| **Amazon CloudWatch** | 4 Standard-Resolution Alarm Metrics | `$0.40` | `$4.80` | `$0.00` (within 10 free Standard Alarms/month) |
| **Amazon SNS** | 100 Email/Email-JSON Notifications/month | `$0.00` | `$0.00` | `$0.00` (1,000 free email notifications/month) |
| **Amazon Redshift Serverless** | 4 Base RPUs (`0.0656h/day` ≈ 2h/month) + 1 GB Managed Storage | `$3.02` | `$36.24` | Auto-sleeps `$0` idle; covered by `$200` AWS Credits |
| **AWS CloudFormation & IAM** | 1 Stack (15–19 resources) & 3 IAM groups | `$0.00` | `$0.00` | 100% Free |
| **Total (Pricing Calculator)** | **Upfront `$0.00`** • **AWS Budgets Alert `$5.00/mo`** | **`$3.51 / month`** | **`$42.12 / year`** | **Effective net spend after Free Tier `~$3.02/mo`** |

* **Official AWS Pricing Calculator Link:** <https://calculator.aws/#/estimate?id=77a934d9011f072f5b013defff9c61ac61f895ef>
* **Exported Calculator Reports:** [Download Official PDF](/fcaj-workshop-template/files/aws_pricing_calculator.pdf) | [Download Detailed CSV](/fcaj-workshop-template/files/aws_pricing_calculator.csv)

![AWS Pricing Calculator Cost Estimate for V5 Data Pipeline](/fcaj-workshop-template/images/workshop/aws_pricing_calculator.png)

---

### 7. Risks & Mitigations

| Technical / Operational Risk | Severity | V5 Mitigation Mechanism |
| :--- | :---: | :--- |
| **Empty (`0`-row) or malformed CSV wiping target tables** | High | Dual guard (`validate_csv_header` + `ensure_non_empty_stage`) on `TEMP TABLE` inside `autocommit=False` transaction with automatic `rollback()`. |
| **Poison messages causing infinite retry loops** | High | `SqsDeadLetterQueue` (`maxReceiveCount=3`) + `DlqMessagesVisibleAlarm` SNS alerting + controlled recovery via `StartMessageMoveTask` (DLQ Redrive). |
| **Premature SQS ACK on missing raw objects or partial failures** | High | Fail-closed `ops.pipeline_ingestion_audit` ledger, deterministic `archive/<safe_audit_id>/` paths, `VisibilityTimeout=900s` heartbeat, and `verify_safe_to_ack_when_raw_missing`. |
| **Unintended Redshift / AWS idle compute spend** | Medium | **4 RPUs** auto-sleep when idle, `DeployRedshift=false` default, paused full-refresh DAG, offline Power BI CSV exports, `$5/mo` AWS Budget, and 1-command `make cfn-delete` to `$0`. |
| **`BucketNotEmpty` failure when deleting a versioned S3 stack** | Medium | Automated paginated `scripts/empty_versioned_bucket.py` purging all `Versions` and `DeleteMarkers` prior to `delete-stack`. |
