---
title: "Argo CD"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 2.2 </b>"
---

#### Argo CD

Chuẩn bị môi trường cho phần này:

```bash timeout=300 wait=120 hook=install
$ prepare-environment automation/gitops/argocd
```

Điều này sẽ thực hiện các thay đổi sau đây trong môi trường lab của bạn:

- Tạo một kho lưu trữ AWS CodeCommit
- Cài đặt ArgoCD trong cụm Amazon EKS

[Argo CD](https://argoproj.github.io/cd/) là một công cụ continuous delivery GitOps có tính khai báo cho Kubernetes. Bộ điều khiển Argo CD trong cụm Kubernetes liên tục giám sát trạng thái của cụm của bạn và so sánh nó với trạng thái mong muốn được xác định trong Git. Nếu trạng thái của cụm không khớp với trạng thái mong muốn, Argo CD báo cáo sự sai lệch và cung cấp các hình ảnh minh họa để giúp các nhà phát triển đồng bộ trạng thái của cụm một cách thủ công hoặc tự động với trạng thái mong muốn.

Argo CD cung cấp 3 cách để quản lý trạng thái ứng dụng của bạn:

- CLI - Một CLI mạnh mẽ cho phép bạn tạo các định nghĩa tài nguyên YAML cho các ứng dụng của bạn và đồng bộ chúng với cụm của bạn.
- Giao diện người dùng đô họa - Một giao diện người dùng dựa trên web cho phép bạn làm những điều tương tự như bạn có thể làm với CLI. Nó cũng cho phép bạn hình dung các tài nguyên Kubernetes thuộc các ứng dụng Argo CD mà bạn tạo.
- Tài liệu Kubernetes và Helm Chart được áp dụng vào cụm.

![EKS](/images/0006/00041.png?featherlight=false&width=90pc)