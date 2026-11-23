---
title: "Bài viết 3: Nâng cấp V5 Production CloudOps với CloudFormation IaC, SQS DLQ & CloudWatch/SNS"
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

- **Chủ đề:** Đưa Data Pipeline lên chuẩn **Production CloudOps (V5)** với **100% AWS CloudFormation IaC**, **SQS Dead-Letter Queue**, **4 CloudWatch Alarms + SNS** và **Sổ cái Kiểm toán `ops.pipeline_ingestion_audit`**
- **Nhóm cộng đồng dự kiến đăng:** AWS Study Group VN (First Cloud AI Journey)
- **Trạng thái:** ⏳ Bản nháp hoàn chỉnh — chờ đăng lên AWS Study Group VN & gắn permalink chính thức
- **Link bài viết chính thức (Facebook):** *(Chèn permalink sau khi đăng lên AWS Study Group VN)*
- **Mã nguồn dự án (GitHub):** <https://github.com/AOMINHTAM/aws-data-pipeline>

---

### Giới thiệu

Sau khi hoàn thiện tầng Data Engineering Core ở V4, hệ thống vẫn còn 5 khoảng trống nếu đặt vào môi trường Production thực tế: hạ tầng dựng tay khó rollback, gói tin lỗi có thể lặp vô hạn, thiếu cảnh báo chủ động, quyền IAM chưa tách vai trò, và nguy cơ ACK nhầm khi retry. Bản nâng cấp **V5 (`5.0.0` / `5.0.1`)** giải quyết trọn vẹn 5 bài toán này theo chuẩn **AWS First Cloud AI Journey (FCAJ)** mà vẫn nằm gọn trong hạn mức Always Free (`~$0.003/tháng` S3).

![Sơ đồ kiến trúc V5 — Production CloudOps & Resilience](/fcaj-workshop-template/images/workshop/architecture_v5.png)

---

### 1. Phá bẫy Circular Dependency kép trong AWS CloudFormation IaC

Khi viết `infra/cloudformation/template.yaml` (15 tài nguyên cốt lõi + 4 tùy chọn), mình đụng ngay 2 lỗi vòng lặp phụ thuộc kinh điển của CloudFormation:
* **Vòng lặp 1 (`S3RawBucket ↔ SqsMainQueuePolicy`):** S3 yêu cầu Queue Policy tồn tại trước để bắn `s3:TestEvent`, nhưng nếu Policy gọi `!GetAtt S3RawBucket.Arn` thì CloudFormation báo Circular Dependency. **Cách phá:** Ghép trước ARN bucket bằng `!Sub` trong `SqsMainQueuePolicy` và gắn `DependsOn: SqsMainQueuePolicy` cho `S3RawBucket`.
* **Vòng lặp 2 (`SqsMainQueue ↔ SqsDeadLetterQueue`):** Tương tự, `SqsDeadLetterQueue` dùng `!Sub` trong `RedriveAllowPolicy: byQueue` để CloudFormation khởi tạo trơn tru theo thứ tự `DLQ → Main Queue → Queue Policy → S3 Bucket`.
* **Điểm mù khi xóa Stack có S3 Versioning:** Lệnh `aws s3 rm --recursive` chỉ gắn *DeleteMarkers* nên `delete-stack` bị lỗi `BucketNotEmpty`. Mình viết script phân trang `scripts/empty_versioned_bucket.py` tích hợp vào `make cfn-delete` để xóa sạch `Versions` + `DeleteMarkers` và đưa tài khoản về `$0` trong 1 lệnh.

---

### 2. Phân tách 3 nhóm Least-Privilege IAM & 4 CloudWatch Alarms + SNS

* **Separation of Duties (IAM):** Tách rõ `RedshiftS3ReadOnlyRole` (chỉ đọc `raw/*`), `AirflowRuntimePolicy` (không có quyền `DELETE` trên `archive/*`), và `DlqOperatorPolicy` (chuyên trách chạy `StartMessageMoveTask` redrive từ DLQ).
* **4 CloudWatch Alarms (`TreatMissingData: notBreaching`):** Giám sát `DlqMessagesVisible >= 1` (1 phút), `OldestMessage > 600s` (SLA 10 phút), `InFlight >= 1` liên tục 15 phút, và `Backlog >= 20` (3 phút), gắn `PipelineAlertTopicPolicy` publish email cảnh báo thời gian thực qua **Amazon SNS**.

---

### 3. Bắt sống Bug Airflow 3.3 & Sổ cái Kiểm toán `ops.pipeline_ingestion_audit`

* **Bug thực tế khi test live trên Airflow 3.3:** Hàm đọc sự kiện `context["triggering_asset_events"]` bị trả về rỗng vì trong Airflow 3 Task SDK, đối tượng này là **`TriggeringAssetEventsAccessor`** (kế thừa `collections.abc.Mapping`, **không phải `dict`**). Sửa sang `isinstance(..., Mapping)` trong `include/sqs_parser.py` đã giúp pipeline nhận trọn vẹn event (kèm 21/21 unit tests PASS).
* **Audit Ledger & Safe ACK:** Mọi file nạp vào Redshift đều đi qua máy trạng thái `SWAPPED → ARCHIVED → COMPLETED / FAILED` trong `ops.pipeline_ingestion_audit`, lưu đường dẫn cố định `archive/<safe_audit_id>/<filename>`, kiểm tra CSV Header (`validate_csv_header`), gia hạn `VisibilityTimeout=900s` fail-closed và chặn ACK nhầm bằng `verify_safe_to_ack_when_raw_missing`.

---

### 4. Nghiệm thu thực tế (Live Verified trên AWS)

* **Happy Path:** Replay CSV vào `raw/` → Airflow 3 `AssetWatcher` chạy **17/17 tasks PASS** (`00:03:52`), 11 dbt tests PASS trên Redshift, audit ghi đủ `SWAPPED → ARCHIVED → COMPLETED`.
* **Fault Tolerance & DLQ Redrive:** Upload CSV 0 dòng → `ensure_non_empty_stage` rollback bảo toàn nguyên vẹn 71 dòng dữ liệu → sau 3 lần thử message nhảy sang DLQ → CloudWatch Alarms báo đỏ `In alarm` + SNS bắn email về Gmail → sửa CSV + chạy `aws sqs start-message-move-task` → DAG phục hồi **17/17 tasks PASS** và **4/4 Alarms tự quay về xanh `OK`**.

* **GitHub Repository (Release `5.0.0` / `5.0.1`):** <https://github.com/AOMINHTAM/aws-data-pipeline>
* **Portfolio Case Study:** <https://aominhtam-portfolio.vercel.app/projects/aws-data-pipeline.html>
