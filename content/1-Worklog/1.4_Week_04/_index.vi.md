---
title: "Worklog Tuần 4"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:
* Thiết kế kiến trúc **Data Lake đa tầng (Medallion Architecture)** trên **Amazon S3** kết hợp mã hóa **SSE-KMS** và tối ưu chi phí lưu trữ bằng **S3 Lifecycle Rules**.
* Tự động hóa thu thập và quản lý dữ liệu đặc tả (Metadata/Schema) với **AWS Glue Data Catalog** và **AWS Glue Crawler**.
* Thực thi truy vấn phân tích dữ liệu lớn không máy chủ (Serverless SQL) bằng **Amazon Athena** và đánh giá hiệu năng định dạng nén cột **Apache Parquet**.

### Các công việc đã thực hiện trong tuần:
* **Xây dựng Medallion Data Lake trên Amazon S3:** Khởi tạo cấu trúc S3 Bucket phân tầng theo chuẩn Medallion gồm `bronze-raw/` (dữ liệu thô), `silver-cleaned/` (dữ liệu làm sạch) và `gold-curated/` (dữ liệu tổng hợp phân tích); kích hoạt **Block Public Access 100%** và mã hóa lưu trữ an toàn bằng **SSE-KMS**.
* **Thiết lập vòng đời dữ liệu (S3 Lifecycle Management):** Cấu hình tự động chuyển đổi lớp lưu trữ của dữ liệu tầng Bronze sang `S3 Standard-IA` sau 30 ngày và lưu trữ lạnh `S3 Glacier Flexible Retrieval` sau 90 ngày; tải các tập dữ liệu mẫu dạng CSV/JSON và Apache Parquet có phân vùng (`year=YYYY/month=MM`) lên S3.
* **Tự động hóa Schema Discovery với AWS Glue:** Tạo cơ sở dữ liệu trên **AWS Glue Data Catalog**, phân quyền IAM Role cho Glue Service và vận hành **AWS Glue Crawler** để tự động quét cấu trúc S3, suy luận kiểu dữ liệu và ghi nhận các cột phân vùng (Partitions).
* **Phân tích dữ liệu Serverless với Amazon Athena:** Cấu hình Athena với S3 Query Result Bucket chuyên biệt; thực thi các truy vấn SQL phân tích, tính toán tổng hợp trực tiếp trên dữ liệu S3 thông qua Metadata của Glue Data Catalog.
* **Đo lường và tối ưu hiệu năng định dạng cột Parquet:** So sánh hiệu năng giữa CSV thô và **Apache Parquet** đã phân vùng, ghi nhận mức giảm vượt trội lên tới **~85% dung lượng dữ liệu quét (Data Scanned)** và cải thiện rõ rệt tốc độ truy vấn; hoàn tất dọn dẹp các tài nguyên thử nghiệm.

### Kết quả đạt được tuần 4:
* Xây dựng hoàn chỉnh nền tảng Data Lake 3 tầng (`Bronze`, `Silver`, `Gold`) bảo mật và tối ưu chi phí vòng đời lưu trữ trên Amazon S3.
* Tự động hóa quy trình khám phá lược đồ (Schema Discovery) và quản lý Metadata tập trung bằng AWS Glue Crawler & Data Catalog.
* Kiểm chứng thực tế lợi ích của định dạng cột **Apache Parquet** và Partitioning trên Amazon Athena: giảm **~85% dung lượng dữ liệu quét (Data Scanned)** và rút ngắn đáng kể thời gian phản hồi truy vấn.

### Tài liệu tham khảo:
* <https://cloudjourney.awsstudygroup.com/>
* <https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html>
* <https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html>
* <https://docs.aws.amazon.com/athena/latest/ug/what-is.html>
* <https://docs.aws.amazon.com/athena/latest/ug/performance-tuning.html>
