---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Hiện đã có tính năng cài đặt hệ điều hành tùy chỉnh trên các thiết bị AWS DeepRacer.

Trong quá trình tìm hiểu các bài viết mới trên AWS Blog, tôi đọc được một cập nhật khá thú vị liên quan đến **AWS DeepRacer**.

Trước đây, tôi chỉ biết AWS DeepRacer là một chiếc xe tự hành có tỷ lệ 1/18, được sử dụng để học Machine Learning và Reinforcement Learning. Người dùng có thể huấn luyện mô hình trên môi trường đám mây, sau đó đưa mô hình xuống thiết bị để thử nghiệm trên đường đua.

Tuy nhiên, sau khi đọc bài viết này, tôi mới biết rằng các thiết bị DeepRacer trước đây chủ yếu khởi động những hệ điều hành được AWS ký xác thực, bao gồm Ubuntu 16.04 và Ubuntu 20.04. Khi những phiên bản này không còn được hỗ trợ, việc cài đặt phần mềm mới và tiếp tục nghiên cứu trên thiết bị sẽ gặp nhiều hạn chế.

Link bài viết: https://www.facebook.com/groups/awsstudygroupfcj/posts/2220039238761036

## Developer Bootloader

Điểm tôi thấy đáng chú ý nhất là AWS đã phát hành **Developer Bootloader**, cho phép nhà phát triển cài đặt hệ điều hành tùy chỉnh hoặc bản phân phối Linux của bên thứ ba trên thiết bị AWS DeepRacer.

Developer Bootloader được xây dựng dựa trên dự án mã nguồn mở `shim`. Khi chữ ký mặc định không được xác thực, Bootloader sẽ kiểm tra chứng chỉ do nhà phát triển cung cấp trong thư mục:

```text
/EFI/DEVELOPER/certs/
```

Nhờ đó, người dùng có thể tự quản lý chứng chỉ bằng các công cụ như OpenSSL và `sbsign`, đồng thời vẫn duy trì cơ chế kiểm tra tính toàn vẹn trong quá trình khởi động.

## Dấu hiệu nhận biết Developer Mode

Khi thiết bị sử dụng chứng chỉ của nhà phát triển, hệ thống sẽ hiển thị những cảnh báo rõ ràng:

- Hiển thị thông báo Developer Mode qua màn hình HDMI.
- Đèn trên xe nhấp nháy thông điệp “DEVELOPER MODE” bằng mã Morse.
- Thêm thời gian chờ trong quá trình khởi động để người dùng nhận biết chế độ đang hoạt động.

Theo tôi, đây là một thiết kế hợp lý vì người dùng có quyền tùy chỉnh thiết bị nhưng vẫn biết rõ trạng thái bảo mật hiện tại.

## Các phương án cài đặt

AWS giới thiệu ba phương án chính để sử dụng Developer Bootloader.

### 1. Sử dụng bản phân phối của cộng đồng

Cộng đồng AWS DeepRacer đã xây dựng một bản phân phối mới dựa trên Ubuntu 24.04 và ROS2 Jazzy. Bản phân phối này đã tích hợp sẵn Developer Bootloader, giúp người dùng bắt đầu nhanh hơn mà không cần tự cấu hình toàn bộ quá trình khởi động Linux.

### 2. Cài đặt bản Linux của bên thứ ba

Người dùng có thể cài đặt Ubuntu hoặc một bản Linux khác, sau đó thay tệp `BOOTX64.EFI` bằng Developer Shim và bổ sung chứng chỉ phù hợp.

Phương án này phù hợp với những người muốn kiểm soát phiên bản hệ điều hành, thư viện, Driver và các công cụ được cài đặt trên thiết bị.

### 3. Tự xây dựng hệ điều hành

Đối với những nhà phát triển muốn tùy chỉnh sâu hơn, AWS cũng cho phép tự xây dựng hệ điều hành dành cho DeepRacer.

Developer Shim được đặt tại:

```text
EFI/BOOT/BOOTX64.EFI
```

Kernel hoặc Bootloader của hệ điều hành được đặt tại:

```text
EFI/BOOT/GRUBX64.EFI
```

Chứng chỉ công khai được lưu trong thư mục `DEVELOPER/certs` để xác thực trước khi khởi động.

## Điều tôi rút ra

Sau khi tìm hiểu bài viết, tôi nhận thấy cập nhật này không chỉ giúp cài một phiên bản Ubuntu mới hơn mà còn mở rộng đáng kể khả năng sử dụng của AWS DeepRacer.

Người dùng có thể:

- Cài đặt các hệ điều hành Linux hiện đại.
- Thử nghiệm các gói ROS2 mới.
- Bổ sung Driver cho phần cứng tùy chỉnh.
- Phát triển thuật toán điều khiển xe.
- Xây dựng các dự án Machine Learning và Robotics.
- Kéo dài thời gian sử dụng của thiết bị DeepRacer.

Một điểm quan trọng khác là quá trình này có thể đảo ngược. Khi cần thiết, người dùng vẫn có thể khôi phục thiết bị về cấu hình ban đầu của AWS.

## Kết luận

Qua bài viết này, tôi hiểu rõ hơn cách AWS mở rộng khả năng phát triển trên thiết bị DeepRacer bằng Developer Bootloader.

Thay vì chỉ sử dụng DeepRacer để học Reinforcement Learning và tham gia các cuộc đua, nhà phát triển giờ đây có thể kiểm soát hệ điều hành, phần mềm và môi trường phát triển trên thiết bị.

Theo tôi, đây là một cập nhật hữu ích đối với cộng đồng AWS DeepRacer vì vừa kéo dài vòng đời phần cứng, vừa tạo thêm nhiều cơ hội nghiên cứu về Machine Learning, ROS2 và Robotics.

<p align="center">
  <strong>Bài viết tham khảo:</strong>
  <a href="https://aws.amazon.com/vi/blogs/machine-learning/custom-os-installation-now-available-on-aws-deepracer-devices/" target="_blank">
    AWS Artificial Intelligence Blog
  </a>
</p>