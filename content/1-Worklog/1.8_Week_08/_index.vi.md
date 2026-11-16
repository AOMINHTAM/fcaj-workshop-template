---
title: "Worklog Tuần 8"
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:
* Tối ưu hóa hiệu năng chuyển đổi dữ liệu bằng mô hình hóa gia tăng (**Incremental Modeling**) với **dbt Core (`dbt-redshift`)**, áp dụng chiến lược `delete+insert` để tiết kiệm tối đa tài nguyên tính toán **Amazon Redshift Serverless**.
* Xử lý bài toán dữ liệu đến trễ (**Late-arriving data**) bằng cửa sổ nhìn lại 3 ngày (**3-day lookback window**) trên bảng Fact (`fct_orders`), đồng thời chuẩn hóa grain ở cấp `customer_unique_id` trên bảng Dimension (`dim_customers`) để tính toán chính xác giá trị vòng đời khách hàng (**LTV**).
* Chuẩn hóa chất lượng mã nguồn theo quy chuẩn kỹ nghệ phần mềm: Phát triển bộ kiểm thử đơn vị (**Unit Tests**) bằng **Pytest**, tự động hóa quy trình kiểm thử liên tục (**CI/CD**) với **GitHub Actions**, và kết xuất 4 tập dữ liệu sạch bàn giao cho giai đoạn **Data Analytics (DA)**.

### Các công việc đã thực hiện trong tuần:
* **Nâng cấp dbt Incremental Modeling cho bảng Fact:** Cập nhật DDL bổ sung trường siêu dữ liệu `_ingested_at TIMESTAMP DEFAULT SYSDATE` trên toàn bộ 7 bảng Staging tại Redshift Serverless; tái cấu trúc mô hình `fct_orders.sql` chuyển sang `materialized='incremental'` với `unique_key='order_id'`, cấu hình chiến lược `delete+insert` kết hợp CTE `affected_order_ids` từ 4 bảng nguồn (`stg_orders`, `stg_order_items`, `stg_payments`, `stg_reviews`) kèm cửa sổ nhìn lại 3 ngày (`3-day lookback window`).
* **Chuẩn hóa Grain Dimension và DAG đối soát định kỳ:** Tái cấu trúc mô hình `dim_customers.sql` chuyển sang `materialized='incremental'` với `unique_key='customer_id'` và `delete+insert`; phân rã và tái tổng hợp tại mức định danh duy nhất của khách hàng (`customer_unique_id`), đảm bảo chỉ số tổng chi tiêu trọn đời (**Lifetime Value - LTV**) và phân khúc khách hàng (`Returning` vs `One-Time`) được cập nhật chính xác; xây dựng DAG đối soát định kỳ `dags/ecommerce_weekly_full_refresh_dag.py` (chế độ tạm dừng mặc định) nhằm xử lý triệt để sai lệch dữ liệu (data drift).
* **Phát triển bộ kiểm thử đơn vị với Pytest:** Xây dựng 7 kịch bản kiểm thử độc lập cho module phân tích tin nhắn SQS (`tests/test_sqs_parser.py`) bao gồm: parse tệp CSV đơn lẻ, xử lý nhiều bản ghi (multiple records), bỏ qua tệp không phải CSV, giải mã ký tự URL-encoded trên S3 key, và xử lý payload SQS bị lỗi định dạng; đồng thời viết các ca kiểm thử cho module bảo vệ nạp dữ liệu (`tests/test_ingestion_guards.py`) xác thực hoạt động của hàm `ensure_non_empty_stage`.
* **Thiết lập CI/CD Pipeline và kết xuất dữ liệu phân tích:** Thiết lập luồng tự động hóa kiểm thử tích hợp liên tục (**CI Pipeline**) bằng **GitHub Actions** (`.github/workflows/ci.yml`), tự động kích hoạt kiểm tra cú pháp (Flake8), kiểm thử đơn vị (Pytest) và kiểm tra cú pháp SQL dbt (`dbt parse`) trên mỗi pull request và commit lên branch `main`; xây dựng module Python kết xuất dữ liệu `scripts/export_to_csv.py` kết nối bảo mật SSL tới Redshift Serverless, trích xuất 4 tập dữ liệu phân tích sạch (~54 MB, 344.483 dòng) vào thư mục `exports/`.
* **Nghiệm thu toàn trình và bàn giao Data Analytics:** Vận hành kiểm thử toàn diện end-to-end: Xác nhận 100% Unit Tests (7/7 tests passed), 100% dbt Tests (9/9 tests passed), và GitHub Actions CI workflow báo xanh (All-Green); hoàn tất báo cáo nghiệm thu kỹ thuật phần Data Engineering, bàn giao các tập dữ liệu sạch cho bộ phận Data Analytics (DA) và chuẩn bị kế hoạch phân tích chuyên sâu cho Tuần 9.

### Kết quả đạt được tuần 8:
* Tối ưu hóa hiệu năng chuyển đổi dữ liệu vượt bậc với dbt Incremental Modeling (`delete+insert`), rút ngắn thời gian xử lý và giảm thiểu chi phí tính toán Redshift Serverless so với cơ chế Full Refresh cũ.
* Giải quyết triệt để 2 bài toán lớn trong xử lý dữ liệu: Xử lý dữ liệu cập nhật muộn (Late-arriving data) qua CTE đa bảng 3-day lookback, và bảo toàn độ chính xác chỉ số LTV khách hàng qua `customer_unique_id`.
* Nâng cao tiêu chuẩn kỹ thuật phần mềm cho Data Pipeline với 7 kịch bản Pytest, Ingestion Guards và pipeline CI/CD tự động bằng GitHub Actions đạt 100% All-Green.
* Trích xuất thành công 4 bộ dữ liệu sạch (~54 MB, 344.483 dòng), hoàn thành trọn vẹn hạ tầng Data Engineering và chuyển giao sang giai đoạn Data Analytics (Tuần 9).

### Tài liệu tham khảo:
* <https://docs.getdbt.com/docs/build/incremental-models>
* <https://docs.getdbt.com/terms/kimball-dimensional-modeling>
* <https://docs.pytest.org/en/stable/>
* <https://docs.github.com/en/actions>
