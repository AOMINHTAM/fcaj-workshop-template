---
title: "Worklog Tuần 4"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:
* Tìm hiểu kiến trúc lưu trữ đối tượng trên **Amazon S3** (Buckets, Objects, Prefixes, Regions và quy tắc đặt tên duy nhất toàn cầu).
* Thực hành khởi tạo và bảo mật **Amazon S3 Bucket**: bật **Block Public Access 100%**, cấu hình mã hóa dữ liệu (**SSE-S3 / SSE-KMS**) và thử nghiệm tổ chức phân tầng thư mục dữ liệu trên S3.
* Tìm hiểu các lớp lưu trữ (**S3 Storage Classes**) và thiết lập quy tắc vòng đời (**S3 Lifecycle Rules**) cơ bản kết hợp dọn dẹp tài nguyên sau khi thử nghiệm.

### Các công việc đã thực hiện trong tuần:
* **Khảo sát & tìm hiểu lý thuyết Amazon S3:** Nghiên cứu cách hoạt động của Object Storage trên Amazon S3, phân biệt các lớp lưu trữ (`S3 Standard`, `S3 Standard-IA`, `S3 Glacier`), cơ chế bảo mật truy cập (Block Public Access, Bucket Policy) và mã hóa lưu trữ.
* **Thực hành khởi tạo và cấu hình S3 Bucket đầu tiên:** Tạo S3 Bucket thử nghiệm trên AWS Console, bật **Block Public Access 100%** để chặn truy cập công khai và cấu hình mã hóa mặc định (**SSE-S3 / SSE-KMS**).
* **Thử nghiệm tổ chức cấu trúc thư mục & thao tác Object:** Tạo các tiền tố thư mục phân tầng (`bronze-raw/`, `silver-cleaned/`, `gold-curated/`), thực hành upload, kiểm tra metadata, tải xuống và xóa thử các tập dữ liệu mẫu (CSV/JSON) thông qua AWS Console và AWS CLI cơ bản.
* **Cấu hình S3 Lifecycle Management & Versioning:** Thử nghiệm bật S3 Versioning và cấu hình quy tắc Lifecycle chuyển đổi lớp lưu trữ (`S3 Standard-IA` sau 30 ngày, `S3 Glacier Flexible Retrieval` sau 90 ngày) để làm quen tư duy tối ưu chi phí lưu trữ dữ liệu trên Cloud.
* **Dọn dẹp tài nguyên thử nghiệm:** Xóa sạch toàn bộ các object, version thử nghiệm và bucket tạm sau khi hoàn tất bài thực hành để tránh phát sinh chi phí ngoài ý muốn.

### Kết quả đạt được tuần 4:
* Nắm vững kiến thức nền tảng và thao tác thực tế với **Amazon S3** (tạo bucket, phân quyền riêng tư Block Public Access, mã hóa và quản lý prefix/object).
* Hiểu cách thiết kế cấu trúc phân vùng thư mục trên S3 và áp dụng **S3 Lifecycle Rules** để tối ưu chi phí lưu trữ cho các bước xây dựng Data Lake ở tuần tiếp theo.

### Tài liệu tham khảo:
* <https://cloudjourney.awsstudygroup.com/>
* <https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html>
* <https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html>
* <https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html>
