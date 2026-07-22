---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---


# Giới thiệu bộ công cụ di chuyển API LBC Ingress-to-Gateway

Trong quá trình tìm hiểu về Amazon EKS và cách định tuyến lưu lượng trong Kubernetes, tôi đọc được một bài viết của AWS giới thiệu **LBC Ingress-to-Gateway API Migration Toolkit**.

Trước đây, Ingress thường được sử dụng để đưa lưu lượng từ bên ngoài vào các Service trong Kubernetes. Khi triển khai trên AWS, AWS Load Balancer Controller có thể đọc tài nguyên Ingress và tự động tạo Application Load Balancer, Listener, Target Group cùng các quy tắc định tuyến phù hợp.

Tuy nhiên, khi hệ thống ngày càng phức tạp, Ingress bắt đầu bộc lộ một số hạn chế. Nhiều tính năng phải được cấu hình thông qua Annotation, khiến tệp YAML dài, khó kiểm tra và phụ thuộc nhiều vào từng Controller.

Gateway API được phát triển để giải quyết vấn đề này bằng cách cung cấp mô hình tài nguyên rõ ràng, có cấu trúc và dễ mở rộng hơn.

Link bài viết: https://www.facebook.com/groups/awsstudygroupfcj/posts/2220037625427864

## Vì sao nên chuyển sang Gateway API?

Gateway API được xem là hướng phát triển lâu dài cho việc quản lý lưu lượng trong Kubernetes.

Thay vì tập trung phần lớn cấu hình trong một tài nguyên Ingress, Gateway API chia trách nhiệm thành nhiều loại tài nguyên khác nhau:

- `GatewayClass` xác định loại Controller chịu trách nhiệm xử lý Gateway.
- `Gateway` mô tả điểm tiếp nhận lưu lượng, Listener và giao thức.
- `HTTPRoute` mô tả Hostname, Path, Backend và quy tắc định tuyến.
- `ReferenceGrant` cho phép tham chiếu tài nguyên giữa các Namespace.
- Các Policy Resource hỗ trợ mở rộng hành vi theo từng nhu cầu cụ thể.

Cách tổ chức này giúp phân tách rõ vai trò giữa nhóm quản trị hạ tầng và nhóm phát triển ứng dụng.

Nhóm hạ tầng có thể quản lý Gateway, Load Balancer và Listener, trong khi nhóm ứng dụng chỉ cần quản lý HTTPRoute của dịch vụ mình phụ trách.

Gateway API còn hỗ trợ các khả năng như:

- Chia lưu lượng theo trọng số.
- Canary Deployment.
- Định tuyến giữa nhiều Namespace.
- Kiểm tra Schema ngay khi áp dụng tài nguyên.
- Mở rộng bằng các tài nguyên có kiểu dữ liệu rõ ràng.
- Quản lý quyền truy cập tốt hơn trong môi trường nhiều nhóm phát triển.

## Khó khăn khi chuyển đổi thủ công

Việc chuyển từ LBC Ingress sang Gateway API không chỉ đơn giản là đổi `kind: Ingress` thành `kind: Gateway`.

Một cấu hình Ingress thực tế có thể chứa nhiều Annotation liên quan đến:

- Scheme của Load Balancer.
- Target Type.
- HTTP và HTTPS Listener.
- Chứng chỉ AWS Certificate Manager.
- Chuyển hướng HTTP sang HTTPS.
- Health Check Path.
- Quy tắc định tuyến theo Host và Path.
- URI Rewrite.
- IngressGroup.
- Cấu hình Target Group.

Khi chuyển đổi bằng tay, người quản trị phải xác định từng Annotation tương ứng với tài nguyên nào trong Gateway API.

Ví dụ:

- `ingressClassName: alb` được ánh xạ thành GatewayClass.
- Scheme và Certificate ARN được đưa vào LoadBalancerConfiguration.
- HTTP và HTTPS Port được chuyển thành các Listener trong Gateway.
- SSL Redirect được chuyển thành HTTPRoute với RequestRedirect Filter.
- Hostname, Path và Service được chuyển thành HTTPRoute.
- Health Check và Target Type được chuyển thành TargetGroupConfiguration.

Nếu chuyển đổi sai một Listener, Certificate hoặc Routing Rule, lưu lượng Production có thể bị gián đoạn. Vì vậy, AWS giới thiệu Migration Toolkit nhằm giảm thao tác thủ công và hỗ trợ kiểm tra trước khi thay đổi hệ thống thật.

## Các thành phần của Migration Toolkit

Bộ công cụ gồm hai thành phần chính.

### 1. lbc-migrate CLI

`lbc-migrate` là công cụ dòng lệnh giúp đọc cấu hình LBC Ingress và sinh ra các tài nguyên Gateway API tương ứng.

Công cụ có thể xử lý:

- Annotation của AWS Load Balancer Controller.
- Host và Path Rule.
- TLS Termination.
- SSL Redirect.
- IngressGroup.
- Load Balancer Configuration.
- Target Group Configuration.

Công cụ có thể đọc từ tệp YAML hoặc đọc trực tiếp từ một Kubernetes Cluster đang hoạt động.

Các Deployment và Service hiện có vẫn được giữ nguyên. `lbc-migrate` chủ yếu tạo lại lớp định tuyến, bao gồm GatewayClass, Gateway, HTTPRoute và những tài nguyên cấu hình liên quan.

### 2. Migration Console

Migration Console là giao diện Web chạy cục bộ và được tích hợp trong `lbc-migrate`.

Giao diện này cho phép người quản trị:

- Xem cấu hình Ingress hiện tại.
- Xem Gateway API Manifest được tạo.
- So sánh kế hoạch trước và sau chuyển đổi.
- Kiểm tra Dry-run Plan.
- Phát hiện những Annotation cần chỉnh sửa thủ công.

Migration Console chỉ hỗ trợ xem và kiểm tra, không tự động thay đổi tài nguyên Production.

## Điều kiện chuẩn bị

Trước khi thực hiện chuyển đổi, cần chuẩn bị:

- Amazon EKS Cluster đang hoạt động.
- Managed Node Group hoặc Node phù hợp cho Workload.
- AWS Load Balancer Controller hỗ trợ Gateway API.
- Gateway API Standard CRDs.
- LBC Gateway CRDs.
- Ứng dụng đang chạy sau một ALB Ingress.
- Chứng chỉ AWS Certificate Manager nếu sử dụng HTTPS.
- Quyền `kubectl` phù hợp để tạo và kiểm tra tài nguyên.

Có thể cài Gateway API Standard CRDs bằng lệnh:

```bash
kubectl apply --server-side=true \
  -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.5.0/standard-install.yaml
```

Cài các CRD dành cho AWS Load Balancer Controller:

```bash
kubectl apply \
  -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v3.4.0/config/crd/gateway/gateway-crds.yaml
```

Sau đó, kiểm tra Controller đã nhận Gateway API hay chưa:

```bash
kubectl -n kube-system logs deploy/aws-load-balancer-controller \
  | grep "gateway.k8s.aws/alb"
```

## Quy trình chuyển đổi

Quá trình chuyển đổi có thể thực hiện theo sáu bước.

### Bước 1 – Cài đặt lbc-migrate

Tải mã nguồn AWS Load Balancer Controller và build công cụ:

```bash
git clone https://github.com/kubernetes-sigs/aws-load-balancer-controller.git
cd aws-load-balancer-controller

make lbc-migrate
make install-lbc-migrate
```

Sau khi hoàn thành, có thể sử dụng lệnh `lbc-migrate` trong Terminal.

### Bước 2 – Chuyển Ingress thành Gateway API Manifest

Có thể đọc cấu hình trực tiếp từ Cluster:

```bash
lbc-migrate \
  --from-cluster \
  --namespaces demo \
  --output-dir ./gateway-output
```

Công cụ sẽ đọc các tài nguyên liên quan như:

- Ingress.
- Service.
- IngressClass.
- IngressClassParams.

Sau đó, công cụ tạo tệp YAML chứa các tài nguyên Gateway API tương ứng.

Một kết quả chuyển đổi có thể gồm:

- GatewayClass.
- Gateway.
- HTTPRoute dành cho SSL Redirect.
- HTTPRoute dành cho định tuyến ứng dụng.
- LoadBalancerConfiguration.
- TargetGroupConfiguration.

Nếu có Annotation chưa được hỗ trợ đầy đủ, công cụ sẽ hiển thị cảnh báo thay vì cố tạo cấu hình có thể sai.

## Bước 3 – Kiểm tra bằng Dry Run

Theo mặc định, Gateway được tạo có Annotation Dry Run:

```yaml
gateway.k8s.aws/dry-run: "true"
```

Có thể áp dụng Manifest như sau:

```bash
kubectl apply -f ./gateway-output/gateway-resources.yaml
```

Ở chế độ Dry Run, Controller sẽ phân tích và mô phỏng các tài nguyên AWS cần tạo nhưng chưa tạo Application Load Balancer thật.

Bước này giúp kiểm tra:

- Listener.
- Routing Rule.
- Certificate.
- Target Group.
- Health Check.
- Annotation Mapping.
- Những phần cần chỉnh sửa thủ công.

Đây là bước nên thực hiện trước khi triển khai vào môi trường Production.

## Bước 4 – Xử lý cấu hình chưa chuyển đổi hoàn toàn

Không phải Annotation nào của Ingress cũng có thể chuyển tự động sang Gateway API.

Một ví dụ là URI Rewrite sử dụng biểu thức chính quy và Capture Group.

Ingress có thể sử dụng một cấu hình để xóa tiền tố `/api` trước khi gửi Request đến Backend. Tuy nhiên, Gateway API sử dụng URLRewrite Filter có cấu trúc khác.

Trong trường hợp này, người quản trị có thể bổ sung Filter thủ công:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: demo-route
  namespace: demo
spec:
  hostnames:
    - demo.example.com
  parentRefs:
    - name: demo-gateway
      sectionName: https-443
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api
      filters:
        - type: URLRewrite
          urlRewrite:
            path:
              type: ReplacePrefixMatch
              replacePrefixMatch: /
      backendRefs:
        - name: echoserver
          port: 80
```

Cấu hình này sẽ chuyển Request:

```text
/api/hello
```

thành:

```text
/hello
```

trước khi gửi đến Backend Service.

## Bước 5 – Triển khai Gateway thật và kiểm tra

Sau khi kiểm tra Dry Run, tạo lại Manifest với Dry Run bị tắt:

```bash
lbc-migrate \
  --from-cluster \
  --namespaces demo \
  --output-dir ./gateway-live \
  --dry-run=false
```

Áp dụng tài nguyên:

```bash
kubectl apply -f ./gateway-live/gateway-resources.yaml
```

Lúc này, AWS Load Balancer Controller sẽ tạo một ALB mới dành cho Gateway API. ALB cũ của Ingress vẫn tiếp tục hoạt động.

Kiểm tra trạng thái Gateway:

```bash
kubectl get gateway demo-gateway -n demo
```

Gateway nên đạt trạng thái:

```text
PROGRAMMED: True
```

Tiếp theo, cần kiểm tra:

- Target Group có Healthy hay không.
- HTTP có chuyển hướng sang HTTPS hay không.
- Chứng chỉ TLS có hợp lệ hay không.
- Path Routing có hoạt động đúng hay không.
- URI Rewrite có cho kết quả giống hệ thống cũ hay không.
- CloudWatch Metrics của ALB mới có bất thường hay không.

## Bước 6 – Chuyển lưu lượng và dọn tài nguyên cũ

Sau khi Gateway ALB hoạt động ổn định, lưu lượng có thể được chuyển dần từ Ingress ALB sang Gateway ALB.

Có thể sử dụng:

- Amazon Route 53 Weighted Routing.
- AWS Global Accelerator.
- Cơ chế quản lý Traffic khác phù hợp với hệ thống.

Việc chuyển dần lưu lượng giúp phát hiện lỗi trước khi toàn bộ người dùng được chuyển sang kiến trúc mới.

Nếu xảy ra vấn đề, có thể chuyển lưu lượng trở lại Ingress ALB.

Khi hệ thống mới đã ổn định, xóa Ingress cũ:

```bash
kubectl delete ingress demo -n demo
```

AWS Load Balancer Controller sẽ tự động dọn ALB, Listener Rule và Target Group liên quan đến Ingress cũ.

## Khuyến nghị khi thực hiện Migration

Qua quá trình tìm hiểu, tôi rút ra một số điểm quan trọng:

- Luôn thực hiện Dry Run trước khi tạo ALB thật.
- Kiểm tra bảng hỗ trợ Annotation trước khi chuyển đổi.
- Không xóa Ingress cũ ngay sau khi tạo Gateway.
- Giữ hai ALB hoạt động song song trong thời gian kiểm thử.
- Chuyển lưu lượng theo từng phần thay vì chuyển toàn bộ một lần.
- Theo dõi Target Health và CloudWatch Metrics.
- Chuẩn bị sẵn phương án Rollback.
- Kiểm tra kỹ URI Rewrite, Authentication và các Annotation tùy chỉnh.
- Với Workload mới, nên cân nhắc sử dụng HTTPRoute ngay từ đầu.

## Lợi ích của Migration Toolkit

Bộ công cụ mang lại nhiều lợi ích:

- Giảm thời gian chuyển đổi Manifest.
- Hạn chế lỗi khi ánh xạ Annotation.
- Tạo Gateway API Resource có cấu trúc rõ ràng.
- Hỗ trợ kiểm tra Dry Run trước khi tạo tài nguyên AWS.
- Cho phép Ingress và Gateway hoạt động song song.
- Giúp quá trình chuyển lưu lượng an toàn hơn.
- Giữ nguyên Deployment và Service hiện có.
- Cảnh báo những cấu hình cần chỉnh sửa thủ công.
- Hỗ trợ xây dựng lộ trình chuyển đổi theo từng giai đoạn.

## Kết luận

Qua bài viết này, tôi hiểu rõ hơn về cách chuyển đổi từ AWS Load Balancer Controller Ingress sang Gateway API trên Amazon EKS.

Gateway API không chỉ thay đổi cú pháp YAML mà còn cung cấp mô hình quản lý lưu lượng rõ ràng hơn, hỗ trợ phân quyền, mở rộng và triển khai nhiều chiến lược Routing hiện đại.

`lbc-migrate` giúp tự động hóa phần lớn quá trình chuyển đổi, trong khi Migration Console và cơ chế Dry Run giúp kiểm tra cấu hình trước khi tác động đến môi trường thật.

Theo tôi, điểm quan trọng nhất của giải pháp là ALB mới có thể chạy song song với ALB cũ. Nhờ đó, người quản trị có thể kiểm thử, chuyển lưu lượng theo từng giai đoạn và Rollback khi cần thiết.

Đây là một bộ công cụ hữu ích cho những hệ thống đang vận hành Amazon EKS bằng LBC Ingress và có kế hoạch sử dụng Gateway API làm giao diện định tuyến lâu dài.

<p align="center">
  <strong>Bài viết tham khảo:</strong>
  <a href="https://aws.amazon.com/vi/blogs/networking-and-content-delivery/introducing-the-lbc-ingress-to-gateway-api-migration-toolkit/" target="_blank">
    AWS Networking & Content Delivery Blog
  </a>
</p>