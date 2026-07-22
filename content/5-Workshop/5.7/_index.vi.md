---
title: "SQS, Lambda và SES"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

Event pipeline chỉ được tích hợp sau khi database và media flow ổn định. Request web hoàn tất nghiệp vụ trước, sau đó phát event; Lambda xử lý email bất đồng bộ.

5.7.1. SQS main queue và DLQ

| Thuộc tính | Giá trị workshop |
|---|---|
| Main queue | delivery-order-events (Standard) |
| DLQ | delivery-order-events-dlq |
| Visibility timeout | 180 giây |
| Retention | 4 ngày |
| Encryption | SQS-managed SSE hoặc KMS theo phạm vi |
| maxReceiveCount | 3 |
![Hinh31](/workshop-website/images/5-Workshop/image31.png)
![Hinh32](/workshop-website/images/5-Workshop/image32.png)
5.7.2. Event contract
```json
{
  "schemaVersion": "1.0",
  "eventId": "uuid",
  "eventType": "order.created",
  "occurredAtUtc": "2026-07-22T09:10:14Z",
  "orderId": 6,
  "orderCode": "NF-20260721170553-347",
  "status": "created",
  "customerId": 1,
  "driverId": null
}
```
Event phải có `schemaVersion`, `eventId`, `eventType`, thời gian UTC và khóa nghiệp vụ.
Không đưa credential, cookie, connection string hoặc base64 media vào message.
Payload chỉ chứa metadata cần cho consumer; media được tham chiếu bằng object key nếu thật sự cần.

5.7.3. Publisher trong ASP.NET Core
1. Đăng ký `IAmazonSQS` bằng IAM Role; không hard-code Queue URL hoặc access key.
2. Tạo `IOrderEventPublisher` và `OrderEventEnvelope`.
3. Publish sau khi `SaveChanges` hoặc transaction đã commit.
4. Log `eventId`, `eventType`, `orderId`, AWS MessageId; không log toàn bộ payload nhạy cảm.
5. Trong workshop có thể publish trực tiếp; production nên dùng Transactional Outbox để tránh khoảng hở DB commit/SQS send.

Tiêu chí quan trọng: Manual `SendMessage` chỉ chứng minh hạ tầng queue. Tích hợp chỉ hoàn tất khi một hành động đơn hàng thật từ ứng dụng tạo message đúng contract.

5.7.4. Lambda consumer
![Hinh33](/workshop-website/images/5-Workshop/image33.png)
![Hinh34](/workshop-website/images/5-Workshop/image34.png)
![Hinh35](/workshop-website/images/5-Workshop/image35.png)
![Hinh36](/workshop-website/images/5-Workshop/image36.png)
Lambda role cần quyền ReceiveMessage, DeleteMessage, ChangeMessageVisibility, GetQueueAttributes và quyền SES cần thiết.
Handler phải kiểm tra `eventId` để xử lý idempotent vì SQS có thể giao lại message.
Chỉ xóa message sau khi xử lý thành công. Lỗi phải được throw để SQS/Lambda retry và chuyển DLQ khi đủ số lần.
Khi rollout lần đầu, có thể để trigger Disabled, test handler bằng payload mẫu rồi mới Enable.

5.7.5. Amazon SES
![Hinh37](/workshop-website/images/5-Workshop/image37.png)
![Hinh38](/workshop-website/images/5-Workshop/image38.png)
```env
SES_FROM_ADDRESS=<VERIFIED_SENDER>
SES_REPLY_TO=<OPTIONAL_REPLY_TO>
APP_BASE_URL=<ALB_OR_CLOUDFRONT_URL>
```
Trạng thái gửi mail: Nếu SES account còn ở sandbox, chỉ dùng identity/recipient đã được xác minh và ghi rõ giới hạn trong báo cáo. Không ghi nhận là production email nếu chưa được cấp production access.

5.7.6. Kiểm thử pipeline

| Lớp kiểm thử | Bằng chứng |
|---|---|
| Hạ tầng | Manual message vào main queue; redrive policy trỏ đúng DLQ. |
| Publisher | Tạo/đổi trạng thái đơn thật và thấy event đúng contract. |
| Lambda | CloudWatch log có eventId; không xử lý trùng. |
| SES | Send call thành công hoặc ghi nhận sandbox/identity constraint rõ ràng. |
| Failure | Message lỗi retry và đi vào DLQ sau maxReceiveCount. |