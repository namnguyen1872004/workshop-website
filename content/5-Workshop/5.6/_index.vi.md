---
title: "S3 cho POD, chữ ký và minh chứng"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

S3 được tích hợp trước SQS để hoàn tất luồng nghiệp vụ và loại bỏ phụ thuộc local disk. Khi EC2 bị thay thế, media vẫn tồn tại độc lập và ứng dụng có thể mở rộng.

5.6.1. Bucket, encryption và prefixes
Tạo bucket `delivery-dev-pod-<unique-suffix>` tại `ap-southeast-1`, bật Block Public Access và default encryption. Với workshop có thể dùng SSE-S3; nếu dùng SSE-KMS phải cấp đúng quyền key cho EC2 role.

| Loại dữ liệu | Object key chuẩn |
|---|---|
| Ảnh POD | pod/{orderId}/image/{guid}.png |
| Chữ ký | pod/{orderId}/signature/{guid}.png |
| Minh chứng giao thất bại | failed-evidence/{orderId}/{guid}.png |
| Gói triển khai | deployments/{release}.tar.gz |
![Hinh25](/workshop-website/images/5-Workshop/image25.png)
![Hinh26](/workshop-website/images/5-Workshop/image26.png)
5.6.2. S3 Gateway VPC Endpoint
Tạo Gateway Endpoint `com.amazonaws.ap-southeast-1.s3` và associate với `delivery-app-rt`. EC2 private truy cập S3 mà không cần đi qua NAT cho traffic S3.
![Hinh27](/workshop-website/images/5-Workshop/image27.png)
Kiểm tra route: Endpoint phải gắn đúng application route table; bucket policy/KMS policy vẫn phải cho phép IAM Role tương ứng.

5.6.3. Thay đổi trong ứng dụng
```env
Storage__BucketName=delivery-dev-pod-<unique-suffix>
AWS_REGION=ap-southeast-1
```
Đăng ký `IAmazonS3` bằng default credential provider chain.
Tạo service `IMediaStorage` để upload, read và delete theo prefix; controller không gọi SDK trực tiếp.
Database chỉ lưu object key, content type, size và checksum cần thiết; không lưu URL public hoặc base64.
Endpoint `/media/...` phải kiểm tra quyền truy cập rồi stream object hoặc tạo presigned URL ngắn hạn.
Khi transaction DB thất bại sau upload, phải có compensating delete hoặc cleanup job cho orphan objects.

5.6.4. Kiểm thử
```bash
printf "delivery-s3-test" | sudo -u deliveryapp aws s3 cp - s3://<BUCKET_NAME>/pod/_test/connectivity.txt --region ap-southeast-1
sudo -u deliveryapp aws s3api head-object --bucket <BUCKET_NAME> --key pod/_test/connectivity.txt --region ap-southeast-1 --query '{Size:ContentLength,Encryption:ServerSideEncryption,KmsKey:SSEKMSKeyId}'
sudo -u deliveryapp aws s3 rm s3://<BUCKET_NAME>/pod/_test/connectivity.txt --region ap-southeast-1
```

| Kịch bản | Bằng chứng đạt |
|---|---|
| Giao thành công | DB có PodImagePath và DriverSignaturePath; S3 có hai object tương ứng. |
| Giao thất bại | Evidence nằm trong `failed-evidence/{orderId}/`. |
| Bảo mật | Bucket không public; object không truy cập bằng URL công khai. |
| Scale/replace EC2 | Media vẫn đọc được sau khi instance thay đổi. |