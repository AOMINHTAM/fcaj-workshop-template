---
title: "Workshop: AWS E-Commerce Modern Data Pipeline (V5)"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

### Giới thiệu bài thực hành
Bài Workshop này hướng dẫn chi tiết từng bước triển khai một **AWS E-Commerce Modern Data Pipeline (V5 — Production CloudOps & Resilience)** hoàn chỉnh trên bộ dữ liệu thực tế **Olist Brazilian E-Commerce** (~120 MB, 9 bảng quan hệ, 99.441 đơn hàng) theo chuẩn **AWS First Cloud AI Journey (FCAJ)**:

* **100% Infrastructure as Code (AWS CloudFormation):** Triển khai, kiểm thử tĩnh (`cfn-lint`), Native Auto-Rollback và dọn dẹp 1 lệnh (`make cfn-delete`).
* **Pure Event-Driven & Fault-Tolerant ELT:** Amazon S3 Event Notifications → Amazon SQS Main Queue + Dead-Letter Queue (DLQ) → Apache Airflow 3 `AssetWatcher` → Amazon Redshift Serverless (4 RPUs) → dbt Core Kimball Star Schema.
* **Observability, Audit & FinOps:** 4 Amazon CloudWatch Alarms + Amazon SNS Email, sổ cái kiểm toán `ops.pipeline_ingestion_audit` + Deterministic Safe ACK, và tối ưu chi phí theo AWS Pricing Calculator (`$3.51/tháng`, còn `~$3.02/tháng` sau Always Free).
* **Mã nguồn tái lập:** <https://github.com/AOMINHTAM/aws-data-pipeline>

---

### 📑 Cấu trúc 6 chương của Workshop:
1. **[5.1 Tổng quan & Kiến trúc V5](5.1-workshop-overview/):** Bài toán nghiệp vụ, sơ đồ kiến trúc Draw.io tự thiết kế, luồng dữ liệu và đối chiếu 5 trụ cột **AWS Well-Architected Framework**.
2. **[5.2 Chuẩn bị môi trường & Yêu cầu tiên quyết](5.2-prerequiste/):** Cài đặt WSL2 Ubuntu, Docker Desktop, Astro CLI, AWS CLI v2, chuẩn bị dữ liệu Olist và tệp biến môi trường `.env`.
3. **[5.3 Triển khai Hạ tầng AWS bằng CloudFormation IaC](5.3-infra-iac/):** Deploy `infra/cloudformation/template.yaml` (15 tài nguyên cốt lõi + 4 tùy chọn), xử lý Circular Dependency kép, S3 bảo mật 5 lớp, SQS + DLQ, 4 CloudWatch Alarms + SNS và phân tách Least-Privilege IAM.
4. **[5.4 Cấu hình Redshift Serverless, Airflow 3 & dbt Core](5.4-pipeline-dwh/):** Gắn IAM Role, khởi tạo DDL `raw_data` & `ops.pipeline_ingestion_audit`, chạy Airflow 3 `AssetWatcher`, Two-Phase Atomic `TEMP TABLE` Swap và dbt Incremental Star Schema.
5. **[5.5 Kiểm thử E2E, Xử lý Lỗi DLQ, Dự toán Chi phí & Demo](5.5-testing-observability/):** Nghiệm thu Happy Path (`17/17` tasks), mô phỏng CSV 0 dòng → DLQ → CloudWatch Alarm → SNS Email → DLQ Redrive, CI/CD (`21/21` unit tests), bảng **AWS Pricing Calculator** và **Video Demo**.
6. **[5.6 Dọn dẹp tài nguyên về $0.00](5.6-cleanup/):** Tắt Airflow Triggerer, chạy `make cfn-delete` (xóa sạch S3 Versions bằng `empty_versioned_bucket.py`), xóa Redshift Serverless (nếu tạo ngoài Stack) và xác minh không còn tài nguyên tồn đọng.
