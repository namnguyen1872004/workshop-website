---
title: "Amazon Location GeoPlaces V2"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

Sau khi event pipeline ổn định, thay Nominatim bằng Amazon Location để toàn bộ core flow sử dụng dịch vụ mục tiêu trước lần kiểm thử end-to-end cuối.

5.8.1. Thiết kế tích hợp
Backend dùng package `AWSSDK.GeoPlaces` và IAM Role để gọi SearchText/ReverseGeocode.
Trình duyệt chỉ gọi API nội bộ `/api/location/search` và `/api/location/reverse`; không nhận AWS credential.
API trả model ổn định gồm `displayName`, `title`, `latitude`, `longitude` để UI không phụ thuộc trực tiếp SDK response.
Không fallback âm thầm về Nominatim sau cutover; nếu cần rollback phải dùng feature flag rõ ràng và log provider.
![Hinh39](/workshop-website/images/5-Workshop/image39.png)
5.8.2. IAM và biến môi trường
```env
AmazonLocation__Region=ap-southeast-1
AWS_REGION=ap-southeast-1
```
```json
{
  "Effect": "Allow",
  "Action": [
    "geo-places:SearchText",
    "geo-places:ReverseGeocode",
    "geo-places:GetPlace",
    "geo-places:Suggest"
  ],
  "Resource": "arn:aws:geo-places:ap-southeast-1::provider/default"
}
```

5.8.3. Kiểm thử CLI và ứng dụng
```bash
sudo -u deliveryapp aws geo-places search-text   --query-text "BHD Star Lê Văn Việt, Thành phố Hồ Chí Minh"   --bias-position 106.6297 10.8231   --filter 'IncludeCountries=VNM'   --max-results 3   --language vi   --intended-use Storage   --region ap-southeast-1

sudo -u deliveryapp aws geo-places reverse-geocode   --query-position 106.778936 10.845672   --max-results 3   --language vi   --intended-use Storage   --region ap-southeast-1
```
1. Đăng nhập và mở trang tạo đơn.
2. Nhập địa chỉ, gọi `/api/location/search?query=...` và nhận HTTP 200.
3. Chọn một kết quả; latitude/longitude và marker phải cập nhật.
4. Click bản đồ; `/api/location/reverse` phải điền lại địa chỉ.
5. Lưu order và kiểm tra Lat/Lng trong database.

Bằng chứng cần bổ sung: Ảnh Network response của hai API nội bộ, marker sau khi chọn địa chỉ và record order có tọa độ. Không chụp AWS credential vì browser không được nhận credential.