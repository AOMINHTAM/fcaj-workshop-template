---
title: "5.3 Triển khai Hạ tầng AWS bằng CloudFormation IaC"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

### 1. Kiến trúc Template CloudFormation (`infra/cloudformation/template.yaml`)
Toàn bộ hạ tầng V5 được khai báo tường minh trong tệp `infra/cloudformation/template.yaml` gồm **15 tài nguyên cốt lõi**, **4 tài nguyên FinOps/Redshift tùy chọn** (`DeployRedshift=true/false`) và **16 Outputs**:

* **Phá bẫy Circular Dependency kép bằng `!Sub`:**
  * *Vòng lặp 1 (`S3RawBucket ↔ SqsMainQueuePolicy`):* `SqsMainQueuePolicy` ghép trước ARN bucket bằng `!Sub` và `S3RawBucket` đặt `DependsOn: SqsMainQueuePolicy` để S3 bắn `s3:TestEvent` thành công ngay lượt tạo đầu tiên.
  * *Vòng lặp 2 (`SqsMainQueue ↔ SqsDeadLetterQueue`):* `SqsDeadLetterQueue` ghép trước ARN queue chính bằng `!Sub` trong `RedriveAllowPolicy: byQueue`, giúp AWS tạo chuẩn thứ tự `DLQ → Main Queue → Queue Policy → S3 Bucket`.
* **Bảo mật 5 lớp cho Amazon S3 (`S3RawBucket` + `S3RawBucketPolicy`):** Mã hóa `AES256`, chặn 4 cờ Public Access, từ chối mọi truy cập không qua HTTPS (`aws:SecureTransport: "false"`), bật **Versioning**, và Lifecycle (hết hạn `archive/` sau 90 ngày, phiên bản cũ sau 30 ngày, dọn `DeleteMarker` & multipart treo sau 7 ngày).
* **Phân tách 3 nhóm Least-Privilege IAM (có hậu tố `${AWS::Region}`):**
  1. `RedshiftS3ReadOnlyRole`: Khóa `aws:SourceAccount`, chỉ cấp quyền đọc `raw/*`.
  2. `AirflowRuntimePolicy`: Consume/ACK trên `SqsMainQueue`, `GET/DELETE` trên `raw/*`, và `GET/PUT` (**không có `DELETE`**) trên `archive/*`.
  3. `DlqOperatorPolicy`: Tách riêng quyền kiểm tra và chạy `StartMessageMoveTask` (DLQ Redrive).
* **4 CloudWatch Alarms (`TreatMissingData: notBreaching`) + `PipelineAlertTopicPolicy` + SNS:**
  1. `DlqMessagesVisibleAlarm`: `ApproximateNumberOfMessagesVisible >= 1` trong 1 phút.
  2. `MainQueueOldestMessageAlarm`: `ApproximateAgeOfOldestMessage > 600s` (SLA 10 phút).
  3. `MainQueueInFlightAlarm`: `ApproximateNumberOfMessagesNotVisible >= 1` liên tục 15 phút.
  4. `MainQueueBacklogAlarm`: `ApproximateNumberOfMessagesVisible >= 20` liên tục 3 phút.

---

### 2. Bước 1 — Kiểm tra tĩnh & Xem trước Change Set
```bash
# Kiểm tra tĩnh bằng cfn-lint và CloudFormation Engine
cfn-lint infra/cloudformation/template.yaml
make cfn-validate

# Xem trước thay đổi bằng Change Set (Dry-Run an toàn)
aws cloudformation deploy \
  --stack-name olist-v5-pipeline-stack \
  --template-file infra/cloudformation/template.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --no-execute-changeset \
  --region us-east-1 \
  --parameter-overrides \
    ProjectPrefix=olist-v5 \
    EnvironmentName=dev \
    AlertEmail="your-email@example.com" \
    DeployRedshift=false
```

---

### 3. Bước 2 — Triển khai Stack & Kiểm tra Outputs
Chế độ mặc định `DeployRedshift=false` dựng toàn bộ 15 tài nguyên cốt lõi trong hạn mức Always Free (tái sử dụng cụm Redshift Serverless hiện có). Nếu muốn CloudFormation dựng luôn Redshift Serverless (khóa `PubliclyAccessible=false`, `/32` CIDR, Secrets Manager), truyền `DeployRedshift=true` kèm `VpcId` và `SubnetIds`.

```bash
make cfn-deploy ALERT_EMAIL="your-email@example.com"
make cfn-status
```

Khi stack chuyển sang trạng thái **`CREATE_COMPLETE`**, ghi lại 4 Outputs quan trọng (`S3RawBucketName`, `SqsMainQueueName`, `SqsMainQueueUrl`, `RedshiftS3ReadOnlyRoleArn`) vào tệp `.env`.

*(Nếu có bất kỳ lỗi khởi tạo nào xảy ra, CloudFormation tự động kích hoạt **Native Auto-Rollback** thu hồi toàn bộ tài nguyên tạo dở về trạng thái sạch).*

---

### 4. Bước 3 — Xác nhận Email đăng ký Amazon SNS
Mở hộp thư của `ALERT_EMAIL`, tìm email từ `AWS Notifications` và bấm **Confirm subscription** để kích hoạt kênh nhận cảnh báo CloudWatch → SNS.

![Xác nhận đăng ký Amazon SNS Subscription thành công](/fcaj-workshop-template/images/workshop/sns_v5_subscription_confirmed.png)
