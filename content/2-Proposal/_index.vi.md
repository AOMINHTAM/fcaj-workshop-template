---
title: "Đề xuất giải pháp (Proposal)"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

### 1. Tổng quan dự án (Project Overview)
* **Tên đề tài:** Xây dựng **AWS E-Commerce Modern Data Pipeline (V5)** — Kiến trúc ELT hướng sự kiện, chịu lỗi (Fault-Tolerant), quản trị 100% bằng **AWS CloudFormation IaC** và mô hình hóa dữ liệu **Kimball Star Schema** trên **Amazon Redshift Serverless**.
* **Định hướng chuyên sâu:** Data & Analytics / Cloud Data Engineering (Workforce Bootcamp — First Cloud AI Journey 2026).
* **Bộ dữ liệu thực tế:** **Olist Brazilian E-Commerce Dataset** (~120 MB, 9 tệp CSV quan hệ, 99.441 đơn hàng, 112.650 chi tiết đơn hàng).
* **Mã nguồn dự án:** <https://github.com/AOMINHTAM/aws-data-pipeline>

---

### 2. Đặt vấn đề & Bài toán nghiệp vụ (Problem Statement)
Trong hệ thống thương mại điện tử thực tế (như sàn Olist), dữ liệu giao dịch xuất ra từ hệ thống OLTP dưới dạng **9 tệp CSV rời rạc** tồn tại 5 điểm nghẽn lớn nếu đưa trực tiếp vào phân tích hoặc vận hành bằng script thủ công:

1. **Mâu thuẫn số liệu Tài chính & Bán hàng:** Một đơn hàng có nhiều `order_items` và nhiều dòng thanh toán trả góp (`order_payments`) khiến tiền hàng thuần, phí ship, tổng hóa đơn và tiền thực thu không khớp nếu không tách bạch ở tầng Fact.
2. **Đếm sai khách hàng thật & Customer Lifetime Value (LTV):** Hệ thống sinh một `customer_id` mới cho mỗi đơn hàng thay vì gom theo `customer_unique_id`, dẫn tới đánh giá sai hoàn toàn tỷ lệ khách quay lại.
3. **Thiếu chỉ số SLA giao hàng & lỗi dữ liệu gốc:** Chưa có sẵn độ trễ giao hàng (`delay_days`, `delivery_status`) để đối chiếu với `review_score`; danh mục sản phẩm còn dùng tiếng Bồ Đào Nha và có lỗi chính tả cột (`product_name_lenght`).
4. **Rủi ro mất dữ liệu & đốt tiền Cloud khi vận hành:** Cách nạp `TRUNCATE + COPY` không thể rollback trong Redshift khi gặp file CSV rỗng/lỗi; lịch polling định kỳ (`*/5`) gây dư thừa request và lãng phí compute khi không có dữ liệu mới.
5. **Thiếu tính tái lập & giám sát Production:** Hạ tầng tạo thủ công bằng Console khó rollback; gói tin lỗi có thể lặp vô hạn; thiếu cảnh báo chủ động và nhật ký kiểm toán quá trình nạp.

---

### 3. Mục tiêu & Phạm vi (Objectives & Scope)
* **Hạ tầng 100% Infrastructure as Code (IaC):** Tự động khởi tạo, kiểm thử tĩnh (`cfn-lint`), tự động Rollback và dọn dẹp sạch về `$0` bằng **AWS CloudFormation** (`infra/cloudformation/template.yaml`).
* **Kiến trúc ELT hướng sự kiện thuần túy (Zero Time Cron):** Kết hợp **Amazon S3 Event Notifications → Amazon SQS (Main + DLQ) → Apache Airflow 3 `AssetWatcher`** (`Long Polling 20s`), loại bỏ hoàn toàn các lượt chạy DAG rỗng.
* **An toàn giao dịch & Kiểm toán dữ liệu:** Áp dụng **Two-Phase Batch Staging**, **CSV Header Guard**, **Empty-Stage Guard** (`ensure_non_empty_stage`), giao dịch nguyên tử `DELETE + INSERT` (`autocommit=False`), sổ cái kiểm toán **`ops.pipeline_ingestion_audit`** và **Deterministic Safe ACK**.
* **Kho dữ liệu phân tích Kimball Star Schema:** Triển khai **dbt Core (`dbt-redshift`)** gồm 7 Staging Views và 3 Marts Tables (`fct_orders` incremental 4 nguồn, `dim_customers` bảo toàn LTV theo `customer_unique_id`, `dim_products`) vượt qua toàn bộ automated dbt tests.
* **Giám sát chủ động & Phục vụ BI:** Thiết lập **4 CloudWatch Alarms + Amazon SNS Email**, phân tách quyền **Least-Privilege IAM**, và trích xuất **4 bộ CSV sạch** (~56 MB) phục vụ Power BI với `$0` chi phí RPU nhàn rỗi.

---

### 4. Kiến trúc giải pháp sơ bộ (Solution Architecture)

![Sơ đồ kiến trúc V5 — Production CloudOps & Resilience](/fcaj-workshop-template/images/workshop/architecture_v5.png)

#### Danh mục dịch vụ AWS sử dụng (8 dịch vụ — vượt mức tối thiểu 3 dịch vụ):
| Nhóm chức năng | Dịch vụ AWS | Vai trò trong kiến trúc V5 |
| :--- | :--- | :--- |
| **IaC & Governance** | **AWS CloudFormation** | Định nghĩa 15 tài nguyên cốt lõi + 4 tài nguyên tùy chọn, hỗ trợ Change Sets & Native Auto-Rollback. |
| **Data Lake Storage** | **Amazon S3** | Lưu trữ `raw/` (Landing) và `archive/<audit_id>/` (Replay), bật Versioning, mã hóa AES256, ép HTTPS-only và Lifecycle 30/90 ngày. |
| **Event Messaging & Resilience** | **Amazon SQS + DLQ** | Hàng đợi chính (`VisibilityTimeout=900s`, Long Polling 20s) nối `RedrivePolicy` (`maxReceiveCount=3`) sang DLQ (lưu 14 ngày) và hỗ trợ DLQ Redrive. |
| **Observability & Alerting** | **Amazon CloudWatch + Amazon SNS** | 4 Alarms giám sát DLQ, SLA tuổi message (> 600s), In-Flight kéo dài (15 phút) và Backlog (≥ 20), gửi cảnh báo thời gian thực qua SNS Email. |
| **Cloud Data Warehouse** | **Amazon Redshift Serverless** | Cụm 4 RPUs tối ưu FinOps (tự ngủ `$0` compute khi rỗng), lưu 3 tầng dữ liệu + sổ cái `ops.pipeline_ingestion_audit` + Recovery Points mỗi 30 phút. |
| **Security & FinOps** | **AWS IAM, VPC Security Group, AWS Budgets** | Tách 3 nhóm Least-Privilege (`RedshiftS3ReadOnlyRole`, `AirflowRuntimePolicy`, `DlqOperatorPolicy`), khóa cổng `5439`, cảnh báo ngân sách `$5/tháng`. |

---

### 5. Kế hoạch triển khai & Lộ trình 12 tuần (Implementation Plan & Timeline)

| Giai đoạn | Thời gian | Hạng mục triển khai chính | Kết quả đầu ra (Deliverables) |
| :--- | :---: | :--- | :--- |
| **Giai đoạn 1: Nền tảng AWS & FinOps** | Tuần 1 – 4 | Khởi tạo tài khoản, AWS Budgets, IAM Roles/STS, Amazon VPC Multi-AZ, thực hành Amazon S3 & Lifecycle. | Hạ tầng bảo mật cơ bản, `$200` AWS Credit và hàng rào FinOps. |
| **Giai đoạn 2: Data Engineering Core (V2 → V4)** | Tuần 5 – 8 | Nạp Olist lên Redshift Serverless 4 RPUs, dựng Kimball Star Schema bằng dbt Core, nâng cấp Event-Driven S3 → SQS → Airflow 3 `AssetWatcher`, dbt Incremental + GitHub Actions CI + Export 4 CSV. | Release `v4.0.1`, 9/9 dbt tests PASS, CI xanh 100%, 4 CSV sạch (~54–56 MB). |
| **Giai đoạn 3: Production CloudOps (V5)** | Tuần 9 | Chuyển đổi 100% hạ tầng sang CloudFormation IaC, thêm SQS DLQ, 4 CloudWatch Alarms + SNS, `ops.pipeline_ingestion_audit` + Safe ACK, live test E2E + Poison Redrive. | Release `5.0.0` / `5.0.1`, 21/21 unit tests PASS, bộ tài liệu Runbook & Deep Dive song ngữ. |
| **Giai đoạn 4: BI Analytics & Đóng gói FCAJ** | Tuần 10 – 12 | Xây dựng Dashboard BI trên 4 CSV sạch, hoàn thiện 3 Blogs AWS Study Group, Workshop 6 chương, AWS Pricing Calculator, Demo Video và nghiệm thu báo cáo. | Website Portfolio/Workshop song ngữ hoàn chỉnh & nộp duyệt trên Portal FCAJ. |

---

### 6. Dự toán ngân sách & Chi phí AWS (Cost Estimation & FinOps)

> **Lưu ý nộp bài (AWS Pricing Calculator):** Bảng dưới đây là định mức FinOps thực tế của stack V5 (`DeployRedshift=false` hoặc dùng Redshift 4 RPUs theo nhu cầu). Trước khi nộp Portal, cần chèn **link share và ảnh chụp bảng dự toán từ [AWS Pricing Calculator](https://calculator.aws/)** vào mục bên dưới.

| Dịch vụ AWS | Quy mô sử dụng dự kiến / tháng | Hạn mức Always Free | Chi phí ước tính / tháng |
| :--- | :--- | :--- | :---: |
| **AWS CloudFormation & IAM** | 1 Stack (15–19 resources), 3 nhóm IAM | Miễn phí 100% | `$0.00` |
| **Amazon S3 (Standard + Versioning)** | ~120 MB dữ liệu Olist + archive (Lifecycle 30/90 ngày) | Theo GB (`$0.023/GB`) | `~$0.003` |
| **Amazon SQS (Main + DLQ)** | ~103.680 requests (nếu bật polling 24/7; `0` khi `astro dev stop`) | 1.000.000 requests/tháng | `$0.00` |
| **Amazon CloudWatch Alarms** | 4 Standard-resolution Alarms (`Period >= 60s`) | 10 Standard Alarms | `$0.00` |
| **Amazon SNS (Email)** | < 50 email cảnh báo/tháng | 1.000 email/tháng | `$0.00` |
| **Amazon Redshift Serverless (4 RPUs)** | ~1–2 RPU-hours/tháng cho batch nạp & dbt (tự ngủ khi idle) + ~0.2 GB Managed Storage | Tính theo sử dụng thực tế | `~$0.75 – $1.50` *(trong `$200` Credit)* |
| **Tổng cộng dự kiến** | **Vận hành học tập & demo thực tế** | **AWS Budgets cảnh báo `$5/tháng`** | **`< $2.00 / tháng`** |

* **Liên kết dự toán chính thức (AWS Pricing Calculator):** *(Đang chờ gắn link estimate từ `https://calculator.aws/`)*
* **Ảnh chụp bảng ước tính AWS Pricing Calculator:** *(Lưu tại `static/images/workshop/aws_pricing_calculator.png` sau khi xuất)*

---

### 7. Rủi ro & Giải pháp giảm thiểu (Risks & Mitigations)

| Rủi ro kỹ thuật / Vận hành | Mức độ | Giải pháp giảm thiểu đã triển khai trong V5 |
| :--- | :---: | :--- |
| **File CSV rỗng (`0` dòng) hoặc sai schema xóa hỏng bảng đích** | Cao | Khóa 2 lớp `validate_csv_header` + `ensure_non_empty_stage` (`staged_rows > 0`) trên `TEMP TABLE` với giao dịch `autocommit=False` → `rollback()` tự động. |
| **Gói tin lỗi gây lặp vô tận (Infinite Retry Loop)** | Cao | Cấu hình `SqsDeadLetterQueue` (`maxReceiveCount=3`) + CloudWatch `DlqMessagesVisibleAlarm` báo email ngay và xử lý bằng `StartMessageMoveTask` (DLQ Redrive). |
| **Xóa nhầm SQS message khi mất file hoặc lỗi nửa chừng** | Cao | Áp dụng sổ cái `ops.pipeline_ingestion_audit` (fail-closed), đường dẫn `archive/<safe_audit_id>/` cố định, gia hạn `VisibilityTimeout=900s` và `verify_safe_to_ack_when_raw_missing`. |
| **Phát sinh chi phí Redshift/AWS ngoài kiểm soát** | Trung bình | Cấu hình **4 RPUs** tự ngủ khi rỗi, `DeployRedshift=false` mặc định, DAG full-refresh ở chế độ `Paused`, tách Power BI qua CSV `exports/`, AWS Budgets `$5/tháng` và `make cfn-delete` dọn sạch về `$0`. |
| **Lỗi `BucketNotEmpty` khi xóa CloudFormation Stack có Versioning** | Trung bình | Tự động chạy script phân trang `scripts/empty_versioned_bucket.py` xóa toàn bộ `Versions` + `DeleteMarkers` trước khi gọi `delete-stack`. |
