---
title: "Initial setup"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 2.2.3 </b>"
---


#### Triển khai ứng dụng với Argo CD và Kustomize

Argo CD áp dụng phương pháp `GitOps` cho Kubernetes. Nó sử dụng Git như nguồn thông tin chính xác về trạng thái mong muốn của cụm của bạn. Bạn có thể sử dụng Argo CD để triển khai ứng dụng, theo dõi sức khỏe của chúng và đồng bộ hóa chúng với trạng thái mong muốn. Tài liệu Kubernetes có thể được chỉ định theo một số cách:

- Ứng dụng Kustomize
- Bản Helm charts
- Tệp Jsonnet
- Thư mục thuần của các tệp YAML Kubernetes

Trong bài tập thực hành này, chúng ta sẽ triển khai các ứng dụng được chỉ định trong Kustomize bằng cách sử dụng Argo CD. Chúng ta sẽ sử dụng ứng dụng `ui` từ kho [EKS Workshop](https://github.com/aws-samples/eks-workshop-v2/tree/main/manifests/base-application/ui).

Kho Git trong AWS CodeCommit đã được tạo sẵn cho bạn.

:::info
Nếu bạn muốn sử dụng kho GitHub riêng tư của mình, bạn có thể định nghĩa lại

```
export GITOPS_REPO_URL_ARGOCD=https://github.com/username/reponame
```

và sử dụng [hướng dẫn đó](https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/) để tạo một bí mật Argo CD để cung cấp quyền truy cập vào kho Git từ Argo CD
:::

Hãy sao chép kho Git.

```bash
$ git clone $GITOPS_REPO_URL_ARGOCD ~/environment/argocd
$ git -C ~/environment/argocd checkout -b main
Chuyển sang nhánh mới 'main'
$ mkdir ~/environment/argocd/apps && touch ~/environment/argocd/apps/.gitkeep
$ git -C ~/environment/argocd add .
$ git -C ~/environment/argocd commit -am "Commit ban đầu"
$ git -C ~/environment/argocd push --set-upstream origin main
```

Tạo một bí mật Argo CD để cung cấp quyền truy cập vào kho Git từ Argo CD:

```bash
$ argocd repo add $GITOPS_REPO_URL_ARGOCD --ssh-private-key-path ${HOME}/.ssh/gitops_ssh.pem --insecure-ignore-host-key --upsert --name git-repo
Kho 'ssh://...' đã được thêm
```

Ứng dụng Argo CD là một đối tượng tài nguyên CRD Kubernetes biểu diễn một phiên bản ứng dụng triển khai trong một môi trường. Nó xác định thông tin chính về ứng dụng, như tên ứng dụng, kho Git và đường dẫn đến các tài liệu Kubernetes. Tài nguyên ứng dụng cũng xác định trạng thái mong muốn của ứng dụng, như phiên bản mục tiêu, chính sách đồng bộ hóa và chính sách kiểm tra sức khỏe.

Như bước tiếp theo, hãy tạo một ứng dụng Argo CD `Sync` với trạng thái mong muốn trong kho Git:

```bash
$ argocd app create apps --repo $GITOPS_REPO_URL_ARGOCD \
  --path apps --dest-server https://kubernetes.default.svc
Ứng dụng 'apps' đã được tạo
```

Xác minh rằng ứng dụng đã được tạo:

```bash
$ argocd app list
TÊN          CỤM                         KHÔNG GIAN  DỰ ÁN  TRẠNG THÁI  SỨC KHỎE  CHÍNH SÁCH ĐỒNG BỘ  ĐIỀU KIỆN
argocd/apps  https://kubernetes.default.svc             mặc định  Đã đồng bộ  Khỏe mạnh  <không>            <không>
```

Chúng ta cũng có thể thấy Ứng dụng này trong giao diện ArgoCD ngay bây giờ:

![EKS](/images/0006/00043.png?featherlight=false&width=90pc)

Hoặc bạn cũng có thể tương tác với các đối tượng Argo CD trong cụm bằng cách sử dụng lệnh `kubectl`:

```bash
$ kubectl get applications.argoproj.io -n argocd
TÊN    TRẠNG THÁI ĐỒNG BỘ   TRẠNG THÁI SỨC KHỎE
apps   Đã đồng bộ             Khỏe mạnh
```