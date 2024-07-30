---
title: "Cập nhật ứng dụng"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 2.2.5 </b>"
---

#### Cập nhật ứng dụng

Bây giờ chúng ta có thể sử dụng Argo CD và Kustomize để triển khai các bản vá vào các tập tin cấu hình ứng dụng của chúng ta bằng cách sử dụng GitOps

Ví dụ, hãy tăng số lượng `replicas` cho `ui` deployment thành `3`

<!--
```kustomization
modules/automation/gitops/argocd/update-application/deployment-patch.yaml
Deployment/ui
```

Sao chép tập tin vá vào thư mục kho Git:

```bash
$ cp /workspace/modules/automation/gitops/argocd/update-application/deployment-patch.yaml ~/environment/argocd/apps/deployment-patch.yaml
```

Bạn có thể xem lại các thay đổi dự định trong tập tin `apps/deployment-patch.yaml`

Để áp dụng các bản vá, bạn có thể chỉnh sửa tập tin `apps/kustomization.yaml` như trong ví dụ dưới đây:

```file
manifests/modules/automation/gitops/argocd/update-application/kustomization.yaml.example
```

Sao chép tập tin `kustomization.yaml` đã chỉnh sửa vào thư mục kho Git:

```bash
$ cp /workspace/modules/automation/gitops/argocd/update-application/kustomization.yaml.example ~/environment/argocd/apps/kustomization.yaml
```
-->

Bạn có thể thực hiện các lệnh sau để thêm các thay đổi cần thiết vào tập tin `apps/deployment.yaml`:

```bash
$ yq -i '.spec.replicas = 3' ~/environment/argocd/apps/deployment.yaml
```

Đẩy các thay đổi lên kho Git

```bash
$ git -C ~/environment/argocd add .
$ git -C ~/environment/argocd commit -am "Cập nhật số lượng replicas cho dịch vụ UI"
$ git -C ~/environment/argocd push
```

Nhấp vào `Refresh` và `Sync` trong giao diện ArgoCD hoặc sử dụng CLI `argocd` để `Sync` ứng dụng:

```bash
$ argocd app sync apps
```

Bây giờ chúng ta nên có 3 pods trong `ui` deployment

![EKS](/images/0006/00046.png?featherlight=false&width=90pc)

Để xác minh, chạy các lệnh sau:

```bash hook=update
$ kubectl get deployment -n ui ui
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
ui     3/3     3            3           3m33s
$ kubectl get pod -n ui
NAME                 READY   STATUS    RESTARTS   AGE
ui-6d5bb7b95-hzmgp   1/1     Running   0          61s
ui-6d5bb7b95-j28ww   1/1     Running   0          61s
ui-6d5bb7b95-rjfxd   1/1     Running   0          3m34s
```

