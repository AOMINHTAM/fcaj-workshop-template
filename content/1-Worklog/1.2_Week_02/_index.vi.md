---
title: "Worklog Tuần 2"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:
* Nắm vững kiến trúc quản trị định danh và truy cập trên AWS (**AWS IAM**): Users, User Groups, Policies, IAM Roles và cơ chế ủy quyền tạm thời (`sts:AssumeRole`).
* Trực tiếp lên văn phòng FCAJ và tham gia hỗ trợ công tác kỹ thuật chuẩn bị sự kiện cộng đồng.

### Các công việc đã thực hiện trong tuần:
* **Nghiên cứu nguyên lý bảo mật AWS IAM:** Tìm hiểu chi tiết về tài khoản Root, IAM User, IAM User Group, phân loại Managed Policy vs. Inline Policy và nguyên tắc đặc quyền tối thiểu (**Principle of Least Privilege**).
* **Thiết lập định danh và phân quyền chuẩn doanh nghiệp:**
  * Khởi tạo IAM User Group `FCAJ-Admin` và gắn chính sách quản trị `AdministratorAccess`.
  * Khởi tạo IAM User `aotam_admin` để sử dụng cho toàn bộ hoạt động kỹ thuật thường nhật, khóa và ngừng sử dụng tài khoản Root.
  * Kích hoạt quyền truy cập Billing Console an toàn cho IAM User.
* **Xây dựng IAM Role và cơ chế ủy quyền STS:** Khởi tạo IAM Role `AdminRole` với Trust Policy cho phép ủy quyền qua `sts:AssumeRole`, thực hành quy trình **Switch Role** trực tiếp trên AWS Console và kiểm chứng cơ chế cấp phát thông tin xác thực tạm thời (Temporary Security Credentials).
* **Trực văn phòng FCAJ và hỗ trợ sự kiện:** Có mặt tại văn phòng, phối hợp cùng Ban tổ chức kiểm tra đường truyền mạng, thiết bị trình chiếu và chuẩn bị hậu cần kỹ thuật phục vụ cho các buổi workshop chuyên đề của cộng đồng.

### Kết quả đạt được tuần 2:
* Hoàn thành buổi trực văn phòng và đóng góp hỗ trợ kỹ thuật cho sự kiện của cộng đồng FCAJ.
* Thiết lập chuẩn mực bảo mật tài khoản AWS: tách biệt hoàn toàn Root User, quản lý quyền tập trung qua Group `FCAJ-Admin` và User `aotam_admin`.
* Hiểu sâu sự khác biệt giữa định danh dài hạn (IAM User) và định danh tạm thời qua STS (IAM Role), làm nền tảng để phân quyền giữa S3 và Redshift ở các tuần tiếp theo.

### Tài liệu tham khảo:
* <https://cloudjourney.awsstudygroup.com/>
* <https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html>
* <https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html>
