---
title: "Tuần 3: Mạng hạ tầng Amazon VPC & Bảo mật hệ thống Cloud"
weight: 3
---

### 🇻🇳 Báo cáo công việc Tuần 03

**1. Công việc đã thực hiện:**
- **Thiết kế kiến trúc Virtual Private Cloud (VPC):**
  - Khởi tạo VPC tùy chỉnh với dải địa chỉ CIDR `10.0.0.0/16` trải rộng trên 2 Availability Zones (AZs) nhằm đảm bảo tính sẵn sàng cao (High Availability).
  - Phân chia cấu trúc mạng 2 tầng (dual-tier subnets): 2 Public Subnet (dành cho các cổng tiếp nhận lưu lượng Internet) và 2 Private Subnet (cô lập an toàn các dịch vụ xử lý dữ liệu nội bộ).
- **Cấu hình định tuyến và Gateway:**
  - Khởi tạo **Internet Gateway (IGW)**, gắn vào VPC và cập nhật bảng định tuyến công khai (Public Route Table: `0.0.0.0/0 -> igw-xxxx`).
  - Triển khai **NAT Gateway** tại Public Subnet kèm theo Elastic IP (EIP), cho phép các tài nguyên trong Private Subnet truy cập Internet chiều ra (outbound) để tải thư viện/bản vá nhưng chặn hoàn toàn kết nối trực tiếp từ ngoài vào (inbound).
  - Cập nhật Private Route Table điều hướng toàn bộ lưu lượng ra Internet thông qua NAT Gateway.
- **Bảo mật mạng và phân tầng phòng thủ:**
  - Thiết lập **Security Groups (SG)** tuân thủ nguyên tắc Least Privilege (giới hạn cổng SSH 22 chỉ mở cho IP quản trị, lưu lượng nội bộ giới hạn trong CIDR của VPC).
  - Cấu hình **Network ACLs (NACL)** như một lớp tường lửa cấp độ subnet để bảo vệ bổ trợ.
  - Khởi tạo EC2 kiểm thử trong cả hai phân vùng mạng để kiểm tra tính toàn vẹn của luồng định tuyến và độ an toàn mạng.

**2. Kết quả đạt được:**
- Xây dựng thành công hạ tầng mạng Multi-AZ chuẩn doanh nghiệp, tạo nền tảng vững chắc cho các pipeline dữ liệu tiếp theo.
- Thực thi cơ chế cô lập an toàn, triệt tiêu nguy cơ lộ lọt tài nguyên backend ra Internet công cộng.
- Kiểm thử và xác minh kết nối đa phân vùng thành công.

**3. Tài liệu tham khảo:**
- FCAJ Lab Guide: Networking on AWS - Amazon VPC Architecture.
- AWS Documentation: VPC with Public and Private Subnets (NAT).
- AWS Well-Architected Framework: Reliability and Security Pillars.
