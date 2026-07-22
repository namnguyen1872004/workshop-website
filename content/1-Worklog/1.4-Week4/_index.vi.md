---
title: "Worklog Tuần 4"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---



### Mục tiêu tuần 4:
* Bắt đầu Epic 2: Xác thực người dùng (Authentication).
* Bảo vệ API Backend và quản lý phiên đăng nhập.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tích hợp `NextAuth.js` (v5.0.0-beta.31) vào Frontend <br> - Thiết lập Provider: Google OAuth | 08/05/2026 | 08/05/2026 | NextAuth Docs |
| 3 | - Thiết lập xác thực bằng OTP qua Email (sử dụng HMAC signature) <br> - Tạo trang Đăng nhập / Đăng ký | 09/05/2026 | 09/05/2026 | NextAuth Email Auth |
| 4 | - Viết Custom Lambda Authorizer trên Backend <br> - Áp dụng `crypto.subtle` xác thực JWT (thuật toán HS256) | 10/05/2026 | 10/05/2026 | AWS Lambda Authorizer |
| 5 | - Cấu hình API Gateway sử dụng Lambda Authorizer vừa tạo <br> - Ràng buộc Frontend gửi kèm Bearer Token | 11/05/2026 | 11/05/2026 | AWS API Gateway |
| 6 | - Kiểm thử các luồng chặn truy cập chưa đăng nhập (Next.js Middleware) <br> - Chặn tính năng upload khi user chưa login | 12/05/2026 | 12/05/2026 | Next.js Middleware |

### Kết quả đạt được tuần 4:
* Hệ thống đăng nhập Stateless JWT hoạt động với Google OAuth và OTP.
* Backend được bảo vệ chặt chẽ thông qua Custom Lambda Authorizer không phụ thuộc thư viện ngoài.
* API Gateway xác thực thành công các request hợp lệ từ Frontend.


