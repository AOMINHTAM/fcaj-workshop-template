---
title: "Tuần 5: Thiết kế Cloud Data Warehouse với Amazon Redshift Serverless & Nạp dữ liệu từ S3"
weight: 5
---

### 🇻🇳 Báo cáo công việc Tuần 05

**1. Công việc đã thực hiện:**
- **Khảo sát dữ liệu & Chuẩn hóa môi trường phát triển:**
  - Phân tích cấu trúc và mối quan hệ của bộ dữ liệu thực tế **Olist Brazilian E-Commerce Dataset** (~120 MB, gần 100.000 đơn hàng) bao gồm 9 tập dữ liệu quan hệ: `orders`, `order_items`, `order_payments`, `order_reviews`, `customers`, `products`, `sellers`, `geolocation` và `product_category_name_translation`.
  - Thiết lập môi trường phát triển Linux trên **Ubuntu (WSL2)**, cấu hình AWS CLI v2 bảo mật (`chmod 600` cho tệp credentials) và quản lý thông tin nhạy cảm qua tệp `.env` tách biệt khỏi hệ thống quản lý phiên bản Git.
- **Xây dựng S3 Landing Zone & Tự động hóa đồng bộ dữ liệu:**
  - Khởi tạo S3 Bucket lưu trữ dữ liệu thô (`olist-raw-data-amt-us-east-1-...`) tại phân vùng `us-east-1`, kích hoạt toàn bộ chế độ **Block Public Access** để đảm bảo an toàn dữ liệu.
  - Viết kịch bản tự động hóa bằng Bash (`scripts/upload_to_s3.sh`) để kiểm tra sự tồn tại của tệp và đồng bộ hàng loạt 9 tệp CSV nguồn lên đường dẫn `s3://.../raw/`.
- **Triển khai Amazon Redshift Serverless & Tối ưu chi phí (FinOps):**
  - Khởi tạo **Amazon Redshift Serverless** (`ecommerce-workgroup`) với dung lượng tính toán cơ sở tối thiểu **4 RPUs (Redshift Processing Units)** — giúp cắt giảm 50% chi phí so với mức mặc định 8 RPUs và tự động ngắt tính phí khi hệ thống nhàn rỗi (idle).
  - Thiết lập cảnh báo ngân sách **AWS Budgets** (ngưỡng \$5/tháng) để giám sát chặt chẽ chi phí Cloud Data Warehouse.
  - Thực thi bảo mật theo nguyên tắc **Least Privilege**: Khởi tạo IAM Role `RedshiftS3ReadOnlyRole` sử dụng cơ chế ủy quyền tạm thời AWS STS (`AssumeRole`), cho phép Redshift đọc dữ liệu từ S3 mà không cần nhúng cứng (hardcode) Access Key hay Secret Key vào mã nguồn SQL.
  - Cấu hình **VPC Security Group** giới hạn lưu lượng truy cập cổng `5439`, đồng thời chẩn đoán và xử lý sự cố xung đột định tuyến mạng do dịch vụ Cloudflare WARP chặn kết nối tới Redshift Endpoint.
- **Thiết kế lược đồ `raw_data` & Nạp dữ liệu song song (MPP Batch Loading):**
  - Xây dựng kịch bản DDL (`scripts/create_raw_tables.sql`) khởi tạo schema `raw_data` (xử lý lỗi từ khóa bảo lưu `raw` của Redshift) và định nghĩa cấu trúc bảng cho toàn bộ 9 thực thể dữ liệu.
  - Viết và thực thi kịch bản nạp dữ liệu tốc độ cao (`scripts/copy_raw_tables.sql`) sử dụng lệnh **`COPY`** tận dụng kiến trúc xử lý song song hàng loạt (MPP) của Redshift với các tham số `FORMAT AS CSV`, `IGNOREHEADER 1` và `TIMEFORMAT 'auto'`.
  - Kiểm tra đối soát số lượng bản ghi sau khi nạp (**99.441 đơn hàng**, **112.650 chi tiết mục đơn hàng**), đồng thời khắc phục lỗi nhân bản dữ liệu do chạy lệnh `COPY` lặp lại nhiều lần.

**2. Kết quả đạt được:**
- Triển khai thành công hạ tầng Cloud Data Warehouse trên Amazon Redshift Serverless an toàn, bảo mật IAM Role và tối ưu hóa chi phí vận hành với cấu hình 4 RPUs.
- Tự động hóa hoàn toàn khâu tiếp nhận dữ liệu (Data Ingestion) từ môi trường cục bộ lên Amazon S3 và nạp vào tầng `raw_data` của Redshift.
- Rèn luyện kỹ năng gỡ lỗi kết nối mạng (VPC Security Group, Port 5439, VPN/WARP) và đảm bảo tính toàn vẹn dữ liệu (Idempotency) trong quá trình nạp batch.

**3. Tài liệu tham khảo:**
- Amazon Redshift Management Guide: Getting Started with Amazon Redshift Serverless & RPU Capacity.
- Amazon Redshift Database Developer Guide: Loading Data from Amazon S3 using the `COPY` Command.
- AWS IAM Documentation: Authorizing Amazon Redshift to Access Other AWS Services on Your Behalf.
