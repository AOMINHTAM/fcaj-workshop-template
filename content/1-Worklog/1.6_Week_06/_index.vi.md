---
title: "Worklog Tuần 6"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:
* Thiết lập kiến trúc biến đổi dữ liệu hiện đại (**Analytics Engineering**) với **dbt Core (`dbt-redshift`)** kết nối trực tiếp tới Amazon Redshift Serverless.
* Xây dựng tầng **Staging (7 Views)** làm sạch, chuẩn hóa kiểu dữ liệu và tầng **Marts (3 Physical Tables)** theo chuẩn **Kimball Star Schema** (`fct_orders`, `dim_customers`, `dim_products`).
* Triển khai hệ thống kiểm định chất lượng dữ liệu tự động (**9 bài dbt tests**) đảm bảo tính duy nhất, không rỗng và toàn vẹn tham chiếu khóa ngoại.

### Các công việc đã thực hiện trong tuần:
* **Khởi tạo dự án dbt Core và thiết lập chiến lược Materialization:** Cấu hình kết nối `dbt-redshift` qua biến môi trường bảo mật (`profiles.yml` và `.env`); thiết lập chiến lược lưu trữ trong `dbt_project.yml` gồm tầng **Staging** dưới dạng `VIEW` (tiết kiệm lưu trữ) và tầng **Marts** dưới dạng `TABLE` vật lý (tối ưu tốc độ truy vấn cho BI).
* **Xây dựng 7 mô hình Staging Views:** Khai báo nguồn dữ liệu trong `models/staging/sources.yml` ánh xạ tới schema `raw_data`; phát triển 7 Staging Views (`stg_orders`, `stg_order_items`, `stg_customers`, `stg_products`, `stg_payments`, `stg_reviews`, `stg_category_translation`) để chuẩn hóa tên cột, ép kiểu dữ liệu `CAST`, xử lý giá trị `NULL` và sửa lỗi chính tả gốc của Olist (`product_name_lenght` → `product_name_length`).
* **Phát triển bảng Fact trung tâm (`fct_orders`):** Xây dựng mô hình `fct_orders.sql` ở độ mịn từng đơn hàng, tổng hợp các chỉ số doanh thu (`total_item_value`, `total_freight_value`, `total_payment_value`), các chỉ số giao vận (`delivery_duration_days`, `delay_days`, cờ cảnh báo `is_delayed`) và điểm đánh giá của khách hàng (`review_score`).
* **Phát triển 2 bảng Dimension chuẩn Kimball:**
  * Bảng **`dim_customers`:** Xử lý định danh duy nhất `customer_unique_id`, tính toán tổng giá trị chi tiêu trọn đời (**Customer Lifetime Value - LTV**), tổng số lượng đơn hàng và tự động phân khúc khách hàng (`Returning Customer` vs. `One-Time Customer`).
  * Bảng **`dim_products`:** Kết hợp bảng dịch để chuẩn hóa tên ngành hàng sản phẩm từ tiếng Bồ Đào Nha sang tiếng Anh (`category_name`).
* **Thiết lập kiểm thử chất lượng dữ liệu tự động (dbt Tests):** Khai báo 9 bài test tự động trong `models/marts/schema.yml` kiểm tra tính duy nhất (`unique`), không rỗng (`not_null`), và toàn vẹn tham chiếu khóa ngoại (`relationships` giữa `fct_orders.customer_id` và `dim_customers.customer_id`); thực thi `dbt run` và `dbt test` đạt kết quả tuyệt đối **`PASS=9 WARN=0 ERROR=0`** và kết xuất sơ đồ dòng chảy dữ liệu (**dbt Data Lineage**).

### Kết quả đạt được tuần 6:
* Chuyển đổi thành công 9 bảng dữ liệu thô rời rạc thành mô hình kho dữ liệu **Kimball Star Schema** (`fct_orders`, `dim_customers`, `dim_products`) trên Amazon Redshift Serverless.
* Làm chủ phương pháp viết SQL dạng module hóa (CTE + Jinja `{{ ref() }}`), tự động quản lý sự phụ thuộc giữa các bảng và sinh sơ đồ dòng chảy dữ liệu (**Data Lineage**).
* Đảm bảo **100% chất lượng và tính toàn vẹn tham chiếu** của kho dữ liệu phân tích thông qua 9/9 bài kiểm thử tự động trên dbt Core.

### Tài liệu tham khảo:
* <https://docs.getdbt.com/docs/core/connect-data-platform/redshift-setup>
* <https://docs.getdbt.com/best-practices/how-we-structure/2-staging>
* <https://docs.getdbt.com/best-practices/how-we-structure/4-marts>
* <https://docs.getdbt.com/docs/build/data-tests>
