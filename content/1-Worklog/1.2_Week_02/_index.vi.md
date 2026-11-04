---
title: "Worklog Tuần 2"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:
* Nắm vững kiến trúc quản trị định danh và truy cập trên AWS (**AWS IAM**): Users, User Groups, Policies, IAM Roles và cơ chế ủy quyền tạm thời (`sts:AssumeRole`).
* Trực tiếp lên văn phòng FCAJ và tham gia hỗ trợ công tác kỹ thuật chuẩn bị sự kiện cộng đồng.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---: | :--- | :---: | :---: | :--- |
| **Thứ 2** | - Nghiên cứu lý thuyết **AWS Identity and Access Management (IAM)** <br> - Phân biệt tài khoản Root, IAM User, IAM User Group và nguyên tắc **Least Privilege** | 05/10/2026 | 05/10/2026 | <https://cloudjourney.awsstudygroup.com/> |
| **Thứ 3** | - **Thực hành IAM:** <br>&emsp; + Khởi tạo IAM User Group `FCAJ-Admin` gắn chính sách `AdministratorAccess` <br>&emsp; + Tạo IAM User `aotam_admin` phục vụ thao tác kỹ thuật hàng ngày, ngừng sử dụng tài khoản Root | 06/10/2026 | 06/10/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html> |
| **Thứ 4** | - Kích hoạt quyền truy cập Billing Console cho IAM User <br> - Khởi tạo **IAM Role (`AdminRole`)** với Trust Policy cho phép ủy quyền qua `sts:AssumeRole` | 07/10/2026 | 07/10/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html> |
| **Thứ 5** | - **Trực văn phòng FCAJ:** Có mặt tại văn phòng, phối hợp cùng Ban tổ chức kiểm tra đường truyền mạng, thiết bị trình chiếu và hậu cần kỹ thuật cho sự kiện chuyên đề | 08/10/2026 | 08/10/2026 | Nội quy văn phòng FCAJ |
| **Thứ 6** | - Thực hành quy trình **Switch Role** trực tiếp trên AWS Management Console <br> - Kiểm chứng cơ chế cấp phát thông tin xác thực tạm thời (Temporary Security Credentials) qua AWS STS | 09/10/2026 | 09/10/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 2:
* Hoàn thành buổi trực văn phòng và đóng góp hỗ trợ kỹ thuật cho sự kiện của cộng đồng FCAJ.
* Thiết lập chuẩn mực bảo mật tài khoản AWS: tách biệt hoàn toàn Root User, quản lý quyền tập trung qua Group `FCAJ-Admin` và User `aotam_admin`.
* Hiểu sâu sự khác biệt giữa định danh dài hạn (IAM User) và định danh tạm thời qua STS (IAM Role), làm nền tảng để phân quyền giữa S3 và Redshift ở các tuần tiếp theo.
