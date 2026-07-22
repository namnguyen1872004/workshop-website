---
title: "Bản đề xuất: Hệ thống Quản lý Giao hàng AWS"
date: 2026-07-22
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

Tại phần này, tài liệu tóm tắt các nội dung cốt lõi của dự án Triển khai Hệ thống Quản lý Giao hàng trên AWS mà đội ngũ phát triển dự tính sẽ xây dựng và triển khai, phù hợp để tích hợp vào mẫu giao diện xưởng thực hành (workshop template) đang được cấu trúc.

# AWS Delivery Management System
## Kiến trúc hạ tầng đám mây đa tầng & Tự động hóa quy trình nghiệp vụ trên AWS

### 1. Tóm tắt điều hành 
Dự án tập trung triển khai hạ tầng đám mây cho Hệ thống Quản lý Giao hàng (Delivery Management System) sử dụng hệ sinh thái AWS. Nền tảng được xây dựng dựa trên ứng dụng ASP.NET Core, kết hợp với các dịch vụ cốt lõi như RDS MySQL, S3, SQS, Lambda, SES và Amazon Location. Mục tiêu là chuyển đổi một bản ghi cấu hình rời rạc thành một quy trình chuẩn hóa có thể lặp lại: từ chuẩn bị mạng riêng (VPC), bảo mật nhiều tầng, đến triển khai ứng dụng tự động mở rộng (Auto Scaling) và xử lý sự kiện nền. Kiến trúc backend này cung cấp một hệ thống API mạnh mẽ, sẵn sàng hỗ trợ tích hợp với các ứng dụng di động phía client (như Android, Flutter) trong tương lai, tạo điểm nhấn kỹ thuật ấn tượng trong các buổi phỏng vấn chuyên sâu.

### 2. Tuyên bố vấn đề 
*Vấn đề hiện tại* 
Các hệ thống quản lý giao hàng truyền thống thường đối mặt với rào cản lớn về khả năng mở rộng khi lượng đơn hàng tăng đột biến. Việc lưu trữ trực tiếp các minh chứng giao hàng (POD), chữ ký tài xế lên máy chủ cục bộ hoặc ghi thông tin xác thực (credentials) vào mã nguồn tạo ra các lỗ hổng bảo mật nghiêm trọng. Thêm vào đó, việc xử lý đồng bộ các tác vụ nặng như gửi email hay tính toán định vị làm giảm tốc độ phản hồi của hệ thống.

*Giải pháp* 
Dự án giải quyết vấn đề này bằng một kiến trúc đám mây tách biệt và tự động hóa cao. Người dùng chỉ giao tiếp với hệ thống qua Application Load Balancer (ALB), trong khi EC2 và cơ sở dữ liệu RDS được cô lập hoàn toàn trong mạng riêng tư (private subnets) không có IP công cộng. Ứng dụng truy xuất thông tin cấu hình an toàn qua AWS Secrets Manager. Hình ảnh POD và chữ ký được lưu trữ trực tiếp lên Amazon S3 thông qua S3 Gateway VPC Endpoint. Các tác vụ gửi thông báo được đẩy vào hàng đợi SQS và được AWS Lambda tiêu thụ để gửi email qua SES một cách bất đồng bộ.

*Lợi ích và hoàn vốn đầu tư (ROI)* 
Kiến trúc Auto Scaling Group (ASG) giúp tự động điều chỉnh lượng máy chủ theo lưu lượng truy cập thực tế, tối ưu hóa chi phí vận hành. Bằng cách áp dụng nguyên tắc đặc quyền tối thiểu (least privilege) với IAM Roles và mã hóa S3/RDS qua KMS, nền tảng đảm bảo độ an toàn dữ liệu cao nhất, hạn chế tối đa rủi ro thất thoát thông tin.

### 3. Kiến trúc giải pháp 
Nền tảng được triển khai trên AWS Region `ap-southeast-1` (Singapore) với thiết kế mạng phân tán trên hai Availability Zones (Multi-AZ) để đảm bảo tính sẵn sàng cao.

![Architecture](/workshop-website/images/2-Proposal/Picture1.png)

*Dịch vụ AWS và Công nghệ cốt lõi* 
- *AWS IAM & Security*: Quản lý quyền truy cập bằng IAM Roles cho EC2/Lambda và bảo vệ thông tin bằng AWS Secrets Manager.
- *Networking*: VPC gồm 6 subnets, Internet Gateway, NAT Gateway và Application Load Balancer định tuyến lưu lượng HTTP.
- *Compute*: EC2 (chạy ứng dụng ASP.NET Core) quản lý bởi Launch Template và Auto Scaling Group. 
- *Database*: Amazon RDS MySQL 8.4 triển khai trong Private Database Subnet.
- *Storage & Messaging*: Amazon S3 (lưu trữ tệp đa phương tiện, gói triển khai) và Amazon SQS (hàng đợi sự kiện Standard Queue kèm DLQ).
- *Serverless & External*: AWS Lambda (Worker xử lý Email), Amazon SES (Dịch vụ gửi Email) và Amazon Location GeoPlaces V2 (Tìm kiếm tọa độ).

*Thiết kế thành phần* 
- *Chuỗi kết nối (Security Group Chain)*: Request đi từ Internet -> ALB (Cổng 80) -> EC2 (Cổng 5000) -> RDS (Cổng 3306), không cho phép truy cập trực tiếp từ bên ngoài vào cơ sở dữ liệu.
- *Event-Driven Workflow*: Sau khi đơn hàng được commit vào cơ sở dữ liệu, ứng dụng phát một sự kiện JSON vào SQS. Lambda function sẽ lấy sự kiện này, xử lý gửi email và chỉ xóa message khỏi hàng đợi khi thao tác thành công.

### 4. Triển khai kỹ thuật 
*Các giai đoạn triển khai (Phân chia theo Lab)* 
Dự án được chia thành 7 Lab cốt lõi, lý tưởng để trình bày thành các module riêng biệt:
1. *Lab 1 (Network)*: Thiết lập VPC `10.0.0.0/16` với 6 subnets, public/private route tables và NAT Gateway.
2. *Lab 2 (Security)*: Xây dựng chuỗi Security Groups và IAM Roles (gắn quyền SSM, S3, SQS, Location) cho EC2.
3. *Lab 3 (Storage)*: Tạo S3 Bucket, thiết lập tiền tố thư mục cho POD/Signature và cấu hình S3 Gateway Endpoint.
4. *Lab 4 (Database)*: Triển khai DB Subnet Group, RDS MySQL, AWS Secrets Manager và thực hiện Database Migration qua CLI.
5. *Lab 5 (Events)*: Khởi tạo SQS (kèm DLQ), gắn trigger cho Lambda Worker và cấu hình SES Identity.
6. *Lab 6 (Location)*: Chuyển đổi công cụ định vị sang Amazon Location GeoPlaces V2 qua IAM Role thay vì API Key tĩnh.
7. *Lab 7 (Compute)*: Đóng gói Launch Template, liên kết Target Group với ALB và kích hoạt Auto Scaling Group.

*Yêu cầu kỹ thuật* 
- Tuyệt đối không lưu Access Key hay thông tin nhạy cảm vào mã nguồn; mọi tương tác dịch vụ phải thông qua IAM Role của tài nguyên thực thi.
- Luôn tạo Snapshot và Logical Backup trước khi thực hiện bất kỳ thao tác Migration nào có rủi ro thay đổi Schema.

### 5. Lộ trình & Mốc kiểm thử 
- *Mốc 1 (Mạng & Cơ sở dữ liệu)*: Đảm bảo EC2 giao tiếp thành công với RDS thông qua việc đọc Secrets Manager, xác minh bằng trạng thái HTTP 200 từ Health Check nội bộ.
- *Mốc 2 (Storage & Location)*: Người dùng có thể tìm kiếm địa chỉ (SearchText) để sinh tọa độ (Lat/Lng), và ứng dụng lưu thành công POD vào S3 mà không bị AccessDenied.
- *Mốc 3 (Tích hợp luồng sự kiện)*: Khi đơn hàng được tạo, SQS ghi nhận event có `eventType` chuẩn xác, Lambda Worker tiêu thụ event, và SES gửi email thành công mà không để lại thông báo trong Dead-Letter Queue (DLQ).
- *Mốc 4 (Hoàn thiện nền tảng)*: Triển khai bản release mới bằng Symlink, kiểm thử Auto Scaling và lên kế hoạch cho các nâng cấp tương lai như CloudFront, WAF và CloudWatch.

### 6. Đánh giá rủi ro 
*Ma trận rủi ro & Kế hoạch dự phòng* 
- *Rủi ro cấu hình mạng (Lỗi Target Unhealthy)*: Nguyên nhân thường do ứng dụng chưa chạy hoặc cấu hình sai Security Group. *Khắc phục*: Sử dụng công cụ `curl` cục bộ để kiểm tra cổng 5000 và đối chiếu lý do Unhealthy từ Target Group.
- *Lỗi Timeout/Worker Lambda*: Message SQS bị lặp lại nhiều lần. *Khắc phục*: Cấu hình `maxReceiveCount` bằng 3, xử lý logic idempotent theo `eventId` bên trong Lambda, và đảm bảo Lambda có đủ quyền `sqs:DeleteMessage`.
- *Lỗi truy cập S3 (AccessDenied)*: Thiếu quyền Put/Delete hoặc KMS. *Khắc phục*: Kiểm tra IAM Role thực tế của EC2 bằng `aws sts get-caller-identity` để xác minh quyền hạn.

### 7. Kết quả kỳ vọng 
*Cải tiến kỹ thuật*: Xây dựng hoàn chỉnh một kiến trúc chuẩn hóa, tự động và chịu tải tốt trên AWS. Hệ thống loại bỏ hoàn toàn các rủi ro từ việc cấp phát IP công cộng cho máy chủ nội bộ hay lưu trữ key thủ công.
*Giá trị dài hạn*: Nền tảng không chỉ đảm bảo ứng dụng ASP.NET Core vận hành mượt mà với 13 đơn hàng, 5 hubs và 3 users mẫu, mà tài liệu triển khai này còn đóng vai trò như một bộ khung tham chiếu thực tiễn, sẵn sàng tích hợp vào trang tài liệu tĩnh cá nhân để giới thiệu năng lực kiến trúc hệ thống.