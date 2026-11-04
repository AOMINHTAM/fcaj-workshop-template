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

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---: | :--- | :---: | :---: | :--- |
| **Thứ 2** | - Khởi tạo dự án **dbt Core (`dbt-redshift`)**, cấu hình kết nối Redshift Serverless bảo mật qua biến môi trường (`profiles.yml` & `.env`) <br> - Thiết kế chiến lược lưu trữ vật lý trong `dbt_project.yml`: tầng **Staging** dưới dạng `VIEW` (tiết kiệm lưu trữ) và tầng **Marts** dưới dạng `TABLE` (tối ưu tốc độ truy vấn BI) | 02/11/2026 | 02/11/2026 | <https://docs.getdbt.com/docs/core/connect-data-platform/redshift-setup> |
| **Thứ 3** | - Khai báo `models/staging/sources.yml` ánh xạ tới schema `raw_data` <br> - Xây dựng **7 mô hình Staging Views** (`stg_orders`, `stg_order_items`, `stg_customers`, `stg_products`, `stg_payments`, `stg_reviews`, `stg_category_translation`): chuẩn hóa tên cột, ép kiểu `CAST`, xử lý `NULL` và sửa lỗi chính tả gốc của Olist (`product_name_lenght` $\rightarrow$ `product_name_length`) | 03/11/2026 | 03/11/2026 | <https://docs.getdbt.com/best-practices/how-we-structure/2-staging> |
| **Thứ 4** | - Phát triển bảng Fact trung tâm **`fct_orders` (`models/marts/fct_orders.sql`)** ở độ mịn đơn hàng: tổng hợp chỉ số tài chính (`total_item_value`, `total_freight_value`, `total_payment_value`), hiệu suất giao vận (`delivery_duration_days`, `delay_days`, cờ giao trễ `is_delayed`) và điểm đánh giá (`review_score`) | 04/11/2026 | 04/11/2026 | *The Data Warehouse Toolkit (Kimball)* |
| **Thứ 5** | - Phát triển 2 bảng chiều (Dimensions): <br>&emsp; + **`dim_customers`:** Xử lý định danh `customer_unique_id`, tính **Customer Lifetime Value (`lifetime_spend`)**, tổng số đơn và phân khúc tự động (`Returning Customer` vs. `One-Time Customer`) <br>&emsp; + **`dim_products`:** Kết hợp bảng dịch chuyển đổi tên ngành hàng từ tiếng Bồ Đào Nha sang tiếng Anh chuẩn hóa (`category_name`) | 05/11/2026 | 05/11/2026 | <https://docs.getdbt.com/best-practices/how-we-structure/4-marts> |
| **Thứ 6** | - Định nghĩa **9 bài kiểm thử chất lượng dữ liệu tự động** trong `models/marts/schema.yml` (`unique`, `not_null`, và khóa ngoại `relationships` giữa `fct_orders.customer_id` $\rightarrow$ `dim_customers.customer_id`) <br> - Chạy `dbt run` và `dbt test` đạt kết quả **`PASS=9 WARN=0 ERROR=0`** và trích xuất đồ thị **dbt Data Lineage Graph** | 06/11/2026 | 06/11/2026 | <https://docs.getdbt.com/docs/build/data-tests> |

### Kết quả đạt được tuần 6:
* Chuyển đổi thành công 9 bảng dữ liệu thô rời rạc thành mô hình kho dữ liệu **Kimball Star Schema** (`fct_orders`, `dim_customers`, `dim_products`) trên Amazon Redshift Serverless.
* Làm chủ phương pháp viết SQL dạng module hóa (CTE + Jinja `{{ ref() }}`), tự động quản lý sự phụ thuộc giữa các bảng và sinh sơ đồ dòng chảy dữ liệu (**Data Lineage**).
* Đảm bảo **100% chất lượng và tính toàn vẹn tham chiếu** của kho dữ liệu phân tích thông qua 9/9 bài kiểm thử tự động trên dbt Core.
