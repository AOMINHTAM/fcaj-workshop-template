---
title: "Bài viết 1: Di chuyển Data Pipeline từ Snowflake sang AWS S3 & Redshift Serverless"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

- **Tiêu đề bài đăng:** *Chê Snowflake hết Free Trial, mình dọn nguyên cái Pipeline sang AWS S3 & Redshift và cái kết =)))*
- **Nhóm cộng đồng:** AWS Study Group VN (First Cloud AI Journey)
- **Ngày đăng:** 11/11/2026 (20:21)
- **Link bài viết chính thức (Facebook):** <https://www.facebook.com/groups/awsstudygroupfcj/permalink/2334049077360051/>
- **Mã nguồn dự án (GitHub):** <https://github.com/AOMINHTAM/aws-data-pipeline>

---

### Giới thiệu

Nhân dịp tài khoản Snowflake hết hạn free trial, mình quyết định "chuyển nhà" toàn bộ dự án Data Pipeline từ bản `dbt_snowflake` cũ sang hệ sinh thái **AWS** để làm quen kiến trúc thực tế và nhận góp ý từ các tiền bối trong cộng đồng **AWS Study Group VN**.

![Sơ đồ kiến trúc Data Pipeline trên AWS S3 và Redshift Serverless](/fcaj-workshop-template/images/blogs/blog-01-architecture.png)

---

### 1. Tâm điểm đợt "chuyển nhà": Bộ đôi Amazon S3 & Redshift Serverless

* **Chọn Region:** Kinh nghiệm xương máu khi làm project trên AWS là ưu tiên chọn **`us-east-1`** vì hỗ trợ đầy đủ các dịch vụ mới và đồng bộ.
* **Amazon S3 (Data Lake — Landing Zone):** Đóng vai trò là điểm tập kết dữ liệu thô ban đầu (dữ liệu Olist E-Commerce gồm 9 tệp CSV, ~100.000 đơn hàng) được đẩy lên qua Bash script và AWS CLI (`aws s3 sync`) với chi phí lưu trữ rất thấp trước khi đưa vào xử lý.
* **Amazon Redshift Serverless (Data Warehouse):** Sự thay thế vừa vặn sau khi chia tay Snowflake. Điểm mạnh lớn nhất là tối ưu chi phí theo cơ chế **RPU (4 RPUs)** — chỉ tính phí khi thực thi truy vấn, tự động thu hồi về `$0` khi nhàn rỗi.
* **Sự kết hợp giữa S3, Redshift và dbt Core:**
  * Phân quyền **IAM Role (`RedshiftS3ReadOnlyRole`)** thông qua STS `AssumeRole` để Redshift kéo dữ liệu trực tiếp từ S3 vào tầng `raw_data` bằng lệnh **`COPY` (MPP Bulk Load)**.
  * **dbt Core (`dbt-redshift`)** kết nối vào Redshift qua Workgroup Endpoint để đẩy toàn bộ các tác vụ SQL chuyển đổi dữ liệu (từ `raw_data` → `public_staging` → `public_marts` theo mô hình **Kimball Star Schema** kèm 9/9 bài kiểm thử chất lượng) xuống cho Redshift xử lý.
* **Góc Troubleshooting thực tế:** Trong quá trình thao tác nạp dữ liệu từ S3 vào Redshift, mình từng chạy nhầm lệnh `COPY` tới **4 lần**, khiến dữ liệu bị nhân bản trùng lặp gấp 4. Cách xử lý nhanh và dứt điểm ở phiên bản nền tảng này là `DROP` bảng để nạp lại từ đầu sạch sẽ (tạo tiền đề để về sau chuẩn hóa cơ chế nạp Idempotent và Atomic Swap).

---

### 2. "Giải ngố" Ubuntu/Linux cùng Docker và Apache Airflow

* **Hiểu rõ vai trò của Ubuntu/Linux:** Trước đây Ubuntu từng là khái niệm khá mơ hồ, nhưng khi trực tiếp dựng pipeline trên **WSL2 Ubuntu**, mình nắm rõ hơn cách nhân Linux vận hành phía sau và phối hợp nhịp nhàng với **Docker** container.
* **Apache Airflow (Astro CLI & Astronomer Cosmos):** Triển khai điều phối toàn bộ DAG chuyển đổi dbt qua Astronomer Cosmos chạy ổn định, giúp quản lý phụ thuộc giữa các tầng dữ liệu trực quan hơn.

---

### 3. Cảm nhận cá nhân & Hướng phát triển tiếp theo

* **Cảm nhận:** Các dịch vụ AWS được đóng gói và chuẩn hóa rất tốt; phần tốn nhiều thời gian nhất khi triển khai thực tế lại nằm ở khâu cấu hình mạng, phân quyền **IAM** và thiết lập để các dịch vụ giao tiếp đúng với nhau.
* **Điểm còn bỏ ngỏ (tại thời điểm viết bài V2):** Nguồn dữ liệu đầu vào vẫn ở dạng tĩnh (static), nên toàn bộ pipeline chưa khép kín và chưa tự động kích hoạt theo sự kiện với Airflow.
* **Định hướng tiếp theo:** Mở rộng luồng tự động hóa theo hướng sự kiện (Event-Driven) và tích hợp thêm lớp phân tích trực quan hóa dữ liệu (**BI & Analytics**) ở các bài viết tiếp theo.

---

### 4. Lời kết & Tham khảo

Dù sử dụng AWS, GCP hay Azure, điều quan trọng nhất vẫn là **tư duy thiết kế hệ thống và xử lý dữ liệu** đằng sau nền tảng. Xin gửi lời cảm ơn đến các anh chị mentor trong cộng đồng **First Cloud AI Journey (FCAJ)** và **Xóm Data** đã chia sẻ nhiều kiến thức quý báu trong suốt quá trình thực hiện dự án.

* **Bài đăng gốc trên AWS Study Group VN:** <https://www.facebook.com/groups/awsstudygroupfcj/permalink/2334049077360051/>
* **GitHub Repository:** <https://github.com/AOMINHTAM/aws-data-pipeline>
