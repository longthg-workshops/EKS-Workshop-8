---
title: "Setup"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 2.3.1 </b>"
---

#### Setup

Trước khi bắt đầu thiết lập các ứng dụng Argo CD, hãy xóa ứng dụng Argo CD `Application` mà chúng ta đã tạo cho `ui`:

```bash wait=30
$ argocd app delete apps --cascade -y
```

Chúng ta tạo các mẫu cho một tập hợp các ứng dụng ArgoCD bằng cách sử dụng phương pháp DRY trong các biểu đồ Helm:

```
.
|-- app-of-apps
|   |-- Chart.yaml
|   |-- templates
|   |   |-- _application.yaml
|   |   `-- application.yaml
|   `-- values.yaml
`-- apps-kustomization
    ...
```

`Chart.yaml` là một mẫu cơ bản. `templates` chứa một tệp mẫu sẽ được sử dụng để tạo các ứng dụng được định nghĩa trong `values.yaml`.

`values.yaml` cũng chứa các giá trị cụ thể cho một môi trường nhất định và sẽ được áp dụng cho tất cả các mẫu ứng dụng.

```file
manifests/modules/automation/gitops/argocd/app-of-apps/values.yaml
```

Đầu tiên, sao chép cấu hình `App of Apps` như đã mô tả ở trên vào thư mục Repository Git:

```bash
$ cp -R ~/environment/eks-workshop/modules/automation/gitops/argocd/app-of-apps ~/environment/argocd/
$ yq -i ".spec.source.repoURL = env(GITOPS_REPO_URL_ARGOCD)" ~/environment/argocd/app-of-apps/values.yaml
```

Tiếp theo, đẩy các thay đổi lên Repository Git:

```bash wait=10
$ git -C ~/environment/argocd add .
$ git -C ~/environment/argocd commit -am "Adding App of Apps"
$ git -C ~/environment/argocd push
```

Cuối cùng, chúng ta cần tạo một `Application` Argo CD mới để hỗ trợ mẫu `App of Apps`.
Chúng ta xác định một đường dẫn mới đến Argo CD `Application` bằng cách sử dụng `--path app-of-apps`.

Chúng ta cũng kích hoạt `Application` ArgoCD để tự động [đồng bộ hóa](https://argo-cd.readthedocs.io/en/stable/user-guide/auto_sync/) trạng thái trong cụm với cấu hình trong Repository Git bằng cách sử dụng `--sync-policy automated`.

```bash
$ argocd app create apps --repo $GITOPS_REPO_URL_ARGOCD \
  --dest-server https://kubernetes.default.svc \
  --sync-policy automated --self-heal --auto-prune \
  --set-finalizer \
  --upsert \
  --path app-of-apps
 application 'apps' created
```

Khoảng thời gian `Refresh` mặc định là 3 phút (180 giây). Bạn có thể thay đổi khoảng thời gian bằng cách cập nhật giá trị `timeout.reconciliation` trong ConfigMap `argocd-cm`. Nếu khoảng thời gian là 0 thì Argo CD sẽ không tự động kiểm tra Repository Git và phương pháp thay thế như webhooks và / hoặc đồng bộ thủ công nên được sử dụng.

Cho mục đích đào tạo, hãy đặt khoảng thời gian `Refresh` thành 5 giây và khởi động lại bộ điều khiển ứng dụng ArgoCD để triển khai các thay đổi của chúng ta nhanh hơn:

```bash wait=30
$ kubectl patch configmap/argocd-cm -n argocd --type merge \
  -p '{"data":{"timeout.reconciliation":"5s"}}'
$ kubectl -n argocd rollout restart deploy argocd-repo-server
$ kubectl -n argocd rollout status deploy/argocd-repo-server
$ kubectl -n argocd rollout restart statefulset argocd-application-controller
$ kubectl -n argocd rollout status statefulset argocd-application-controller
```

Mở giao diện người dùng Argo CD và điều hướng đến ứng dụng `apps`.

![EKS](/images/0006/00048.png?featherlight=false&width=90pc)

Nhấp vào `Refresh` và `Sync` trong giao diện người dùng ArgoCD, sử dụng `argocd` CLI để `Sync` ứng dụng hoặc chờ cho đến khi `Sync` tự động hoàn thành:

```bash
$ argocd app sync apps
```

Chúng ta đã triển khai và đồng bộ hóa `App of Apps Application` của Argo CD.

Các ứng dụng của chúng tôi, ngoại trừ ứng dụng Argo CD `App of Apps Application`, đang ở trạng thái `Unknown` vì chúng ta chưa triển khai cấu hình của chúng.

![EKS](/images/0006/00049.png?featherlight=false&width=90pc)

Chúng ta sẽ triển khai cấu hình ứng dụng cho các ứng dụng trong bước tiếp theo.