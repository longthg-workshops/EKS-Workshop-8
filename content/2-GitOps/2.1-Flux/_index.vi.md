---
title: "Flux"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 2.1 </b>"
---

#### Flux

Chuẩn bị môi trường cho phần này:

```bash timeout=300 wait=30
$ prepare-environment automation/gitops/flux
```

Quá trình này sẽ thực hiện các thay đổi sau vào môi trường lab của bạn:

- Tạo một kho lưu trữ AWS CodeCommit
- Tạo một người dùng IAM có quyền truy cập vào kho lưu trữ CodeCommit
- Tạo một đường ống tích hợp liên tục cho [thành phần giao diện ứng dụng mẫu](https://github.com/aws-containers/retail-store-sample-app)

Bạn có thể xem Terraform áp dụng các thay đổi này [tại đây](https://github.com/VAR::MANIFESTS_OWNER/VAR::MANIFESTS_REPOSITORY/tree/VAR::MANIFESTS_REF/manifests/modules/automation/gitops/flux/.workshop/terraform).

Flux giữ các cụm Kubernetes đồng bộ với cấu hình được giữ trong quản lý phiên bản như kho lưu trữ Git và tự động hóa cập nhật cho cấu hình đó khi có mã mới để triển khai. Nó được xây dựng bằng cách sử dụng máy chủ mở rộng API của Kubernetes và có thể tích hợp với Prometheus và các thành phần cốt lõi khác của hệ sinh thái Kubernetes. Flux hỗ trợ đa người dùng và đồng bộ hóa một số lượng tùy ý các kho lưu trữ Git.