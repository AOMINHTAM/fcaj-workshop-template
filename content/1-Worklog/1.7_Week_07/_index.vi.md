---
title: "Worklog Tuần 7"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:
* Chuyển đổi kiến trúc nạp dữ liệu từ Polling định kỳ sang mô hình hướng sự kiện (**Event-Driven S3 Ingestion**) nhằm tối ưu chi phí hạ tầng (**FinOps $0 compute** khi nhàn rỗi).
* Tích hợp **Amazon S3 Event Notifications** kết hợp hàng đợi **Amazon SQS** (`olist-s3-raw-arrival-queue`) và điều phối kích hoạt bằng **Apache Airflow 3 `Asset` + `AssetWatcher` (`SqsSensorTrigger` Long Polling 20s)** thay thế hoàn toàn cron `*/5`.
* Thiết kế cơ chế hoán đổi bảng tạm nguyên tử (**Atomic TEMP TABLE Staging Swap**) và chốt chặn an toàn (**Ingestion Guard** `ensure_non_empty_stage`) ngăn ngừa rủi ro file rỗng làm xóa sạch dữ liệu sản xuất trên **Amazon Redshift Serverless**.

### Các công việc đã thực hiện trong tuần:
* **Phân tích chi phí FinOps và thiết kế kiến trúc Event-Driven:** Phân tích hạn chế về chi phí của mô hình Polling truyền thống (`S3KeySensor` và cron `schedule="*/5 * * * *"`) trên môi trường Cloud: lãng phí tài nguyên máy chủ Airflow và phát sinh chi phí tính toán Redshift Serverless RPU không cần thiết khi không có file mới; lập bản thiết kế kiến trúc Event-Driven Ingestion: S3 (`raw/*.csv`) → S3 Event Notification → Amazon SQS Queue → Airflow 3 `AssetWatcher`.
* **Khởi tạo hạ tầng Amazon SQS và phân quyền S3 Event:** Khởi tạo hàng đợi Amazon SQS `olist-s3-raw-arrival-queue` trên AWS Console với Visibility Timeout (300s) và Message Retention (4 ngày); cấu hình S3 Event Notifications trên bucket định tuyến sự kiện `s3:ObjectCreated:*` với prefix filter `raw/` đến ARN của SQS queue, đồng thời gán quyền SQS Access Policy cho S3 Principal.
* **Triển khai Airflow 3 Asset & AssetWatcher:** Cài đặt Astro CLI, nâng cấp môi trường Apache Airflow 3 (Astronomer Runtime) trên Docker và WSL2; phát triển DAG điều phối trung tâm sử dụng **Airflow 3 `Asset` & `AssetWatcher`** với `SqsSensorTrigger` (Long Polling `wait_time_seconds=20`, `poke_interval=20`), kích hoạt DAG tự động ngay khi có tệp dữ liệu Olist mới nạp vào S3 mà không cần lập lịch cron (`schedule=None`).
* **Xây dựng Atomic Staging Swap và Ingestion Guard:** Xây dựng module kiểm soát chất lượng nạp `include/ingestion_guards.py` với hàm `ensure_non_empty_stage` (`staged_rows > 0`); thiết kế cơ chế nạp **Atomic Staging Swap**: Thực thi lệnh `COPY` từ S3 vào bảng tạm (`TEMP TABLE`) với chế độ giao dịch nguyên tử (`autocommit=False`), chỉ ghi đè vào bảng staging chính khi dữ liệu hợp lệ, ngăn chặn tuyệt đối tình trạng file rỗng làm hỏng bảng dữ liệu sản xuất.
* **Quản lý vòng đời tin nhắn SQS và kiểm thử toàn trình:** Xây dựng cơ chế xác nhận và dọn dẹp sau xử lý (Post-verification Acknowledgment): chỉ thực hiện xóa tin nhắn trong hàng đợi SQS (`delete_message`) và lưu trữ tệp trên S3 (`archive/<timestamp>/`) sau khi pipeline và các bài kiểm tra hoàn tất thành công; kiểm thử end-to-end kịch bản đẩy tệp Olist lên S3: DAG Airflow tự động kích hoạt trong vòng < 20 giây, hoàn tất nạp dữ liệu an toàn và xác nhận tin nhắn thành công.

### Kết quả đạt được tuần 7:
* Xây dựng thành công kiến trúc nạp dữ liệu theo hướng sự kiện (Event-Driven Ingestion) trên AWS: S3 → SQS → Airflow 3 `AssetWatcher`, loại bỏ 100% các lần chạy DAG nhàn rỗi và đạt chuẩn FinOps ($0 chi phí RPU tính toán trên Redshift khi không có dữ liệu mới).
* Đảm bảo tính toàn vẹn và bất biến dữ liệu với cơ chế Atomic Staging Swap và Ingestion Guard (`ensure_non_empty_stage`), triệt tiêu hoàn toàn rủi ro lỗi dữ liệu nạp từ tầng raw.
* Thiết lập quy trình quản lý vòng đời tin nhắn SQS an toàn, chỉ xác nhận xóa tin nhắn khi pipeline đã hoàn thành đầy đủ các bước kiểm tra chất lượng.

### Tài liệu tham khảo:
* <https://aws.amazon.com/blogs/big-data/>
* <https://docs.aws.amazon.com/AmazonS3/latest/userguide/EventNotifications.html>
* <https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/assets.html>
* <https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html>
