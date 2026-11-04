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

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---: | :--- | :---: | :---: | :--- |
| **Thứ 2** | - Khảo sát cấu trúc và mối quan hệ của bộ dữ liệu **Olist E-Commerce** (9 tập dữ liệu CSV: `orders`, `order_items`, `order_payments`, `order_reviews`, `customers`, `products`, `sellers`, `geolocation`, `product_category_name_translation`) <br> - Cấu hình môi trường **Ubuntu (WSL2)**, bảo mật AWS CLI v2 (`chmod 600`) và tách biệt thông tin nhạy cảm qua `.env` | 26/10/2026 | 26/10/2026 | <https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce> |
| **Thứ 3** | - Khởi tạo S3 Bucket riêng tư (`olist-raw-data-amt-us-east-1-...`) với chế độ **Block Public Access 100%** <br> - Viết Bash script `scripts/upload_to_s3.sh` tự động kiểm tra và đồng bộ toàn bộ 9 tệp CSV lên `s3://.../raw/` | 27/10/2026 | 27/10/2026 | <https://docs.aws.amazon.com/cli/latest/reference/s3/> |
| **Thứ 4** | - Khởi tạo **Amazon Redshift Serverless** (`ecommerce-workgroup`) với mức năng lực tính toán cơ sở tối thiểu **4 RPUs** (tiết kiệm 50% chi phí so với mặc định 8 RPUs, tự thu hồi về \$0 khi idle) và cài đặt **AWS Budget Alert (\$5/tháng)** <br> - Tạo IAM Role `RedshiftS3ReadOnlyRole` (AWS STS `AssumeRole`, tuân thủ Least Privilege, không hardcode Access Key) | 28/10/2026 | 28/10/2026 | <https://docs.aws.amazon.com/redshift/latest/mgmt/serverless-whatis.html> |
| **Thứ 5** | - Cấu hình **VPC Security Group** mở cổng `5439` cho IP máy trạm và xử lý sự cố xung đột định tuyến mạng do phần mềm Cloudflare WARP chặn kết nối tới Redshift <br> - Viết kịch bản DDL `scripts/create_raw_tables.sql` khởi tạo schema `raw_data` (xử lý lỗi từ khóa bảo lưu `raw` của Redshift) cho 9 bảng nguồn | 29/10/2026 | 29/10/2026 | <https://docs.aws.amazon.com/redshift/latest/dg/r_pg_keywords.html> |
| **Thứ 6** | - Thực thi kịch bản `scripts/copy_raw_tables.sql` nạp dữ liệu song song (MPP) từ S3 vào Redshift bằng lệnh **`COPY`** (`IAM_ROLE`, `FORMAT AS CSV`, `IGNOREHEADER 1`, `TIMEFORMAT 'auto'`) <br> - Đối soát số lượng bản ghi (**99.441 đơn hàng**, **112.650 chi tiết đơn hàng**) và xử lý lỗi nhân bản dữ liệu khi chạy `COPY` nhiều lần (đảm bảo tính Idempotency) | 30/10/2026 | 30/10/2026 | <https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html> |

### Kết quả đạt được tuần 5:
* Triển khai thành công hạ tầng **Cloud Data Warehouse** trên Amazon Redshift Serverless an toàn, bảo mật bằng IAM Role (`sts:AssumeRole`) và tối ưu chi phí vận hành với cấu hình **4 RPUs**.
* Tự động hóa hoàn toàn khâu tiếp nhận dữ liệu (**EL - Extract & Load**) từ máy trạm cục bộ lên Amazon S3 Data Lake và nạp song song tốc độ cao vào schema `raw_data` trên Redshift.
* Đúc kết kinh nghiệm thực chiến khi gỡ lỗi kết nối mạng (VPC Security Group, Port `5439`, xung đột Cloudflare WARP) và kiểm soát tính toàn vẹn không trùng lặp dữ liệu (Idempotency) khi nạp batch.
