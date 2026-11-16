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

### Các công việc đã thực hiện trong tuần:
* **Thiết kế quy hoạch địa chỉ mạng VPC:** Nghiên cứu kiến trúc Amazon VPC, quy hoạch dải địa chỉ IPv4 CIDR (`10.0.0.0/16`), xây dựng mô hình mạng 2 tầng (Public Subnet và Private Subnet) phân bổ cân bằng trên 2 Availability Zones để đảm bảo tính sẵn sàng cao (High Availability).
* **Khởi tạo và phân chia Subnet:** Tạo VPC tùy chỉnh `10.0.0.0/16`, thiết lập 2 Public Subnets và 2 Private Subnets độc lập trải dài trên 2 AZs.
* **Cấu hình Gateway và Bảng định tuyến (Route Tables):**
  * Gắn **Internet Gateway (IGW)** vào VPC và thiết lập Public Route Table định tuyến lưu lượng ra Internet (`0.0.0.0/0 → igw-xxxx`).
  * Khởi tạo **NAT Gateway** kèm Elastic IP (EIP) tại Public Subnet, cấu hình Private Route Table để cho phép các tài nguyên trong Private Subnet truy cập Internet một chiều (chiều tải gói/bản vá) mà không bị lộ diện ra bên ngoài.
* **Bảo vệ mạng đa tầng với Security Groups và Network ACLs:**
  * Cấu hình **Security Groups (Stateful)** theo nguyên tắc Least Privilege, chỉ mở cổng SSH 22 và lưu lượng nội bộ giữa các tầng.
  * Cấu hình **Network ACLs (Stateless)** bảo vệ bổ trợ ở cấp độ biên mạng của từng Subnet.
* **Kiểm thử kết nối và dọn dẹp tài nguyên:** Khởi tạo EC2 kiểm thử trong Public và Private Subnet để kiểm tra định tuyến gói tin; hoàn tất quy trình xóa sạch toàn bộ NAT Gateway, giải phóng Elastic IP và Terminate EC2 kiểm thử để tối ưu chi phí.

### Kết quả đạt được tuần 3:
* Xây dựng hoàn chỉnh hạ tầng mạng Amazon VPC Multi-AZ chuẩn doanh nghiệp, cô lập an toàn các tài nguyên dữ liệu ở tầng Private Subnet.
* Hiểu rõ luồng đi của gói tin qua Internet Gateway, NAT Gateway, Route Tables và sự khác biệt giữa Security Group (Stateful) với Network ACL (Stateless).
* Nắm vững kỹ năng quản trị Security Group và cổng dịch vụ — trực tiếp áp dụng để cấu hình kết nối an toàn cổng `5439` cho Amazon Redshift ở các tuần tiếp theo.

### Tài liệu tham khảo:
* <https://cloudjourney.awsstudygroup.com/>
* <https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html>
* <https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html>
