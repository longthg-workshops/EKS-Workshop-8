---
title: "Accessing Argo CD"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 2.2.2 </b>"
---

#### Accessing Argo CD

Cho mục đích của bài thực hành này, giao diện Argo CD server đã được tiếp cận bên ngoài cụm sử dụng Dịch vụ Kubernetes loại `Load Balancer`. 


Nếu bạn cần đặt lại mật khẩu `admin` của Argo CD, bạn có thể sử dụng các lệnh sau:

```bash
$ kubectl -n argocd patch secret argocd-secret -p '{"data": {"admin.password": null, "admin.passwordMtime": null}}'
$ kubectl -n argocd rollout restart deployment/argocd-server
$ kubectl -n argocd rollout status deploy/argocd-server
```


Để lấy URL từ dịch vụ Argo CD, chạy lệnh sau:

```bash
$ ARGOCD_SERVER=$(kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].hostname')
$ echo "URL ArgoCD: http://$ARGOCD_SERVER"
URL ArgoCD: http://acfac042a61e5467aace45fc66aee1bf-818695545.us-west-2.elb.amazonaws.com
```

Tên người dùng ban đầu là `admin`. Mật khẩu được tạo tự động. Bạn có thể lấy nó bằng cách chạy lệnh sau:

```bash
$ ARGOCD_PWD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
$ echo "Mật khẩu quản trị ArgoCD: $ARGOCD_PWD"
```

Đăng nhập vào giao diện Argo CD sử dụng URL và thông tin đăng nhập bạn vừa lấy được.

Bạn sẽ nhìn thấy một màn hình giống như sau:

![EKS](/images/0006/00042.png?featherlight=false&width=90pc)

Argo CD cũng cung cấp một công cụ CLI mạnh mẽ gọi là `argocd` có thể được sử dụng để quản lý ứng dụng.

:::info
Cho mục đích của bài thực hành này, CLI `argocd` đã được cài đặt cho bạn. Bạn có thể tìm hiểu thêm về cách cài đặt công cụ CLI bằng cách tuân theo [hướng dẫn](https://argoproj.github.io/argo-cd/cli_installation/).
:::

Để tương tác với các đối tượng Argo CD bằng CLI, chúng ta cần đăng nhập vào máy chủ Argo CD bằng cách chạy các lệnh sau:

```bash
$ argocd login $(kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].hostname') --username admin --password $(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d) --insecure
'admin:login' đã đăng nhập thành công
Ngữ cảnh 'acfac042a61e5467aace45fc66aee1bf-818695545.us-west-2.elb.amazonaws.com' được cập nhật
```