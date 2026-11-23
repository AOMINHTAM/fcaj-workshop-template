---
title: "5.2 Các bước chuẩn bị (Prerequisites)"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

### 1. Tài khoản & Quyền AWS cần thiết
* Một tài khoản **AWS** (khuyến nghị Region **`us-east-1`** để đồng bộ toàn bộ dịch vụ) đã thiết lập **AWS Budgets** cảnh báo chi phí (`$5/tháng` hoặc theo ngân sách học tập).
* Một IAM User thực thi CLI (không dùng tài khoản Root) có quyền triển khai CloudFormation, S3, SQS, CloudWatch, SNS, IAM và Redshift Serverless.
* Một địa chỉ Email hợp lệ (`ALERT_EMAIL`) để xác nhận đăng ký nhận cảnh báo từ **Amazon SNS**.

---

### 2. Công cụ cài đặt trên máy trạm (Windows WSL2 Ubuntu / Linux / macOS)
1. **Ubuntu trên WSL2** (nếu dùng Windows) cùng `git`, `make`, `python3` (≥ 3.10).
2. **Docker Desktop** (đã bật tích hợp WSL2 backend).
3. **AWS CLI v2** đã cấu hình thông tin xác thực qua `aws configure`:
   ```bash
   aws --version
   aws sts get-caller-identity
   ```
4. **Astro CLI** để chạy môi trường **Apache Airflow 3 (Astronomer Runtime 3.3)** cục bộ trên Docker:
   ```bash
   astro version
   ```

---

### 3. Clone mã nguồn & Tải bộ dữ liệu Olist E-Commerce
1. Clone repository dự án và kiểm tra môi trường Python / kiểm thử cục bộ:
   ```bash
   git clone https://github.com/AOMINHTAM/aws-data-pipeline.git
   cd aws-data-pipeline

   # Kiểm tra xác thực AWS CLI và bộ test/lint offline ($0 AWS)
   make aws-verify
   make verify
   ```
2. Tải **Brazilian E-Commerce Public Dataset by Olist** từ Kaggle (<https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce>) gồm 9 tệp CSV (~120 MB):
   * `olist_orders_dataset.csv`
   * `olist_order_items_dataset.csv`
   * `olist_order_payments_dataset.csv`
   * `olist_order_reviews_dataset.csv`
   * `olist_customers_dataset.csv`
   * `olist_products_dataset.csv`
   * `olist_sellers_dataset.csv`
   * `olist_geolocation_dataset.csv`
   * `product_category_name_translation.csv`
3. Đặt 9 tệp CSV vào thư mục dữ liệu cục bộ chuẩn bị cho bước tải lên S3 ở Chương 5.4.

---

### 4. Chuẩn bị tệp biến môi trường `.env` (Git-Ignored)
Không bao giờ hard-code khóa AWS hoặc mật khẩu Redshift vào mã nguồn. Sau khi triển khai CloudFormation ở **Chương 5.3**, bạn sẽ điền các giá trị Outputs vào tệp `.env` (đã nằm trong `.gitignore`):

```env
DBT_REDSHIFT_PASSWORD=<your-redshift-admin-password>
AWS_DEFAULT_REGION=us-east-1
AWS_ACCESS_KEY_ID=<your-aws-access-key>
AWS_SECRET_ACCESS_KEY=<your-aws-secret-key>
S3_BUCKET_NAME=<S3RawBucketName-from-stack-outputs>
SQS_QUEUE_NAME=<SqsMainQueueName-from-stack-outputs>
SQS_QUEUE_URL=<SqsMainQueueUrl-from-stack-outputs>
REDSHIFT_IAM_ROLE=<RedshiftS3ReadOnlyRoleArn-from-stack-outputs>
AIRFLOW_CONN_AWS_DEFAULT=aws://
```
