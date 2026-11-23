---
title: "5.4 Cấu hình Redshift Serverless, Airflow 3 & dbt Core"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

### 1. Gắn IAM Role V5 vào Amazon Redshift Serverless Namespace
Khi chạy ở chế độ FinOps mặc định (`DeployRedshift=false`, tái sử dụng cụm Redshift Serverless 4 RPUs có sẵn), thực hiện gắn `RedshiftS3ReadOnlyRole` vừa tạo từ CloudFormation vào Namespace để lệnh `COPY` chỉ đọc dữ liệu từ `raw/*`:

```bash
ROLE_ARN=$(aws cloudformation describe-stacks \
  --stack-name olist-v5-pipeline-stack \
  --region us-east-1 \
  --query "Stacks[0].Outputs[?OutputKey=='RedshiftS3ReadOnlyRoleArn'].OutputValue" \
  --output text)

aws redshift-serverless update-namespace \
  --namespace-name "${REDSHIFT_NAMESPACE:-ecommerce-namespace}" \
  --iam-roles "$ROLE_ARN" \
  --default-iam-role-arn "$ROLE_ARN" \
  --region us-east-1
```

*(Đảm bảo VPC Security Group của Workgroup chỉ mở cổng `5439` cho địa chỉ `/32` của máy trạm và không bật VPN/Cloudflare WARP gây xung đột định tuyến tới Redshift endpoint).*

---

### 2. Khởi tạo Lược đồ Bảng Raw & Sổ cái Kiểm toán Audit Ledger
Thực thi 2 tập lệnh DDL trên Redshift Query Editor v2 (hoặc `psql` qua cổng `5439`):
1. `scripts/create_raw_tables.sql`: Tạo schema `raw_data` gồm 9 bảng Olist kèm cột siêu dữ liệu `_ingested_at TIMESTAMP DEFAULT SYSDATE`.
2. `scripts/create_ops_audit_table.sql`: Tạo bảng kiểm toán **`ops.pipeline_ingestion_audit`** lưu vết trạng thái `SWAPPED → ARCHIVED → COMPLETED / FAILED`, số dòng `rows_loaded`, `s3_key`, đường dẫn lưu trữ cố định `archive_key` và thông điệp lỗi (DAG cũng hỗ trợ tự tạo bảng nếu chưa có).

---

### 3. Kiến trúc Điều phối Airflow 3 (`AssetWatcher`) & Độ tin cậy Nạp dữ liệu
DAG trung tâm `dags/ecommerce_pipeline_dag.py` vận hành theo 3 bước nối tiếp không dùng cron polling (`schedule=None`):

1. **Kích hoạt tức thì bằng `Asset` & `AssetWatcher`:**
   * Sử dụng `SqsSensorTrigger` (`Long Polling 20s`, `delete_message_on_reception=False`, `VisibilityTimeout=900s`).
   * Module `include/sqs_parser.py` đã vá bug thực tế của **Airflow 3.3 Task SDK**: kiểm tra `isinstance(..., collections.abc.Mapping)` để đọc chuẩn đối tượng `TriggeringAssetEventsAccessor` (thay vì `dict`), đồng thời khóa chặt hợp đồng flat prefix `raw/<allowlisted_filename>.csv` và trích xuất `event_token` xuyên DAG Run.
2. **Task 1 — `@task load_raw_from_sqs` (Two-Phase Atomic Staging Swap):**
   * *Pha A (Staging & Guard):* Đọc dòng header CSV trên S3 đối chiếu với `EXPECTED_CSV_HEADERS` (`validate_csv_header`), nạp `COPY` vào `TEMP TABLE tmp_stage_<table>` và kiểm tra `ensure_non_empty_stage` (`staged_rows > 0`) cho toàn bộ batch trước khi chạm vào bảng đích `raw_data.*`.
   * *Pha B (Atomic Commit & Audit):* Thực thi `DELETE + INSERT` và ghi `status = 'SWAPPED'` (fail-closed) trong **cùng một transaction `autocommit=False`** → `conn.commit()`, kèm gia hạn `VisibilityTimeout=900s` (heartbeat). Nếu lỗi hoặc 0 dòng → `conn.rollback()` bảo toàn dữ liệu hiện có và ghi `FAILED`.
3. **Task 2 — Astronomer Cosmos (`dbt Core`) & Kimball Star Schema:**
   * Biến đổi 9 bảng `raw_data` → 7 Staging Views (`public_staging`) → 3 Marts Tables (`public_marts`):
     * **`fct_orders`** (`incremental`, `delete+insert`): CTE `affected_order_ids` bắt thay đổi từ **cả 4 bảng upstream** (`stg_orders`, `stg_order_items`, `stg_payments`, `stg_reviews`) kèm 3-day lookback, tính `total_order_cost`, `primary_payment_type`, `is_delayed` và `delivery_status` (`On Time`, `Delayed`, `Pending Delivery`).
     * **`dim_customers`** (`incremental`, `delete+insert`): Bảo toàn độ mịn `customer_id` để join 1:1 với `fct_orders` trong khi tái tổng hợp **Customer Lifetime Value (LTV)** theo `customer_unique_id`.
     * **`dim_products`** (`table`): Chuẩn hóa tên ngành hàng tiếng Anh và kích thước/khối lượng sản phẩm.
   * Thực thi bộ dbt tests tự động (`unique`, `not_null`, `relationships`, `accepted_values`).
4. **Task 3 — `@task archive_and_ack` (Deterministic Archive & Safe ACK):**
   * Gia hạn `VisibilityTimeout` với `fail_on_error=True`, sao chép file sang `archive/<safe_audit_id>/<filename>`, xác minh `check_for_key(dest_key)`, ghi `ARCHIVED → COMPLETED`, xóa file trong `raw/`, và **cuối cùng mới gọi** `DeleteMessage` xóa SQS message (bảo vệ multi-record SQS message và chặn ACK nhầm qua `verify_safe_to_ack_when_raw_missing`).

![Sơ đồ dòng chảy dữ liệu dbt Data Lineage](/fcaj-workshop-template/images/workshop/dbt_lineage.png)

---

### 4. Khởi chạy Airflow 3 & Kích hoạt Pipeline bằng dữ liệu thực tế
```bash
# Khởi động môi trường Apache Airflow 3 (Astro CLI & Docker)
astro dev start

# Mở Airflow UI tại http://localhost:8080, bật (unpause) DAG ecommerce_aws_data_pipeline
# Giữ DAG ecommerce_dbt_full_refresh_maintenance ở trạng thái Paused ($0 FinOps)

# Đồng bộ tệp CSV Olist lên S3 raw/ để kích hoạt luồng Event-Driven
aws s3 sync ./data/ s3://$S3_BUCKET_NAME/raw/ --exclude "*" --include "*.csv"
```

Sau khi pipeline chạy xong, có thể trích xuất ngay **4 tệp CSV phân tích sạch** (~56 MB) vào thư mục `exports/` phục vụ Power BI offline:
```bash
make export-data
```
