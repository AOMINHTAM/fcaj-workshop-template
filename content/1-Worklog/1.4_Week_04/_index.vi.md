---
title: "Tuần 4: Kiến trúc Data Lake với Amazon S3 & Phân tích Serverless với Athena"
weight: 4
---

### 🇻🇳 Báo cáo công việc Tuần 04

**1. Công việc đã thực hiện:**
- **Xây dựng nền tảng Data Lake trên Amazon S3:**
  - Thiết kế cấu trúc lưu trữ phân tầng theo mô hình Medallion Architecture:
    - `bronze-raw/`: Lưu trữ dữ liệu thô ban đầu (định dạng thô JSON / CSV).
    - `silver-cleaned/`: Dữ liệu đã qua làm sạch, chuẩn hóa kiểu dữ liệu.
    - `gold-curated/`: Dữ liệu tổng hợp chất lượng cao, định dạng nén cột (Apache Parquet) sẵn sàng cho truy vấn phân tích.
  - Thiết lập hàng rào bảo mật: Bật toàn bộ **Block Public Access**, thiết lập Bucket Policy chống truy cập trái phép, kích hoạt mã hóa mặc định Server-Side Encryption (**SSE-KMS**).
  - Cấu hình **S3 Lifecycle Rules** nhằm tối ưu chi phí (tự động chuyển dữ liệu tầng Bronze sang S3 Standard-IA sau 30 ngày và lưu trữ dài hạn vào S3 Glacier Flexible Retrieval sau 90 ngày).
- **Quản lý dữ liệu đặc tả (Metadata) bằng AWS Glue:**
  - Thiết lập cơ sở dữ liệu trên **AWS Glue Data Catalog** để lưu trữ thống nhất thông tin lược đồ dữ liệu (schema).
  - Cấu hình và chạy **AWS Glue Crawler** để tự động quét cấu trúc các tập dữ liệu lưu trên S3, tự động suy luận kiểu dữ liệu, các trường phân vùng (`year=YYYY/month=MM`) và đồng bộ metadata vào Data Catalog.
- **Truy vấn phân tích Serverless với Amazon Athena:**
  - Cấu hình vị trí lưu trữ kết quả truy vấn Athena trên S3 kèm chính sách tự hủy sau 7 ngày.
  - Thực thi các câu lệnh SQL phân tích trực tiếp trên dữ liệu S3 mà không cần khởi chạy máy chủ cơ sở dữ liệu.
  - Đánh giá hiệu năng và chi phí: so sánh dung lượng dữ liệu quét qua (Scanned Data) giữa file CSV thô và định dạng nén cột Apache Parquet phân vùng, ghi nhận giảm ~85% dung lượng quét và tốc độ phản hồi nhanh hơn đáng kể.

**2. Kết quả đạt được:**
- Xây dựng hoàn chỉnh kiến trúc Data Lake 3 tầng an toàn và tối ưu chi phí trên Amazon S3.
- Tự động hóa quy trình trích xuất và cập nhật schema bằng AWS Glue Crawler.
- Thành thạo phương pháp phân tích dữ liệu lớn theo mô hình Serverless bằng Amazon Athena.

**3. Tài liệu tham khảo:**
- FCAJ Lab Guide: Building a Serverless Data Lake with Amazon S3, AWS Glue & Amazon Athena.
- AWS Prescriptive Guidance: Data Lake Storage Best Practices on Amazon S3.
- Amazon Athena User Guide: Performance Tuning and Partitioning.
