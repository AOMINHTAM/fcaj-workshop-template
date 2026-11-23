---
title: "Worklog Tuần 9"
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:
* Nâng cấp hệ thống Data Pipeline từ mốc **V4 (`v4.0.1`)** lên **V5 (`5.0.0` / `5.0.1` — Production CloudOps & Resilience)** theo chuẩn **AWS First Cloud AI Journey (FCAJ)** với 100% hạ tầng quản lý bằng **AWS CloudFormation IaC**.
* Hoàn thiện khả năng chịu lỗi và giám sát chủ động (**Fault Tolerance & Observability**): cô lập gói tin lỗi bằng **Amazon SQS Dead-Letter Queue (DLQ)**, thiết lập **4 Amazon CloudWatch Alarms + Amazon SNS Email Alerting**, và xây dựng bảng kiểm toán **`ops.pipeline_ingestion_audit`** cùng cơ chế **Safe ACK**.
* Nghiệm thu thực tế (Live Verification) toàn trình trên **AWS & Airflow 3** cho cả 2 kịch bản (Happy Path và Poison CSV → DLQ → Alarm → SNS → DLQ Redrive), gia cố các điểm mù kiến trúc (`5.0.1`), và hoàn thiện bộ tài liệu kỹ thuật song ngữ (**V5 Deep Dive, Operations Runbook, Business Case, Retrospective**).

### Các công việc đã thực hiện trong tuần:
* **Thiết kế & Triển khai 100% Infrastructure as Code bằng AWS CloudFormation (`infra/cloudformation/template.yaml`):**
  * Định nghĩa **15 tài nguyên cốt lõi** + **4 tài nguyên FinOps/Redshift tùy chọn** (`DeployRedshift=false` mặc định, `PubliclyAccessible=false`, khóa CIDR `/32`, quản lý mật khẩu tự động qua **AWS Secrets Manager**) + **16 Outputs**, hỗ trợ **Native Stack Auto-Rollback** và kiểm tra tĩnh tự động bằng **`cfn-lint`** trong GitHub Actions CI.
  * Xử lý triệt để bẫy **Circular Dependency kép** giữa `S3RawBucket ↔ SqsMainQueuePolicy` và `SqsMainQueue ↔ SqsDeadLetterQueue` bằng cách ghép ARN qua `!Sub` kết hợp `DependsOn: SqsMainQueuePolicy`.
  * Bảo mật 5 lớp cho S3 Data Lake: mã hóa `AES256`, chặn 4 cờ Public Access, gắn `S3RawBucketPolicy` từ chối mọi truy cập không qua HTTPS (`aws:SecureTransport: "false"`), bật **S3 Versioning** + Lifecycle (`archive/` 90 ngày, phiên bản cũ 30 ngày, dọn `DeleteMarker` & multipart upload treo 7 ngày), và phát triển script phân trang `scripts/empty_versioned_bucket.py` phục vụ dọn sạch 100% tài nguyên bằng 1 lệnh (`make cfn-delete`).
  * Phân tách quyền **Least-Privilege IAM** theo vùng (`*-${AWS::Region}`): `RedshiftS3ReadOnlyRole` (chỉ đọc `raw/*`, khóa `aws:SourceAccount`), `AirflowRuntimePolicy` (không có quyền `DELETE` trên `archive/*`), và `DlqOperatorPolicy` (tách riêng quyền vận hành `StartMessageMoveTask` DLQ Redrive).
* **Xây dựng tầng Fault Tolerance & Giám sát Chủ động (SQS DLQ, CloudWatch Alarms & SNS):**
  * Triển khai **Amazon SQS Dead-Letter Queue** (`olist-v5-s3-raw-arrival-dlq-dev`, `maxReceiveCount=3`, lưu giữ 14 ngày, `RedriveAllowPolicy: byQueue`) giúp cách ly gói tin độc/CSV lỗi mà không gây vòng lặp vô hạn.
  * Thiết lập **4 CloudWatch Alarms** (`TreatMissingData: notBreaching`): `DlqMessagesVisibleAlarm` (≥ 1 trong 1 phút), `MainQueueOldestMessageAlarm` (> 600s SLA), `MainQueueInFlightAlarm` (≥ 1 liên tục 15 phút), và `MainQueueBacklogAlarm` (≥ 20 trong 3 phút), nối qua **`PipelineAlertTopicPolicy`** tới **Amazon SNS Email**.
* **Gia cố Độ tin cậy Ứng dụng, Vá Bug Airflow 3 & Audit Ledger (`5.0.0` → `5.0.1`):**
  * Phát hiện và vá lỗi thực tế của **Airflow 3.3 Task SDK** trong `include/sqs_parser.py`: `context["triggering_asset_events"]` là `TriggeringAssetEventsAccessor` kế thừa `collections.abc.Mapping` (không phải `dict`), chuyển sang kiểm tra `isinstance(..., Mapping)`.
  * Triển khai bảng đối soát **`ops.pipeline_ingestion_audit`** trên Redshift Serverless theo vòng đời `SWAPPED → ARCHIVED → COMPLETED / FAILED`; áp dụng **Two-Phase Batch Staging**, **CSV Header Schema Guard** (`validate_csv_header`), **Empty-Stage Guard** (`ensure_non_empty_stage`), ghi `SWAPPED` cùng transaction `DELETE + INSERT`, đường dẫn lưu trữ cố định (`archive/<safe_audit_id>/<filename>`), gia hạn `VisibilityTimeout=900s` fail-closed, trì hoãn ACK cho multi-record SQS message, và chặn ACK nhầm qua `verify_safe_to_ack_when_raw_missing`.
* **Nghiệm thu Live E2E trên AWS, Phát hành Release `5.0.0` / `5.0.1` & Hoàn thiện Tài liệu:**
  * Triển khai thật stack `olist-v5-pipeline-stack` (`us-east-1`, trạng thái `CREATE_COMPLETE`), xác nhận đăng ký SNS email, và kiểm chứng 2 kịch bản live:
    * *Happy Path E2E:* Replay `product_category_name_translation.csv` vào S3 `raw/` → Airflow 3 `AssetWatcher` tự động kích hoạt **17/17 tasks PASS** (`00:03:52`), 11 dbt schema tests PASS trên Redshift, audit ghi đủ `SWAPPED → ARCHIVED → COMPLETED` (`71` dòng).
    * *Fault Tolerance & DLQ Redrive:* Upload CSV rỗng `0` dòng → `ensure_non_empty_stage` chặn swap và rollback bảo toàn nguyên vẹn `71` dòng, ghi `FAILED` vào audit, đẩy message sang DLQ sau 3 lần thử → CloudWatch Alarms báo đỏ `In alarm` và SNS gửi email cảnh báo thật về Gmail → khôi phục CSV hợp lệ + chạy `aws sqs start-message-move-task` → DAG phục hồi **17/17 tasks PASS**, SQS Main/DLQ về `0/0` và **4/4 Alarms tự quay về xanh `OK`**.
  * Mở rộng bộ kiểm thử lên **21/21 Python unit tests** + **3 DAG pytests** + Ruff + `cfn-lint` + `dbt parse` xanh 100% trên GitHub Actions CI; merge PR #4 (`692050c`), phát hành Release **`5.0.0`** và **`5.0.1`**; hoàn thiện sơ đồ Draw.io `architecture_v5.drawio` cùng bộ tài liệu song ngữ **V5 Deep Dive, Operations Runbook (kèm 14 kịch bản Test Matrix), Business Case và Retrospective**.

### Kết quả đạt được tuần 9:
* Hạ tầng AWS của dự án đạt chuẩn **100% Infrastructure as Code (CloudFormation)**, có khả năng tái lập và dọn dẹp sạch về **$0** chỉ bằng một lệnh, bảo mật IAM tách vai trò và giữ chi phí vận hành trong hạn mức Always Free (~$0.003/tháng lưu trữ S3).
* Kiểm chứng thực tế thành công vòng đời chịu lỗi Production CloudOps toàn diện: **Empty CSV → Empty-Stage Guard Rollback → SQS DLQ → CloudWatch Alarms → SNS Email Alert → DLQ Redrive Recovery**.
* Phát hiện & khắc phục bug thật của **Airflow 3.3 Task SDK** (`TriggeringAssetEventsAccessor`), nâng chất lượng mã nguồn lên **21/21 unit tests PASS** và đóng hoàn chỉnh **Release `5.0.0` / `5.0.1`** cùng đầy đủ tài liệu vận hành & phân tích nghiệp vụ song ngữ.

### Tài liệu tham khảo:
* <https://github.com/AOMINHTAM/aws-data-pipeline>
* <https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html>
* <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html>
* <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html>
* <https://docs.aws.amazon.com/sns/latest/dg/welcome.html>
