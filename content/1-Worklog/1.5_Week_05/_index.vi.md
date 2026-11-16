---
title: "Worklog Tuần 5"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:
* Khảo sát bộ dữ liệu thực tế **Olist Brazilian E-Commerce Dataset** (~120 MB, gần 100.000 đơn hàng, 9 bảng quan hệ) và thiết lập môi trường phát triển Linux chuẩn hóa trên **Ubuntu (WSL2)**.
* Xây dựng kịch bản tự động đồng bộ dữ liệu thô lên **Amazon S3 Landing Zone** (`s3://.../raw/`).
* Triển khai **Amazon Redshift Serverless** tối ưu chi phí theo chiến lược **FinOps (4 RPUs)**, phân quyền IAM Role qua AWS STS (`RedshiftS3ReadOnlyRole`) và nạp dữ liệu hàng loạt tốc độ cao bằng lệnh **`COPY` (MPP)**.

### Các công việc đã thực hiện trong tuần:
* **Khảo sát dữ liệu Olist và chuẩn hóa môi trường phát triển:** Khảo sát cấu trúc thực tế của 9 tập dữ liệu Olist E-commerce (`orders`, `order_items`, `order_payments`, `order_reviews`, `customers`, `products`, `sellers`, `geolocation`, `product_category_name_translation`); thiết lập môi trường Ubuntu (WSL2), phân quyền bảo mật AWS CLI v2 (`chmod 600`) và quản lý biến môi trường nhạy cảm qua `.env`.
* **Tự động hóa đồng bộ dữ liệu thô lên S3:** Khởi tạo S3 Bucket riêng tư với chế độ Block Public Access 100%; viết Bash script `scripts/upload_to_s3.sh` tự động rà soát định dạng và đẩy toàn bộ 9 tệp CSV lên vùng đệm `s3://.../raw/`.
* **Khởi tạo Amazon Redshift Serverless và thiết lập FinOps:** Khởi tạo `ecommerce-workgroup` với năng lực tính toán cơ sở **4 RPUs** (giảm 50% chi phí so với mặc định 8 RPUs, tự thu hồi về $0 khi nhàn rỗi) kèm ngân sách cảnh báo **AWS Budgets ($5/tháng)**; tạo IAM Role `RedshiftS3ReadOnlyRole` thông qua AWS STS (`sts:AssumeRole`) tuân thủ nghiêm ngặt nguyên tắc đặc quyền tối thiểu.
* **Xử lý sự cố mạng và khởi tạo lược đồ DDL:** Cấu hình VPC Security Group mở cổng kết nối `5439` cho IP máy trạm; phân tích và xử lý thành công xung đột định tuyến mạng do phần mềm Cloudflare WARP chặn truy cập vào endpoint Redshift; viết kịch bản DDL `scripts/create_raw_tables.sql` khởi tạo schema `raw_data` cho 9 bảng nguồn.
* **Thực thi nạp dữ liệu MPP song song và kiểm soát Idempotency:** Sử dụng lệnh **`COPY`** từ S3 vào Redshift với tham số `IAM_ROLE`, `FORMAT AS CSV`, `IGNOREHEADER 1`, `TIMEFORMAT 'auto'`; đối soát tính toàn vẹn đạt **99.441 đơn hàng** và **112.650 chi tiết đơn hàng**, đồng thời thiết lập cơ chế xóa bảng/nạp lại để đảm bảo tính bất biến (Idempotency) khi nạp batch.

### Kết quả đạt được tuần 5:
* Triển khai thành công hạ tầng **Cloud Data Warehouse** trên Amazon Redshift Serverless an toàn, bảo mật bằng IAM Role (`sts:AssumeRole`) và tối ưu chi phí vận hành với cấu hình **4 RPUs**.
* Tự động hóa hoàn toàn khâu tiếp nhận dữ liệu (**EL - Extract & Load**) từ máy trạm cục bộ lên Amazon S3 Data Lake và nạp song song tốc độ cao vào schema `raw_data` trên Redshift.
* Đúc kết kinh nghiệm thực chiến khi gỡ lỗi kết nối mạng (VPC Security Group, Port `5439`, xung đột Cloudflare WARP) và kiểm soát tính toàn vẹn không trùng lặp dữ liệu (Idempotency) khi nạp batch.

### Tài liệu tham khảo:
* <https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce>
* <https://docs.aws.amazon.com/cli/latest/reference/s3/>
* <https://docs.aws.amazon.com/redshift/latest/mgmt/serverless-whatis.html>
* <https://docs.aws.amazon.com/redshift/latest/dg/r_pg_keywords.html>
* <https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html>
