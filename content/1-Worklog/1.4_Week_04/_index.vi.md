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

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---: | :--- | :---: | :---: | :--- |
| **Thứ 2** | - Nghiên cứu kiến trúc **Medallion Data Lake** (`Bronze` → `Silver` → `Gold`) trên Amazon S3 <br> - Khởi tạo S3 Bucket phân tầng (`bronze-raw/`, `silver-cleaned/`, `gold-curated/`), bật **Block Public Access** và mã hóa **SSE-KMS** | 19/10/2026 | 19/10/2026 | <https://cloudjourney.awsstudygroup.com/> |
| **Thứ 3** | - Cấu hình **S3 Lifecycle Rules** tự động chuyển dữ liệu tầng Bronze sang `S3 Standard-IA` sau 30 ngày và lưu trữ lạnh `S3 Glacier Flexible Retrieval` sau 90 ngày <br> - Tải tập dữ liệu mẫu định dạng CSV/JSON và Apache Parquet phân vùng (`year=YYYY/month=MM`) lên S3 | 20/10/2026 | 20/10/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html> |
| **Thứ 4** | - Khởi tạo cơ sở dữ liệu trên **AWS Glue Data Catalog** <br> - Cấu hình IAM Role cho Glue và chạy **AWS Glue Crawler** tự động quét S3, suy luận kiểu dữ liệu và nhận diện các cột phân vùng (Partitions) | 21/10/2026 | 21/10/2026 | <https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html> |
| **Thứ 5** | - Cấu hình **Amazon Athena** với S3 Query Result Bucket (kèm Lifecycle tự xóa sau 7 ngày) <br> - Viết các câu lệnh SQL tổng hợp, lọc theo phân vùng trực tiếp trên dữ liệu S3 thông qua Glue Data Catalog | 22/10/2026 | 22/10/2026 | <https://docs.aws.amazon.com/athena/latest/ug/what-is.html> |
| **Thứ 6** | - Đo lường và so sánh hiệu năng truy vấn trên Athena giữa tập dữ liệu CSV thô và **Apache Parquet** đã phân vùng <br> - Rà soát và dọn dẹp các Crawler/Object thử nghiệm | 23/10/2026 | 23/10/2026 | <https://docs.aws.amazon.com/athena/latest/ug/performance-tuning.html> |

### Kết quả đạt được tuần 4:
* Xây dựng hoàn chỉnh nền tảng Data Lake 3 tầng (`Bronze`, `Silver`, `Gold`) bảo mật và tối ưu chi phí vòng đời lưu trữ trên Amazon S3.
* Tự động hóa quy trình khám phá lược đồ (Schema Discovery) và quản lý Metadata tập trung bằng AWS Glue Crawler & Data Catalog.
* Kiểm chứng thực tế lợi ích của định dạng cột **Apache Parquet** và Partitioning trên Amazon Athena: giảm **~85% dung lượng dữ liệu quét (Data Scanned)** và rút ngắn đáng kể thời gian phản hồi truy vấn.
