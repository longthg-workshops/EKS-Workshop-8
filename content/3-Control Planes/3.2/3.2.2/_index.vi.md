---
title: "Crossplane"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 3.2.2 </b>"
---

Mặc định, thành phần **Carts** trong ứng dụng mẫu sử dụng một phiên bản cục bộ của DynamoDB chạy như một pod trong cụm EKS được gọi là `carts-dynamodb`. Trong phần này của lab, chúng ta sẽ cung cấp một bảng dựa trên đám mây của Amazon DynamoDB cho ứng dụng của chúng ta bằng cách sử dụng các tài nguyên được quản lý bởi Crossplane và trỏ **Carts** deployment để sử dụng bảng DynamoDB mới được cung cấp thay vì bản sao cục bộ.

![EKS](/images/0006/00058.png?featherlight=false&width=90pc)

SDK Java của AWS trong thành phần **Carts** có thể sử dụng IAM Roles để EKS tương tác với các dịch vụ AWS, điều này có nghĩa là chúng ta không cần phải truyền thông tin đăng nhập, giảm thiểu bề mặt tấn công. Trong ngữ cảnh của EKS, IRSA cho phép chúng ta xác định IAM Roles theo pod cho các ứng dụng sử dụng. Để tận dụng IRSA, trước tiên chúng ta cần:

- Tạo một Kubernetes Service Account trong namespace Carts
- Tạo một IAM Policy với các quyền DynamoDB cần thiết
- Tạo một IAM Role trong AWS với các quyền trên
- Ánh xạ Service Account để sử dụng IAM Role bằng cách sử dụng Annotations trong định nghĩa của Service Account.

May mắn thay, chúng ta có một dòng lệnh tiện lợi để hỗ trợ quá trình này. Chạy dòng lệnh sau:

```bash
$ eksctl create iamserviceaccount --name carts-crossplane \
  --namespace carts --cluster $EKS_CLUSTER_NAME \
  --role-name ${EKS_CLUSTER_NAME}-carts-crossplane \
  --attach-policy-arn $DYNAMODB_POLICY_ARN --approve

2023-10-30 12:45:17 [i]  1 iamserviceaccount (carts/carts-crossplane) đã được bao gồm (dựa trên các quy tắc bao gồm/loại trừ)
2023-10-30 12:45:17 [!]  các tài khoản dịch vụ tồn tại trong Kubernetes sẽ bị loại bỏ, sử dụng --override-existing-serviceaccounts để ghi đè
2023-10-30 12:45:17 [i]  1 nhiệm vụ: {
    2 nhiệm vụ phụ tuần tự: {
        tạo IAM role cho serviceaccount "carts/carts-crossplane",
        tạo serviceaccount "carts/carts-crossplane",
    } }2023-10-30 12:45:17 [i]  đang xây dựng ngăn xếp iamserviceaccount "eksctl-eks-workshop-addon-iamserviceaccount-carts-carts-crossplane"
2023-10-30 12:45:18 [i]  triển khai ngăn xếp "eksctl-eks-workshop-addon-iamserviceaccount-carts-carts-crossplane"
2023-10-30 12:45:18 [i]  đang chờ ngăn xếp CloudFormation "eksctl-eks-workshop-addon-iamserviceaccount-carts-carts-crossplane"
```

`eksctl` cung cấp một ngăn xếp CloudFormation để hỗ trợ quản lý các tài nguyên này có thể được thấy trong đầu ra ở trên.

Để tìm hiểu thêm về cách IRSA hoạt động, hãy [đi đến đây](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html).

---

Bây giờ, hãy khám phá cách chúng ta sẽ tạo bảng DynamoDB thông qua một biểu mẫu tài nguyên được quản lý bởi Crossplane

```file
manifests/modules/automation/controlplanes/crossplane/managed/table.yaml
```

Cuối cùng, chúng ta có thể tạo cấu hình cho DynamoDB chính nó với một tài nguyên `dynamodb.aws.upbound.io`.

```bash wait=10 timeout=400 hook=table
$ kubectl kustomize ~/environment/eks-workshop/modules/automation/controlplanes/crossplane/managed \
  | envsubst | kubectl apply -f-
table.dynamodb.aws.upbound.io/eks-workshop-carts-crossplane đã được tạo
$ kubectl wait tables.dynamodb.aws.upbound.io ${EKS_CLUSTER_NAME}-carts-crossplane \
  --for=condition=Ready --timeout=5m
```

Mất một thời gian để cung cấp các dịch vụ được quản lý bởi AWS, trong trường hợp của DynamoDB có thể lên đến 2 phút. Crossplane sẽ báo cáo trạng thái của việc điều hòa trong trường `status` của các tài nguyên tùy chỉnh Kubernetes.

```bash
$ kubectl get tables.dynamodb.aws.upbound.io
TÊN                                        SẴN SÀNG  ĐỒNG BỘ   TÊN NGOẠI VI                      TUỔI
eks-workshop-carts-crossplane               Đúng   Đúng     eks-workshop-carts-crossplane   6s
```

Khi các tài nguyên mới được tạo hoặc cập nhật, cấu hình ứng dụng cũng cần được cập nhật để sử dụng các tài nguyên mới này. Cập nhật ứng dụng để sử dụng điểm cuối của DynamoDB:

```bash timeout=180
$ kubectl kustomize ~/environment/eks-workshop/modules/automation/controlplanes/crossplane/application \
  | envsubst | kubectl apply -f-
namespace/carts không thay đổi


serviceaccount/carts không thay đổi
configmap/carts không thay đổi
configmap/carts-crossplane đã được tạo
service/carts không thay đổi
service/carts-dynamodb không thay đổi
deployment.apps/carts được cấu hình
deployment.apps/carts-dynamodb không thay đổi
$ kubectl rollout status -n carts deployment/carts --timeout=2m
deployment "carts" đã được triển khai thành công
```

---

Bây giờ, làm thế nào chúng ta biết rằng ứng dụng đang hoạt động với bảng DynamoDB mới?

Một NLB đã được tạo ra để tiếp cận ứng dụng mẫu để kiểm tra, cho phép chúng ta tương tác trực tiếp với ứng dụng thông qua trình duyệt:

```bash
$ kubectl get service -n ui ui-nlb -o jsonpath="{.status.loadBalancer.ingress[*].hostname}{'\n'}"
k8s-ui-uinlb-a9797f0f61.elb.us-west-2.amazonaws.com
```

:::info
Vui lòng lưu ý rằng địa chỉ cuối thực tế sẽ khác khi bạn chạy lệnh này vì một điểm cuối Network Load Balancer mới sẽ được cung cấp.
:::

Để chờ cho đến khi load balancer hoàn thành việc cung cấp, bạn có thể chạy lệnh này:

```bash timeout=610
$ wait-for-lb $(kubectl get service -n ui ui-nlb -o jsonpath="{.status.loadBalancer.ingress[*].hostname}{'\n'}")
```

Sau khi load balancer được cung cấp, bạn có thể truy cập nó bằng cách dán URL vào trình duyệt web của bạn. Bạn sẽ thấy giao diện người dùng từ cửa hàng web được hiển thị và có thể di chuyển xung quanh trang web như một người dùng.

![EKS](/images/0006/00059.png?featherlight=false&width=90pc)

Để xác minh rằng mô-đun **Carts** thực sự đang sử dụng bảng DynamoDB chúng ta vừa cung cấp, hãy thử thêm một vài mục vào giỏ hàng.

![EKS](/images/0006/00060.png?featherlight=false&width=90pc)

Và để kiểm tra xem các mục có trong bảng DynamoDB không, chạy

```bash
$ aws dynamodb scan --table-name "${EKS_CLUSTER_NAME}-carts-crossplane" \
  --query 'Items[].{itemId:itemId,Price:unitPrice}' --output text
GIÁ    795
MÃ MỤC  510a0d7e-8e83-4193-b483-e27e09ddc34d
GIÁ    385
MÃ MỤC  6d62d909-f957-430e-8689-b5129c0bb75e
GIÁ    50
MÃ MỤC  a0a4f044-b040-410d-8ead-4de0446aec7e
```

Chúc mừng! Bạn đã thành công trong việc tạo các Tài nguyên AWS mà không cần rời khỏi API Kubernetes!

