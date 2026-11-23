---
title: "5.6 Dọn dẹp tài nguyên về $0.00 (Clean-up)"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

### Tại sao cần dọn dẹp đúng thứ tự?
Vì `S3RawBucket` trong CloudFormation bật **`VersioningConfiguration: Enabled`**, lệnh `aws s3 rm --recursive` thông thường chỉ gắn *DeleteMarkers* chứ không xóa *Object Versions* — nếu gọi `delete-stack` ngay sẽ gặp lỗi `BucketNotEmpty`. Quy trình 4 bước dưới đây đảm bảo dọn sạch 100% tài nguyên AWS và đưa chi phí về **`$0.00`**.

---

### Bước 1 — Tắt Apache Airflow Triggerer trên máy cục bộ
Ngừng toàn bộ tiến trình Long Polling SQS ngay lập tức:
```bash
astro dev stop
```

---

### Bước 2 — Xóa sạch S3 Object Versions & Xóa toàn bộ CloudFormation Stack bằng 1 lệnh
```bash
make cfn-delete
```
Lệnh `make cfn-delete` tự động thực hiện tuần tự:
1. Chạy `scripts/empty_versioned_bucket.py` (lặp phân trang xóa sạch toàn bộ `Versions` và `DeleteMarkers` theo lô 500 objects, kể cả khi bucket có > 1.000 phiên bản).
2. Gọi `aws cloudformation delete-stack --stack-name olist-v5-pipeline-stack --region us-east-1`.
3. Chờ đến khi trạng thái xác nhận `stack-delete-complete` (xóa sạch S3 Bucket, SQS Main Queue, SQS DLQ, 4 CloudWatch Alarms, SNS Topic & IAM Policies/Roles trong Stack).

---

### Bước 3 — Xóa Redshift Serverless (nếu bạn khởi tạo ngoài Stack với `DeployRedshift=false`)
```bash
# 1. Xóa Workgroup trước
aws redshift-serverless delete-workgroup \
  --workgroup-name ecommerce-workgroup \
  --region us-east-1

# 2. Sau khi Workgroup xóa xong, xóa Namespace (không giữ final snapshot để không tốn phí lưu trữ)
aws redshift-serverless delete-namespace \
  --namespace-name ecommerce-namespace \
  --region us-east-1
```

---

### Bước 4 — Kiểm tra xác nhận không còn tài nguyên tồn đọng
```bash
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE DELETE_FAILED \
  --region us-east-1 \
  --query "StackSummaries[].{Name:StackName,Status:StackStatus}" \
  --output table
```
Kiểm tra thêm trong **AWS Billing & Cost Management** và **AWS Budgets** để chắc chắn không còn tài nguyên chạy ngầm phát sinh chi phí.
