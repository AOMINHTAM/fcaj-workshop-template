---
title: "Worklog Tuần 1"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu tuần 1:
* Tham gia sự kiện Kick-off chương trình **Workforce Bootcamp - First Cloud AI Journey (FCAJ)**, nắm vững nội quy, quy chế và định hướng chuyên sâu **Data & Analytics**.
* Khởi tạo tài khoản AWS Free Tier, xử lý sự cố xác minh tài khoản và hoàn thành chuỗi tác vụ **Explore AWS** để nhận **$200 AWS Credit**.
* Thiết lập hàng rào kiểm soát chi phí **FinOps** với **AWS Budgets** (4 ngưỡng cảnh báo tự động).

### Các công việc đã thực hiện trong tuần:
* **Tham gia sự kiện Kick-off Bootcamp:** Dự lễ khai mạc chương trình, tìm hiểu các quy định thực tập, tiêu chí đánh giá dự án và lộ trình kỹ thuật 12 tuần của chương trình FCAJ.
* **Khởi tạo và gỡ rối xác minh tài khoản AWS:** Đăng ký tài khoản AWS Free Tier cá nhân; chủ động mở phiếu hỗ trợ AWS Support (Case ID: `179040404500475`) để xử lý lỗi xác minh cuộc gọi/SMS và chuẩn bị phương án tài khoản dự phòng để đảm bảo tiến độ học tập.
* **Hoàn thành chuỗi tác vụ Explore AWS săn $200 Credit:**
  * *Task 1 (Amazon EC2):* Khởi tạo máy chủ ảo EC2 (`t2/t3.micro`, cặp khóa `first-kp.pem`) và thực hiện quy trình Terminate an toàn (nhận $20).
  * *Task 2 (Amazon Bedrock):* Khảo sát giao diện Text Playground và trải nghiệm thử nghiệm mô hình nền tảng Claude Haiku / Amazon Titan (nhận $20).
  * *Task 3 (AWS Budgets):* Thiết lập ngân sách kiểm soát chi phí cơ bản đầu tiên (nhận $20).
  * *Task 4 (AWS Lambda):* Triển khai hàm serverless mẫu `http-function-url-tutorial` qua blueprint HTTP và tiến hành dọn dẹp tài nguyên (nhận $20).
  * *Task 5 (Amazon RDS):* Khởi tạo cơ sở dữ liệu quan hệ Aurora PostgreSQL và thực hành quy trình xóa dọn dẹp 2 tầng gồm DB Instance và DB Cluster (nhận $20).
* **Thiết lập hệ thống FinOps bảo vệ tài khoản:** Cấu hình **AWS Budgets** với tổng ngân sách **$200** kèm 4 nấc cảnh báo tự động qua email (`12.5% - $25`, `25% - $50`, `50% - $100`, `75% - $150`) nhằm ngăn chặn mọi nguy cơ phát sinh chi phí ngoài ý muốn.

### Kết quả đạt được tuần 1:
* Xử lý linh hoạt sự cố xác thực tài khoản thông qua AWS Support mà không làm gián đoạn tiến độ học tập.
* Hoàn thành 5/5 tác vụ trên widget **Explore AWS**, xác minh tổng số dư **$200 Credit** đã được cộng vào mục Billing Credits.
* Thiết lập thành công hệ thống cảnh báo chi phí **AWS Budgets 4 cấp độ**, đảm bảo an toàn tài chính tuyệt đối cho tài khoản trong suốt kỳ thực tập.
* Rèn luyện thói quen **Clean Up tài nguyên đa tầng** (EC2, Lambda, RDS Instance & Cluster) ngay sau khi hoàn tất bài thực hành.

### Tài liệu tham khảo:
* <https://hcm-rules.awsfcaj.com/>
* <https://cloudjourney.awsstudygroup.com/>
* <https://aws.amazon.com/free/>
* <https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html>
