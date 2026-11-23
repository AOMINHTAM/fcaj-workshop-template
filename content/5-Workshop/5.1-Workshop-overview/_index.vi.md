---
title: "5.1 Tổng quan & Kiến trúc V5"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### Mục tiêu bài thực hành
Sau khi hoàn thành Workshop, bạn sẽ tự tay dựng và kiểm chứng một hệ thống Data Pipeline chuẩn Production trên AWS có khả năng:
* Tự động tiếp nhận và xử lý dữ liệu thương mại điện tử ngay khi tệp CSV mới được tải lên **Amazon S3** (không dùng cron polling lãng phí tài nguyên).
* Bảo vệ kho dữ liệu **Amazon Redshift Serverless** trước dữ liệu rỗng (`0` dòng) hoặc sai schema, cô lập gói tin lỗi vào **SQS DLQ** và bắn cảnh báo email thời gian thực qua **CloudWatch + SNS**.
* Biến đổi dữ liệu theo mô hình **Kimball Star Schema** gia tăng (**dbt Incremental `delete+insert`**) kèm kiểm thử tự động và sổ cái đối soát `ops.pipeline_ingestion_audit`.

---

### Kiến trúc tổng thể V5 (Tự thiết kế bằng Draw.io & AWS Icons)

![Sơ đồ Kiến trúc V5 — Production CloudOps & Resilience](/fcaj-workshop-template/images/workshop/architecture_v5.png)

#### Luồng dữ liệu toàn trình (End-to-End Data Flow):
1. **Landing & Event Push:** Tệp CSV Olist được đồng bộ vào `s3://<bucket>/raw/*.csv` → S3 phát sự kiện `s3:ObjectCreated:*` đẩy JSON message vào **Amazon SQS Main Queue** (`VisibilityTimeout=900s`).
2. **Zero-Cron Trigger:** **Airflow 3 `AssetWatcher`** (`SqsSensorTrigger`, Long Polling 20s) tự động đánh thức DAG `ecommerce_aws_data_pipeline` ngay khi hàng đợi có message (nhàn rỗi = `0` DAG run, `$0` Redshift compute).
3. **Two-Phase Atomic Ingestion:** Task `load_raw_from_sqs` kiểm tra CSV header (`validate_csv_header`), nạp `COPY` từ S3 (qua STS `RedshiftS3ReadOnlyRole`) vào `TEMP TABLE`, kiểm tra `staged_rows > 0` (`ensure_non_empty_stage`), sau đó thực thi `DELETE + INSERT` và ghi `status = 'SWAPPED'` vào `ops.pipeline_ingestion_audit` trong **cùng một transaction `autocommit=False`**.
4. **Incremental Star Schema Transformation:** **Astronomer Cosmos** chạy **dbt Core** biến đổi qua 7 Staging Views (`public_staging`) và 3 Marts Tables (`fct_orders`, `dim_customers`, `dim_products`) cùng bộ dbt schema tests (`unique`, `not_null`, `relationships`, `accepted_values`).
5. **Deterministic Archive & Safe ACK:** Sau khi dbt đạt 100%, Task `archive_and_ack` sao chép tệp sang `archive/<safe_audit_id>/<filename>`, xác minh tồn tại, cập nhật `ARCHIVED → COMPLETED`, xóa tệp trong `raw/` và gọi `DeleteMessage` xóa SQS message.
6. **Fault Isolation & Active Alerting:** Nếu gặp tệp lỗi quá `maxReceiveCount=3`, SQS chuyển message sang **SQS DLQ** (lưu 14 ngày) → **CloudWatch Alarms** chuyển trạng thái `ALARM` → **Amazon SNS** gửi email cảnh báo tức thì; kỹ sư sửa dữ liệu và chạy **DLQ Redrive** để khôi phục.
7. **Decoupled BI Export:** Module `scripts/export_to_csv.py` kết xuất 4 tệp CSV sạch (~56 MB) phục vụ Power BI offline với `$0` chi phí RPU nhàn rỗi.

---

### Đối chiếu 5 trụ cột AWS Well-Architected Framework

| Trụ cột Well-Architected | Cách áp dụng cụ thể trong V5 |
| :--- | :--- |
| **Operational Excellence** | 100% hạ tầng khai báo bằng **CloudFormation IaC** (`cfn-lint` CI, Change Sets, Native Auto-Rollback), sổ tay **Operations Runbook** song ngữ và bảng kiểm toán `ops.pipeline_ingestion_audit`. |
| **Security** | Tách 3 nhóm **Least-Privilege IAM** (`RedshiftS3ReadOnlyRole` chỉ đọc `raw/*`, `AirflowRuntimePolicy` không được xóa `archive/*`, `DlqOperatorPolicy`); S3 Block Public Access 100%, mã hóa `AES256`, ép HTTPS-only (`S3RawBucketPolicy`); Redshift khóa VPC Security Group `/32` và Secrets Manager. |
| **Reliability** | **Two-Phase TEMP TABLE Swap** (`autocommit=False`), **CSV Header + Empty-Stage Guard**, **SQS DLQ (`maxReceiveCount=3`) + DLQ Redrive**, **4 CloudWatch Alarms + SNS**, S3 Versioning, và **Redshift Serverless Recovery Points** mỗi 30 phút. |
| **Performance Efficiency** | Nạp song song **MPP `COPY`** từ S3 vào Redshift Serverless, mô hình **dbt Incremental (`delete+insert`)** với CTE 4 nguồn `affected_order_ids` tránh full rebuild, và SQS Long Polling 20s. |
| **Cost Optimization** | **Redshift Serverless 4 RPUs** tự ngủ `$0` compute khi rỗi, `DeployRedshift=false` mặc định cho môi trường lab, DAG full-refresh `Paused`, S3 Lifecycle (30/90 ngày), AWS Budgets `$5/tháng`, và `make cfn-delete` dọn sạch về `$0`. |
