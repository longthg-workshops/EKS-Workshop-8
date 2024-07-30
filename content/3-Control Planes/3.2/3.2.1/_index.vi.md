---
title: "Crossplane"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 3.2.1 </b>"
---

Chạy Crossplane trong một cụm bao gồm hai phần chính:

1. Bộ điều khiển Crossplane cung cấp các thành phần cốt lõi
2. Một hoặc nhiều nhà cung cấp Crossplane mỗi nhà cung cấp một bộ điều khiển và Định nghĩa Tài nguyên Tùy chỉnh để tích hợp với một nhà cung cấp cụ thể, chẳng hạn như AWS

Bộ điều khiển Crossplane, nhà cung cấp AWS của Upbound đã được cài đặt trước trong cụm EKS của chúng tôi, mỗi bộ điều khiển đều chạy dưới dạng một bản triển khai trong namespace `crossplane-system` cùng với `crossplane-rbac-manager`:

```bash
$ kubectl get deployment -n crossplane-system
NAME                                         READY   UP-TO-DATE   AVAILABLE   AGE
crossplane                                   1/1     1            1           3h7m
crossplane-rbac-manager                      1/1     1            1           3h7m
upbound-aws-provider-dynamodb-23a48a51e223   1/1     1            1           3h6m
upbound-provider-family-aws-1ac09674120f     1/1     1            1           21h
```

Ở đây, `upbound-provider-family-aws` đại diện cho nhà cung cấp Crossplane cho Amazon Web Services (AWS) được phát triển và hỗ trợ bởi Upbound. `upbound-aws-provider-dynamodb` là một phần con của trước đó được dành riêng để triển khai DynamoDB thông qua Crossplane.

Crossplane cung cấp một giao diện đơn giản hóa cho các nhà phát triển để yêu cầu tài nguyên cơ sở hạ tầng thông qua các tuyên bố Kubernetes gọi là yêu cầu. Như được hiển thị trong biểu đồ này, các yêu cầu là các tài nguyên Crossplane giới hạn trong phạm vi không gian tên, đóng vai trò là giao diện của nhà phát triển và trừu tượng hóa các chi tiết triển khai. Khi một yêu cầu được triển khai vào cụm, nó tạo ra một Tài nguyên Composite (XR), một tài nguyên tùy chỉnh Kubernetes đại diện cho một hoặc nhiều tài nguyên đám mây được xác định thông qua các mẫu gọi là Sáng tạo. Tài nguyên Composite tạo ra một hoặc nhiều Tài nguyên Quản lý tương tác với API AWS để yêu cầu tạo ra các tài nguyên cơ sở hạ tầng mong muốn.

![EKS](/images/0006/00057.png?featherlight=false&width=90pc)