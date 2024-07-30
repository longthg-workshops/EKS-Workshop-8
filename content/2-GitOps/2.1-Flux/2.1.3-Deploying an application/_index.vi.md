---
title: "Triển khai ứng dụng"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 2.1.3 </b>"
---

Chúng ta đã thành công trong việc khởi động Flux trên cụm của chúng ta, giờ đây chúng ta có thể triển khai một ứng dụng. Để minh họa sự khác biệt giữa việc triển khai một ứng dụng dựa trên GitOps và các phương pháp khác, chúng ta sẽ chuyển đổi thành phần giao diện người dùng của ứng dụng mẫu hiện đang sử dụng phương pháp triển khai `kubectl apply -k` sang phương pháp triển khai Flux mới.

Đầu tiên, hãy gỡ bỏ thành phần giao diện người dùng hiện tại:

```bash
$ kubectl delete -k ~/environment/eks-workshop/base-application/ui
```

Tiếp theo, sao chép kho lưu trữ chúng ta đã sử dụng để khởi động Flux trong phần trước:

```bash
$ git clone ssh://${GITOPS_IAM_SSH_KEY_ID}@git-codecommit.${AWS_REGION}.amazonaws.com/v1/repos/${EKS_CLUSTER_NAME}-gitops ~/environment/flux
```

Bây giờ, hãy vào thư mục đã sao chép và bắt đầu tạo cấu hình GitOps của chúng ta. Sao chép cấu hình kustomize hiện có cho dịch vụ giao diện người dùng:

```bash
$ mkdir ~/environment/flux/apps
$ cp -R ~/environment/eks-workshop/base-application/ui ~/environment/flux/apps
```

Sau đó, chúng ta cần tạo một tệp kustomization trong thư mục `apps`:

```file
manifests/modules/automation/gitops/flux/apps-kustomization.yaml
```

Sao chép tệp này vào thư mục kho lưu trữ Git:

```bash
$ cp ~/environment/eks-workshop/modules/automation/gitops/flux/apps-kustomization.yaml \
  ~/environment/flux/apps/kustomization.yaml
```

Bước cuối cùng trước khi đẩy các thay đổi là đảm bảo rằng Flux biết về thư mục `apps` của chúng ta. Điều đó có thể thực hiện bằng cách tạo một tệp bổ sung trong thư mục `flux`:

```file
manifests/modules/automation/gitops/flux/flux-kustomization.yaml
```

Sao chép tệp này vào thư mục kho lưu trữ Git:

```bash
$ cp ~/environment/eks-workshop/modules/automation/gitops/flux/flux-kustomization.yaml \
  ~/environment/flux/apps.yaml
```

Thư mục Git của bạn bây giờ nên trông giống như thế này mà bạn có thể xác nhận bằng cách chạy `tree ~/environment/flux`:

```
.
├── apps
│   ├── kustomization.yaml
│   └── ui
│       ├── configMap.yaml
│       ├── deployment.yaml
│       ├── kustomization.yaml
│       ├── namespace.yaml
│       ├── serviceAccount.yaml
│       └── service.yaml
├── apps.yaml
└── flux-system
    ├── gotk-components.yaml
    ├── gotk-sync.yaml
    └── kustomization.yaml


3 directories, 11 files
```

Cuối cùng, chúng ta có thể đẩy cấu hình đã tạo lên CodeCommit:

```bash
$ git -C ~/environment/flux add .
$ git -C ~/environment/flux commit -am "Adding the UI service"
$ git -C ~/environment/flux push origin main
```

Flux sẽ mất một ít thời gian để nhận thấy các thay đổi trong CodeCommit và làm đồng bộ. Bạn có thể sử dụng CLI Flux để theo dõi tệp kustomization `apps` mới của chúng ta xuất hiện:

```bash test=false
$ flux get kustomization --watch
NAMESPACE     NAME          AGE   READY   STATUS
flux-system   flux-system   14h   True    Applied revision: main/f39f67e6fb870eed5997c65a58c35f8a58515969
flux-system   apps          34s   True    Applied revision: main/f39f67e6fb870eed5997c65a58c35f8a58515969
```

Bạn cũng có thể kích hoạt Flux để làm đồng bộ thủ công như sau:

```bash wait=30 hook=flux-deployment
$ flux reconcile source git flux-system -n flux-system
```

Khi `apps` xuất hiện như đã chỉ ra ở trên, sử dụng `Ctrl+C` để hủy lệnh. Bây giờ bạn nên đã triển khai lại tất cả các tài nguyên liên quan đến các dịch vụ giao diện người dùng. Để xác minh, hãy chạy các lệnh sau:

```bash
$ kubectl get deployment -n ui ui
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
ui     1/1     1            1           5m
$ kubectl get pod -n ui
NAME                  READY   STATUS    RESTARTS   AGE
ui-54ff78779b-qnrrc   1/1     Running   0          5m
```

Chúng ta đã thành công trong việc di chuyển thành phần giao diện người dùng để triển khai bằng cách sử dụng Flux, và bất kỳ thay đổi nào được đẩy vào kho lưu trữ Git sẽ được tự động làm đồng bộ với cụm EKS của chúng ta.