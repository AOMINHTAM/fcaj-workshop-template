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

Bảng dự toán chính thức được lập trên **[AWS Pricing Calculator](https://calculator.aws/#/estimate?id=77a934d9011f072f5b013defff9c61ac61f895ef)** tại Region **`us-east-1` (US East — N. Virginia)** cho toàn bộ 5 nhóm dịch vụ vận hành V5:

| Dịch vụ AWS | Cấu hình nhập trên AWS Pricing Calculator | Chi phí chuẩn / tháng | Chi phí 12 tháng | Thực tế áp dụng Always Free / Credit |
| :--- | :--- | :---: | :---: | :--- |
| **Amazon S3 (S3 Standard)** | 1 GB/tháng + 1.000 PUT/COPY/LIST + 5.000 GET/SELECT | `$0.03` | `$0.36` | `~$0.003` cho ~120 MB Olist + Lifecycle 30/90 ngày |
| **Amazon SQS** | Standard queue: 0.15 triệu requests/tháng | `$0.06` | `$0.72` | `$0.00` (nằm gọn trong 1 triệu requests/tháng Free Tier) |
| **Amazon CloudWatch** | 4 Standard-Resolution Alarm Metrics | `$0.40` | `$4.80` | `$0.00` (nằm trong 10 Standard Alarms miễn phí/tháng) |
| **Amazon SNS** | 100 Email/Email-JSON Notifications/tháng | `$0.00` | `$0.00` | `$0.00` (miễn phí 1.000 emails/tháng) |
| **Amazon Redshift Serverless** | 4 Base RPUs (`0.0656h/ngày` ≈ 2h/tháng) + 1 GB Managed Storage | `$3.02` | `$36.24` | Tự ngủ `$0` khi idle; chi trả từ `$200` AWS Credit |
| **AWS CloudFormation & IAM** | 1 Stack (15–19 resources) & 3 nhóm IAM | `$0.00` | `$0.00` | Miễn phí 100% |
| **Tổng cộng (Pricing Calculator)** | **Upfront `$0.00`** • **Ngân sách AWS Budgets `$5.00/tháng`** | **`$3.51 / tháng`** | **`$42.12 / năm`** | **Thực chi sau Free Tier chỉ `~$3.02/tháng` (từ Credit)** |

* **Liên kết dự toán chính thức (AWS Pricing Calculator):** <https://calculator.aws/#/estimate?id=77a934d9011f072f5b013defff9c61ac61f895ef>
* **Tài liệu xuất từ Calculator:** [Tải PDF chính thức](/fcaj-workshop-template/files/aws_pricing_calculator.pdf) | [Tải CSV chi tiết](/fcaj-workshop-template/files/aws_pricing_calculator.csv)

![Bảng dự toán chi phí AWS Pricing Calculator cho dự án V5](/fcaj-workshop-template/images/workshop/aws_pricing_calculator.png)

---

### 7. Rủi ro & Giải pháp giảm thiểu (Risks & Mitigations)

| Rủi ro kỹ thuật / Vận hành | Mức độ | Giải pháp giảm thiểu đã triển khai trong V5 |
| :--- | :---: | :--- |
| **File CSV rỗng (`0` dòng) hoặc sai schema xóa hỏng bảng đích** | Cao | Khóa 2 lớp `validate_csv_header` + `ensure_non_empty_stage` (`staged_rows > 0`) trên `TEMP TABLE` với giao dịch `autocommit=False` → `rollback()` tự động. |
| **Gói tin lỗi gây lặp vô tận (Infinite Retry Loop)** | Cao | Cấu hình `SqsDeadLetterQueue` (`maxReceiveCount=3`) + CloudWatch `DlqMessagesVisibleAlarm` báo email ngay và xử lý bằng `StartMessageMoveTask` (DLQ Redrive). |
| **Xóa nhầm SQS message khi mất file hoặc lỗi nửa chừng** | Cao | Áp dụng sổ cái `ops.pipeline_ingestion_audit` (fail-closed), đường dẫn `archive/<safe_audit_id>/` cố định, gia hạn `VisibilityTimeout=900s` và `verify_safe_to_ack_when_raw_missing`. |
| **Phát sinh chi phí Redshift/AWS ngoài kiểm soát** | Trung bình | Cấu hình **4 RPUs** tự ngủ khi rỗi, `DeployRedshift=false` mặc định, DAG full-refresh ở chế độ `Paused`, tách Power BI qua CSV `exports/`, AWS Budgets `$5/tháng` và `make cfn-delete` dọn sạch về `$0`. |
| **Lỗi `BucketNotEmpty` khi xóa CloudFormation Stack có Versioning** | Trung bình | Tự động chạy script phân trang `scripts/empty_versioned_bucket.py` xóa toàn bộ `Versions` + `DeleteMarkers` trước khi gọi `delete-stack`. |
