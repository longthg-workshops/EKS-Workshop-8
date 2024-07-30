---
title: "Triển khai ứng dụng"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 2.3.2 </b>"
---

#### Triển khai ứng dụng

Chúng ta đã cấu hình thành công ứng dụng Argo CD App of Apps, vì vậy bây giờ chúng ta có thể triển khai tùy chỉnh cho từng môi trường cho tập hợp ứng dụng.

Đầu tiên, hãy xóa các ứng dụng hiện có để chúng ta có thể thay thế chúng:

```bash
$ kubectl delete -k ~/environment/eks-workshop/base-application --ignore-not-found=true
namespace "assets" đã được xóa
namespace "carts" đã được xóa
namespace "catalog" đã được xóa
namespace "checkout" đã được xóa
namespace "orders" đã được xóa
namespace "other" đã được xóa
namespace "rabbitmq" đã được xóa
namespace "ui" đã được xóa
...
```

Sau đó, chúng ta cần tạo một tùy chỉnh cho mỗi ứng dụng:

```
.
|-- app-of-apps
|   |-- ...
`-- apps-kustomization
    |-- assets
    |   `-- kustomization.yaml
    |-- carts
    |   `-- kustomization.yaml
    |-- catalog
    |   `-- kustomization.yaml
    |-- checkout
    |   `-- kustomization.yaml
    |-- orders
    |   `-- kustomization.yaml
    |-- other
    |   `-- kustomization.yaml
    |-- rabbitmq
    |   `-- kustomization.yaml
    `-- ui
        |-- deployment-patch.yaml
        `-- kustomization.yaml
```

```file
manifests/modules/automation/gitops/argocd/apps-kustomization/ui/kustomization.yaml
```

Chúng ta xác định một đường dẫn đến các tập tin mẫu Kubernetes cơ bản cho một ứng dụng, trong trường hợp này là `ui`, bằng cách sử dụng `resources`. Chúng ta cũng xác định cấu hình nào sẽ được áp dụng cho ứng dụng `ui` trên cụm EKS bằng cách sử dụng `patches`.

```file
manifests/modules/automation/gitops/argocd/apps-kustomization/ui/deployment-patch.yaml
```

Chúng tôi muốn có `1` bản sao cho ứng dụng `ui`. Tất cả các ứng dụng khác sẽ sử dụng cấu hình từ các tập tin mẫu Kubernetes cơ bản.

Sao chép các tập tin vào thư mục kho lưu trữ Git:

```bash
$ cp -R ~/environment/eks-workshop/modules/automation/gitops/argocd/apps-kustomization ~/environment/argocd/
```

Thư mục Git của bạn cuối cùng bây giờ sẽ trông như sau. Bạn có thể xác nhận điều này bằng cách chạy `tree ~/environment/argocd`:

```
|-- app-of-apps
|   |-- Chart.yaml
|   |-- templates
|   |   |-- _application.yaml
|   |   `-- application.yaml
|   `-- values.yaml
|-- apps
|   ...
`-- apps-kustomization
    |-- assets
    |   `-- kustomization.yaml
    |-- carts
    |   `-- kustomization.yaml
    |-- catalog
    |   `-- kustomization.yaml
    |-- checkout
    |   `-- kustomization.yaml
    |-- orders
    |   `-- kustomization.yaml
    |-- other
    |   `-- kustomization.yaml
    |-- rabbitmq
    |   `-- kustomization.yaml
    `-- ui
        |-- deployment-patch.yaml
        `-- kustomization.yaml

12 thư mục, 19 tập tin
```

Đẩy các thay đổi lên kho lưu trữ Git:

```bash
$ git -C ~/environment/argocd add .
$ git -C ~/environment/argocd commit -am "Thêm tùy chỉnh ứng dụng"
$ git -C ~/environment/argocd push
```

Nhấp vào `Refresh` và `Sync` trong giao diện ArgoCD, sử dụng `argocd` CLI để `Sync` ứng dụng hoặc đợi cho đến khi `Sync` tự động hoàn tất:

```bash
$ argocd app sync apps
$ argocd app sync ui
```

Bây giờ chúng ta đã chuyển toàn bộ các ứng dụng để triển khai bằng Argo CD, và bất kỳ thay đổi nào tiếp theo được đẩy lên kho lưu trữ Git sẽ được tự động điều chỉnh với cụm EKS.

Khi Argo CD hoàn tất quá trình đồng bộ, tất cả các ứng dụng của chúng ta sẽ ở trạng thái `Synced`.

![EKS](/images/0006/00050.png?featherlight=false&width=90pc)

Bạn cũng nên có tất cả các tài nguyên liên quan đến ứng dụng `ui` được triển khai. Để xác minh, hãy chạy các lệnh sau:

```bash hook=deploy
$ kubectl get deployment -n ui ui
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
ui     1/1     1            1           61 giây
$ kubectl get pod -n ui
NAME                 READY   STATUS   RESTARTS   AGE
ui-6d5bb7b95-rjfxd   1/1     Running  0          62 giây
```

![EKS](/images/0006/00051.png?featherlight=false&width=90pc)