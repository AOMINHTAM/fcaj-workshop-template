---
title: "Worklog Tuần 3"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:
* Hiểu và thiết kế kiến trúc mạng đám mây **Amazon Virtual Private Cloud (VPC)** đa phân vùng khả dụng (Multi-AZ).
* Nắm vững cơ chế định tuyến giữa Public Subnet và Private Subnet thông qua **Internet Gateway (IGW)** và **NAT Gateway**.
* Thiết lập bảo mật mạng đa lớp bằng **Security Groups (Stateful)** và **Network ACLs (Stateless)**.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---: | :--- | :---: | :---: | :--- |
| **Thứ 2** | - Nghiên cứu kiến trúc **Amazon VPC**, quy hoạch dải địa chỉ IPv4 CIDR (`10.0.0.0/16`) <br> - Phân tích mô hình phân tách mạng 2 tầng (Public Subnet & Private Subnet) trên 2 Availability Zones | 12/10/2026 | 12/10/2026 | <https://cloudjourney.awsstudygroup.com/> |
| **Thứ 3** | - **Thực hành khởi tạo VPC:** <br>&emsp; + Tạo VPC tùy chỉnh `10.0.0.0/16` <br>&emsp; + Chia 2 Public Subnets và 2 Private Subnets trải đều trên 2 AZs để đảm bảo High Availability | 13/10/2026 | 13/10/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html> |
| **Thứ 4** | - **Cấu hình Gateway & Route Tables:** <br>&emsp; + Gắn **Internet Gateway (IGW)** vào VPC và cấu hình Public Route Table (`0.0.0.0/0 -> igw-xxxx`) <br>&emsp; + Khởi tạo **NAT Gateway** kèm Elastic IP tại Public Subnet và định tuyến chiều ra cho Private Route Table | 14/10/2026 | 14/10/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html> |
| **Thứ 5** | - Thiết lập **Security Groups** (giới hạn cổng SSH 22 và lưu lượng nội bộ theo nguyên tắc Least Privilege) <br> - Cấu hình **Network ACLs (NACL)** bảo vệ bổ trợ ở cấp độ Subnet | 15/10/2026 | 15/10/2026 | <https://cloudjourney.awsstudygroup.com/> |
| **Thứ 6** | - Khởi tạo máy chủ EC2 kiểm thử tại Public và Private Subnet để xác minh luồng kết nối mạng <br> - Thực hiện **Clean Up toàn bộ NAT Gateway, Elastic IP và EC2** để tránh phát sinh phí theo giờ | 16/10/2026 | 16/10/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 3:
* Xây dựng hoàn chỉnh hạ tầng mạng Amazon VPC Multi-AZ chuẩn doanh nghiệp, cô lập an toàn các tài nguyên dữ liệu ở tầng Private Subnet.
* Hiểu rõ luồng đi của gói tin qua Internet Gateway, NAT Gateway, Route Tables và sự khác biệt giữa Security Group (Stateful) với Network ACL (Stateless).
* Nắm vững kỹ năng quản trị Security Group và cổng dịch vụ — trực tiếp áp dụng để cấu hình kết nối an toàn cổng `5439` cho Amazon Redshift ở các tuần tiếp theo.
