---
title: "Bài viết 2: Chuyển đổi Pipeline sang Event-Driven (S3 → SQS → Airflow 3) & dbt Incremental"
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

- **Chủ đề:** Nâng cấp AWS Data Pipeline từ Scheduled Polling sang **Pure Event-Driven (`AssetWatcher`)**, **Atomic `TEMP TABLE` Swap** và **dbt Incremental (`delete+insert`)**
- **Nhóm cộng đồng dự kiến đăng:** AWS Study Group VN (First Cloud AI Journey)
- **Trạng thái:** ⏳ Bản nháp hoàn chỉnh — chờ đăng lên AWS Study Group VN & gắn permalink chính thức
- **Link bài viết chính thức (Facebook):** *(Chèn permalink sau khi đăng lên AWS Study Group VN)*
- **Mã nguồn dự án (GitHub):** <https://github.com/AOMINHTAM/aws-data-pipeline>

---

### Giới thiệu

Ở bài chia sẻ đầu tiên khi chuyển pipeline từ Snowflake sang AWS S3 & Redshift Serverless, hạn chế lớn nhất của hệ thống là dữ liệu vẫn nạp thủ công và phải dùng lịch quét định kỳ. Trong bản nâng cấp **V4 (`v4.0.1`)**, mình đã giải quyết triệt để 3 bài toán kinh điển của Data Engineering trên Cloud: **loại bỏ cron polling lãng phí**, **chống mất dữ liệu khi `COPY` gặp file rỗng**, và **tối ưu chi phí chuyển đổi bằng dbt Incremental**.

---

### 1. Từ Polling (`*/5`) sang Pure Event-Driven với S3 → SQS → Airflow 3 `AssetWatcher`

* **Vấn đề của Polling:** Dùng `S3KeySensor` hoặc lịch chạy mỗi 5–15 phút sinh ra hàng trăm DAG Run `SKIPPED` vô ích khi không có file mới, tăng chi phí API `LIST` trên S3 và dễ đánh thức compute không cần thiết.
* **Giải pháp Push-based Event-Driven:**
  * Cấu hình **Amazon S3 Event Notifications** (`s3:ObjectCreated:*`, bộ lọc `prefix: raw/`, `suffix: .csv`) đẩy thẳng sự kiện sang **Amazon SQS (`olist-s3-raw-arrival-queue`)** qua Resource-based Queue Policy khóa theo `aws:SourceArn` và `aws:SourceAccount`.
  * Nâng cấp Airflow lên **Airflow 3 `Asset` + `AssetWatcher`** dùng `SqsSensorTrigger` (`Long Polling 20s`, `delete_message_on_reception=False`, `VisibilityTimeout=900s`, `schedule=None`).
  * **Kết quả FinOps:** Khi queue trống, Airflow tạo **`0` DAG Run** và Redshift Serverless ngủ ở mức **`$0` compute**; khi có file mới lên S3, pipeline tự thức dậy xử lý trong vòng chưa đầy 20 giây.

---

### 2. Vì sao không nên dùng `TRUNCATE + COPY` trên Amazon Redshift?

* Trong Redshift, lệnh `TRUNCATE` tự động commit ngay lập tức và **không thể `ROLLBACK`**. Nếu file CSV mới upload lên S3 bị rỗng (`0` dòng) hoặc lỗi cột, bảng dữ liệu sản xuất sẽ bị xóa trắng!
* **Cách xử lý trong V4 (Atomic `TEMP TABLE` Swap + Empty-Stage Guard):**
  1. Nạp lệnh `COPY` vào bảng tạm `TEMP TABLE` trước (chưa đụng tới bảng chính).
  2. Chạy chốt chặn `ensure_non_empty_stage` (`include/ingestion_guards.py`) xác nhận `staged_rows > 0`.
  3. Mở giao dịch `autocommit=False`, chạy `DELETE + INSERT` vào bảng đích rồi mới `commit()`. Nếu bất kỳ bước nào lỗi → `conn.rollback()` giữ nguyên 100% dữ liệu cũ.
  4. Chỉ thực hiện lưu trữ file sang `archive/` và gọi `DeleteMessage` (SQS ACK) sau khi toàn bộ dbt tests đạt.

---

### 3. Hai bài toán khó khi nâng cấp `dbt Core` sang Incremental (`delete+insert`)

1. **Dữ liệu cập nhật muộn (Late-arriving data) trên `fct_orders`:** Đơn hàng có thể đã mua từ tuần trước nhưng hôm nay mới giao xong hoặc mới có đánh giá (`review_score`). Mình xây dựng CTE `affected_order_ids` bắt thay đổi từ **cả 4 bảng nguồn** (`stg_orders`, `stg_order_items`, `stg_payments`, `stg_reviews`) kèm **3-day lookback window**, áp dụng `incremental_strategy='delete+insert'` trên Redshift.
2. **Bảo toàn Customer Lifetime Value (LTV) trên `dim_customers`:** Olist cấp một `customer_id` mới cho mỗi đơn hàng dù cùng một người mua (`customer_unique_id`). Model `dim_customers` giữ grain `customer_id` để join `1:1` chuẩn Star Schema với `fct_orders`, đồng thời gom lại toàn bộ lịch sử theo `customer_unique_id` cho những khách bị ảnh hưởng để tổng chi tiêu LTV và nhãn `Returning Customer` luôn chính xác tuyệt đối.

---

### 4. Kết quả đạt được & Bài học rút ra
* Đạt `PASS 100%` trên toàn bộ bài test dbt (`unique`, `not_null`, `relationships`, `accepted_values`) và tự động hóa kiểm thử CI trên GitHub Actions với `$0` chi phí AWS.
* Xuất thành công **4 bộ CSV phân tích sạch** (~54–56 MB, 344.483 dòng) tách rời lớp BI khỏi Redshift Serverless để chạy Power BI không tốn RPU.
* **Hẹn gặp ở bài viết tiếp theo (V5):** Đưa toàn bộ hạ tầng lên **100% AWS CloudFormation IaC**, cô lập gói tin độc bằng **SQS DLQ**, giám sát chủ động với **4 CloudWatch Alarms + SNS Email** và sổ cái kiểm toán **`ops.pipeline_ingestion_audit`**.

* **GitHub Repository:** <https://github.com/AOMINHTAM/aws-data-pipeline>
