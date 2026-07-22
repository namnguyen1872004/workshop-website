---
title: "Giới thiệu và kiến trúc giải pháp"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

Tuyên bố vấn đề
NightFury Express là hệ thống quản lý giao hàng có các luồng nghiệp vụ tạo đơn, phân công tài xế, cập nhật trạng thái, ghi nhận bằng chứng giao hàng và gửi thông báo. Khi chuyển từ môi trường local lên AWS, các điểm yếu cần được xử lý gồm:
Ứng dụng chưa có hợp đồng runtime thống nhất cho EC2 và ALB: thiếu endpoint health riêng, port chưa cố định và forwarded headers chưa được xử lý đầy đủ.
Thông tin kết nối database có nguy cơ phụ thuộc cấu hình cục bộ; migration có thể làm lệch schema nếu chạy không kiểm soát.
Ảnh POD, chữ ký và minh chứng nếu lưu trên local disk sẽ mất khi thay instance hoặc mở rộng Auto Scaling.
Gửi email trực tiếp trong request web làm tăng thời gian phản hồi và khó retry khi dịch vụ ngoài lỗi.
Nominatim là phụ thuộc bên ngoài không nằm trong kiến trúc AWS mục tiêu.
CloudFront, WAF và monitoring nếu cấu hình quá sớm có thể che khuất lỗi ở lớp ứng dụng, làm tăng phạm vi debug và chi phí.
Workshop giải quyết các vấn đề trên bằng cách tách rõ các lớp runtime, dữ liệu, media, messaging, serverless, location, edge và monitoring. Mỗi lớp chỉ được xem là hoàn tất khi có bước kiểm thử và bằng chứng tương ứng.

Kiến trúc giải pháp
Hệ thống được triển khai trong một VPC riêng trên hai Availability Zones. ALB tiếp nhận request và chuyển đến ASP.NET Core trên EC2 cổng 5000. EC2 dùng IAM Role để đọc secret RDS, ghi media vào S3, gửi sự kiện vào SQS và gọi Amazon Location. Lambda nhận message từ SQS và gửi email qua SES.

Luồng request và dữ liệu
1. Người dùng truy cập ứng dụng qua CloudFront/WAF ở giai đoạn cuối hoặc trực tiếp qua ALB trong giai đoạn core flow.
2. ALB chuyển request đến Target Group trên cổng 5000; `/health` dùng làm liveness check.
3. Ứng dụng đọc database credential từ Secrets Manager và kết nối RDS MySQL qua Security Group chain.
4. POD, chữ ký và minh chứng được ghi vào S3; database chỉ giữ object key và metadata.
5. Sau khi transaction nghiệp vụ commit, ứng dụng phát event vào SQS.
6. Lambda xử lý message, thực hiện idempotency và gọi SES. Message lỗi đủ số lần được chuyển vào DLQ.
7. Geocoding và reverse geocoding đi qua API nội bộ sử dụng Amazon Location GeoPlaces V2.

Sơ đồ kiến trúc

![Hinh1](/workshop-website/images/5-Workshop/image1.png)
*Hình 1. Sơ đồ kiến trúc hiện tại của NightFury Express được giữ nguyên làm kiến trúc mục tiêu.*

Cách đọc sơ đồ: Sơ đồ được giữ nguyên theo yêu cầu. Tuy nhiên, sự xuất hiện của CloudFront, WAF, Auto Scaling hoặc monitoring trong sơ đồ không đồng nghĩa các thành phần đó đã hoàn tất. Trạng thái thực tế phải được đối chiếu với bảng trạng thái và evidence trong từng section.

Công nghệ chính

| Thành phần | Vai trò trong hệ thống |
|---|---|
| ASP.NET Core / .NET 9 | Web application, REST API, business logic và health endpoint. |
| Amazon EC2 + systemd | Runtime của ứng dụng; quản trị ưu tiên bằng Systems Manager Session Manager. |
| Application Load Balancer | Điểm vào HTTP(S), chuyển request tới Target Group và health check. |
| Amazon RDS for MySQL | Lưu dữ liệu nghiệp vụ và lịch sử migration. |
| AWS Secrets Manager | Quản lý credential RDS; ứng dụng đọc bằng IAM Role. |
| Amazon S3 | Lưu POD, chữ ký, minh chứng và gói deploy. |
| Amazon SQS + DLQ | Tách luồng request khỏi xử lý email bất đồng bộ. |
| AWS Lambda + Amazon SES | Consumer sự kiện và gửi email. |
| Amazon Location GeoPlaces V2 | Tìm địa chỉ và reverse geocoding thay Nominatim. |
| CloudFront + WAF + CloudWatch | Edge protection và quan sát, chỉ triển khai sau core flow. |

Danh mục tài nguyên mẫu

| Lớp | Tên tài nguyên | Vai trò |
|---|---|---|
| Network | delivery-dev-vpc | VPC 10.0.0.0/16 |
| Load balancing | delivery-alb / delivery-app-tg | Nhận HTTP(S), health check `/health` trên cổng 5000 |
| Compute | delivery-ec2-lt / delivery-ec2-asg | Launch Template, Auto Scaling và runtime ASP.NET Core |
| Database | delivery-dev-rds | RDS MySQL 8.4 trong private DB subnets |
| Storage | delivery-dev-pod-nm2026a | POD, chữ ký, minh chứng và gói triển khai |
| Messaging | delivery-order-events | Standard queue phát sự kiện đơn hàng |
| DLQ | delivery-order-events-dlq | Message lỗi sau số lần retry cấu hình |
| Serverless | delivery-email-worker | Lambda xử lý email qua SES |
| IAM | delivery-ec2-role | Quyền runtime theo least privilege |

Trạng thái triển khai hiện tại

| Hạng mục | Trạng thái tài liệu | Nguyên tắc trình bày |
|---|---|---|
| Network và security baseline | Đã có nội dung/evidence | Giữ nguyên sơ đồ và ảnh cấu hình hiện tại. |
| Chuẩn hóa app, RDS, S3, SQS, Lambda/SES, Location | Đang tích hợp theo thứ tự | Chỉ đánh dấu hoàn tất khi có smoke test và evidence. |
| EC2 + ALB/Target Group | Giai đoạn chốt triển khai | Target Group phải Healthy trước khi sang Edge. |
| CloudFront/WAF/Monitoring nâng cao | Planned | Không ghi `Done` nếu chưa có cấu hình và ảnh chụp thật. |

Thứ tự tích hợp phù hợp nhất

Thứ tự dưới đây là dependency order chứ không chỉ là danh sách công việc. Mỗi bước tạo nền tảng ổn định cho bước tiếp theo và giảm số lượng lỗi phải xử lý đồng thời.

| # | Giai đoạn | Lý do thực hiện ở vị trí này | Điều kiện qua bước |
|---|---|---|---|
| 1 | Chuẩn hóa ứng dụng cho EC2 | Tạo hợp đồng runtime ổn định: `/health`, cổng 5000, forwarded headers, systemd, biến môi trường và tắt VNPay bằng feature flag. | Local curl trả HTTP 200; service khởi động lại không lỗi; source không phụ thuộc credential tĩnh. |
| 2 | RDS qua Secrets Manager và migration | Ổn định dữ liệu nghiệp vụ trước khi chuyển media và phát sự kiện. Secret được đọc bằng IAM Role, không đặt password trong source hoặc file public. | Đọc secret thành công; migration up-to-date; dữ liệu kiểm chứng không mất. |
| 3 | Chuyển POD/chữ ký/minh chứng sang S3 | Loại bỏ phụ thuộc local disk trước khi scale EC2. Database chỉ lưu object key. | Upload/read/delete đúng prefix; bucket private; object được mã hóa. |
| 4 | Tích hợp SQS để phát sự kiện đơn hàng | Chốt event contract sau khi luồng database và media đã ổn định; publish sau khi transaction nghiệp vụ commit. | Một thao tác đơn hàng thật tạo message đúng `eventType`, `eventId`, `orderId`. |
| 5 | Hoàn thiện Lambda → SES | Consumer chỉ được triển khai sau khi event contract ổn định để tránh sửa nhiều lần và tránh gửi email sai. | Lambda xử lý idempotent; message thành công bị xóa; lỗi đủ số lần đi vào DLQ. |
| 6 | Thay Nominatim bằng Amazon Location | Thực hiện trước kiểm thử end-to-end cuối cùng để địa chỉ và tọa độ được kiểm chứng trên dịch vụ mục tiêu. | Search và reverse geocode hoạt động qua API nội bộ; trình duyệt không nhận AWS credential. |
| 7 | Deploy lên EC2 và kiểm tra Target Group | Đóng gói release hoàn chỉnh, triển khai theo symlink, gắn ALB/Target Group và kiểm tra health. | Local `/health` HTTP 200; Target Group `Healthy`; có lệnh rollback. |
| 8 | CloudFront, WAF và Monitoring | Chỉ cấu hình edge và quan sát nâng cao sau khi backend ổn định để không che khuất lỗi ứng dụng và không mở rộng phạm vi quá sớm. | Có evidence distribution/Web ACL/alarm thực tế; nếu chưa có phải ghi rõ `Planned`. |

Điều chỉnh quan trọng: Bước 1 chỉ chuẩn hóa hợp đồng chạy trên EC2, chưa phải lần deploy cuối. Các tích hợp RDS, S3, SQS, Lambda/SES và Location có thể được phát triển, smoke test trên local hoặc staging; release hoàn chỉnh chỉ được đưa vào Target Group ở bước 7.