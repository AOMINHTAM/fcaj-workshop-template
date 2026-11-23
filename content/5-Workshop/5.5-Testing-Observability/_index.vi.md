---
title: "5.5 Kiểm thử E2E, Xử lý Lỗi DLQ, Dự toán Chi phí & Demo"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

### 1. Nghiệm thu Live Kịch bản 1 — Happy Path E2E (`17/17` Tasks PASS)
Thực hiện replay tệp `product_category_name_translation.csv` từ `archive/` về `s3://<bucket>/raw/`:
```bash
aws s3 cp \
  s3://$S3_BUCKET_NAME/archive/product_category_name_translation.csv \
  s3://$S3_BUCKET_NAME/raw/product_category_name_translation.csv
```
* **Kết quả thực tế (18/11/2026):** `AssetWatcher` tự động kích hoạt không có DAG run rỗng, hoàn tất **17/17 tasks PASS** (`00:03:52`), chạy đạt **11 dbt schema tests** trên Redshift Serverless, bảng `ops.pipeline_ingestion_audit` ghi đầy đủ 3 nấc **`SWAPPED → ARCHIVED → COMPLETED`** (`71` dòng), tệp lưu đúng đường dẫn deterministic và SQS ACK sạch về `0`.

![Airflow 3 Event-Driven Run 17/17 Tasks PASS](/fcaj-workshop-template/images/workshop/airflow_v5_asset_watcher_run.png)

![Bảng kiểm toán ops.pipeline_ingestion_audit trên Redshift Serverless](/fcaj-workshop-template/images/workshop/redshift_v5_ops_audit_table.png)

---

### 2. Nghiệm thu Live Kịch bản 2 — Fault Tolerance (CSV 0 dòng → DLQ → Alarm → SNS → Redrive)
1. **Giả lập gói tin độc (Empty CSV — chỉ có header, 0 dòng dữ liệu):** Tải file CSV rỗng lên `s3://<bucket>/raw/product_category_name_translation.csv`.
   * Hàm `ensure_non_empty_stage` lập tức kích hoạt `ValueError`, **rollback giao dịch bảo toàn nguyên vẹn `71` dòng** trong `raw_data.product_category_name_translation` và ghi `status='FAILED', rows_loaded=0` vào bảng kiểm toán.

![Log Empty-Stage Guard chặn file CSV rỗng bảo vệ bảng đích](/fcaj-workshop-template/images/workshop/airflow_v5_empty_stage_guard_log.png)

2. **Cô lập sang SQS DLQ & Kích hoạt 4 CloudWatch Alarms + SNS Email:**
   * Sau `maxReceiveCount=3`, Amazon SQS tự động chuyển gói tin lỗi sang hàng đợi cách ly **`olist-v5-s3-raw-arrival-dlq-dev`** (`1` message available).
   * Trong vòng 1 phút, các **CloudWatch Alarms** (`dlq-messages-visible`, `backlog-age`, `inflight-sustained`) chuyển đỏ **`In alarm`** và **Amazon SNS** gửi email cảnh báo tức thì về Gmail.

![Amazon SQS Dead-Letter Queue cách ly gói tin độc](/fcaj-workshop-template/images/workshop/sqs_v5_dlq_messages.png)

![4 CloudWatch Alarms chuyển trạng thái In alarm](/fcaj-workshop-template/images/workshop/cloudwatch_v5_alarms_in_alarm.png)

![Email cảnh báo thời gian thực từ Amazon SNS gửi về Gmail](/fcaj-workshop-template/images/workshop/gmail_v5_sns_dlq_alarm.png)

3. **Khôi phục bằng SQS DLQ Redrive (`StartMessageMoveTask`):**
   * Khôi phục tệp CSV hợp lệ (`71` dòng) và thực thi lệnh Redrive đẩy message từ DLQ về Main Queue:
   ```bash
   aws sqs start-message-move-task \
     --source-arn "$DLQ_ARN" \
     --destination-arn "$MAIN_QUEUE_ARN" \
     --region us-east-1
   ```
   * **Kết quả:** DAG Run phục hồi đạt **17/17 tasks PASS**, sổ cái ghi tiếp `SWAPPED → ARCHIVED → COMPLETED`, cả SQS Main & DLQ về `0/0`, và **toàn bộ 4/4 CloudWatch Alarms tự động quay về xanh `OK`**.

---

### 3. Kiểm thử Tự động CI/CD & Phiên bản Hardening (`5.0.1`)
Trên mọi Push/PR vào nhánh `main`, GitHub Actions (`.github/workflows/ci.yml`) chạy tự động với chi phí `$0` AWS:
* **Ruff Lint** (`0` lỗi format) + **`cfn-lint`** kiểm tra chuẩn `infra/cloudformation/template.yaml`.
* **21/21 Python Unit Tests PASS** (`tests/test_ingestion_guards.py` 11 tests + `tests/test_sqs_parser.py` 10 tests).
* **Offline `dbt parse`** kiểm tra cú pháp toàn bộ SQL/YAML + **Astro Docker Build & 3 DAG Pytests**.

---

### 4. Dự toán Chi phí & Tối ưu hóa FinOps (AWS Pricing Calculator)

> **Lưu ý hoàn thiện trước khi nộp Portal (Tránh 0 điểm mục Trình bày):** Hãy tạo bảng ước tính trực tiếp trên [AWS Pricing Calculator](https://calculator.aws/) cho cấu hình bên dưới, dán link chia sẻ (Share link) và lưu ảnh chụp vào `static/images/workshop/aws_pricing_calculator.png`.

| Dịch vụ AWS | Cấu hình tính phí | Chi phí ước tính / tháng | Chiến lược tối ưu FinOps đã áp dụng |
| :--- | :--- | :---: | :--- |
| **AWS CloudFormation & IAM** | 1 Stack (15–19 resources) | `$0.00` | Miễn phí hoàn toàn; tái lập & xóa nhanh tránh bỏ quên tài nguyên. |
| **Amazon S3 (Standard + Versioning)** | ~120 MB dữ liệu + archive | `~$0.003` | Lifecycle hết hạn `archive/` 90 ngày, xóa old versions sau 30 ngày & multipart treo sau 7 ngày. |
| **Amazon SQS (Main + DLQ)** | ≤ 104k requests/tháng | `$0.00` | Nằm trong Always Free 1 triệu requests; Long Polling 20s; `0` reqs khi `astro dev stop`. |
| **Amazon CloudWatch & SNS** | 4 Standard Alarms + Email | `$0.00` | Nằm trong Always Free (10 Alarms + 1.000 emails/tháng). |
| **Amazon Redshift Serverless** | 4 RPUs (~1–2 RPU-hours/tháng) | `~$0.75 – $1.50` | Tự ngủ `$0` khi idle; `DeployRedshift=false` mặc định; DAG full-refresh `Paused`; tách BI qua CSV `exports/`. |

* **Link ước tính AWS Pricing Calculator:** *(Chèn link share từ `https://calculator.aws/` tại đây)*
* **Ảnh chụp AWS Pricing Calculator:** *(Đang chờ bổ sung `static/images/workshop/aws_pricing_calculator.png`)*

---

### 5. Minh chứng Sản phẩm thực tế (Live Demo / Video Walkthrough)

> **Lưu ý hoàn thiện trước khi nộp Portal (Tránh 0 điểm mục Demo):** Quay video demo E2E dài 5–8 phút minh họa luồng chạy khớp với sơ đồ `architecture_v5.png` (Deploy/Outputs CloudFormation → Upload CSV → Airflow 3 `AssetWatcher` → Redshift `ops.pipeline_ingestion_audit` & dbt → Empty CSV DLQ/Alarm/Redrive → Clean-up) và gắn link YouTube/Google Drive tại đây.

* **Video Demo E2E (5–8 phút):** *(Chèn link YouTube / Google Drive công khai tại đây)*
* **Portfolio Case Study trực tuyến:** <https://aominhtam-portfolio.vercel.app/projects/aws-data-pipeline.html>
* **GitHub Repository (Release `5.0.0` / `5.0.1`):** <https://github.com/AOMINHTAM/aws-data-pipeline>
