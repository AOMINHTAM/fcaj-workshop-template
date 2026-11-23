---
title: "Tự đánh giá (Self-evaluation)"
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

### Bảng tự đánh giá năng lực theo 8 tiêu chí FCAJ

Trong suốt thời gian thực tập tại **Công ty TNHH Amazon Web Services Việt Nam** (chương trình **Workforce Bootcamp — First Cloud AI Journey 2026**, định hướng **Data & Analytics**), tôi tự đánh giá kết quả rèn luyện của bản thân theo 8 tiêu chí chuẩn của chương trình:

| STT | Tiêu chí đánh giá | Mức độ tự đánh giá | Nhận xét & Minh chứng thực tế |
| :-: | :--- | :---: | :--- |
| **1** | **Kiến thức chuyên môn** | **Tốt** | Làm chủ kiến trúc AWS cho Data Engineering & CloudOps: IAM Roles/STS, VPC Multi-AZ, S3 (Versioning, Lifecycle, HTTPS Policy), SQS + DLQ, CloudWatch Alarms, SNS, Redshift Serverless (4 RPUs), CloudFormation IaC, dbt Core (Kimball Star Schema, Incremental `delete+insert`) và Apache Airflow 3 (`AssetWatcher`). |
| **2** | **Tự học & nghiên cứu** | **Tốt** | Chủ động nghiên cứu tài liệu chính thức AWS, dbt, Airflow 3 Task SDK và tiến hóa liên tục dự án cá nhân qua 4 thế hệ kiến trúc (V2 → V3 → V4 → V5 `5.0.0`/`5.0.1`), kèm viết tài liệu phân tích chuyên sâu (`V5_DEEP_DIVE`, `OPERATIONS_RUNBOOK`, `BUSINESS_CASE`). |
| **3** | **Tính chủ động** | **Tốt** | Chủ động mở ticket AWS Support xử lý lỗi xác minh từ Tuần 1, hoàn thành 5/5 task nhận `$200` Credit, tự xây dựng CI/CD (GitHub Actions + `cfn-lint` + `21/21` unit tests), và chia sẻ bài học thực tế lên cộng đồng AWS Study Group VN. |
| **4** | **Kỷ luật & tuân thủ** | **Tốt** | Tuân thủ nghiêm túc nội quy văn phòng tầng 26 Bitexco (giờ giấc, trang phục, check-in/check-out Portal 2 lần/ngày), tham gia trực văn phòng hỗ trợ sự kiện từ Tuần 2 và duy trì kỷ luật FinOps (AWS Budgets, Clean-up tài nguyên về `$0`). |
| **5** | **Giao tiếp** | **Khá** | Trình bày rõ ràng tiến độ qua Worklog song ngữ VI/EN, trao đổi tích cực với mentor và các thành viên trong cộng đồng AWS Study Group VN; tiếp tục rèn luyện kỹ năng thuyết trình kỹ thuật trực tiếp trước đám đông. |
| **6** | **Teamwork (Làm việc nhóm)** | **Khá** | Phối hợp tốt với Ban tổ chức trong công tác chuẩn bị hậu cần kỹ thuật sự kiện cộng đồng, hỗ trợ trao đổi troubleshooting cùng các bạn học viên Data & Analytics và tiếp thu góp ý cross-review để vá 13 điểm mù ở bản `5.0.1`. |
| **7** | **Giải quyết vấn đề** | **Tốt** | Gỡ lỗi thực chiến thành công nhiều tình huống khó: xung đột mạng Cloudflare WARP cổng `5439`, lỗi duplicate `COPY` 4x, Circular Dependency kép trong CloudFormation, lỗi `BucketNotEmpty` khi xóa S3 Versioning, và bắt sống bug Airflow 3.3 `TriggeringAssetEventsAccessor` (`Mapping` vs. `dict`). |
| **8** | **Đóng góp cho Workshop** | **Tốt** | Tự thiết kế sơ đồ kiến trúc Draw.io, triển khai mã nguồn mở `aws-data-pipeline` V5 có nghiệm thu live E2E (`17/17` tasks PASS cả Happy Path và DLQ Redrive), và đóng gói thành bộ Workshop 6 chương song ngữ tái lập được 100% kèm quy trình dọn dẹp về `$0`. |

---

### Điểm mạnh & Hướng phát triển sau kỳ thực tập
* **Điểm mạnh:** Tư duy hệ thống tốt (gắn chặt giữa bài toán Business, Data Engineering và CloudOps/FinOps), cẩn trọng trong kiểm thử tự động và an toàn dữ liệu (Audit Ledger, Rollback, Least-Privilege IAM).
* **Điểm cần tiếp tục cải thiện:** Nâng cao khả năng kể chuyện bằng dữ liệu (Data Storytelling & BI Dashboard chuyên sâu), tìm hiểu thêm về Streaming real-time (Kinesis/MSK) và chuẩn bị cho chứng chỉ AWS quốc tế (**AWS Certified Solutions Architect – Associate / Data Engineer – Associate**).
