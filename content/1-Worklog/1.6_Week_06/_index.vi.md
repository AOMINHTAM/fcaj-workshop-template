---
title: "Tuần 6: Mô hình hóa dữ liệu Kimball Star Schema & Kiểm định chất lượng với dbt Core"
weight: 6
---

### 🇻🇳 Báo cáo công việc Tuần 06

**1. Công việc đã thực hiện:**
- **Thiết lập kiến trúc biến đổi dữ liệu với dbt Core (`dbt-redshift`):**
  - Khởi tạo dự án **dbt Core** kết nối trực tiếp tới cụm **Amazon Redshift Serverless** thông qua `profiles.yml` sử dụng biến môi trường bảo mật (`DBT_REDSHIFT_PASSWORD`).
  - Cấu hình `dbt_project.yml` phân tách rõ ràng chiến lược lưu trữ vật lý (materialization) theo kiến trúc đa tầng: tầng **Staging** khởi tạo dưới dạng `VIEW` (tiết kiệm chi phí lưu trữ) và tầng **Marts** khởi tạo dưới dạng `TABLE` vật lý (tối ưu tốc độ truy vấn phân tích BI).
- **Xây dựng Tầng Staging (`public_staging` - 7 Views):**
  - Định nghĩa tệp đặc tả nguồn `models/staging/sources.yml` liên kết trực tiếp tới 9 bảng dữ liệu trong schema `raw_data`.
  - Phát triển 7 mô hình làm sạch và chuẩn hóa dữ liệu (`stg_orders`, `stg_order_items`, `stg_customers`, `stg_products`, `stg_payments`, `stg_reviews`, `stg_category_translation`):
    - Chuẩn hóa tên cột đồng nhất, ép kiểu dữ liệu (`CAST`) về đúng định dạng số thực/thời gian và xử lý các giá trị `NULL`.
    - Khắc phục các lỗi chính tả gốc trong bộ dữ liệu Olist (như sửa `product_name_lenght` $\rightarrow$ `product_name_length`, `product_description_lenght` $\rightarrow$ `product_description_length`) và tính toán sẵn giá trị mục hàng (`total_item_value = price + freight_value`).
- **Xây dựng Tầng Marts theo chuẩn Kimball Star Schema (`public_marts` - 3 Physical Tables):**
  - **`fct_orders` (`models/marts/fct_orders.sql`):** Thiết kế bảng Fact trung tâm ở độ mịn từng đơn hàng (order granularity), kết hợp:
    - *Chỉ số tài chính:* Tổng số lượng mặt hàng (`total_items`), tổng tiền hàng (`total_item_value`), tổng phí vận chuyển (`total_freight_value`), tổng giá trị thanh toán thực tế (`total_payment_value`) và phương thức thanh toán chính (`primary_payment_type`).
    - *Hiệu suất giao vận:* Tính toán số ngày giao hàng thực tế (`delivery_duration_days`), số ngày giao dự kiến (`estimated_delivery_duration_days`), độ trễ (`delay_days`) và gắn cờ phân loại giao trễ (`is_delayed = 1` khi ngày giao thực tế vượt quá ngày dự kiến).
    - *Chỉ số hài lòng khách hàng:* Điểm đánh giá trung bình của đơn hàng (`review_score`).
  - **`dim_customers` (`models/marts/dim_customers.sql`):** Thiết kế bảng chiều Khách hàng, xử lý bài toán phân biệt giữa mã phiên đơn hàng (`customer_id`) và định danh người dùng duy nhất (`customer_unique_id`). Tổng hợp ngày mua đầu tiên (`first_order_date`), ngày mua gần nhất (`most_recent_order_date`), tổng số đơn (`total_orders`), tính toán **Giá trị vòng đời khách hàng (Customer Lifetime Value - `lifetime_spend`)** và tự động phân khúc khách hàng (`Returning Customer` nếu `total_orders > 1` so với `One-Time Customer`).
  - **`dim_products` (`models/marts/dim_products.sql`):** Thiết kế bảng chiều Sản phẩm, thực hiện `LEFT JOIN` với bảng dịch danh mục để chuyển đổi tên ngành hàng từ tiếng Bồ Đào Nha sang tiếng Anh chuẩn hóa (`category_name`, mặc định `'unknown'` nếu khuyết dữ liệu) kèm đầy đủ thông số kích thước và trọng lượng.
- **Kiểm định chất lượng dữ liệu tự động (Automated Data Quality Testing):**
  - Thiết lập 9 bài kiểm thử chất lượng dữ liệu tự động trong `models/marts/schema.yml`:
    - Kiểm tra tính duy nhất (`unique`) và không rỗng (`not_null`) cho khóa chính của cả 3 bảng Marts (`order_id`, `customer_id`, `product_id`) cùng định danh `customer_unique_id`.
    - Kiểm tra toàn vẹn tham chiếu khóa ngoại (`relationships`) đảm bảo mọi `customer_id` trong `fct_orders` đều tồn tại hợp lệ trong `dim_customers`.
  - Thực thi `dbt run` và `dbt test` thành công tuyệt đối (**9/9 Tests Passed: `PASS=9 WARN=0 ERROR=0`**) và sinh tài liệu đồ thị dòng chảy dữ liệu (**dbt Data Lineage Graph**).

**2. Kết quả đạt được:**
- Chuyển đổi thành công dữ liệu thô phân mảnh thành mô hình **Kimball Star Schema** (`fct_orders`, `dim_customers`, `dim_products`) sẵn sàng cho các bài toán phân tích chuyên sâu.
- Áp dụng tư duy **Analytics Engineering** hiện đại: viết SQL theo hướng module hóa (CTE + Jinja `{{ ref() }}`), quản lý phiên bản bằng Git và tự động hóa kiểm thử chất lượng dữ liệu.
- Đảm bảo 100% độ tin cậy và tính toàn vẹn tham chiếu của kho dữ liệu thông qua 9 bài test tự động trên dbt Core.

**3. Tài liệu tham khảo:**
- Kimball, R., & Ross, M.: *The Data Warehouse Toolkit: The Definitive Guide to Dimensional Modeling*.
- dbt Labs Documentation: Best Practices for Structuring dbt Projects (Staging vs. Marts) & Data Tests.
- dbt-redshift Adapter Reference: Materializations and Redshift Serverless Configuration.
