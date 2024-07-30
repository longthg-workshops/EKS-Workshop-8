---
title: "Crossplane"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 3.2 </b>"
---

#### Crossplane

#### Chuẩn bị môi trường cho phần này:

```bash timeout=300 wait=120
$ prepare-environment automation/controlplanes/crossplane
```

Điều này sẽ thực hiện các thay đổi sau đây trong môi trường lab của bạn:

- Cài đặt Crossplane và nhà cung cấp AWS trong cụm Amazon EKS
---

[Crossplane](https://crossplane.io/) là một dự án mã nguồn mở trong CNCF giúp biến cụm Kubernetes của bạn thành một trung tâm kiểm soát phổ quát. Crossplane cho phép các nhóm nền tảng tổ hợp cơ sở hạ tầng từ nhiều nhà cung cấp và tiếp cận các API tự phục vụ cấp cao hơn cho các nhóm ứng dụng tiêu thụ mà không cần phải viết mã.

Crossplane mở rộng cụm Kubernetes của bạn để hỗ trợ việc điều phối bất kỳ cơ sở hạ tầng hoặc dịch vụ quản lý nào. Hãy tổ hợp các tài nguyên cụm Crossplane thành các trừơng tự cấp cao hơn có thể được quản lý, triển khai và tiêu thụ bằng cách sử dụng các công cụ yêu thích và quy trình hiện có của bạn.

![EKS](/images/0006/00056.png?featherlight=false&width=90pc)