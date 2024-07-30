---
title: "App of Apps"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 2.3 </b>"
---

#### App of Apps

[Argo CD](https://argoproj.github.io/cd/) có thể triển khai một tập hợp các ứng dụng lên các môi trường khác nhau (DEV, TEST, PROD ...) bằng cách sử dụng `base` Kubernetes manifests cho các ứng dụng và các tùy chỉnh cụ thể cho một môi trường.

Chúng ta có thể tận dụng [mẫu ứng dụng của Argo CD](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/) để triển khai trường hợp sử dụng này. Mẫu này cho phép chúng ta chỉ định một ứng dụng Argo CD bao gồm các ứng dụng khác.

![EKS](/images/0006/00047.png?featherlight=false&width=90pc)

Chúng ta tham chiếu [kho Git EKS Workshop](https://github.com/aws-samples/eks-workshop-v2/tree/main/environment/eks-workshop/manifests/base) như một kho Git chứa các `base` manifests cho tài nguyên Kubernetes của bạn. Kho này sẽ chứa trạng thái tài nguyên ban đầu cho mỗi ứng dụng.

```
.
|-- manifests
| |-- assets
| |-- carts
| |-- catalog
| |-- checkout
| |-- orders
| |-- other
| |-- rabbitmq
| `-- ui
```

Ví dụ này cho thấy cách sử dụng Helm để tạo cấu hình cho một môi trường cụ thể, ví dụ DEV.
Một bố cục thông thường của một kho Git có thể là:

```
.
|-- app-of-apps
|   |-- ...
`-- apps-kustomization
    ...
```