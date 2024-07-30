---
title: "Cập nhật ứng dụng"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 2.3.3 </b>"
---

Bây giờ chúng ta có thể sử dụng Argo CD và Kustomize để triển khai các bản vá vào các tệp mô tả ứng dụng của chúng ta bằng cách sử dụng GitOps. Ví dụ, chúng ta sẽ tăng số lượng `replicas` cho việc triển khai `ui` lên `3`.

Bạn có thể thực thi các lệnh để thêm các thay đổi cần thiết vào tệp `apps-kustomization/ui/deployment-patch.yaml`:

```bash
$ yq -i '.spec.replicas = 3' ~/environment/argocd/apps-kustomization/ui/deployment-patch.yaml
```

Bạn có thể xem lại các thay đổi dự định trong tệp `apps-kustomization/ui/deployment-patch.yaml`.

```kustomization
modules/automation/gitops/argocd/update-application/deployment-patch.yaml
Deployment/ui
```

Đẩy các thay đổi lên kho Git:

```bash
$ git -C ~/environment/argocd add .
$ git -C ~/environment/argocd commit -am "Cập nhật số bản sao dịch vụ UI"
$ git -C ~/environment/argocd push
```

Nhấp vào `Refresh` và `Sync` trong giao diện người dùng Argo CD, sử dụng CLI `argocd` để `Sync` ứng dụng hoặc đợi cho đến khi `Sync` tự động hoàn thành:

```bash
$ argocd app sync ui
```

![argocd-update-application](../assets/argocd-update-application.png)

Để xác minh, chạy các lệnh sau:

```bash hook=update
$ kubectl get deployment -n ui ui
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
ui     3/3     3            3           3m33s
$ kubectl get pod -n ui
NAME                  READY   STATUS    RESTARTS   AGE
ui-6d5bb7b95-hzmgp   1/1     Running   0          61s
ui-6d5bb7b95-j28ww   1/1     Running   0          61s
ui-6d5bb7b95-rjfxd   1/1     Running   0          3m34s
```

