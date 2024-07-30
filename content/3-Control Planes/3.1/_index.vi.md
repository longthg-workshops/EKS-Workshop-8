---
title: "ACK"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 3.1 </b>"
---

#### ACK

#### Chuẩn bị môi trường cho phần này:

```bash timeout=300 wait=30
$ prepare-environment automation/controlplanes/ack
```

Điều này sẽ thực hiện các thay đổi sau vào môi trường lab của bạn:

- Cài đặt AWS Controllers for DynamoDB trong cụm Amazon EKS
- Cài đặt AWS Load Balancer controller trong cụm Amazon EKS

Dự án [AWS Controllers for Kubernetes (ACK)](https://aws-controllers-k8s.github.io/community/) cho phép bạn xác định và sử dụng tài nguyên dịch vụ AWS trực tiếp từ Kubernetes bằng cách sử dụng các cấu trúc YAML quen thuộc.

Với ACK, bạn có thể tận dụng việc sử dụng các dịch vụ AWS như cơ sở dữ liệu ([RDS](https://aws-controllers-k8s.github.io/community/docs/tutorials/rds-example/) hoặc các dịch vụ khác) và/hoặc hàng đợi ([SQS](https://aws-controllers-k8s.github.io/community/docs/tutorials/sqs-example/) vv) cho các ứng dụng Kubernetes của bạn mà không cần phải xác định tài nguyên một cách thủ công bên ngoài cụm. Điều này giảm thiểu tổng thể sự phức tạp cho việc quản lý các phụ thuộc của ứng dụng của bạn.

Ứng dụng mẫu có thể chạy hoàn toàn trong cụm của bạn, bao gồm các tải trọng có trạng thái như cơ sở dữ liệu và hàng đợi tin nhắn. Điều này là một phương pháp tốt khi bạn đang phát triển ứng dụng. Tuy nhiên, khi nhóm muốn làm cho ứng dụng có sẵn trong các giai đoạn khác như kiểm thử và sản xuất, họ sẽ sử dụng các dịch vụ được quản lý bởi AWS như cơ sở dữ liệu Amazon DynamoDB và môi trường lưu trữ Amazon MQ. Điều này cho phép nhóm tập trung vào khách hàng và các dự án kinh doanh của mình mà không cần phải quản lý cơ sở dữ liệu hoặc các trình quản lý hàng đợi tin nhắn.

Trong lab này, chúng ta sẽ tận dụng ACK để cung cấp các dịch vụ này và tạo các secrets và configmaps chứa thông tin kết nối ứng dụng với các dịch vụ được quản lý bởi AWS này.

![EKS](/images/0006/00052.png?featherlight=false&width=90pc)