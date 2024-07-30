---
title: "Deploying an application"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 2.2.4 </b>"
---

#### Deploying an application

Chúng ta đã cấu hình thành công Argo CD trên cụm của chúng ta nên bây giờ chúng ta có thể triển khai một ứng dụng. Để minh họa sự khác biệt giữa việc triển khai ứng dụng dựa trên GitOps và các phương pháp khác, chúng ta sẽ di chuyển thành phần giao diện người dùng của ứng dụng mẫu hiện đang sử dụng phương pháp `kubectl apply -k` sang phương pháp triển khai Argo CD mới.

Đầu tiên hãy xóa bỏ thành phần giao diện người dùng hiện tại để chúng ta có thể thay thế nó:

```bash
$ kubectl delete -k ~/environment/eks-workshop/base-application/ui --ignore-not-found=true
namespace "ui" đã được xóa
serviceaccount "ui" đã được xóa
configmap "ui" đã được xóa
service "ui" đã được xóa
deployment.apps "ui" đã được xóa
```

Bây giờ, hãy vào thư mục Git đã được nhân bản và bắt đầu tạo cấu hình GitOps của chúng ta. Sao chép cấu hình kustomize hiện có cho dịch vụ UI:

```bash
$ cp -R ~/environment/eks-workshop/base-application/ui/* ~/environment/argocd/apps
```

Thư mục Git của bạn bây giờ nên trông giống như sau, bạn có thể xác nhận bằng cách chạy `tree ~/environment/argocd`:

```
.
└── apps
    ├── configMap.yaml
    ├── deployment.yaml
    ├── kustomization.yaml
    ├── namespace.yaml
    ├── serviceAccount.yaml
    └── service.yaml

1 thư mục, 6 tập tin
```

Mở giao diện Argo CD và điều hướng đến ứng dụng `apps`.

![EKS](/images/0006/00044.png?featherlight=false&width=90pc)

Cuối cùng, chúng ta có thể đẩy cấu hình của chúng ta lên kho Git:

```bash
$ git -C ~/environment/argocd add .
$ git -C ~/environment/argocd commit -am "Thêm dịch vụ UI"
$ git -C ~/environment/argocd push
```

Nhấp vào `Refresh` và `Sync` trong giao diện ArgoCD hoặc sử dụng CLI `argocd` để `Sync` ứng dụng:

```bash
$ argocd app sync apps
```

Sau một khoảng thời gian ngắn, ứng dụng nên ở trạng thái `Synced` và các tài nguyên nên được triển khai, giao diện người dùng nên trông như sau:

![EKS](/images/0006/00045.png?featherlight=false&width=90pc)

Điều đó cho thấy rằng Argo CD đã tạo ra cấu hình kustomize cơ bản và nó đang được đồng bộ với cụm.

Chúng ta đã thành công di chuyển thành phần giao diện người dùng để triển khai bằng cách sử dụng Argo CD và bất kỳ thay đổi nào khác được đẩy lên kho Git sẽ được tự động điều hòa với cụm EKS của chúng ta.

Bây giờ bạn nên có tất cả các tài nguyên liên quan đến các dịch vụ giao diện người dùng được triển khai. Để xác minh, hãy chạy các lệnh sau:

```bash hook=deploy
$ kubectl get deployment -n ui ui
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
ui     1/1     1            1           61 giây
$ kubectl get pod -n ui
NAME                 READY   STATUS   RESTARTS   AGE
ui-6d5bb7b95-rjfxd   1/1     Running  0          62 giây
```

