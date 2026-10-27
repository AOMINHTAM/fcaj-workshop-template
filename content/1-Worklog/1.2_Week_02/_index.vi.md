---
title: "Tuần 2: Quản trị bảo mật IAM & Hỗ trợ kỹ thuật sự kiện"
weight: 2
---

### 🇻🇳 Báo cáo công việc Tuần 02

**1. Công việc đã thực hiện:**
- **Hoạt động trực tiếp tại văn phòng:** Có mặt tại văn phòng vào Thứ Năm (01/10/2026), tham gia hỗ trợ Ban tổ chức chuẩn bị cơ sở vật chất, đường truyền và thiết bị kỹ thuật phục vụ cho sự kiện diễn ra vào Thứ Sáu (02/10/2026).
- **Quản trị định danh và truy cập (AWS IAM):**
  - Khởi tạo IAM User Group mang tên chính xác **`FCAJ-Admin`** và đính kèm chính sách `AdministratorAccess`.
  - Tạo IAM User (`aotam_admin`) phục vụ thao tác kỹ thuật hàng ngày, dừng hoàn toàn việc dùng tài khoản Root.
  - Kích hoạt tính năng truy cập thanh toán (Billing Access) cho danh tính IAM từ cấu hình tài khoản.
  - Khởi tạo IAM Role (`AdminRole`) với chính sách tin cậy (Trust Policy) cho phép ủy quyền `sts:AssumeRole`.
  - Thực hành quy trình **Switch Role** trực tiếp trên thanh điều hướng AWS Management Console để thẩm tra luồng cấp phát quyền tạm thời (Temporary Credentials) và kiểm chứng nguyên tắc Least Privilege.

**2. Kết quả đạt được:**
- Hoàn thành 01 buổi trực văn phòng và đóng góp hỗ trợ công tác tổ chức sự kiện cộng đồng.
- Nắm vững kiến trúc phân quyền IAM: phân biệt vai trò giữa IAM User (định danh dài hạn cho con người) và IAM Role (cấp quyền tạm thời cho dịch vụ và cơ chế ủy quyền chéo).
- Vận hành thành công cơ chế Switch Role an toàn trên AWS Console.

**3. Tài liệu tham khảo:**
- FCAJ Lab Guide: Access Management with AWS Identity and Access Management (AWS IAM).
- AWS Documentation: Security Best Practices in IAM & Role Switching.